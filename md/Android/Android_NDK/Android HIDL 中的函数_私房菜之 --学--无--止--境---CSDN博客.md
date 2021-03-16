Android HIDL 中的函数_私房菜之 --学--无--止--境---CSDN博客

## <a id="t1"></a><a id="t1"></a>函数参数

`.hal` 文件中列出的参数会映射到 C++ 数据类型。未映射到基元 C++ 类型的参数会通过常量引用进行传递。

对于具有返回值（具有 `generates` 语句）的每个 HIDL 函数，该函数的 C++ 参数列表中都有一个附加参数：使用 HIDL 函数的返回值调用的回调函数。有**一种情况例外**：如果 `generates` 子句包含直接映射到 C++ 基元的单个参数，则使用回调省略（回调会被移除，而返回值则会通过正常的 `return` 语句从函数返回）。

## <a id="t3"></a><a id="t3"></a>函数返回值

### <a id="t4"></a><a id="t4"></a>传输错误和返回类型

`generates` 语句可以产生三种类型的函数签名：

- 如果只有一个作为 C++ 基元的返回值，`generates` 返回值会由 `Return<T>` 对象中函数的值返回。
- 如果情况更复杂，`generates` 返回值则会通过随函数调用本身一起提供的回调参数返回，而函数则返回 `Return<void>`。
- 如果不存在 `generates` 语句，函数则会返回 `Return<void>`。

RPC 调用可能偶尔会遇到传输错误，例如服务器终止，传输资源不足以完成调用，或传递的参数不允许完成调用（例如缺少必需的回调函数）。`Return` 对象会存储传输错误指示以及 `T` 值（`Return<void>` 除外）。

由于客户端和服务器端函数具有相同的签名，因此服务器端函数必须返回 `Return` 类型（即使其实现并不会指出传输错误）。`Return<T>` 对象会使用 `Return(myTValue)` 进行构建（也可以通过 `mTValue` 隐式构建，例如在 `return` 语句中），而 `Return<void>` 对象则使用 `Void()` 进行构建。

`Return<T>` 对象可以从其 `T` 值执行隐式转换，也可以执行到该值的隐式转换。您可以检查 `Return` 对象是否存在传输错误，只需调用其 `isOk()` 方法即可。这项检查不是必需的；不过，如果发生了一个错误，而您未在 `Return` 对象销毁前对该错误进行检查，或尝试进行了 `T` 值转换，则客户端进程将会终止并记录一个错误。如果 `isOk()` 表明存在由开发者代码中的逻辑错误（例如将 `nullptr` 作为同步回调进行传递）导致的传输错误或失败调用，则可以对 Return 对象调用 `description()` 以返回适合日志记录的字符串。在这种情况下，您无法确定因调用失败而在服务器上执行的代码可能有多少。另外，您还可以使用 `isDeadObject()` 方法。此方法表明，之所以会显示 `!isOk()` 是因为远程对象已崩溃或已不存在。`isDeadObject()` 一律表示 `!isOk()`。

### <a id="t5"></a><a id="t5"></a>由值返回

如果 `generates` 语句映射到单个 C++ 基元，则参数列表中不会有任何回调参数，而实现会在 `Return<T>` 对象中提供返回值 `T`，该值可以从基元类型 `T` 隐式生成。例如：

```
Return<uint32_t> someMethod() {uint32_t return_data = ...; 
```

另外，您还可以使用 `Return<*>::withDefault` 方法。此方法会在返回值为 `!isOk()` 的情况下提供一个值。此方法还会自动将返回对象标记为正常，以免客户端进程遭到终止。

### <a id="t6"></a><a id="t6"></a>使用回调参数返回

回调可以将 HIDL 函数的返回值回传给调用方。回调的原型是 `std::function` 对象，其参数（从 `generates` 语句中获取）会映射到 C++ 类型。它的返回值为 void（回调本身并不会返回任何值）。

具有回调参数的 C++ 函数的返回值具有 `Return<void>` 类型。服务器实现仅负责提供返回值。由于返回值已使用回调传输，因此 `T` 模板参数为 `void`：

