Android HIDL 接口和软件包使用_私房菜之 --学--无--止--境---CSDN博客

HIDL 是围绕接口进行编译的，接口是面向对象的语言使用的一种用来定义行为的抽象类型。每个接口都是软件包的一部分。

## <a id="t0"></a><a id="t0"></a>软件包

软件包名称可以具有子级，例如 `package.subpackage`。已发布的 HIDL 软件包的根目录是 `hardware/interfaces` 或 `vendor/vendorName`（例如 Pixel 设备为 `vendor/google`）。软件包名称在根目录下形成一个或多个子目录；定义软件包的所有文件都位于同一目录下。例如，`package android.hardware.example.extension.light@2.0` 可以在 `hardware/interfaces/example/extension/light/2.0` 下找到。

例如：

```
package android.hardware.nfc@1.0;import INfcClientCallback;    @callflow(next={"write", "coreInitialized", "prediscover", "powerCycle", "controlGranted"})    open(INfcClientCallback clientCallback) generates (NfcStatus status);
```

要使用接口INfc，必须要确定其package。

通过[Android HIDL 编程规范](https://blog.csdn.net/jingerppp/article/details/86525761) 知道包名的命名规则：

```
PACKAGE.MODULE[.SUBMODULE[.SUBMODULE[…]]]@VERSION
```

结合例子得到：

- PACKAGE 为android.hardware
- MODULE 为nfc
- VERSION 为1.0

其中的PACKAGE 是通过ROOT-DIRECTORY 隐射来得到的。

下表列出了软件包前缀和位置：

| 软件包前缀 | 位置  |
| --- | --- |
| `android.hardware.*` | `hardware/interfaces/*` |
| `android.frameworks.*` | `frameworks/hardware/interfaces/*` |
| `android.system.*` | `system/hardware/interfaces/*` |
| `android.hidl.*` | `system/libhidl/transport/*` |

所以，上面例子中android.hardware.nfc 路径应该是来源于hardware/interfaces/nfc，那么如何确定这个ROOT-DIRECTORY？其实这个是在编译的时候mk 或bp 设定的，例如android.hardware定义在hardware/interfaces/Android.bp 中：

```
    name: "android.hardware",    path: "hardware/interfaces",
```

当然，OEMs也可以在vendor 的目录下定义自己的HIDL，例如OEM定义vendor名为shift（路径为vendor/shift） 目录下定义：

```
    name: "vendor.shift.hardware.camera",    path: "vendor/shift/opensource/interfaces/camera",    name: "vendor.shift.hardware.display",    path: "vendor/shift/opensource/interfaces/display",    name: "vendor.shift.hardware.wifi",    path: "vendor/shift/opensource/interfaces/wifi",    path: "vendor/shift/opensource/interfaces/display",
```

这样如果定义display 的HIDL 文件可以是：

```
package vendor.display.config@1.0;                                                                  interface IDisplayConfig {                                                                          enum DisplayType : int32_t {                                                                    
```

通过这里得知：

- PACKAGE 为vendor.display
- MODULE 为display
- SUBMODULE 为config
- VERSION 为1.0

## <a id="t1"></a><a id="t1"></a>接口

除了 `types.hal` 之外，其他 `.hal` 文件均定义一个接口。接口通常定义如下：

```
interface IBar extends IFoo {     create(int32_t id) generates (MyStruct s);
```

不含显式 `extends` 声明的接口会从 `android.hidl.base@1.0::IBase`（类似于 Java 中的 `java.lang.Object`）隐式扩展。隐式导入的 IBase 接口声明了多种不应也不能在用户定义的接口中重新声明或以其他方式使用的预留方法。这些方法包括：

- `ping`
- `interfaceChain`
- `interfaceDescriptor`
- `notifySyspropsChanged`
- `linkToDeath`
- `unlinkToDeath`
- `setHALInstrumentation`
- `getDebugInfo`
- `debug`
- `getHashChain`

## <a id="t2"></a><a id="t2"></a>导入

导入采用以下 3 种格式之一：

- 完整软件包导入：`import PACKAGE-NAME;`
- 部分导入：`import PACKAGE-NAME::UDT;`（或者，如果导入的类型是在同一个软件包中，则为 `import UDT;`）
- 仅类型导入：`import PACKAGE-NAME::types;`

**完整导入：**

PACKAGE-NAME 为软件包的完成名称，例如：

```
import android.hardware.nfc@1.0;            
```

**部分导入：**

```
import INfcClientCallback;
```

因为是在一个软件包中，可以直接导入接口名称。当然也可以改为：

```
import android.hardware.nfc@1.0::INfcClientCallback;
```

当然，也可以导入types.hal中定义的UDT（user definition types），例如：

```
import android.hardware.nfc@1.0::NfcStatus;
```

NfcStatus 为types.hal 中定义的枚举类型的名称，如果在INfc.hal 中只是使用这一个type，可以这样导入。一般情况下，types.hal 中定义的类型都是给整个软件包使用，如果在Android.bp中声明，那么在文件中是不需要导入的，例如：

```
name: "android.hardware.nfc@1.0",root: "android.hardware","INfcClientCallback.hal",gen_java_constants: true,
```

**仅类型导入：**

```
import android.hardware.example@1.0::types; 
```

## <a id="t3"></a><a id="t3"></a>接口继承

接口可以是之前定义的接口的扩展。扩展可以是以下三种类型中的一种：

- 接口可以向其他接口添加功能，并按原样纳入其 API。
- 软件包可以向其他软件包添加功能，并按原样纳入其 API。
- 接口可以从软件包或特定接口导入类型。

接口只能扩展一个其他接口（不支持多重继承）。具有非零 Minor 版本号的软件包中的每个接口必须扩展一个以前版本的软件包中的接口。例如，如果 4.0 版本的软件包 `derivative` 中的接口 `IBar` 是基于（扩展了）1.2 版本的软件包 `original` 中的接口 `IFoo`，并且您又创建了 1.3 版本的软件包 `original`，则 4.1 版本的 `IBar` 不能扩展 1.3 版本的 `IFoo`。相反，4.1 版本的 `IBar` 必须扩展 4.0 版本的 `IBar`，因为后者是与 1.2 版本的 `IFoo` 绑定的。 如果需要，5.0 版本的 `IBar` 可以扩展 1.3 版本的 `IFoo`。

接口扩展并不意味着生成的代码中存在代码库依赖关系或跨 HAL 包含关系，接口扩展只是在 HIDL 级别导入数据结构和方法定义。HAL 中的每个方法必须在相应 HAL 中实现。

例如：

```
enum NfcEvent : @1.0::NfcEvent {
```

这里就继承了1.0 版本中的NfcEvent，在之前的基础上多加了一个枚举值（1.0中是0~6）。

例如：

```
package android.hardware.nfc@1.1;import @1.1::INfcClientCallback;interface INfc extends @1.0::INfc {
```

这里是1.1版本的INfc 继承了1.0版本，在1.0的接口基础上多加了一个接口factoryReset()。

## <a id="t4"></a><a id="t4"></a>接口布局总结

| 术语  | 定义  |
| --- | --- |
| 应用二进制接口 (ABI) | 应用编程接口 \+ 所需的任何二进制链接。 |
| 完全限定名称 (fqName) | 用于区分 hidl 类型的名称。例如：`android.hardware.foo@1.0::IFoo`。 |
| 软件包 | 包含 HIDL 接口和类型的软件包。例如：`android.hardware.foo@1.0`。 |
| 软件包根目录 | 包含 HIDL 接口的根目录软件包。例如：HIDL 接口 `android.hardware` 在软件包根目录 `android.hardware.foo@1.0` 中。 |
| 软件包根目录路径 | 软件包根目录映射到的 Android 源代码树中的位置。 |

### <a id="t5"></a><a id="t5"></a>每个文件都可以通过软件包根目录映射及其完全限定名称找到

软件包根目录以参数 `-r android.hardware:hardware/interfaces` 的形式指定给 `hidl-gen`。例如，如果软件包为 `vendor.awesome.foo@1.0::IFoo` 并且向 `hidl-gen` 发送了 `-r vendor.awesome:some/device/independent/path/interfaces`，那么接口文件应该位于 `$ANDROID_BUILD_TOP/some/device/independent/path/interfaces/foo/1.0/IFoo.hal`。

详细关于hidl-gen 可以看另一篇：[Android HIDL 中 hidl-gen使用](https://blog.csdn.net/jingerppp/article/details/86525079)

在实践中，建议称为 `awesome` 的供应商或原始设备制造商 (OEM) 将其标准接口放在 `vendor.awesome` 中。在选择了软件包路径之后，不能再更改该路径，因为它已写入接口的 ABI。

### <a id="t6"></a><a id="t6"></a>软件包路径映射不得重复

例如，如果您有 `-rsome.package:$PATH_A` 和 `-rsome.package:$PATH_B`，则 `$PATH_A` 必须等于 `$PATH_B` 以实现一致的接口目录。

### <a id="t7"></a><a id="t7"></a>软件包根目录必须有版本控制文件

如果您创建一个软件包路径（如 `-r vendor.awesome:vendor/awesome/interfaces`），则还应创建文件 `$ANDROID_BUILD_TOP/vendor/awesome/interfaces/current.txt`，它应包含使用 `hidl-gen`中的 `-Lhash` 选项创建的接口的哈希。

### <a id="t8"></a><a id="t8"></a>接口位于设备无关的位置

在实践中，建议在分支之间共享接口。这样可以最大限度地重复使用代码，并在不同的设备和用例中对代码进行最大程度的测试。

## <a id="t9"></a><a id="t9"></a>Java 中导出常量

在接口不兼容 Java（例如由于使用联合类型而不兼容 Java）的情况下，可能仍需将常量（枚举值）导出到 Java 环境。这种情况需要用到 `hidl-gen -Ljava-constants …`，它会将已添加注释的枚举声明从软件包的接口文件提取出来，并生成一个名为 `[PACKAGE-NAME]-V[PACKAGE-VERSION]-java-constants` 的 java 库。请为每个要导出的枚举声明添加注释，如下所示：

如有必要，这一类型导出到 Java 环境时所使用的名称可以不同于接口声明中选定的名称，方法是添加注释参数 `name`：

如果依据 Java 惯例或个人偏好需要将一个公共前缀添加到枚举类型的值，请使用注释参数 `value_prefix`：

```
package android.hardware.bar@1.0;@export(name="JavaFoo", value_prefix="JAVA_")
```

生成的 Java 类如下所示：

```
package android.hardware.bar.V1_0;public final class JavaFoo {public static final int JAVA_SOME_VALUE = 0;public static final int JAVA_SOME_OTHER_VALUE = 1;
```

当然，如果注释中的name 设为空，则常量将直接定义与Constants类中，例如：

```
package android.hardware.bar@1.0;@export(name="", value_prefix="JAVA_")
```

生成的 Java 类如下所示：

```
package android.hardware.bar.V1_0;public static final int JAVA_SOME_VALUE = 0;public static final int JAVA_SOME_OTHER_VALUE = 1;
```

另外，需要将gen\_java\_constants 属性设为true 添加到bp 文件中即可生成：

```
    name: "android.hardware.nfc@1.0",    root: "android.hardware","INfcClientCallback.hal",    gen_java_constants: true,
```

**相关文章：**

[Android Treble 简介](https://blog.csdn.net/jingerppp/article/details/86513675)

[Android HIDL 详解](https://blog.csdn.net/jingerppp/article/details/86514997)

[Android HIDL 中 hidl-gen使用](https://blog.csdn.net/jingerppp/article/details/86525079)

[Android HIDL 编程规范](https://blog.csdn.net/jingerppp/article/details/86525761)

[Android HIDL 实例](https://blog.csdn.net/jingerppp/article/details/86530600)

[Android HIDL 中的函数](https://blog.csdn.net/jingerppp/article/details/86531137)

[Android HIDL 中的数据类型](https://blog.csdn.net/jingerppp/article/details/86531179)