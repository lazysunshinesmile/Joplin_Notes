ActivityThread

[![](https://upload.jianshu.io/users/upload_avatars/3879403/08838752-5adf-4938-99cd-32fc3ffec233.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/96/h/96/format/webp)](https://www.jianshu.com/u/f788647332b4)

0.8012017.03.27 14:57:41字數 840閱讀 3,289

我们学习 Android 过程中会发现，我们的文件都是 `.java` 文件，也就是说 Android 开发还是用的 Java 语言来编写的。也正是这样，所以你们来学 Android ，也会让你们先学习一段时间 Java 。掌握好了 Java 的相关知识，学起 Android 来可谓是事半功倍。好了，你们是不是感觉有点扯远了啊，不是说好讲 `ActivityThread` 类的么，其实并不如此。

你们在刚开始从 Java 学习转到 Android 学习的过程中，有一个重大的改变不知道你们又没有发现。那就是 Java 中的 `main()` 方法，程序的入口不见了，取而代之的是 `onCreate()` 方法。你们没有一点疑惑么？初学阶段直接无脑接受是对的，但是作为一个工作几年了的人来说，就有必要去深入研究一下了。明明 Android 也就是 Java 语言也编写的，差别咋就这么大呢？

其实呢， Android 中还是有 main() 方法的，只是隐藏的比较深而已。今天，就由我 AIqingfeng 来带你们一探究竟~！

我们先找到 ActivityThread 这个类，看一下注释（**较少**，值得一看）：

> This manages the execution of the main thread in an  
> application process, scheduling and executing activities,  
> broadcasts, and other operations on it as the activity  
> manager requests.

翻译一下就是：在 Application 进程中**管理执行主线程，调度和执行 活动和广播**，和活动管理请求的其它操作。

Android 上一个应用的入口，应该是 ActivityThread 类，和普通的Java 类一样，入口是一个 main() 方法。

```
public static void main(String[] args) {
    
    ···
    
    
    Looper.prepareMainLooper();

    
    ActivityThread thread = new ActivityThread();
    thread.attach(false);

    
    
    if (sMainThreadHandler == null) {
        sMainThreadHandler = thread.getHandler();
    }
    Looper.loop()；
    
    
    throw new RuntimeException("Main thread loop unexpectedly exited");
}

```

好了，现在我们解决了我们开始的疑惑后，再来深度学习一下这个类的一些知识吧。

ActivityThread 有几个比较重要的成员变量，会在创建ActivityThread对象时初始化。

`final ApplicationThread mAppThread = new ApplicationThread();`

ApplicationThread继承自ApplicationThreadNative， 而ApplicationThreadNative又继承自Binder并实现了IApplicationThread接口。IApplicationThread继承自IInterface。这是一个很明显的binder结构，用于与Ams通信。IApplicationThread接口定义了对一个程序（Linux的进程）操作的接口。ApplicationThread通过binder与Ams通信，并将Ams的调用，通过下面的H类（也就是Hnalder）将消息发送到消息队列，然后进行相应的操作，入activity的start， stop。

`final H mH = new H();`

这个 `H` 大家首先会想到什么啊，不要开车哈。看到 `H` 想到了 `Handler` 。发现 `H` 是 `ActivityThread` 内部类，继承自 `Handler` ，果然没错。所以大家遇到不清楚的，不要怕，大胆的猜测一下。`Handler` 最重要的的也就是 `handleMessage()` 方法了。查看一下其方法：

ActivityThread.java

```
public void handleMessage(Message ms ) {
    switch (msg.what) {
        
        case LAUNCH_ACTIVITY: {
            
            handleLaunchActivity(r, null, "LAUNCH_ACTIVITY");
        } break;
        case RESUME_ACTIVITY:
        case STOP_ACTIVITY:
    }
}

```

点进来咯。 ActivityThread.java

```
private void handleLaunchActivity(···) {
    
    Activity a = performLaunchActivity(r, custonIntent);
}

```

兴趣是最好的老师。ActivityThread.java

```
private Activity performLaunchActivity(···) {
    Activity activity = null;
    try {
         
         activity = mInstrumentation.newActivity(···);
    }
    if (r.isPersistable()) {
                    
                    mInstrumentation.callActivityOnCreate(activity, r.state, r.persistentState);
                } else {
                    mInstrumentation.callActivityOnCreate(activity, r.state);
                }
}

```

先探索一下 Activity 创建这条路吧。最底层啦。Instrumentation.java

```
public Activity newActivity(ClassLoader cl, String className, Intent intent) throws InstantiationException, illegalAccessException, ClassNotFoundException {
    
    return (Activity) cl.loadClass(className).newInstance();
}

```

Native方法，C语言啦，活动创建之路结束了。Class.java

```
public native T newInstance() throws InstantiationException, IllegalAccessException;

```

再来看看 Activity 中 onCreate() 方法执行之路吧。 Instrumentation.java

```
public void callActivityOnCreate(Activity activity, Bundle icicle,
            PersistableBundle persistentState) {
        
        prePerformCreate(activity);
        
        activity.performCreate(icicle, persistentState);
        postPerformCreate(activity);
    }

```

到了 Activity 了，哪里我们自己 Activity 还远么~！ Activity.java

```
final void performCreate(Bundle icicle) {
        restoreHasCurrentPermissionRequest(icicle);
        
        onCreate(icicle);
        mActivityTransitionState.readState(icicle);
        performCreateCommon();
    }

```

来，仔细瞅瞅~！ Activity.java

```
@MainThread 
@CallSuper 
protected void onCreate(@Nullable Bundle savedInstanceState) {
            
            if (savedInstanceState != null) {
            Parcelable p = savedInstanceState.getParcelable(FRAGMENTS_TAG);
            
            mFragments.restoreAllState(p, mLastNonConfigurationInstances != null
                    ? mLastNonConfigurationInstances.fragments : null);
        }
       
        mFragments.dispatchCreate();
}

```

"小禮物走一走，來簡書關注我"

还没有人赞赏，支持一下

[![  ](https://upload.jianshu.io/users/upload_avatars/3879403/08838752-5adf-4938-99cd-32fc3ffec233.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/100/h/100/format/webp)](https://www.jianshu.com/u/f788647332b4)

總資產3 (約0.33元)共写了6.2W字获得135个赞共48个粉丝

### 被以下專題收入，發現更多相似內容

### 推薦閱讀[更多精彩內容](https://www.jianshu.com/)

- 什么是事件分发？ 简单来说，就是我们通过屏幕与手机进行交互的时候，每次的点击，移动，长按等会产生一个个的事件。每一...
    
    [![](https://upload-images.jianshu.io/upload_images/6065113-09f3f4d14e20cf5b.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)](https://www.jianshu.com/p/4e85392552af)
- 前言 回顾一下自己这段时间的经历，三月份的时候，疫情原因公司通知了裁员，我匆匆忙忙地出去面了几家，但最终都没有拿到...
    
    [![](https://upload-images.jianshu.io/upload_images/25052971-77367e6a084a36eb?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)](https://www.jianshu.com/p/95dacea50488)
- Android面试季必问——AMS的核心原理 系列Android启动流程 https://www.jianshu...
    
    [![](https://upload-images.jianshu.io/upload_images/14355128-ca9054d66d261430.png?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)](https://www.jianshu.com/p/8f0d372b0a34)
- 在 序文 【Android 插件化的过去 现在 未来】\[https://kymjs.com/code/2016/0...
    
    [![](https://upload-images.jianshu.io/upload_images/24388310-b03e8ddb83b66114.png?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)](https://www.jianshu.com/p/cc91f17aa91e)
- 前言 现在大厂面试一般都有 3-4 轮技术面，1 轮的 HR 面。就字节跳动而言的话，是有 4 轮技术面的，前两轮...
    
    [![](https://upload-images.jianshu.io/upload_images/12798471-b7f208a4ebcdb82f.png?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)](https://www.jianshu.com/p/741af8afe2ab)