深入PMS源码（一）—— PMS的启动过程和执行流程_浅谈Android-CSDN博客

## <a id="t0"></a><a id="t0"></a><a id="1PMS_0"></a>1、PMS简介

作为Android开发者，或多或少的都接触过Android的framework层架构，这也是开发者从使用Android到了解安卓的过程，framework层的核心功能有AMS、PMS、WMS等，这三个也是系统中最基础的使用，笔者之前分析过[Android进阶知识树——Android系统的启动过程](https://blog.csdn.net/Alexwll/article/details/100133524) 和 [Android进阶知识树——应用进程的启动过程](https://blog.csdn.net/Alexwll/article/details/100133553)，在Android程序启动完成后回启动一系列的核心服务，AMS、PMS、WMS就是在此过程中启动的，之后的系列文章之后会一次介绍他们，本篇主要介绍PMS关于PMS文章共氛围3篇，本篇最为首篇也是PMS的主要逻辑部分；

- PMS主要功能

1.  管理设备上安装的所有应用程序，并在系统启动时加载应用程序；
2.  根据请求的Intent匹配到对应的Activity、Provider、Service，提供包含包名和Component的信息对象；
3.  调用需要权限的系统函数时，检查程序是否具备相应权限从而保证系统安全；
4.  提供应用程序的安装、卸载的接口；

- PMS包管理

1.  应用程序层：使用getPackageManager（）获取包的管理对象PackageManager，PMS使用的也是Binder通信，PackageManager是从ServiceManager中获取注册的Binder对象，具体的实现为PackageManagerService，PMS实现对所有程序的安装和加载；
2.  PMS服务层：PMS运行在SystemServer进程中，主要使用/data/system/packages.xml([/data/system/packages.xml](../../Android/Android_Frameworks/_data_system_packages.xml.md))管理包信息；
3.  数据文件管理：PMS负责对系统的配置文件、apk安装文件、apk的数据文件执行管理、读写、创建和删除等功能；  
    （1）程序文件：所有系统程序的文件处于/system/app/目录下，第三方程序文件处于/data/app/目录下，在程序安装过程中PMS会将要安装的apk文件复制到/data/app/目录下，以包名命名apk文件并添加“-x”后缀，在文件更新时会修改后缀编号；  
    （2）/data/dalvik-cache/ 目录保存了程序中的执行代码，在应用程序运行前PMS会从apk中提取dex文件并保存在该目录下，以便之后能快速运行；  
    （3）对framework库文件，PMS会将其中所有的apk、jar文件中提取dex文件，将dex文件保存在/data/dalvik-cache/目录下；  
    （4）应用程序所使用的数据文件：数据以键值对保存、数据库保存、File保存所产生的文件都保存带/data/data/xxx/目录下，PMS在程序卸载会删除相应文件；

- /data/system/packages.xml ：系统的配置文件，记录所有的应用程序的包管理信息，PMS根据此文件管理所有程序

1.  last-platform-version：记录系统最后一次修改的版本信息；
2.  permissions：保存系统中所有的权限信息列表，系统权限以androd开头、自定义权限以包名开头；
3.  sigs：签名标签，一个程序只能有一个签名但可以有多个证书，包含count属性表示证书数量；
4.  cert：表示签名证书，包含index、key属性，index表示证书的下标；
5.  perms：表示一个程序中声明使用的权限列表，存在package标签之下；
6.  package：包含一个应用程序的对应关系：  
    （1）name：应用程序的包名  
    （2）codePath：程序apk文件所在的路径  
    （3）nativeLibraryPath：程序中使用的native文件路径，一般指程序包下的lib文件中导入的依赖  
    （4）flags：表示应用程序的类型  
    （5）it、ut：分别表示程序首次安装的install time、更新时间update time  
    （6）userId：表示应用程序在Linux下的用户id  
    （7）shareId：表示应用程序所共享的Linux用户Id，与userId互斥  
    （8）installer：安装器的名称，在调用PackageManager.installPackage()方法时设置的名称

## <a id="t1"></a><a id="t1"></a><a id="2PMS_34"></a>2、PMS的启动过程

在Android系统启动过程中，程序会执行到SystemServer中，然后调用startBootstrapServices()方法启动核心服务，在startBootstrapServices（）方法中完成PMS的启动：

```
private void startBootstrapServices() {
       mPackageManagerService = PackageManagerService.main(mSystemContext, installer,mFactoryTestMode != FactoryTest.FACTORY_TEST_OFF, mOnlyCore); 
    
mPackageManager = mSystemContext.getPackageManager(); 
      }

```

1.  调用PackageManagerService.main()方法，在main（）方法中创建PMS的对象，并向ServiceManager注册Binder

```
public static PackageManagerService main(Context context, Installer installer, boolean factoryTest, boolean onlyCore) {
        
        PackageManagerService m = new PackageManagerService(context, installer,factoryTest, onlyCore);
        m.enableSystemUserPackages();
        ServiceManager.addService("package", m); 
        final PackageManagerNative pmn = m.new PackageManagerNative(); 
        ServiceManager.addService("package_native", pmn); 
        return m;
    }


```

2.  调用ContextImpl.getPackageManager（）获取PackageManager对象，getPackageManager（）中使用ActivityThread.getPackageManager()获取前面创建并注册的Binder对象，然后创建ApplicationPackageManager实例

```
 @Override
public PackageManager getPackageManager() {
  IPackageManager pm = ActivityThread.getPackageManager(); 
     if (pm != null) {
     return (mPackageManager = new ApplicationPackageManager(this, pm)); 
     }
  return null;
}

```

3.  程序在获取PMS对象时会调用ActivityThread.getPackageManager()，从ServiceManager中获取Binder，并获取BInder代理对象PMS实例

```
public static IPackageManager getPackageManager() {
    if (sPackageManager != null) {
        return sPackageManager;
    }
    IBinder b = ServiceManager.getService("package”);  
    sPackageManager = IPackageManager.Stub.asInterface(b); 
    return sPackageManager;
}

```

从上面的3个过程可以得出以下结论：

1.  PMS使用Binder通信机制，最终IPackageManager接口的实现类为PackageManagerService类；
2.  系统中获取的PackageManager对象具体实现的子类是ApplicationPackageManager对象；

## <a id="t2"></a><a id="t2"></a><a id="3PMS_84"></a>3、PMS构造函数

由上面的分析知道，在系统启动后程序执行PMS的构造函数创建对象，整个系统对程序的管理就从这里开始，先介绍下相关类和属性信息：

- PMS中属性

1.  ArrayMap&lt;String, PackageParser.Package&gt; mPackages：在扫描程序文件目录时会将信息保存在Package对象中，然后将所有程序包名极其的package保存在此集合中；
2.  Settings mSettings：保存整个系统信息的Setting对象；
3.  ActivityIntentResolver mActivities：遍历所有程序的目录，并解析所有的注册清单文件，将提取所有的Intent-filter数据保存在对应的集合中；
4.  ActivityIntentResolver mReceivers：同上
5.  ServiceIntentResolver mServices：同上
6.  ProviderIntentResolver mProviders：同上

- PackageParser：解析apk文件的主要类，执行解析操作；
- PackageParser.Package：PackageParser的内部类，保存apk文件中的解析信息，每个应用程序对应一个Package对象，属性信息如下：

1.  String packageName：程序包名
2.  String codePath： 软件包的路径
3.  ApplicationInfo applicationInfo ：applicationInfo对象
4.  final ArrayList permissions：申请权限的集合
5.  final ArrayList activities：Activity标签解析的结合
6.  final ArrayList receivers：Receiver标签解析的结合
7.  final ArrayList providers ：Provider标签解析的结合
8.  final ArrayList services ：Service标签解析的结合
9.  Bundle mAppMetaData = null：注册清单中设置的信息
10. int mVersionCode：版本Code
11. String mVersionName：版本名称
12. int mCompileSdkVersion：Sdk版本

- Settings：PMS内部主要保存信息的类，主要属性如下：

1.  mSettingsFilename：配置系统目录下的package.xml文件

```
mSystemDir = new File(dataDir, "system"); 
mSettingsFilename = new File(mSystemDir, "packages.xml"); 

```

2.  mBackupSettingsFilename：配置系统目录下的packages-backup.xml文件，一般在创建和修改package文件前，会先创建packages-backup保存原来信息，在操作读写后会删除此文件；

```
mBackupSettingsFilename = new File(mSystemDir, "packages-backup.xml");

```

3.  mPackageListFilename：配置系统目录下的packages.list文件，保存了所有应用程序列表，每一行对应一个应用程序

```
mPackageListFilename = new File(mSystemDir, "packages.list”);
如：com.android.hai 10032 1 data/data/com.android.hai ，第一项：应用程序包名、第二项：Linux用户Id、第三项：1表示可以debug，0表示不能debug、第四项：程序数据文件目录

```

4.  ArrayMap&lt;String, PackageSetting&gt; mPackages：解析package.xml文件中的每个程序信息保存在PackageSetting对象中，将所有程序的PackageSetting都填充到集合中
5.  mDisabledSysPackages：保存那些没有经过正常程序卸载的应用程序列表，按照正常卸载程序时，PMS会自动删除package.xml文件中的信息，使用adb命令或其他方法删除时package文件信息则不会删除，系统启动时PMS会检查package文件，并检查对应的应用程序的文件目录，从而判断是否意外删除，如果意外删除则加入mDisabledSysPackages集合；
6.  mUserIds：保存Linux下所有的用户Id列表
7.  mPendingPackages：在解析每个PackageSetting时如果是使用sharedId，则将此Setting加入此集合，等相应的share-user标签后再补充Setting
8.  mPastSignatures：保存所有的签名文件信息
9.  mPermissions：保存所有的权限信息
10. ArraySet mInstallerPackages：已安装的应用软件包

### <a id="t3"></a><a id="t3"></a><a id="31PMS_138"></a>3.1、PMS的工作过程

- 构造函数

```
public PackageManagerService(Context context, Installer installer, 
        boolean factoryTest, boolean onlyCore) {
sUserManager = new UserManagerService(context, this, 
new UserDataPreparer(mInstaller, mInstallLock, mContext, mOnlyCore), mPackages);
mSettings = new Settings(mPermissionManager.getPermissionSettings(), mPackages); 

mHandlerThread = new ServiceThread(TAG,Process.THREAD_PRIORITY_BACKGROUND, true );
mHandlerThread.start();
mHandler = new PackageHandler(mHandlerThread.getLooper()); 

mFirstBoot = !mSettings.readLPw(sUserManager.getUsers(false)); 
}

final int packageSettingCount = mSettings.mPackages.size();
for (int i = packageSettingCount - 1; i >= 0; i--) {
    PackageSetting ps = mSettings.mPackages.valueAt(i); 
    if (!isExternal(ps) && (ps.codePath == null || !ps.codePath.exists())
            && mSettings.getDisabledSystemPkgLPr(ps.name) != null) { 
        mSettings.mPackages.removeAt(i); 
        mSettings.enableSystemPackageLPw(ps.name); 
    }
}
File frameworkDir = new File(Environment.getRootDirectory(), "framework”); 
Iterator<PackageSetting> pkgSettingIter = mSettings.mPackages.values().iterator();
while (pkgSettingIter.hasNext()) {
    PackageSetting ps = pkgSettingIter.next();
    if (isSystemApp(ps)) { 
        mExistingSystemPackages.add(ps.name);
    }
}
scanDirTracedLI(frameworkDir, 
        mDefParseFlags
        | PackageParser.PARSE_IS_SYSTEM_DIR,
        scanFlags
        | SCAN_NO_DEX
        | SCAN_AS_SYSTEM
        | SCAN_AS_PRIVILEGED,
        0);
final File systemAppDir = new File(Environment.getRootDirectory(), "app");
scanDirTracedLI(systemAppDir, 
        mDefParseFlags
        | PackageParser.PARSE_IS_SYSTEM_DIR,
        scanFlags
        | SCAN_AS_SYSTEM,
        0);
。。。。。。扫描各种目录下的文件信息
scanDirTracedLI(sAppInstallDir, 0, scanFlags | SCAN_REQUIRE_KNOWN, 0); 

mSettings.writeLPr(); 
Runtime.getRuntime().gc();

```

在启动程序后，PMS的所有工作基本都在构造函数中执行的，具体的执行过程见上面代码注释，这里列出几点主要的执行步骤：

1.  创建Settings对象，将PMS中的mPackage集合传入，此集合保存所有apk的解析数据
2.  调用readLPw（）方法解析系统配置文件package.xml
3.  调用scanDirTracedLI（）扫描系统app
4.  调用scanDirTracedLI（）扫描/data/app/下安装的第三方啊app
5.  执行mSettings.writeLPr()将扫描后的结果，重新写入配置文件

上面的几个主要过程即可实现PMS对所有安装程序的执行和管理，下面从源码的角度分析下PMS具体的执行细节；

### <a id="t4"></a><a id="t4"></a><a id="32packagexml_201"></a>3.2、解析配置文件package.xml

- readLPw()

```
 boolean readLPw(@NonNull List<UserInfo> users) {
        FileInputStream str = null; 
        if (mBackupSettingsFilename.exists()) { 
                str = new FileInputStream(mBackupSettingsFilename); 
        }
     
str = new FileInputStream(mSettingsFilename); 
XmlPullParser parser = Xml.newPullParser(); 
parser.setInput(str, StandardCharsets.UTF_8.name());

while ((type = parser.next()) != XmlPullParser.END_DOCUMENT
        && (type != XmlPullParser.END_TAG || parser.getDepth() > outerDepth)) { 
                    String tagName = parser.getName(); 
3044                if (tagName.equals("package")) {
3045                    readPackageLPw(parser);  
3046                } else if (tagName.equals("permissions")) {
3047                    mPermissions.readPermissions(parser);  
3048                } else if (tagName.equals("shared-user")) {
3051                    readSharedUserLPw(parser); 
                    }
 
 final int N = mPendingPackages.size();
3172        for (int i = 0; i < N; i++) { 
3173            final PackageSetting p = mPendingPackages.get(i);
3174            final int sharedUserId = p.getSharedUserId();
3175            final Object idObj = getUserIdLPr(sharedUserId); 
3176            if (idObj instanceof SharedUserSetting) {
3177                final SharedUserSetting sharedUser = (SharedUserSetting) idObj;
3178                p.sharedUser = sharedUser; 
3179                p.appId = sharedUser.userId;
3180                addPackageSettingLPw(p, sharedUser);  
3181            }
3192        }
3193        mPendingPackages.clear(); 
}
}

```

文件在readLPw（）中首先判断mBackupSettingsFilename文件是否存在，前面提到当更新配置文件时，系统会先将package.xml文件重名为backup文件，然后创建package.xml并将mSettings内容写入文件，写入完成之后将back-up为难删除，如果此过程发生意外则系统会保留back-up文件，再此重启PMS时会优先读取backup文件，一般情况会直接读取package.xml，读取信息后使用Xml解析文件中信息，在解析中提取每个标签中的属性，这里需要注意的是package 标签，其中包好应用程序的基本信息，程序回调用readPackageLPw（）解析；

- readPackageLPw（）：解析并获取package标签下的信息，并创建PackageSetting对象保存信息；

```
 private void readPackageLPw(XmlPullParser parser) throws XmlPullParserException, IOException {
name = parser.getAttributeValue(null, ATTR_NAME); 
realName = parser.getAttributeValue(null, "realName");
idStr = parser.getAttributeValue(null, "userId");
uidError = parser.getAttributeValue(null, "uidError");
sharedIdStr = parser.getAttributeValue(null, "sharedUserId");
codePathStr = parser.getAttributeValue(null, "codePath");
.......
version = parser.getAttributeValue(null, "version");
timeStampStr = parser.getAttributeValue(null, "it");
timeStampStr = parser.getAttributeValue(null, "ut");

packageSetting = addPackageLPw(name.intern(), realName, new File(codePathStr), 
3836                        new File(resourcePathStr), legacyNativeLibraryPathStr, primaryCpuAbiString,
3837                        secondaryCpuAbiString, cpuAbiOverrideString, userId, versionCode, pkgFlags,
3838                        pkgPrivateFlags, parentPackageName, null ,
3839                        null , null );
packageSetting.setTimeStamp(timeStamp);
packageSetting.firstInstallTime = firstInstallTime;
packageSetting.lastUpdateTime = lastUpdateTime;

if (sharedIdStr != null) {
3853                if (sharedUserId > 0) {
3854                    packageSetting = new PackageSetting(name.intern(), realName, new File(
3855                            codePathStr), new File(resourcePathStr), legacyNativeLibraryPathStr,
3856                            primaryCpuAbiString, secondaryCpuAbiString, cpuAbiOverrideString,
3857                            versionCode, pkgFlags, pkgPrivateFlags, parentPackageName,
3858                            null , sharedUserId,
3859                            null , null );
3860                    packageSetting.setTimeStamp(timeStamp);
3861                    packageSetting.firstInstallTime = firstInstallTime;
3862                    packageSetting.lastUpdateTime = lastUpdateTime;
3863                    mPendingPackages.add(packageSetting); 
3873            }
if (installerPackageName != null) {
    mInstallerPackages.add(installerPackageName); 
}

```

在readPackageLPw（）中首先从解析器parser中提取所有的标签信息，然后调用addPackageLPw（）方法保存这些属性，最后当程序使用shareId时先暂时将对象的解析 添加到mPendingPackages集合中，等共享的应用执行结束后再补充信息；

- addPackageLPw（）

```
PackageSetting addPackageLPw(String name,......) {
598        PackageSetting p = mPackages.get(name); 
599        if (p != null) {
600            if (p.appId == uid) {
601                return p;
602            }
606        }
607    p = new PackageSetting(name, realName, codePath, resourcePath, 
608                legacyNativeLibraryPathString, primaryCpuAbiString, secondaryCpuAbiString,
609                cpuAbiOverrideString, vc, pkgFlags, pkgPrivateFlags, parentPackageName,
610                childPackageNames, 0 , usesStaticLibraries, usesStaticLibraryNames);
611        p.appId = uid;
612        if (addUserIdLPw(uid, p, name)) {
613            mPackages.put(name, p); 
614            return p;
615        }
616        return null;
617    }
}

```

addPackageLPw（）中直接创建PackageSetting对象，将解析的信息封装起来，然后以程序的name为Key将PackageSetting对象添加的mPackages集合中，那此时mPackages就保存了手机中所有app的应用信息；

### <a id="t5"></a><a id="t5"></a><a id="33_311"></a>3.3、扫描安装的应用程序

- scanDirTracedLI()：扫描/data/app/目录文件下的所有apk文件信息，在scanDirTracedLI（）中直接调用scanDirLI（）方法

```
private void scanDirLI(File scanDir, int parseFlags, int scanFlags, long currentTime) {
final File[] files = scanDir.listFiles(); 
try (ParallelPackageParser parallelPackageParser = new ParallelPackageParser(
        mSeparateProcesses, mOnlyCore, mMetrics, mCacheDir,
        mParallelPackageParserCallback)) { 
    int fileCount = 0;
    for (File file : files) {
        parallelPackageParser.submit(file, parseFlags); 
        fileCount++;
    }
    for (; fileCount > 0; fileCount--) {
        ParallelPackageParser.ParseResult parseResult = parallelPackageParser.take();
        scanPackageChildLI(parseResult.pkg, parseFlags, scanFlags,currentTime, null); 
  }
}

```

scanDirLI（）方法执行的逻辑很简单：

1.  遍历文件目录下的所有文件
2.  创建ParallelPackageParser对象，调用submit（）方法提交请求，执行解析每个apk文件
3.  调用parallelPackageParser.take()逐个取出每个解析的结果

到这里我们知道PMS是对/data/app/目录中所有apk文件进行解析，在之前的版本中会直接创建PackageParser对象执行解析，在Androip P版本中引入ParallelPackageParser类，使用线程池和队列执行程序的解析；

- ParallelPackageParser：内部使用线程池和队列执行文件目录中apk扫描解析

```
private final BlockingQueue<ParseResult> mQueue = new ArrayBlockingQueue<>(QUEUE_CAPACITY); 
private final ExecutorService mService = ConcurrentUtils.newFixedThreadPool(MAX_THREADS,
        "package-parsing-thread", Process.THREAD_PRIORITY_FOREGROUND); 
        
public void submit(File scanFile, int parseFlags) {
    mService.submit(() -> { 
        ParseResult pr = new ParseResult();
        try {
            PackageParser pp = new PackageParser();
            pp.setSeparateProcesses(mSeparateProcesses);
            pp.setOnlyCoreApps(mOnlyCore);
            pp.setDisplayMetrics(mMetrics);
            pp.setCacheDir(mCacheDir);
            pp.setCallback(mPackageParserCallback);
            pr.scanFile = scanFile;
            pr.pkg = parsePackage(pp, scanFile, parseFlags); 
        }
        try {
            mQueue.put(pr); 
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            mInterruptedInThread = Thread.currentThread().getName();
        }
    });
}
public ParseResult take() {
    try {
        return mQueue.take(); 
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
        throw new IllegalStateException(e);
    }
}

```

ParallelPackageParser中利用线程池处理并发问题，执行多个apk文件的解析，并使用阻塞队列的方式同步线程的数据，在submit提交的任务run（）方法中，创建了PackageParser对象并调用parser（）方法解析apk，之后将解析的结果封装在ParseResult中，最后添加到mQueue队列中，PMS中在依次调用take（）方法从mQueue队列中获取执行的结果；

- PackageParser.parsePackage（）：解析每个apk文件

```
public Package parsePackage(File packageFile, int flags, boolean useCaches)
        throws PackageParserException {
    Package parsed = useCaches ? getCachedResult(packageFile, flags) : null; 
    if (parsed != null) {
        return parsed;
    }
    if (packageFile.isDirectory()) { 
        parsed = parseClusterPackage(packageFile, flags); 
    } else {
        parsed = parseMonolithicPackage(packageFile, flags); 
    }
    cacheResult(packageFile, flags, parsed); 
    return parsed;
}

```

ParserPackage.parser（）中开始执行apk文件的解析，对于apk文件执行parseMonolithicPackage（），在执行解析结束后会缓存解析结果；

```
public Package parseMonolithicPackage(File apkFile, int flags) throws PackageParserException {
    final PackageLite lite = parseMonolithicPackageLite(apkFile, flags);
    final SplitAssetLoader assetLoader = new DefaultSplitAssetLoader(lite, flags); 
    try {
        final Package pkg = parseBaseApk(apkFile, assetLoader.getBaseAssetManager(), flags); 
        pkg.setCodePath(apkFile.getCanonicalPath());
        pkg.setUse32bitAbi(lite.use32bitAbi);
        return pkg;
    } 
}

```

- parseMonolithicPackageLite（）:先解析出apk文件的基本信息

```
private static PackageLite parseMonolithicPackageLite(File packageFile, int flags) throws PackageParserException {
        final ApkLite baseApk = parseApkLite(packageFile, flags);
        final String packagePath = packageFile.getAbsolutePath();
    
        return new PackageLite(packagePath, baseApk, null, null, null, null, null, null);
    }


 private static ApkLite parseApkLite(String codePath, XmlPullParser parser, AttributeSet attrs,SigningDetails signingDetails)
            throws IOException, XmlPullParserException, PackageParserException {
        final Pair<String, String> packageSplit = parsePackageSplitNames(parser, attrs);
        for (int i = 0; i < attrs.getAttributeCount(); i++) {
            final String attr = attrs.getAttributeName(i);
            if (attr.equals("installLocation")) {
                installLocation = attrs.getAttributeIntValue(i,
                        PARSE_DEFAULT_INSTALL_LOCATION);
            } else if (attr.equals("versionCode")) {
                versionCode = attrs.getAttributeIntValue(i, 0);
            } else if (attr.equals("versionCodeMajor")) {
                versionCodeMajor = attrs.getAttributeIntValue(i, 0);
            } else if (attr.equals("revisionCode")) {
                revisionCode = attrs.getAttributeIntValue(i, 0);
            } 
            .....
        }
         ......
        return new ApkLite(codePath, packageSplit.first, packageSplit.second, isFeatureSplit,
                configForSplit, usesSplitName, versionCode, versionCodeMajor, revisionCode,
                installLocation, verifiers, signingDetails, coreApp, debuggable,
                multiArch, use32bitAbi, extractNativeLibs, isolatedSplits);
    }

```

在parseMonolithicPackage（）中先调用parseApkLite（）将File先简单的解析以下，这里的解析只是获取注册清单中的基础信息，并将信息保存在ApkLite对象中，然后将ApkLite和文件路径  
封装在PackageLite对象中；

- DefaultSplitAssetLoader：内部使用AssestManager加载apk文件的资源，并缓存AssestManager对象信息，主要针对split APK；

- parseBaseApk（）：解析每个apk文件

```
private Package parseBaseApk(File apkFile, AssetManager assets, int flags)
        throws PackageParserException {
        final String apkPath = apkFile.getAbsolutePath(); 
         mArchiveSourcePath = sourceFile.getPath(); 
399        int cookie = assmgr.findCookieForPath(mArchiveSourcePath); 
401        parser = assmgr.openXmlResourceParser(cookie, "AndroidManifest.xml”); 
           Resources res = new Resources(assmgr, metrics, null); 
           pkg = parseBaseApk(res, parser, flags, errorText); 
pkg.setVolumeUuid(volumeUuid);
pkg.setApplicationVolumeUuid(volumeUuid);
pkg.setBaseCodePath(apkPath);
pkg.setSigningDetails(SigningDetails.UNKNOWN);
return pkg;
}

```

parseBaseApk（）中从apkFile对象中获取apk文件路径，然后使用assmgr加载apk文件中的资源，从文件中读取注册清单文件，然后调用parseBaseApk解析注册清单；

1.  parseBaseApk（）：从Parser对象中获取解析的信息，保存在Package对象中

```
 private Package parseBaseApk(String apkPath,Resources res, XmlResourceParser parser, int flags, String[] outError){
Pair<String, String> packageSplit = parsePackageSplitNames(parser, parser); 
pkgName = packageSplit.first;
splitName = packageSplit.second;
Package pkg = new Package(pkgName); 

 pkg.mVersionCode = sa.getInteger(…...
pkg.mVersionName = sa.getNonConfigurationString(…...
 pkg.mSharedUserId = str.intern(…...);
pkg.mSharedUserLabel = sa.getResourceId(…...
pkg.installLocation = sa.getInteger(…...
pkg.applicationInfo.installLocation = pkg.installLocation;
return parseBaseApkCommon(pkg, null, res, parser, flags, outError); 
}

```

parseBaseApk中从parser中提取注册清单中的基础信息，并封装保存在Pakage对象中，然后调用parseBaseApkCommon（）方法继续解析清单文件中内容

- parseBaseApkCommon（）

```

int outerDepth = parser.getDepth(); 
while ((type=parser.next()) != parser.END_DOCUMENT. 
804               && (type != parser.END_TAG || parser.getDepth() > outerDepth)) {
 if (tagName.equals("application")) {
if (!parseBaseApplication(pkg, res, parser, attrs, flags, outError)) { 
825                    return null;
}else if (tagName.equals("permission")) { 
832                if (parsePermission(pkg, res, parser, attrs, outError) == null) {
833                    return null;
834                }
} else if (tagName.equals("uses-feature")) { 
 FeatureInfo fi = new FeatureInfo();
    …….
pkg.reqFeatures.add(fi);
}else if (tagName.equals("uses-sdk")) { 
}else if (tagName.equals("supports-screens")) { 
}
}
}

```

parseBaseApkCommon（）中主要负责解析清单文件中的各种标签信息，其中最主要的就是解析标签下的四大组件的信息，在遇到applicaiton标签时直接调用了parseBaseApplication（）执行解析；

- parseBaseApplication（），主要的解析工作

```
String tagName = parser.getName();
final ApplicationInfo ai = owner.applicationInfo;
ai.theme = sa.getResourceId(
        com.android.internal.R.styleable.AndroidManifestApplication_theme, 0);
ai.descriptionRes = sa.getResourceId(
        com.android.internal.R.styleable.AndroidManifestApplication_description, 0);
ai.maxAspectRatio = sa.getFloat(R.styleable.AndroidManifestApplication_maxAspectRatio, 0);


1644            if (tagName.equals("activity")) { 
1645                Activity a = parseActivity(owner, res, parser, attrs, flags, outError, false);
1651                owner.activities.add(a);
1653            } else if (tagName.equals("receiver")) {
1654                Activity a = parseActivity(owner, res, parser, attrs, flags, outError, true);
1660                owner.receivers.add(a);
1662            } else if (tagName.equals("service")) {
1663                Service s = parseService(owner, res, parser, attrs, flags, outError);
1669                owner.services.add(s);
1671            } else if (tagName.equals("provider")) {
1672                Provider p = parseProvider(owner, res, parser, attrs, flags, outError);
1678                owner.providers.add(p);
1680            }

```

清单文件解析共分两部分：

1.  解析出application标签下设置的name类名、icon、theme、targetSdk、processName等属性标签并保存在ApplicationInfo对象中
2.  循环解析activity、receiver、service、provider四个标签，并将信息到保存在Package中对应的集合中

下面逐个分析下四大组件是如何解析保存的：

- parserActivity（）：解析Activity和Receiver标签并返回Activity对象封装所有属性；

```
private Activity parseActivity(Package owner,...)
        throws XmlPullParserException, IOException {
TypedArray sa = res.obtainAttributes(parser, R.styleable.AndroidManifestActivity);
cachedArgs.mActivityArgs.tag = receiver ? "<receiver>" : "<activity>”; 
cachedArgs.mActivityArgs.sa = sa;
cachedArgs.mActivityArgs.flags = flags;
Activity a = new Activity(cachedArgs.mActivityArgs, new ActivityInfo()); 
a.info.theme = sa.getResourceId(R.styleable.AndroidManifestActivity_theme, 0);
a.info.taskAffinity = buildTaskAffinityName(owner.applicationInfo.packageName,
        owner.applicationInfo.taskAffinity, str, outError);
a.info.launchMode = ...
if (parser.getName().equals("intent-filter")) { 
    ActivityIntentInfo intent = new ActivityIntentInfo(a);
    if (!parseIntent(res, parser, true , true ,
            intent, outError)) {
        return null;
    }
        a.order = Math.max(intent.getOrder(), a.order);
        a.intents.add(intent); 
} 
return a;

```

在解析Activity和Receiver标签时，当标签设置intent-filter时则创建一个ActivityIntentInfo对象，并调用parseIntent（）将intent-filter标签下的信息解析到ActivityIntentInfo中，并将ActivityIntentInfo对象保存在a.intents的集合中，简单的说一个intent-filter对应一个ActivityIntentInfo对象，一个Activity和Receiver可以包好多个intent-filter；

1.  parseIntent()：解析每个intent-filter标签下的action、name等属性值，并将所有的属性值保存在outInfo对象中，这里的outInfo是ActivityIntentInfo对象；

```
while ((type = parser.next()) != XmlPullParser.END_DOCUMENT
        && (type != XmlPullParser.END_TAG || parser.getDepth() > outerDepth)) {
    String nodeName = parser.getName(); 
    if (nodeName.equals("action")) { 
    String value = parser.getAttributeValue( 
        ANDROID_RESOURCES, "name");
    outInfo.addAction(value); 
} else if (nodeName.equals("category")) {
    String value = parser.getAttributeValue( 
            ANDROID_RESOURCES, "name");
    outInfo.addCategory(value); 
} else if (nodeName.equals("data")) {
sa = res.obtainAttributes(parser,
        com.android.internal.R.styleable.AndroidManifestData); 
String str = sa.getNonConfigurationString(
        com.android.internal.R.styleable.AndroidManifestData_mimeType, 0);
        outInfo.addDataType(str); 
str = sa.getNonConfigurationString(
        com.android.internal.R.styleable.AndroidManifestData_scheme, 0);
    outInfo.addDataScheme(str); 
String host = sa.getNonConfigurationString(
        com.android.internal.R.styleable.AndroidManifestData_host, 0); 
String port = sa.getNonConfigurationString(
        com.android.internal.R.styleable.AndroidManifestData_port, 0);
    outInfo.addDataAuthority(host, port); 
outInfo.hasDefault = outInfo.hasCategory(Intent.CATEGORY_DEFAULT); 
}

```

2.  ActivityIntentInfo 继承 IntentInfo ，IntentInfo继承IntentFilter属性，所以outInfo保存的数据都保存在IntentFilter中，在IntentFilter中有标签对应的集合，如actions、mDataSchemes等，所以遍历的所有intent-filter中设置的数据都保存在其中；

```
public final static class ActivityIntentInfo extends IntentInfo {}
public static abstract class IntentInfo extends IntentFilter {}
public final void addAction(String action) {
    if (!mActions.contains(action)) {
        mActions.add(action.intern()); 
    }
}
public final void addCategory(String category) {
    if (mCategories == null) mCategories = new ArrayList<String>();
    if (!mCategories.contains(category)) {
        mCategories.add(category.intern()); 
    }
}
public final void addDataScheme(String scheme) {
    if (mDataSchemes == null) mDataSchemes = new ArrayList<String>();
    if (!mDataSchemes.contains(scheme)) {
        mDataSchemes.add(scheme.intern()); 
    }
}

```

- parserService（）：解析Service标签并返回Service对象，对应service标签下的信息保存在ServiceIntentInfo对象中，ServiceIntentInfo的作用和保存和ActivityIntentInfo一样

```
TypedArray sa = res.obtainAttributes(parser,
        com.android.internal.R.styleable.AndroidManifestService);
cachedArgs.mServiceArgs.sa = sa;
cachedArgs.mServiceArgs.flags = flags;
Service s = new Service(cachedArgs.mServiceArgs, new ServiceInfo()); 
if (parser.getName().equals("intent-filter")) { 
    ServiceIntentInfo intent = new ServiceIntentInfo(s);
    if (!parseIntent(res, parser, true , false ,
            intent, outError)) {
        return null;
    }
    s.order = Math.max(intent.getOrder(), s.order);
    s.intents.add(intent);
} else if (parser.getName().equals("meta-data")) { 
    if ((s.metaData=parseMetaData(res, parser, s.metaData,
            outError)) == null) {
        return null;
    }
}

```

- parserProvider（）：解析provider标签并返回provider对象

```
TypedArray sa = res.obtainAttributes(parser,
        com.android.internal.R.styleable.AndroidManifestProvider);
cachedArgs.mProviderArgs.tag = "<provider>";
cachedArgs.mProviderArgs.sa = sa;
cachedArgs.mProviderArgs.flags = flags;

Provider p = new Provider(cachedArgs.mProviderArgs, new ProviderInfo()); 
p.info.exported = sa.getBoolean(
        com.android.internal.R.styleable.AndroidManifestProvider_exported,
        providerExportedDefault);
String permission = sa.getNonConfigurationString(
        com.android.internal.R.styleable.AndroidManifestProvider_permission, 0);
p.info.multiprocess = sa.getBoolean(
        com.android.internal.R.styleable.AndroidManifestProvider_multiprocess,
        false);
p.info.authority = cpname.intern();
if (!parseProviderTags( 
        res, parser, visibleToEphemeral, p, outError)) {
    return null;
}

```

在解析provider标签中，创建ProviderInfo对象保存设置的属性信息，如export、permission等，然后调用parseProviderTags解析provider标签中使用的其他标签信息

1.  parseProviderTags（）：

```
private boolean parseProviderTags(Resources res, XmlResourceParser parser,
        boolean visibleToEphemeral, Provider outInfo, String[] outError){
    int outerDepth = parser.getDepth();
    int type;
    while ((type=parser.next()) != XmlPullParser.END_DOCUMENT
           && (type != XmlPullParser.END_TAG
                   || parser.getDepth() > outerDepth)) {
if (parser.getName().equals("intent-filter")) {
    ProviderIntentInfo intent = new ProviderIntentInfo(outInfo);
    if (!parseIntent(res, parser, true , false ,
            intent, outError)) {
        return false;
    }
    outInfo.order = Math.max(intent.getOrder(), outInfo.order);
    outInfo.intents.add(intent);
} 
if (parser.getName().equals("meta-data")) {
    if ((outInfo.metaData=parseMetaData(res, parser,
            outInfo.metaData, outError)) == null) {
        return false;
    }
} 
if (parser.getName().equals("grant-uri-permission")) {
String str = sa.getNonConfigurationString(
        com.android.internal.R.styleable.AndroidManifestGrantUriPermission_path, 0);
PatternMatcher   pa = new PatternMatcher(str, PatternMatcher.PATTERN_LITERAL); 
PatternMatcher[] newp = new PatternMatcher[N+1];
newp[N] = pa;
outInfo.info.uriPermissionPatterns = newp; 
}
if (parser.getName().equals("path-permission")) { 
newp[N] = pa;
outInfo.info.pathPermissions = newp;
}
}
}

```

主要解析如下：

1.  对于intent-filter标签，调用parserIntent（）解析保存在ProviderIntentInfo对象中，并添加到intent集合中
2.  解析设置的meta-data值
3.  解析grant-uri-permission和path-permission等权限的匹配状况保存在Provider对象中的数组中；

到此apk文件已经解析完成，相应的文件和属性都封装在Package对象中，并都保存在PMS属性集合mPackages集合中；

### <a id="t6"></a><a id="t6"></a><a id="33apkPMS_720"></a>3.3、将apk解析数据同步到PMS的属性中

在使用线程池执行所有的apk解析后，所有的解析结果都保存在队列中，系统会循环调用take（）方法取出解析的结果

```
for (; fileCount > 0; fileCount--) {
                ParallelPackageParser.ParseResult parseResult = parallelPackageParser.take();
                
scanPackageChildLI(parseResult.pkg, parseFlags, scanFlags,
                                    currentTime, null);
                }

```

取出apk文件解析结果后，调用scanPackageChildLI（）扫描获取到的ParseResult中的Pakcage对象，scanPackageChildLI（）中直接调用addForInitLI（）方法

- addForInitLI（）

```
synchronized (mPackages) {
final PackageSetting installedPkgSetting = mSettings.getPackageLPr(pkg.packageName); 

final PackageParser.Package scannedPkg = scanPackageNewLI(pkg, parseFlags, scanFlags | SCAN_UPDATE_SIGNATURE, currentTime, user)； 
}


final ScanRequest request = new ScanRequest(pkg, sharedUserSetting,
        pkgSetting == null ? null : pkgSetting.pkg, pkgSetting, disabledPkgSetting,
        originalPkgSetting, realPkgName, parseFlags, scanFlags,
        (pkg == mPlatformPackage), user);
final ScanResult result = scanPackageOnlyLI(request, mFactoryTest, currentTime); 
if (result.success) {
    commitScanResultsLocked(request, result); 
}

```

commitScanResultsLocked（）中直接调用commitPackageSettings（）调教apk的解析数据

```
commitPackageSettings(pkg, oldPkg, pkgSetting, user, scanFlags,
(parseFlags & PackageParser.PARSE_CHATTY) != 0 ); 

```

- commitPackageSettings（）：将解析得到的Package中的信息保存到PMS内部变量中，并创建程序包所需的各种文件信息

```
final String pkgName = pkg.packageName;
if (pkg.packageName.equals("android")) { 
                   mPlatformPackage = pkg;
2897                pkg.mVersionCode = mSdkVersion;
2898                mAndroidApplication = pkg.applicationInfo;
2899                mResolveActivity.applicationInfo = mAndroidApplication;
2900                mResolveActivity.name = ResolverActivity.class.getName();
。。。。。。
2912                mResolveComponentName = new ComponentName(
2913             mAndroidApplication.packageName, mResolveActivity.name);
}
                int N = pkg.usesLibraries != null ? pkg.usesLibraries.size() : 0;
2950                for (int i=0; i<N; i++) {
2951                String file = mSharedLibraries.get(pkg.usesLibraries.get(i)); 
                    mTmpSharedLibraries[num] = file;   
2960                num++;
2961                }
            if (!verifySignaturesLP(pkgSetting, pkg)) { 
  ……. 
}
                int N = pkg.providers.size();
3404            StringBuilder r = null;
3405            int i;
3406            for (i=0; i<N; i++) {
3407            PackageParser.Provider p = pkg.providers.get(i); 
3408                p.info.processName = fixProcessName(pkg.applicationInfo.processName,
3409                        p.info.processName, pkg.applicationInfo.uid);
3410                mProvidersByComponent.put(new ComponentName(p.info.packageName,
3411                        p.info.name), p); 
3413                if (p.info.authority != null) {
3414                    String names[] = p.info.authority.split(";”); 
3415                    p.info.authority = null;
3416                    for (int j = 0; j < names.length; j++) {
3417                        if (j == 1 && p.syncable) {
3425                            p = new PackageParser.Provider(p);
3426                            p.syncable = false;
3427                        }
3428                        if (!mProviders.containsKey(names[j])) {
3429                            mProviders.put(names[j], p); 
3447                    }
3448                }
3457            }
3460            }

N = pkg.services.size();
3463            r = null;
3464            for (i=0; i<N; i++) {
3465                PackageParser.Service s = pkg.services.get(i);
3466                s.info.processName = fixProcessName(pkg.applicationInfo.processName,
3467                        s.info.processName, pkg.applicationInfo.uid);
3468                mServices.addService(s);
N = pkg.receivers.size();
3483            r = null;
3484            for (i=0; i<N; i++) {
3485                PackageParser.Activity a = pkg.receivers.get(i);
3486                a.info.processName = fixProcessName(pkg.applicationInfo.processName,
3487                        a.info.processName, pkg.applicationInfo.uid);
3488                mReceivers.addActivity(a, "receiver");
3497            }

 N = pkg.activities.size();
3503            r = null;
3504            for (i=0; i<N; i++) {
3505                PackageParser.Activity a = pkg.activities.get(i);
3506                a.info.processName = fixProcessName(pkg.applicationInfo.processName,
3507                        a.info.processName, pkg.applicationInfo.uid);
3508                mActivities.addActivity(a, "activity");
}


```

在commitPackageSettings中，主要是将每个apk文件获得的Package对象中保存的四大组件信息分别提取保存在PMS内部对应的属性中，在PMS内部有4个专门储存四大组件的属性：

```
final ActivityIntentResolver mActivities = new ActivityIntentResolver();
final ActivityIntentResolver mReceivers = new ActivityIntentResolver();
final ServiceIntentResolver mServices = new ServiceIntentResolver();
final ProviderIntentResolver mProviders = new ProviderIntentResolver();

```

- ActivityIntentResolver.addActivity()：处理Activity和Receiver分别保存在各自的ArrayMap中

```
public final void addActivity(PackageParser.Activity a, String type) {
    mActivities.put(a.getComponentName(), a); 
    final int NI = a.intents.size();
    for (int j=0; j<NI; j++) {
        PackageParser.ActivityIntentInfo intent = a.intents.get(j); 
        addFilter(intent); 
    }
}

```

- ServiceIntentResolver.addActivity()：将Package中极细获取的Service对象，保存在ArrayMap中

```
public final void addService(PackageParser.Service s) {
    mServices.put(s.getComponentName(), s); 
    final int NI = s.intents.size();
    int j;
    for (j=0; j<NI; j++) {
        PackageParser.ServiceIntentInfo intent = s.intents.get(j);
        addFilter(intent);
    }
}

```

- ProviderIntentResolver.addActivity()：将Package中极细获取的Provider对象，保存在ArrayMap中

```
public final void addProvider(PackageParser.Provider p) {
    if (mProviders.containsKey(p.getComponentName())) {
        return;
    }
    mProviders.put(p.getComponentName(), p); 
  
    final int NI = p.intents.size();
    int j;
    for (j = 0; j < NI; j++) {
        PackageParser.ProviderIntentInfo intent = p.intents.get(j);
        addFilter(intent); 
    }
}

```

### <a id="t7"></a><a id="t7"></a><a id="34_882"></a>3.4、更新配置文件

- mSettings.writeLPr（）：将mPackages中的数据分别写入package.xml和package.list文件中

```
  void writeLPr() {
        if (mSettingsFilename.exists()) {
            if (!mBackupSettingsFilename.exists()) { 
                if (!mSettingsFilename.renameTo(mBackupSettingsFilename)) {
                    Slog.wtf(PackageManagerService.TAG,
                            "Unable to backup package manager settings, "
                            + " current changes will be lost at reboot");
                    return;
                }
            } else {
                mSettingsFilename.delete(); 
            }
        }
FileOutputStream fstr = new FileOutputStream(mSettingsFilename);
BufferedOutputStream str = new BufferedOutputStream(fstr);
 。。。。。。
 for (final PackageSetting pkg : mPackages.values()) {
 writePackageLPr(serializer, pkg); 
 }
 .......
}
writePackageListLPr(); 

```

1.  先判断package.xml和backup.xml文件是否存在，如果两个都存在则删除package.xml
2.  如果backup.xml文件不存在，则将package.xml重命名为backup。xml
3.  创建新的package.xml文件，并将mSetting中的内容写入文件夹
4.  删除backup文件，并重新生成package.list文件

到此PMS的启动过程介绍完毕，简单来说系统在启动会会创建PMS对象，使用PMS对象读取配置文件，然后扫描手机上所有的app程序，并将所有的程序的内容信息都封装在Package对象中，然后将Package集合中信息转换为PMS的属性供系统使用，最后并更新配置文件；