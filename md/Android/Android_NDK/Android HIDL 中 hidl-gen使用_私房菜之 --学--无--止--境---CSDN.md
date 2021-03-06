Android HIDL 中 hidl-gen使用_私房菜之 --学--无--止--境---CSDN博客

在 [Android HIDL 详解](https://blog.csdn.net/jingerppp/article/details/86514997) 一文提到HIDL 使用的整个过程都是跟其工具hidl-gen 分不开，这一篇来详细分析hidl-gen 的使用。

代码基于：Android P

hidl-gen 的代码路径为：system/tools/hidl

```
    defaults: ["hidl-gen-defaults"],"ConstantExpression.cpp","DeathRecipientType.cpp",
```

编译之后会在out 下生成，详细看out/host/linux-x86/bin/hidl-gen

编译好之后可以通过详细路径直接使用，也可以设置环境变量、lunch之后使用：

```
usage: hidl-gen [-p <root path>] -o <output path> -L <language> [-O <owner>] (-r <interface root>)+ [-v] [-d <depfile>] FQNAME...Process FQNAME, PACKAGE(.SUBPACKAGE)*@[0-9]+.[0-9]+(::TYPE)?, to create output.         -L <language>: The following options are available:            check           : Parses the interface to see if valid but doesn't write any files.            c++             : (internal) (deprecated) Generates C++ interface files for talking to HIDL interfaces.            c++-headers     : (internal) Generates C++ headers for interface files for talking to HIDL interfaces.            c++-sources     : (internal) Generates C++ sources for interface files for talking to HIDL interfaces.            export-header   : Generates a header file from @export enumerations to help maintain legacy code.            c++-impl        : Generates boilerplate implementation of a hidl interface in C++ (for convenience).            c++-impl-headers: c++-impl but headers only            c++-impl-sources: c++-impl but sources only            c++-adapter     : Takes a x.(y+n) interface and mocks an x.y interface.            c++-adapter-headers: c++-adapter but helper headers only            c++-adapter-sources: c++-adapter but helper sources only            c++-adapter-main: c++-adapter but the adapter binary source only            java            : (internal) Generates Java library for talking to HIDL interfaces in Java.            java-constants  : (internal) Like export-header but for Java (always created by -Lmakefile if @export exists).            vts             : (internal) Generates vts proto files for use in vtsd.            makefile        : (removed) Used to generate makefiles for -Ljava and -Ljava-constants.            androidbp       : (internal) Generates Soong bp files for -Lc++-headers, -Lc++-sources, -Ljava, -Ljava-constants, and -Lc++-adapter.            androidbp-impl  : Generates boilerplate bp files for implementation created with -Lc++-impl.            hash            : Prints hashes of interface in `current.txt` format to standard out.         -O <owner>: The owner of the module for -Landroidbp(-impl)?.         -o <output path>: Location to output files.         -p <root path>: Android build root, defaults to $ANDROID_BUILD_TOP or pwd.         -r <package:path root>: E.g., android.hardware:hardware/interfaces.         -d <depfile>: location of depfile to write to.
```

在创建对应模块的hal 文件之后，通过hidl-gen 就可以简单的创建一些开发的文件。

## <a id="t2"></a><a id="t2"></a>update-makefiles.sh

通过/hardware/interfaces/update-makefiles.sh 可以创建编译HIDL 文件的Android.bp。

假设我们创建一个helloworld的模块，在hardware/interfaces 下创建helloworld/1.0/IHelloWorld.hal：

```
package android.hardware.helloworld@1.0;                                                               
```

通过update-makefiles.sh 就可以在对应package 的目录下创建Android.bp：

```
name: "android.hardware.helloworld@1.0",root: "android.hardware",
```

name：FQName 的全名

root：定义好的package root name，详细看[Android HIDL 接口和软件包使用](https://blog.csdn.net/jingerppp/article/details/86526547)

interfaces：编译过程中依赖的接口名称，如c 中的shared library

gen_java：是否编译为Java 使用的接口

当然，还有其他的参数，例如gen\_java\_constants设为true 的时候会生成为Java 使用的Constants类。

可以根据实际情况修改该Android.bp

## <a id="t3"></a><a id="t3"></a>update-files.sh

IHelloWorld.hal 和对应的Android.bp 创建好后，就可以根据需要实现FQName-impl 和FQName-service所需要的文件，而这些文件也是通过hidl-gen 创建的，就是下面这个脚本。

```
PACKAGE=android.hardware.helloworld@1.0LOC=hardware/interfaces/helloworld/1.0/default/hidl-gen -o $LOC -Lc++-impl -randroid.hardware:hardware/interfaces \        -randroid.hidl:system/libhidl/transport $PACKAGEhidl-gen -o $LOC -Landroidbp-impl -randroid.hardware:hardware/interfaces \        -randroid.hidl:system/libhidl/transport $PACKAGE
```

可以看到创建的是c++-impl 所需要的文件以及对应的Android.bp。执行后如下：

```
~/work/LA.UM.7.6/LINUX/android/hardware/interfaces/helloworld/1.0/default:drwxr-xr-x 2 shift shift 4096 1月  17 20:05 ./drwxrwxr-x 3 shift shift 4096 1月  17 20:05 ../-rw-rw-r-- 1 shift shift  973 1月  17 20:05 Android.bp-rw-rw-r-- 1 shift shift  605 1月  17 20:05 HelloWorld.cpp-rw-rw-r-- 1 shift shift 1159 1月  17 20:05 HelloWorld.h
```

创建后的Android.bp如下：

```
    // FIXME: this should only be -impl for a passthrough hal.                                             // In most cases, to convert this to a binderized implementation, you should:                          // - change '-impl' to '-service' here and make it a cc_binary instead of a                            // - add a *.rc file for this module.                                                                  // - delete HIDL_FETCH_I* functions.                                                                   // - call configureRpcThreadpool and registerAsService on the instance.                                // You may also want to append '-impl/-service' with a specific identifier like                        // '-vendor' or '-<hardware identifier>' etc to distinguish it.                                        name: "android.hardware.helloworld@1.0-impl",                                                          relative_install_path: "hw",                                                                           // FIXME: this should be 'vendor: true' for modules that will eventually be                        "android.hardware.helloworld@1.0",                                                             
```

如注释部分，可以根据特殊的需要进行修改，例如，vendor 需要设为true，这样编译出来的so位于vendor下面，而不是system。也可以使用同样的方式为FQName-service 创建对应的规则。

关于HIDL 在helloworld的详细使用可以看另一篇博文：[Android HIDL 实例](https://blog.csdn.net/jingerppp/article/details/86530600)

相关文章：

[Android Treble 简介](https://blog.csdn.net/jingerppp/article/details/86513675)

[Android HIDL 详解](https://blog.csdn.net/jingerppp/article/details/86514997)

[Android HIDL 编程规范](https://blog.csdn.net/jingerppp/article/details/86525761)

[Android HIDL 接口和软件包使用](https://blog.csdn.net/jingerppp/article/details/86526547)

[Android HIDL 实例](https://blog.csdn.net/jingerppp/article/details/86530600)

[Android HIDL 中的函数](https://blog.csdn.net/jingerppp/article/details/86531137)

[Android HIDL 中的数据类型](https://blog.csdn.net/jingerppp/article/details/86531179)