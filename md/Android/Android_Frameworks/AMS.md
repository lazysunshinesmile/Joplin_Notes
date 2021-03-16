AMS

staic SystemServer.main 实例化SystemServer 并调用run  
1. 设置系统属性,例如：`SystemProperties.set("persist.sys.timezone", "GMT");`
2. Looper.prepareMainLooper();
3. System.loadLibrary("android_servers"); // Initialize native services.
4. createSystemContext
-->
	4.1 ActivityThread.systemMain();
	4.2 mSystemContext = activityThread.getSystemContext();
	4.3 Context systemUiContext = activityThread.getSystemUiContext();
	
	
上面的4.1展开：
1. ActivityThread thread = new ActivityThread(); //1. 常量ApplicationThread mAppThread = new ApplicationThread();  2.初始化ResourcesManager 3.此类型变量每一个app一个包括系统
2. thread.attach(true, 0); 或者thread.attach(false, startSeq);// true false代表是不是系统进程。false的调用来自Activity.main方法。
-> 2.1 分系统进程还是其他进程。
3. 初始化ViewRootImpl.ConfigChangedCallback 
4. ViewRootImpl.addConfigCallback(configChangedCallback);//performConfigurationChange 会调用到。

展开2.1
1. 系统进程：
	1. 创建容器，并绑定容器。
	```java
		mInstrumentation = new Instrumentation();
		mInstrumentation.basicInit(this);//容器与ActivityThread绑定
	```
	2. 创建app上下文：
	```java
	ContextImpl context = ContextImpl.createAppContext(
                        this, getSystemContext().mPackageInfo);
	```
	--> getSystemContext() 会创建ContextImpl.createSystemContext()
		--> 初始化LoadedApk，app的信息集合
	3. 创建application
	```
	mInitialApplication = context.mPackageInfo.makeApplication(true, null);
	```
	
	--> LoadedApk makeApplication
	把contextImpl和application互相绑定。
	```java
	ContextImpl appContext = ContextImpl.createAppContext(mActivityThread, this);
	app = mActivityThread.mInstrumentation.newApplication(
			cl, appClass, appContext);
	appContext.setOuterContext(app); 
	```
	4. mActivityThread.mAllApplications.add(app);//app加入所有applications记录中，mAllApplications是final。
	5. instrumentation.callApplicationOnCreate(app);

2. 普通app
	1. final IActivityManager mgr = ActivityManager.getService();
	2. mgr.attachApplication(mAppThread, startSeq); //mAppThread看上面展开4.1的1
	3. 添加GC watcher，当内存达到3/4时，释放一些activity。ActivityManagerService.releaseSomeActivities() //1. 有activity正在destroy就不release了。正在显示，没有在stop状态的，还没有状态的，在resumed，pasuing，paused，stopping状态的，都不release。2. 如果一个activity在多个task中，把两个task拉出来比较，如果有相同的activity，且在相同的进程里，并且是可以destroy的，就destroy掉。

把普通app中2展开：ActivityManagerService.attachApplicationLocked，参数包含唤起的进程id（pid），唤起的用户id（uid）。

1. generateApplicationProvidersLocked(app) 获取app中的ContentProviders
2. thread.bindApplication




