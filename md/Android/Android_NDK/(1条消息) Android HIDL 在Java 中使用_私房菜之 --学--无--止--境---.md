(1条消息) Android HIDL 在Java 中使用_私房菜之 --学--无--止--境---CSDN博客

<img width="36" height="32" src="../../_resources/61e34a11d07b471e91b44261e82bc2af.png"/>

[私房菜](https://justinwei.blog.csdn.net/) 2019-01-23 20:21:43 <img width="24" height="24" src="../../_resources/9d56d5043aa7430e845065a513212372.png"/>3613 <a id="blog_detail_zk_collection"></a><img width="20" height="20" src="../../_resources/8f41e88f696144d0a8ef720e4b433776.png"/>收藏 3 

最后发布:2019-01-23 20:21:43首次发布:2019-01-23 20:21:43

版权声明：本文为博主原创文章，遵循 [CC 4.0 BY-SA](http://creativecommons.org/licenses/by-sa/4.0/) 版权协议，转载请附上原文出处链接和本声明。

## <a id="t0"></a><a id="t0"></a>前言：

HIDL 接口主要通过本机代码使用，因此 HIDL 专注于自动生成高效的 C++ 代码。不过，HIDL 接口也必须能够直接通过 Java 使用，因为有些 Android 子系统（如 Telephony）很可能具有 Java HIDL 接口。本文介绍了 HIDL 接口的 Java 前端，详细说明了如何创建、注册和使用服务，以及使用 Java 编写的 HAL 和 HAL 客户端如何与 HIDL RPC 系统进行交互。

## <a id="t1"></a><a id="t1"></a>作为客户端：

以HelloWorld 为例（详细看[Android HIDL 实例](https://blog.csdn.net/jingerppp/article/details/86530600) 一文），需要将hidl 的java 库导入，在Android.mk 中添加：

```
LOCAL_JAVA_LIBRARIES := \                                                                           android.hardware.helloworld-V1.0-java
```

添加代码：

```
        IHelloWorld service = null;            service = IHelloWorld.getService(true);        } catch (RemoteException e) {            Log.e(TAG, "test failed, service is null...");
```

**注意**：不含参数的 Java `getService` 将不会等待服务启动。

## <a id="t2"></a><a id="t2"></a>作为服务端：

Java 中的框架代码可能需要提供接口才能接收来自 HAL 的异步回调。

**注意**：

- 请勿用 Java 实现驱动程序 (HAL)。我们强烈建议您用 C++ 实现驱动程序。
- Java 驱动程序必须与其客户端处于不同的进程中（不支持同一进程通信）。

对于 1.0 版软件包 `android.hardware.foo` 中的接口 `IFooCallback`，您可以按照以下步骤用 Java 实现接口。

1.  用 HIDL 定义您的接口。
2.  打开 `/tmp/android/hardware/foo/IFooCallback.java` 作为参考。
3.  为您的 Java 实现创建一个新模块。
4.  检查抽象类 `android.hardware.foo.V1_0.IFooCallback.Stub`，然后编写一个新类以对其进行扩展，并实现抽象方法。

### <a id="t3"></a><a id="t3"></a>查看自动生成的文件

要查看自动生成的文件，请运行以下命令：

```
hidl-gen -o /tmp -Ljava \  -randroid.hardware:hardware/interfaces \  -randroid.hidl:system/libhidl/transport android.hardware.foo@1.0
```

这些命令会生成目录 `/tmp/android/hardware/foo/1.0`。对于文件 `hardware/interfaces/foo/1.0/IFooCallback.hal`，这会生成文件 `/tmp/android/hardware/foo/1.0/IFooCallback.java`，其中包含 Java 接口、代理代码和存根（代理和存根均与接口吻合）。

`-Lmakefile` 会生成在构建时运行此命令的规则，并允许您包含 `android.hardware.foo-V1.0-java` 并链接到相应的文件。您可以在 `hardware/interfaces/update-makefiles.sh` 中找到自动为充满接口的项目执行此操作的脚本。 本示例中的路径是相对路径；硬件/接口可能是代码树下的一个临时目录，让您能够先开发 HAL 然后再进行发布。

### <a id="t5"></a><a id="t5"></a>运行服务

HAL 提供了一个接口 `IFoo`，它必须通过接口 `IFooCallback` 对框架进行异步回调。`IFooCallback` 接口不按名称注册为可检测到的服务；相反，`IFoo` 必须包含一个诸如 `setFooCallback(IFooCallback x)` 的方法。

要通过软件包 `android.hardware.foo` 版本 1.0 设置 `IFooCallback`，请将 `android.hardware.foo-V1.0-java` 添加到 `Android.mk` 中。运行服务的代码为：

```
import android.hardware.foo.V1_0.IFoo;import android.hardware.foo.V1_0.IFooCallback.Stub;class FooCallback extends IFooCallback.Stub {IFoo server = IFoo.getService(true ); FooCallback mFooCallback = new FooCallback();server.setFooCallback(mFooCallback);
```

相关文章：

[Android Treble 简介](https://blog.csdn.net/jingerppp/article/details/86513675)

[Android HIDL 详解](https://blog.csdn.net/jingerppp/article/details/86514997)

[Android HIDL 中 hidl-gen使用](https://blog.csdn.net/jingerppp/article/details/86525079)

[Android HIDL 实例](https://blog.csdn.net/jingerppp/article/details/86530600)

[Android HIDL 编程规范](https://blog.csdn.net/jingerppp/article/details/86525761)

[Android HIDL 接口和软件包使用](https://blog.csdn.net/jingerppp/article/details/86526547)

[Android HIDL 中的函数](https://blog.csdn.net/jingerppp/article/details/86531137)

[Android HIDL 中的数据类型](https://blog.csdn.net/jingerppp/article/details/86531179)