```
Return<void> someMethod(someMethod_cb _cb);
```

服务器实现应从其 C++ 实现中返回 `Void()`（这是一个可返回 `Return<void>` 对象的静态内嵌函数）。具有回调参数的典型服务器方法实现示例：

```
Return<void> someMethod(someMethod_cb _cb) {    hidl_vec<uint32_t> vec = ...
```

## <a id="t8"></a><a id="t8"></a>没有返回值的函数

没有 `generates` 语句的函数的 C++ 签名将不会在参数列表中有任何回调参数。它的返回类型将为 `Return<void>.`

## <a id="t9"></a><a id="t9"></a>单向函数

以 `oneway` 关键字标记的函数是异步函数（其执行不会阻塞客户端），而且没有任何返回值。`oneway` 函数的 C++ 签名将不会在参数列表中有任何回调参数，而且其 C++ 返回值将为 `Return<void>`。

## <a id="t11"></a><a id="t11"></a>Void 方法

不返回结果的方法将转换为返回 `void` 的 Java 方法。例如，HIDL 声明：

```
doThisWith(float param)
```

…会变为：

```
void doThisWith(float param);
```

## <a id="t12"></a><a id="t12"></a>单结果方法

返回单个结果的方法将转换为同样返回单个结果的 Java 等效方法。例如，以下方法：

```
doQuiteABit(int32_t a, int64_t b,float c, double d) generates (double something);
```

…会变为：

```
double doQuiteABit(int a, long b, float c, double d);
```

## <a id="t13"></a><a id="t13"></a>多结果方法

对于返回多个结果的每个方法，系统都会生成一个回调类，在其 `onValues` 方法中提供所有结果。 该回调会用作此方法的附加参数。例如，以下方法：

```
justTest(string name) generates (string result, HelloTest value)
```

…会变为：

```
@java.lang.FunctionalInterfacepublic interface justTestCallback {public void onValues(String result, byte value);void justTest(String name, justTestCallback _hidl_cb) throws android.os.RemoteException;
```

## <a id="t15"></a><a id="t15"></a>传输错误和终止通知接收方

由于服务实现可以在不同的进程中运行，在某些情况下，即使实现接口的进程已终止，客户端也可以保持活动状态。调用已终止进程中托管的接口对象会失败并返回传输错误（调用的方法抛出的运行时异常）。可以通过调用 `I<InterfaceName>.getService()` 以请求服务的新实例，从此类失败的调用中恢复。不过，仅当崩溃的进程已重新启动且已向 servicemanager 重新注册其服务时，这种方法才有效（对 HAL 实现而言通常如此）。

接口的客户端也可以注册一个终止通知接收方，以便在服务终止时收到通知。如果调用在服务器刚终止时发出，仍然可能发生传输错误。要在检索的 `IFoo` 接口上注册此类通知，客户端可以执行以下操作：

```
foo.linkToDeath(recipient, 1481 );
```

`recipient` 参数必须是由 HIDL 提供的 `HwBinder.DeathRecipient` 接口的实现。该接口包含会在托管该接口的进程终止时调用的单个方法 `serviceDied()`。

```
final class DeathRecipient implements HwBinder.DeathRecipient {public void serviceDied(long cookie) {
```

`cookie` 参数包含使用 `linkToDeath()` 调用传递的 Cookie。您也可以在注册服务终止通知接收方后将其取消注册：

```
foo.unlinkToDeath(recipient);
```

相关文章：

[Android Treble 简介](https://blog.csdn.net/jingerppp/article/details/86513675)

[Android HIDL 详解](https://blog.csdn.net/jingerppp/article/details/86514997)

[Android HIDL 中 hidl-gen使用](https://blog.csdn.net/jingerppp/article/details/86525079)

[Android HIDL 编程规范](https://blog.csdn.net/jingerppp/article/details/86525761)

[Android HIDL 接口和软件包使用](https://blog.csdn.net/jingerppp/article/details/86526547)

[Android HIDL 实例](https://blog.csdn.net/jingerppp/article/details/86530600)

[Android HIDL 中的数据类型](https://blog.csdn.net/jingerppp/article/details/86531179)