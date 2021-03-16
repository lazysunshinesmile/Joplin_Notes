Android HIDL 编程规范_私房菜之 --学--无--止--境---CSDN博客

        [命名规范](#%E5%91%BD%E5%90%8D%E8%A7%84%E8%8C%83)

[目录结构和文件命名](#%E7%9B%AE%E5%BD%95%E7%BB%93%E6%9E%84%E5%92%8C%E6%96%87%E4%BB%B6%E5%91%BD%E5%90%8D)

[软件包名称](#%E8%BD%AF%E4%BB%B6%E5%8C%85%E5%90%8D%E7%A7%B0)

[版本](#%E7%89%88%E6%9C%AC)

[导入](#%E5%AF%BC%E5%85%A5)

[接口名称](#%E6%8E%A5%E5%8F%A3%E5%90%8D%E7%A7%B0)

[函数](#%E5%87%BD%E6%95%B0)

[结构体/联合字段名称](#%E7%BB%93%E6%9E%84%E4%BD%93%2F%E8%81%94%E5%90%88%E5%AD%97%E6%AE%B5%E5%90%8D%E7%A7%B0)

[类型名称](#%E7%B1%BB%E5%9E%8B%E5%90%8D%E7%A7%B0)

[枚举值](#%E6%9E%9A%E4%B8%BE%E5%80%BC)

[备注](#%E5%A4%87%E6%B3%A8)

[文件备注](#%E6%96%87%E4%BB%B6%E5%A4%87%E6%B3%A8)

[TODO 备注](#TODO%20%E5%A4%87%E6%B3%A8)

[接口/函数备注（文档字符串）](#%E6%8E%A5%E5%8F%A3%2F%E5%87%BD%E6%95%B0%E5%A4%87%E6%B3%A8%EF%BC%88%E6%96%87%E6%A1%A3%E5%AD%97%E7%AC%A6%E4%B8%B2%EF%BC%89)

[格式](#%E6%A0%BC%E5%BC%8F)

[软件包声明](#%E8%BD%AF%E4%BB%B6%E5%8C%85%E5%A3%B0%E6%98%8E)

[函数声明](#%E5%87%BD%E6%95%B0%E5%A3%B0%E6%98%8E)

[注释](#%E6%B3%A8%E9%87%8A)

[枚举声明](#%E6%9E%9A%E4%B8%BE%E5%A3%B0%E6%98%8E)

[结构体声明](#%E7%BB%93%E6%9E%84%E4%BD%93%E5%A3%B0%E6%98%8E)

[数组声明](#%E6%95%B0%E7%BB%84%E5%A3%B0%E6%98%8E)

[矢量](#%E7%9F%A2%E9%87%8F)

* * *

HIDL 代码样式类似于 Android 框架中的 C++ 代码，缩进 4 个空格，并且采用混用大小写的文件名。软件包声明、导入和文档字符串与 Java 中的类似，只有些微差别。

下面针对 `IFoo.hal` 和 `types.hal` 的示例展示了 HIDL 代码样式，并提供了指向每种样式（`IFooClientCallback.hal`、`IBar.hal` 和 `IBaz.hal` 已省略）详细信息的快速链接。

**hardware/interfaces/foo/1.0/IFoo.hal**

```
package android.hardware.foo@1.0;import android.hardware.bar@1.0::IBar;import IFooClientCallback;     foo() generates (FooStatus result);    powerCycle(IBar bar) generates (FooStatus result);    bar(IFooClientCallback clientCallback,
```

**hardware/interfaces/foo/1.0/types.hal**

```
package android.hardware.foo@1.0;
```

## <a id="t1"></a><a id="t1"></a>命名规范

函数名称、变量名称和文件名应该是描述性名称；避免过度缩写。将首字母缩略词视为字词（例如，请使用 `INfc`，而非 `INFC`）。

### <a id="t2"></a><a id="t2"></a>目录结构和文件命名

目录结构应如下所示：

- **`ROOT-DIRECTORY`**
    - **`MODULE`**
        - **`SUBMODULE`**（可选，可以有多层）
            - **`VERSION`**
                - `Android.mk`
                - `IINTERFACE_1.hal`
                - `IINTERFACE_2.hal`
                - `…`
                - `IINTERFACE_N.hal`
                - `types.hal`（可选）

其中：

- `ROOT-DIRECTORY` 为：
    - `hardware/interfaces`（如果是核心 HIDL 软件包）。
    - `vendor/VENDOR/interfaces`（如果是供应商软件包），其中 `VENDOR` 指 SoC 供应商或原始设备制造商 (OEM)/原始设计制造商 (ODM)。
- `MODULE` 应该是一个描述子系统的小写字词（例如 `nfc`）。如果需要多个字词，请使用嵌套式 `SUBMODULE`。可以嵌套多层。
- `VERSION` 应该与[版本](https://source.android.com/devices/architecture/hidl/code-style#versions)中所述的版本完全相同 (major.minor)。
- `IINTERFACE_X` 应该是含有 `UpperCamelCase`/`PascalCase` 的接口名称（例如 `INfc`），如[接口名称](https://source.android.com/devices/architecture/hidl/code-style#interface-names)中所述。

例如：

- `hardware/interfaces`
    - `nfc`
        - `1.0`
            - `Android.mk`
            - `INfc.hal`
            - `INfcClientCallback.hal`
            - `types.hal`

### <a id="t3"></a><a id="t3"></a>软件包名称

软件包名称必须采用以下[完全限定名称 (FQN)](https://source.android.com/devices/architecture/hidl/code-style#fqn) 格式（称为 `PACKAGE-NAME`）：

```
PACKAGE.MODULE[.SUBMODULE[.SUBMODULE[…]]]@VERSION
```

其中：

- `PACKAGE` 是映射到 `ROOT-DIRECTORY` 的软件包。具体来说，`PACKAGE` 是：
    - `android.hardware`（如果是核心 HIDL 软件包）（映射到 `hardware/interfaces`）。
    - `vendor.VENDOR.hardware`（如果是供应商软件包），其中 `VENDOR` 指 SoC 供应商或 OEM/ODM（映射到 `vendor/VENDOR/interfaces`）。
- `MODULE[.SUBMODULE[.SUBMODULE[…]]]@VERSION` 与[目录结构](https://source.android.com/devices/architecture/hidl/code-style#dir-structure)中所述结构内的文件夹名称完全相同。
- 软件包名称应为小写。如果软件包名称包含多个字词，则这些字词应用作子模块或以 `snake_case` 形式书写。
- 不允许使用空格。

软件包声明中始终使用 FQN。

### <a id="t4"></a><a id="t4"></a>版本

版本应具有以下格式：

```
MAJOR.MINOR
```

MAJOR 和 MINOR 版本都应该是一个整数。HIDL 使用[语义化版本编号](http://semver.org/)规则。

### <a id="t5"></a><a id="t5"></a>导入

导入采用以下 3 种格式之一：

- 完整软件包导入：`import PACKAGE-NAME;`
- 部分导入：`import PACKAGE-NAME::UDT;`（或者，如果导入的类型是在同一个软件包中，则为 `import UDT;`）
- 仅类型导入：`import PACKAGE-NAME::types;`

`PACKAGE-NAME` 遵循[软件包名称](https://source.android.com/devices/architecture/hidl/code-style#package-names)中的格式。当前软件包的 `types.hal`（如果存在）是自动导入的（请勿对其进行显式导入）。

完全限定名称 (FQN)

仅在必要时对用户定义的类型导入使用完全限定名称。如果导入类型是在同一个软件包中，则省略 `PACKAGE-NAME`。FQN 不得含有空格。完全限定名称示例：

```
android.hardware.nfc@1.0::INfcClientCallback
```

如果是在 `android.hardware.nfc@1.0` 下的另一个文件中，可以使用 `INfcClientCallback` 引用上述接口。否则，只能使用完全限定名称。

对导入进行分组和排序

在软件包声明之后（在导入之前）添加一个空行。每个导入都应占用一行，且不应缩进。按以下顺序对导入进行分组：

1.  其他 `android.hardware` 软件包（使用完全限定名称）。
2.  其他 `vendor.VENDOR` 软件包（使用完全限定名称）。
    - 每个供应商都应是一个组。
    - 按字母顺序对供应商排序。
3.  源自同一个软件包中其他接口的导入（使用简单名称）。

在组与组之间添加一个空行。在每个组内，按字母顺序对导入排序。例如：

```
import android.hardware.nfc@1.0::INfc;import android.hardware.nfc@1.0::INfcClientCallback;import vendor.barvendor.bar@3.1;import vendor.foovendor.foo@2.2::IFooBar;import vendor.foovendor.foo@2.2::IFooFoo;
```

### <a id="t6"></a><a id="t6"></a>接口名称

接口名称必须以 `I` 开头，后跟 `UpperCamelCase`/`PascalCase` 名称。名称为 `IFoo` 的接口必须在文件 `IFoo.hal` 中定义。此文件只能包含 `IFoo` 接口的定义（接口 `INAME` 应位于 `INAME.hal` 中）。

### <a id="t7"></a><a id="t7"></a>函数

对于函数名称、参数和返回变量名称，请使用 `lowerCamelCase`。例如：

```
open(INfcClientCallback clientCallback) generates (int32_t retVal);oneway pingAlive(IFooCallback cb);
```

### <a id="t8"></a><a id="t8"></a>结构体/联合字段名称

对于结构体/联合字段名称，请使用 `lowerCamelCase`。例如：

### <a id="t9"></a><a id="t9"></a>类型名称

类型名称指结构体/联合定义、枚举类型定义和 `typedef`。对于这些名称，请使用 `UpperCamelCase`/`PascalCase`。例如：

```
enum NfcStatus : int32_t {
```

### <a id="t10"></a><a id="t10"></a>枚举值

枚举值应为 `UPPER_CASE_WITH_UNDERSCORES`。将枚举值作为函数参数传递以及作为函数返回项返回时，请使用实际枚举类型（而不是基础整数类型）。例如：

```
enum NfcStatus : int32_t {    HAL_NFC_STATUS_FAILED           = 1,    HAL_NFC_STATUS_ERR_TRANSPORT    = 2,    HAL_NFC_STATUS_ERR_CMD_TIMEOUT  = 3,    HAL_NFC_STATUS_REFUSED          = 4
```

**注意**：枚举类型的基础类型是在冒号后显式声明的。因为它不依赖于编译器，所以使用实际枚举类型会更明晰。

对于枚举值的完全限定名称，请在枚举类型名称和枚举值名称之间使用**冒号**：

```
PACKAGE-NAME::UDT[.UDT[.UDT[…]]:ENUM_VALUE_NAME
```

完全限定名称内不得含有空格。仅在必要时使用完全限定名称，其他情况下可以省略不必要的部分。例如：

```
android.hardware.foo@1.0::IFoo.IFooInternal.FooEnum:ENUM_OK
```

## <a id="t11"></a><a id="t11"></a>备注

对于单行备注，使用 `// `和 `/** */` 都可以。

- `//` 主要用于：
    - 尾随备注
    - 将不会针对生成的文档使用的备注
    - TODO
- 针对生成的文档使用 `/** */`。此样式只能应用于类型、方法、字段和枚举值声明。例如：

- 多行备注的第一行应为 `/**`，每行的开头应使用 `*`，并且应将 `*/` 单独放在最后一行（各行的星号应对齐）。例如：

- 许可通知和变更日志的第一行应为 `/*`（一个星号），每行的开头应使用 `*`，并且应将 `*/` 单独放在最后一行（各行的星号应对齐）。例如：

```
 * Copyright (C) 2017 The Android Open Source Project
```

### <a id="t12"></a><a id="t12"></a>文件备注

每个文件的开头都应为相应的许可通知。对于核心 HAL，该通知应为 [`development/docs/copyright-templates/c.txt`](https://android.googlesource.com/platform/development/+/master/docs/copyright-templates/c.txt) 中的 AOSP Apache 许可。请务必更新年份，并使用 `/* */` 样式的多行备注（如上所述）。

您可以视需要在许可通知后空一行，后跟变更日志/版本编号信息。使用 `/* */` 样式的多行备注（如上所述），在变更日志后空一行，后跟软件包声明。

### <a id="t13"></a><a id="t13"></a>TODO 备注

TODO 备注应包含全部大写的字符串 `TODO`，后跟一个冒号。例如：

```
// TODO: remove this code before foo is checked in.
```

只有在开发期间才允许使用 TODO 备注；TODO 备注不得存在于已发布的接口中。

### <a id="t14"></a><a id="t14"></a>接口/函数备注（文档字符串）

对于多行和单行文档字符串，请使用 `/** */`。对于文档字符串，请勿使用 `//`。

接口的文档字符串应描述接口的一般机制、设计原理、目的等。函数的文档字符串应针对特定函数（软件包级文档位于软件包目录下的 README 文件中）。

```
interface IFooController {    open() generates (FooStatus status);
```

您必须为每个参数/返回值添加 `@param` 和 `@return`：

- 必须为每个参数添加 `@param`。其后应跟参数的名称，然后是文档字符串。
- 必须为每个返回值添加 `@return`。其后应跟返回值的名称，然后是文档字符串。

例如：

```
 * @param arg1 explain what arg1 is * @param arg2 explain what arg2 is * @return ret1 explain what ret1 is * @return ret2 explain what ret2 isfoo(T arg1, T arg2) generates (S ret1, S ret2);
```

## <a id="t16"></a><a id="t16"></a>格式

一般格式规则包括：

- **行长**：每行文字最长不应超过 **80** 列。
- **空格**：各行不得包含尾随空格；空行不得包含空格。
- **空格与制表符**：仅使用空格。
- **缩进大小**：数据块缩进 **4** 个空格，换行缩进 **8** 个空格。
- **大括号**：（[注释值](https://source.android.com/devices/architecture/hidl/code-style#annotations)除外）**左**大括号与前面的代码在同一行，**右**大括号与后面的分号占一整行。例如：

### <a id="t17"></a><a id="t17"></a>软件包声明

软件包声明应位于文件顶部，在许可通知之后，应占一整行，并且不应缩进。声明软件包时需采用以下格式（有关名称格式，请参阅[软件包名称](https://source.android.com/devices/architecture/hidl/code-style#package-names)）：

```
package PACKAGE-NAME;
```

例如：

```
package android.hardware.nfc@1.0;
```

### <a id="t18"></a><a id="t18"></a>函数声明

函数名称、参数、`generates` 和返回值应在同一行中（如果放得下）。例如：

```
    easyMethod(int32_t data) generates (int32_t result);
```

如果一行中放不下，则尝试按相同的缩进量放置参数和返回值，并突出 `generate`，以便读取器快速看到参数和返回值。例如：

```
    suchALongMethodThatCannotFitInOneLine(int32_t theFirstVeryLongParameter,int32_t anotherVeryLongParameter);    anEvenLongerMethodThatCannotFitInOneLine(int32_t theFirstLongParameter,int32_t anotherVeryLongParameter)                                  generates (int32_t theFirstReturnValue,int32_t anotherReturnValue);    superSuperSuperSuperSuperSuperSuperLongMethodThatYouWillHateToType(int32_t theFirstVeryLongParameter, int32_t anotherVeryLongParameterint32_t theFirstReturnValue,int32_t anotherReturnValue    foobar(AReallyReallyLongType aReallyReallyLongParameter,           AReallyReallyLongType anotherReallyReallyLongParameter)        generates (ASuperLongType aSuperLongReturnValue,                    ASuperLongType anotherSuperLongReturnValue);
```

其他细节：

- 左括号始终与函数名称在同一行。
- 函数名称和左括号之间不能有空格。
- 括号和参数之间不能有空格，它们之间出现换行时除外。
- 如果 `generates` 与前面的右括号在同一行，则前面加一个空格。如果 `generates` 与接下来的左括号在同一行，则后面加一个空格。
- 将所有参数和返回值对齐（如果可能）。
- 默认缩进 4 个空格。
- 将换行的参数与上一行的第一个参数对齐，如果不能对齐，则这些参数缩进 8 个空格。

### <a id="t19"></a><a id="t19"></a>注释

对于注释，请采用以下格式：

```
@annotate(keyword = value, keyword = {value, value, value})
```

按字母顺序对注释进行排序，并在等号两边加空格。例如：

如果注释在同一行中放不下，则缩进 8 个空格。例如：

如果整个值数组在同一行中放不下，则在左大括号 `{` 后和数组内的每个逗号后换行。在最后一个值后紧跟着添加一个右括号。如果只有一个值，请勿使用大括号。

如果整个值数组可以放到同一行，则请勿在左大括号后和右大括号前加空格，并在每个逗号后加一个空格。例如：

```
@callflow(key = {"val", "val"})@callflow(key = { "val","val" })
```

注释和函数声明之间不得有空行。例如：

### <a id="t20"></a><a id="t20"></a>枚举声明

对于枚举声明，请遵循以下规则：

- 如果与其他软件包共用枚举声明，则将声明放在 `types.hal` 中，而不是嵌入到接口内。
- 在冒号前后加空格，并在基础类型后和左大括号前加空格。
- 最后一个枚举值可以有也可以没有额外的逗号。

### <a id="t21"></a><a id="t21"></a>结构体声明

对于结构体声明，请遵循以下规则：

- 如果与其他软件包共用结构体声明，则将声明放在 `types.hal` 中，而不是嵌入到接口内。
- 在结构体类型名称后和左大括号前加空格。
- 对齐字段名称（可选）。例如：

### <a id="t22"></a><a id="t22"></a>数组声明

请勿在以下内容之间加空格：

- 元素类型和左方括号。
- 左方括号和数组大小。
- 数组大小和右方括号。
- 右方括号和接下来的左方括号（如果存在多个维度）。

例如：

```
int32_t[5][6] multiDimArray;int32_t [ 5 ] [ 6 ] array;
```

### <a id="t23"></a><a id="t23"></a>矢量

请勿在以下内容之间加空格：

- `vec` 和左尖括号。
- 左尖括号和元素类型（例外情况：元素类型也是 `vec`）。
- 元素类型和右尖括号（例外情况：元素类型也是 `vec`）。

例如：

```
vec< vec<int32_t> > array;vec < vec < int32_t > > array;
```

相关文章：

[Android Treble 简介](https://blog.csdn.net/jingerppp/article/details/86513675)

[Android HIDL 详解](https://blog.csdn.net/jingerppp/article/details/86514997)

[Android HIDL 中 hidl-gen使用](https://blog.csdn.net/jingerppp/article/details/86525079)

[Android HIDL 接口和软件包使用](https://blog.csdn.net/jingerppp/article/details/86526547)

[Android HIDL 实例](https://blog.csdn.net/jingerppp/article/details/86530600)

[Android HIDL 中的函数](https://blog.csdn.net/jingerppp/article/details/86531137)

[Android HIDL 中的数据类型](https://blog.csdn.net/jingerppp/article/details/86531179)

参考：

[https://source.android.com/devices/architecture/hidl/code-style](https://source.android.com/devices/architecture/hidl/code-style)