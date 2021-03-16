Android WiFiDirect 学习（二）——Service Discovery

Android WiFiDirect 学习（二）——Service Discovery

### Service Discovery 简介

在Android WifiDirect学习（一 ）中，简单介绍了如何使用WifiDirect进行搜索——连接——传输。

这样会有一个问题，那就是你会搜索到到附近所有处于WifiDirect搜索状态的网络设备，而这些设备中不一定都是你想进行连接的。

Android WifiDirect Api提供了一个仅搜索特定网络设备的搜索方式，叫做Service Discovery，它是Wi-Fi Direct API在Android 4.1中被增强以支持在[WifiP2pManager](http://developer.android.com/reference/android/net/wifi/p2p/WifiP2pManager.html)中的预先关联服务发现。这允许在连接之前使用Wi-Fi Direct通过服务(Service)发现和筛选周围的设备。

### 常见Service：

Android已经提供了两种Service，一种是 Bonjour Service，一种是Upnp Service，开发者也可以自己设计一套Service，一般情况下，这两种Service已经足够用了。

### Service基本使用方法

为了广播你的应用作为一个Wi-Fi上的服务，以便于其他设备能够发现并连接你的应用，需要调用[addLocalService()](http://developer.android.com/reference/android/net/wifi/p2p/WifiP2pManager.html#addLocalService(android.net.wifi.p2p.WifiP2pManager.Channel,%20android.net.wifi.p2p.nsd.WifiP2pServiceInfo,%20android.net.wifi.p2p.WifiP2pManager.ActionListener))方法并传输一个[WifiP2pServiceInfo](http://developer.android.com/reference/android/net/wifi/p2p/nsd/WifiP2pServiceInfo.htmlWifiP2pServiceInfo "WifiP2pServiceInfo")对象。这个对象描述了你的应用服务。

为了通过Wi-Fi开始发现附近的设备，首先你应当决定使用Bonjour Service还是Upnp Service实现通信。如果使用Bonjour，首先要用[setDnsSdResponseListeners()](http://developer.android.com/reference/android/net/wifi/p2p/WifiP2pManager.html#setDnsSdResponseListeners(android.net.wifi.p2p.WifiP2pManager.Channel,%20android.net.wifi.p2p.WifiP2pManager.DnsSdServiceResponseListener,%20android.net.wifi.p2p.WifiP2pManager.DnsSdTxtRecordListener))设置好一些回调监听器。这个方法需要[WifiP2pManager.DnsSdServiceResponseListener](http://developer.android.com/reference/android/net/wifi/p2p/WifiP2pManager.DnsSdServiceResponseListener.html)和[DnsSdTxtRecordListener](http://developer.android.com/reference/android/net/wifi/p2p/WifiP2pManager.DnsSdTxtRecordListener.htmlWifiP2pManager.DnsSdTxtRecordListener)两个作参数。如果使用Upnp，则要调用[setUpnpServiceResponseListener()](http://developer.android.com/reference/android/net/wifi/p2p/WifiP2pManager.html#setUpnpServiceResponseListener(android.net.wifi.p2p.WifiP2pManager.Channel,%20android.net.wifi.p2p.WifiP2pManager.UpnpServiceResponseListener))。这个方法需要[UpnpServiceResponseListener](http://developer.android.com/reference/android/net/wifi/p2p/WifiP2pManager.UpnpServiceResponseListener.htmlWifiP2pManager.UpnpServiceResponseListener)作为参数。

在你开始发现本地设备上的服务之前，你也需要调用[addServiceRequest()](http://developer.android.com/reference/android/net/wifi/p2p/WifiP2pManager.html#addServiceRequest(android.net.wifi.p2p.WifiP2pManager.Channel,%20android.net.wifi.p2p.nsd.WifiP2pServiceRequest,%20android.net.wifi.p2p.WifiP2pManager.ActionListener)addServiceRequest())方法。当你传递给这个方法的接口[ActionListener](http://developer.android.com/reference/android/net/wifi/p2p/WifiP2pManager.ActionListener.htmlWifiP2pManager.ActionListener)收到一个成功的回调，接着你可以通过调用[discoverServices()](http://developer.android.com/reference/android/net/wifi/p2p/WifiP2pManager.html#discoverServices(android.net.wifi.p2p.WifiP2pManager.Channel,%20android.net.wifi.p2p.WifiP2pManager.ActionListener)discoverServices())开始探索在本地设备上的服务。

当本地服务被发现，你会收到一个回调，来自[WifiP2pManager.DnsSdServiceResponseListener](http://developer.android.com/reference/android/net/wifi/p2p/WifiP2pManager.DnsSdServiceResponseListener.html)或者[WifiP2pManager.UpnpServiceResponseListener](http://developer.android.com/reference/android/net/wifi/p2p/WifiP2pManager.UpnpServiceResponseListener.html)，这个取决于你注册使用的是Bonjour还是Upnp。收到的回调都会包含一个[WifiP2pDevice](http://developer.android.com/reference/android/net/wifi/p2p/WifiP2pDevice.html)对象，代表了对应的设备。

### Service使用案例（以Bonjour Service为例）

#### **设置**Manifest

为了使用Wi-Fi P2P，要在你的androidmanifest中添加[CHANGE\_WIFI\_STATE](http://developer.android.com/reference/android/Manifest.permission.html#CHANGE_WIFI_STATE)、[ACCESS\_WIFI\_STATE](http://developer.android.com/reference/android/Manifest.permission.html#ACCESS_WIFI_STATE)、和[INTERNET](http://developer.android.com/reference/android/Manifest.permission.html#INTERNET)权限。即使Wi-FiP2P不要求互联网连接，但它要使用标准的Java socket，并且Android在使用这些套接字时要求申请这些权限，因此要在清单中申请INTERNET权限。

 1 <manifestxmlns:android="http://schemas.android.com/apk/res/android" 
 2     package="com.example.android.nsdchat" 
 3     ... 
 4     <uses-permission 
 5         android:required="true" 
 6         android:name="android.permission.ACCESS\_WIFI\_STATE"/> 
 7     <uses-permission 
 8         android:required="true" 
 9         android:name="android.permission.CHANGE\_WIFI\_STATE"/> 
10     <uses-permission 11         android:required="true" 
12         android:name="android.permission.INTERNET"/> 
13     ...

#### **添加Local Service**

如果你要提供一个Local Service，那么就需要把这个服务注册为可发现的。本地服务被注册后，框架就会自动的响应来自对等点的服务发现请求。

以下是创建本地服务的步骤：

1\. 创建一个[WifiP2pServiceInfo](http://developer.android.com/reference/android/net/wifi/p2p/nsd/WifiP2pServiceInfo.html)对象；

2\. 填入你的服务相关的信息；

3\. 调用addLocalService()方法来注册本地服务，让其可发现。

 1 private void startRegistration(){ 
 2 // Create a string map containing information about your service. 
 3         Map record = new HashMap(); 4         record.put("listenport", String.valueOf(SERVER_PORT)); 
 5         record.put("buddyname", "John Doe" + (int) (Math.random() * 1000)); 
 6         record.put("available", "visible"); 
 7         // Serviceinformation.  Pass it an instance name, service type 8         //\_protocol.\_transportlayer , and the map containing 
 9         //information other devices will want once they connect to this one. 
10         WifiP2pDnsSdServiceInfo serviceInfo = 
11                 WifiP2pDnsSdServiceInfo.newInstance("\_test", "\_presence._tcp", record); 12         // Add thelocal service, sending the service info, network channel, 13         // andlistener that will be used to indicate success or failure of 14         // therequest. 
15         mManager.addLocalService(channel, serviceInfo, new ActionListener() { 16 @Override 17             public void onSuccess() { 18                 // Command successful! Code isn't necessarily needed here, 19                 // Unless you want to update the UI or add logging statements. 
20 } 21 @Override 22             public void onFailure(int arg0) { 23                 // Command failed.  Check for P2P_UNSUPPORTED, ERROR, or BUSY 
24 } 25 }); 26     } 

  
**发现附近的服务**

Android使用回调方法来通知你的应用程序可用的服务，因此首先要做的就是要建立回调方法。创建一个[WifiP2pManager.DnsSdTxtRecordListener](http://developer.android.com/reference/android/net/wifi/p2p/WifiP2pManager.DnsSdTxtRecordListener.html)对象来监听传入的记录。这个记录可以是其他设备的任意广播。当一个记录进入时，你可以把设备地址和其他你想要的其他相关信息复制到当前方法外部的数据结构中，以便后续可以访问它。下面的例子假设记录中包含一个“buddyname”字段，它带有用户的标识。

 1 finalHashMap&lt;String,String&gt; buddies =newHashMap&lt;String,String&gt;(); 
 2 ... 
 3 private void discoverService() { 4     DnsSdTxtRecordListener txtListener = new DnsSdTxtRecordListener() { 5         @Override 
 6         /* Callbackincludes: 7          \* fullDomain: full domainname: e.g "printer.\_ipp.\_tcp.local." 
 8          \* record: TXT record dta as amap of key/value pairs. 
 9 \* device: The device runningthe advertised service. 10          */ 
11         public voidonDnsSdTxtRecordAvailable( 12 String fullDomain, Map record, WifiP2pDevice device) { 13                 Log.d(TAG, "DnsSdTxtRecord available -" + record.toString()); 14                 buddies.put(device.deviceAddress, record.get("buddyname")); 15 } 16 }; 17 ... 18 }

  
实现一个[WifiP2pManager.DnsSdServiceResponseListener](http://developer.android.com/reference/android/net/wifi/p2p/WifiP2pManager.DnsSdServiceResponseListener.html)接口，来获取服务信息。这个接口会接收实际的描述和连接信息。上面的代码中使用了[Map](http://developer.android.com/reference/java/util/Map.html)对象把设备地址和用户标识组成一对。服务响应监听器使用这个接口把DNS记录和对应的服务信息连接到一起。实现上述两个监听器后，使用[setDnsSdResponseListener()](http://developer.android.com/reference/android/net/wifi/p2p/WifiP2pManager.html#setDnsSdResponseListeners(android.net.wifi.p2p.WifiP2pManager.Channel,%20android.net.wifi.p2p.WifiP2pManager.DnsSdServiceResponseListener,%20android.net.wifi.p2p.WifiP2pManager.Dns)方法把它们添加给[WifiP2pManager](http://developer.android.com/reference/android/net/wifi/p2p/WifiP2pManager.html)对象。

 1 private void discoverService(){ 2 ... 
 3     DnsSdServiceResponseListener servListener = new DnsSdServiceResponseListener() { 4         @Override 
 5         public voidonDnsSdServiceAvailable(String instanceName, String registrationType, 6                 WifiP2pDevice resourceType) { 
 7                 // Update the device name with the human-friendly version from 8                 // the DnsTxtRecord, assuming one arrived. 
 9                resourceType.deviceName = buddies 10                        .containsKey(resourceType.deviceAddress) ? buddies 11 .get(resourceType.deviceAddress) : resourceType.deviceName; 12                 // Add to the custom adapter defined specifically for showing 13                 // wifi devices. 
14                 WiFiDirectServicesList fragment = (WiFiDirectServicesList)getFragmentManager() 15 .findFragmentById(R.id.frag_peerlist); 16                 WiFiDevicesAdapter adapter = ((WiFiDevicesAdapter)fragment 17 .getListAdapter()); 18 adapter.add(resourceType); 19 adapter.notifyDataSetChanged(); 20                 Log.d(TAG, "onBonjourServiceAvailable " + instanceName); 21 } 22 }; 23 mManager.setDnsSdResponseListeners(channel, servListener, txtListener); 24 ... 25 }

现在创建一个服务请求并调用[addServiceRequest()](http://developer.android.com/reference/android/net/wifi/p2p/WifiP2pManager.html#addServiceRequest(android.net.wifi.p2p.WifiP2pManager.Channel,%20android.net.wifi.p2p.nsd.WifiP2pServiceRequest,%20android.net.wifi.p2p.WifiP2pManager.ActionListener))方法，这个方法也需要一个监听器来包括成功或失败。

 1 serviceRequest =WifiP2pDnsSdServiceRequest.newInstance(); 
 2         mManager.addServiceRequest(channel, 
 3                 serviceRequest, 
 4                 new ActionListener() { 5                    @Override 
 6                    public void onSuccess() { 7                        // Success! 
 8                    } 
 9 @Override 10                    public void onFailure(int code) { 11                        // Command failed.  Check forP2P_UNSUPPORTED, ERROR, or BUSY 
12 } 13                 });

最后，调用的[discoverServices()](http://developer.android.com/reference/android/net/wifi/p2p/WifiP2pManager.html#discoverServices(android.net.wifi.p2p.WifiP2pManager.Channel,%20android.net.wifi.p2p.WifiP2pManager.ActionListener))方法。

 1 mManager.discoverServices(channel,newActionListener(){ 
 2 @Override 
 3             public void onSuccess() { 4                 // Success! 
 5             } 
 6             @Override 
 7             public void onFailure(int code) { 8                 // Command failed.  Check for P2P_UNSUPPORTED, ERROR, or BUSY 
 9                 if (code == WifiP2pManager.P2P_UNSUPPORTED) { 10                    Log.d(TAG, "P2P isn'tsupported on this device."); 11                 else if(...) 12 ... 13 } 14         });

如果一切顺利，恭喜你大功告成。如果遇到问题，前面异步调用的参数[WifiP2pManager.ActionListener](http://developer.android.com/reference/android/net/wifi/p2p/WifiP2pManager.ActionListener.html)参数，它提供了指示成功或失败的回调方法。把调试断点设置在onFailure()方法中来诊断问题。这个方法提供了错误代码，以下是可能发生的错误：

[P2P_UNSUPPORTED](http://developer.android.com/reference/android/net/wifi/p2p/WifiP2pManager.html#P2P_UNSUPPORTED)

运行app的设备上不支持Wi-Fi P2P

[BUSY](http://developer.android.com/reference/android/net/wifi/p2p/WifiP2pManager.html#BUSY)

系统忙于处理请求

[ERROR](http://developer.android.com/reference/android/net/wifi/p2p/WifiP2pManager.html#ERROR)

由于内部错误导致操作失败

参考文章

[Android 4.1 API中文详解-Android 4.1 APIs](http://wiki.eoeandroid.com/Android_4.1_APIs "Android 4.1 APIs")  http://wiki.eoeandroid.com/Android\_4.1\_APIs

 [http://developer.android.com/](http://developer.android.com/about/versions/android-4.1.html)