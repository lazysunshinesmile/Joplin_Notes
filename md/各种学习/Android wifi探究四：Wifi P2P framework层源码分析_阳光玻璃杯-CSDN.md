Android wifi探究四：Wifi P2P framework层源码分析_阳光玻璃杯-CSDN博客

上一篇博客对应用程序下使用Wi-Fi P2P Api连接附近的设备的过程做了一个简单的梳理，我们只是学会了怎么使用api,但对api背后的机制一无所知。那么这篇博客就开始尝试分析api背后的实现机制，也就是android framework中Wi-Fi P2P的工作机制。  
Wifi P2P在framework层也是一个Service，它的启动过程和WifiService一样：

```
                mSystemServiceManager.startService(WIFI_P2P_SERVICE_CLASS)
                mSystemServiceManager.startService(WIFI_SERVICE_CLASS)
```

所以就在不啰嗦了，感兴趣可惜浏览下 [Android wifi探究二：java层的wifi框架](http://blog.csdn.net/u011913612/article/details/52785722) 这篇文章，我们感兴趣的是WifiP2pServiceImpl是怎么工作的，怎么和应用程序交互的。比如当应用程序调用discoverPeers方法发起查找时，WifiP2pServiceImpl做了什么工作。  
回顾上一篇博客的内容，我们使用Wifi P2P服务时，首先要获取到WifiP2pManager 的实例：

```
mManager = (WifiP2pManager) getSystemService(Context.WIFI_P2P_SERVICE);
```

之后的操作都是调用WifiP2pManager 中的方法实现的。我们知道我们向系统注册的Wi-Fi P2P服务是WifiP2pServiceImpl，他们之间会使用一定的机制进行通信，当然主要就是Binder了，不过，还有一个很重要的知识点，那就是Messenger。

浏览下WifiP2pServiceImpl的实现，因为之前已经分析过WifiServiceImpl的实现，对比我们就知道WifiP2pServiceImpl内部也有个状态机，状态机是消息驱动的，我们需要给状态机发消息才能促使状态机切换状态，那么我们能不能在WifiP2pManager 中直接给WifiP2pServiceImpl发消息呢？答案是能。怎么发呢？就是这里要介绍的Messenger。  
我们看一下Api文档中对它的介绍：  
Reference to a Handler, which others can use to send messages to it. This allows for the implementation of message-based communication across processes, by creating a Messenger pointing to a Handler in one process, and handing that Messenger to another process.

Note: the implementation underneath is just a simple wrapper around a Binder that is used to perform the communication. This means semantically you should treat it as such: this class does not impact process lifecycle management (you must be using some higher-level component to tell the system that your process needs to continue running), the connection will break if your process goes away for any reason,

翻译（英语太菜，翻译有误请包涵）：  
我们可以使用Handler引用来发消息，我们可以创建一个Messenger实例，这个实例指向一个进程中的Handler,这样我们可以在另一个进程中处理Messenger 发过来的消息。这样就实现了基于消息的进程间的通信。  
注意：实现的原理就是对Binder做一个封装。这意味着：这个类不影响进程的生命周期（你必须使用高级别的组件告诉系统的进程需要继续运行），如果另一进程消失了的话，连接就会断开。  
那么，Messanger是怎么获取的呢？  
WifiP2pManager 中：

```
    public Messenger getMessenger() {
        try {
            return mService.getMessenger();
        } catch (RemoteException e) {
            return null;
        }
    }
```

mService就是WifiP2pServiceImpl的客户端，我们看下WifiP2pServiceImpl中的getMessenger方法：

```
    public Messenger getMessenger() {
        enforceAccessPermission();
        enforceChangePermission();
        return new Messenger(mClientHandler);
    }
```

mClientHandler就是WifiP2pServiceImpl用来发送和处理消息的Handler实例，因此我们可以知道，WifiP2pManager 中的Messenger 是Binder的一个客户端，调用WifiP2pManager 中Messenger 的send方法发送消息，最终会导致WifiP2pServiceImpl中ClientHandler的handleMessage方法被调用。理解了这一点，我们就可以往下看代码了。

从构造函数看起：

```
    public WifiP2pServiceImpl(Context context) {
        mContext = context

        //STOPSHIP: get this from native side
        mInterface = "p2p0"
        mNetworkInfo = new NetworkInfo(ConnectivityManager.TYPE_WIFI_P2P, 0, NETWORKTYPE, "")

        mP2pSupported = mContext.getPackageManager().hasSystemFeature(
                PackageManager.FEATURE_WIFI_DIRECT)

        mThisDevice.primaryDeviceType = mContext.getResources().getString(
                com.android.internal.R.string.config_wifi_p2p_device_type)

        HandlerThread wifiP2pThread = new HandlerThread("WifiP2pService")
        wifiP2pThread.start()
        mClientHandler = new ClientHandler(wifiP2pThread.getLooper())

        mP2pStateMachine = new P2pStateMachine(TAG, wifiP2pThread.getLooper(), mP2pSupported)
        mP2pStateMachine.start()
    }
```

构造函数主要做了两件事：  
1.创建mP2pStateMachine 状态机，并且启动这个状态机。  
2.创建HandlerThread ，用来监听消息，并绑定一个mClientHandler 来发送和处理消息。  
因此，重点还是在mP2pStateMachine这个状态机上面。

P2pStateMachine是一个状态机，他负责维护Wi-Fi P2P的各种状态，并通过消息的驱动，在状态之间切换，和WifiStateMachine相同的是，它也有下面连个成员：

```
        private WifiNative mWifiNative = new WifiNative(mInterface);
        private WifiMonitor mWifiMonitor = new WifiMonitor(this, mWifiNative);
```

这就意味着，它的底层还是使用WifiNative来和c/c++代码交互的，并且wpa_supplicant时间还是由WifiMonitor来监听的。也就是说，P2pStateMachine的工作原理和WifiStateMachine完全相同。因此这里就不展开分析它的工作原理了，WifiStateMachine的工作原理之前的博客已经分析过了。  
四.自上而下的调用过程

当我们在应用程序中连接附近设备时，会有如下api调用：  
1\. mChannel = mManager.initialize(this, getMainLooper(), null);  
2\. mManager.discoverPeers  
3\. mManager.requestPeers(mChannel, peerListListener);  
4\. mManager.connect  
5\. mManager.requestConnectionInfo(mChannel, connectionListener);  
下面，我们典型的分析几个api的内部实现原理：

## <a id="t3"></a><a id="t3"></a>3.1initialize

也就是所谓的初始化工作：

```
    public Channel initialize(Context srcContext, Looper srcLooper, ChannelListener listener) {
        return initalizeChannel(srcContext, srcLooper, listener, getMessenger());
    }
    private Channel initalizeChannel(Context srcContext, Looper srcLooper, ChannelListener listener,
                                     Messenger messenger) {
        if (messenger == null) return null;

        Channel c = new Channel(srcContext, srcLooper, listener);
        if (c.mAsyncChannel.connectSync(srcContext, c.mHandler, messenger)
                == AsyncChannel.STATUS_SUCCESSFUL) {
            return c;
        } else {
            return null;
        }
    }

```

这里的核心工作就是构建Channel 对象，然后调用Channel 的connectSync方法，这个方法如下：

```
    public int connectSync(Context srcContext, Handler srcHandler, Messenger dstMessenger) {
        if (DBG) log("halfConnectSync srcHandler to the dstMessenger  E");

        
        connected(srcContext, srcHandler, dstMessenger);

        if (DBG) log("halfConnectSync srcHandler to the dstMessenger X");
        return STATUS_SUCCESSFUL;
    }
```

调用connected方法进一步处理：

```
    public void connected(Context srcContext, Handler srcHandler, Messenger dstMessenger) {
        if (DBG) log("connected srcHandler to the dstMessenger  E");

        
        mSrcContext = srcContext;
        mSrcHandler = srcHandler;
        mSrcMessenger = new Messenger(mSrcHandler);

        
        mDstMessenger = dstMessenger;

        if (DBG) log("connected srcHandler to the dstMessenger X");
    }
```

我们看到这里有两个Messenger，一个是“源”，一个是“目的”，这个很好理解，我们使用mDstMessenger 向WifiP2pServiceImpl发送消息，也需要接收WifiP2pServiceImpl返回的消息呀，这不就得用mSrcMessenger 吗？mSrcHandler 当然是用来处理消息的呀。  
也就是说，通过初始化，我们创建了Channel ，而Channel封装了两个Messenger，分别用于发送和接受消息，因此Channel 是WifiP2PManager和WifiP2pServiceImpl之间的桥梁，正是它把二者联系在一起，因此几乎所有的API都要使用Channel 对象。或者说的严谨一点，所有需要向WifiP2pServiceImpl发送消息的api都需要这个Channel 的实例。

那么，接下来，我们通过具体的api来分析Channel 在这个过程中的核心作用。

## <a id="t4"></a><a id="t4"></a>3.2 discoverPeers

看看discoverPeers的实现：

```
    public void discoverPeers(Channel c, ActionListener listener) {
        checkChannel(c);
        c.mAsyncChannel.sendMessage(DISCOVER_PEERS, 0, c.putListener(listener));
    }

```

看到了吧：c就是一个Channel实例，使我们initialize方法中构造好并且返回的一个值，把它作为discoverPeers的第一个参数，就是要使用它来和WifiP2pServiceImpl通信的，我们具体看看通信的过程，也就是c.mAsyncChannel.sendMessage这个方法的实现：

```
    public void sendMessage(int what, int arg1, int arg2) {
        Message msg = Message.obtain();
        msg.what = what;
        msg.arg1 = arg1;
        msg.arg2 = arg2;
        sendMessage(msg);
    }
```

调用sendMessage进一步处理：

```
    public void sendMessage(Message msg) {
        msg.replyTo = mSrcMessenger;
        try {
            mDstMessenger.send(msg);
        } catch (RemoteException e) {
            replyDisconnected(STATUS_SEND_UNSUCCESSFUL);
        }
    }
```

看到了吧：msg.replyTo就是接收WifiP2pServiceImpl返回的消息的，要把它也发送给WifiP2pServiceImpl，真正发送消息的是mDstMessenger.send(msg);mDstMessenger我们在initialize方法中讲过了，它就是真正用来向WifiP2pServiceImpl发送消息的Messenger实例。发送完消息以后WifiP2pServiceImpl中的ClientHandler的handleMessage方法会被调用，着我们之前也说过了。  
ClientHandler的handleMessage方法如下：

```
        @Override
        public void handleMessage(Message msg) {
            switch (msg.what) {
              case WifiP2pManager.SET_DEVICE_NAME:
              case WifiP2pManager.SET_WFD_INFO:
              case WifiP2pManager.DISCOVER_PEERS:
              case WifiP2pManager.STOP_DISCOVERY:
              case WifiP2pManager.CONNECT:
              case WifiP2pManager.CANCEL_CONNECT:
              case WifiP2pManager.CREATE_GROUP:
              case WifiP2pManager.REMOVE_GROUP:
              case WifiP2pManager.START_LISTEN:
              case WifiP2pManager.STOP_LISTEN:
              case WifiP2pManager.SET_CHANNEL:
              case WifiP2pManager.START_WPS:
              case WifiP2pManager.ADD_LOCAL_SERVICE:
              case WifiP2pManager.REMOVE_LOCAL_SERVICE:
              case WifiP2pManager.CLEAR_LOCAL_SERVICES:
              case WifiP2pManager.DISCOVER_SERVICES:
              case WifiP2pManager.ADD_SERVICE_REQUEST:
              case WifiP2pManager.REMOVE_SERVICE_REQUEST:
              case WifiP2pManager.CLEAR_SERVICE_REQUESTS:
              case WifiP2pManager.REQUEST_PEERS:
              case WifiP2pManager.REQUEST_CONNECTION_INFO:
              case WifiP2pManager.REQUEST_GROUP_INFO:
              case WifiP2pManager.DELETE_PERSISTENT_GROUP:
              case WifiP2pManager.REQUEST_PERSISTENT_GROUP_INFO:
              
              case WifiP2pManager.MCAST_LISTEN:
              
                mP2pStateMachine.sendMessage(Message.obtain(msg));
                break;
              default:
                Slog.d(TAG, "ClientHandler.handleMessage ignoring msg=" + msg);
                break;
            }
        }
    }
```

它的作用就是把消息转发给mP2pStateMachine。  
当我们打开wifi以后，系统也会使能Wi-Fi P2P，这个时候mP2pStateMachine处于P2pEnabledState状态，这个时候，如果收到DISCOVER_PEERS消息，就会执行如下代码：

```
                case WifiP2pManager.DISCOVER_PEERS:
                    if (mDiscoveryBlocked) {
                        replyToMessage(message, WifiP2pManager.DISCOVER_PEERS_FAILED,
                                WifiP2pManager.BUSY);
                        break;
                    }
                    // do not send service discovery request while normal find operation.
                    clearSupplicantServiceRequest();
                    if (mWifiNative.p2pFind(DISCOVER_TIMEOUT_S)) {
                        replyToMessage(message, WifiP2pManager.DISCOVER_PEERS_SUCCEEDED);
                        sendP2pDiscoveryChangedBroadcast(true);
                    } else {
                        replyToMessage(message, WifiP2pManager.DISCOVER_PEERS_FAILED,
                                WifiP2pManager.ERROR);
                    }
                    break;
```

这里做了下面几件事：  
1\. 清楚wpa_supplicant原有的查找请求。  
2\. 调用mWifiNative.p2pFind(DISCOVER\_TIMEOUT\_S)开始查找。  
3\. 处理查找结果：如果查找成功，向WifiP2pManager返回成功的消息，并且发送查找结果改变的广播，也就是如下广播： public static final String WIFI\_P2P\_DISCOVERY\_CHANGED\_ACTION =  
“android.net.wifi.p2p.DISCOVERY\_STATE\_CHANGE”; 如果查找失败，则给WifiP2pManager返回查找失败的消息。

虽然我们只分析了两个方法的实现原理，还有很多的api都没有分析，当然，也不可能都分析一遍了，因为我们关注的是框架，而不是具体的细节。虽然这里面还有很多东西我们都忽略了，这也是因为能力有限，不能分析的面面俱到，但是通过这种框架层面的源码追踪和分析，我们能理解这些api工作的原理，能理解WifiP2pManager与WifiP2pServiceImpl的交互过程，以及WifiMonitor、P2pStateMachine和WifiNative各自的扮演的角色和作用。对比WifiService，我们发现他们的设计思路都是一样的，WifiServiceImpl的分析过程中，我尝试这绘制了一些图来帮助理解，这里就不再又一遍区绘制这些图了，因为他们设计的思路是相同的。感兴趣的可以先搞明白WifiService的框架，然后再来梳理WifiP2pService的框架，这时就会发现，一切都会变简单了。