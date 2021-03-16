Android HIDL 中的数据类型_私房菜之 --学--无--止--境---CSDN博客

HIDL 的数据类型分两种情况：HIDL C++(C++实现)、HIDL Java(Java 实现)

## <a id="t0"></a><a id="t0"></a>用户定义类型（UDT）

对于Java，在 `types.hal` 中声明的每个顶级类型都有自己的 .java 输出文件（根据 Java 要求）。例如：

```
package android.hardware.helloworld@1.0;enum HelloTest : uint8_t {
```

最终会在out 下生成HelloTest.java 文件，如下：

```
package android.hardware.helloworld.V1_0;public final class HelloTest {public static final byte V_TEST1 = 0;public static final byte V_TEST2 = 1;public static final String toString(byte o) {return "0x" + Integer.toHexString(Byte.toUnsignedInt((byte)(o)));public static final String dumpBitfield(byte o) {        java.util.ArrayList<String> list = new java.util.ArrayList<>();if ((o & V_TEST2) == V_TEST2) {            list.add("0x" + Integer.toHexString(Byte.toUnsignedInt((byte)(o & (~flipped)))));return String.join(" | ", list);
```

对于C++ 会在out 下生成一个types.h，在引用的地方inclue进去，例如：

```
#include <android/hardware/helloworld/1.0/types.h>
```

## <a id="t1"></a><a id="t1"></a>枚举类型

对于Java 实现枚举不会生成Java的枚举类，而是转换为包含各个枚举用例的静态常量定义的内部类。如果枚举类派生自其他枚举类，则会**沿用**后者的存储类型。 基于无符号整数类型的枚举将重写为其**有符号**的等效项。

例如，一个类型为 `uint8_t` 的 `SomeBaseEnum`：

```
enum SomeBaseEnum : uint8_t { foo = 3 };enum SomeEnum : SomeBaseEnum {
```

…会变为：

```
public final class SomeBaseEnum { public static final byte foo = 3; }public final class SomeEnum {public static final byte foo = 3;public static final byte quux = 33;public static final byte goober = 127;
```

且：

```
enum SomeEnum : uint8_t {
```

…重写为：

```
public final class SomeEnum {static public final byte FIRST_CASE  = 10;  static public final byte SECOND_CASE = -64;
```

**注意：**

- 例子中SECOND_CASE 为192，对于8位的数来说第7位上为1，而Java 不支持无符号的类型，HIDL 中的无符号数都会映射到Java 中的有符号数。
- hidl 中虽然SomeEnum 继承于SomeBaseEnum，但是生成的两个java类之间并不存在继承关系。

对于C++会变成C++形式的枚举。例如：

```
enum Mode : uint8_t { WRITE = 1 << 0, READ = 1 << 1 };
```

…会变为：

```
enum class Mode : uint8_t { WRITE = 1, READ = 2 };
```

## <a id="t2"></a><a id="t2"></a>结构体类型

HIDL 不支持匿名结构体。另一方面，HIDL 中的结构体与 C 非常类似。

HIDL 不支持完全包含在结构体内且长度可变的数据结构。

HIDL `vec<T>` 表示数据存储在单独的缓冲区中且大小动态变化的数组；此类实例由 `struct` 中的 `vec<T>` 的实例表示。

`string` 可包含在 `struct` 中（关联的缓冲区是相互独立的）。在生成的 C++ 代码中，HIDL 句柄类型的实例通过指向实际原生句柄的指针来表示，因为基础数据类型的实例的长度可变。

另外，HIDL 中结构体映射到Java 生成Java 类，例如：

…会变为：

```
public final ArrayList<Boolean> someBools = new ArrayList();public final float[] c = new float[10];public final Bar d = new Bar();
```

## <a id="t3"></a><a id="t3"></a>联合体类型

Java 目前不支持联合体。

HIDL 不支持匿名联合。另一方面，联合与 C 类似。

联合不能包含修正类型（指针、文件描述符、Binder 对象，等等）。它们不需要特殊字段或关联的类型，只需通过 `memcpy()` 或等效函数即可复制。联合不能直接包含（或通过其他数据结构包含）需要设置 Binder 偏移量（即句柄或 Binder 接口引用）的任何内容。例如：

```
//  vec<uint32_t> r;  // Error: can't contain a vec<T>fun8(UnionType info); // Legal
```

联合还可以在结构体中进行声明。例如：

## <a id="t4"></a><a id="t4"></a>数组

数组会映射到 Java 数组，例如：

```
int32_t[] array
```

…会变为：

```
int[] array
```

对于C++，hidl 中的常量数组由 `libhidlbase` 中的 `hidl_array` 类表示。`hidl_array<T, S1, S2, …, SN>` 表示具有固定大小的 N 维数组 `T[S1][S2]…[SN]`。

字符串在 C++ 和 Java 中的显示方式不同，但基础传输存储类型是 C++ 结构。

## <a id="t5"></a><a id="t5"></a>矢量

`vec<T>` 模板用于表示包含 `T` 实例且大小可变的缓冲区。`T` 可以是任何由 HIDL 提供的或由用户定义的类型，**句柄**除外。（`vec<T>` 的 `vec<>` 将指向 `vec<T>` 结构体数组，而不是指向内部 T 缓冲区数组。）

`T` 可以是以下项之一：

- 基本类型（例如 uint32_t）
- 字符串
- 用户定义的枚举
- 用户定义的结构体
- 接口，或 `interface` 关键字（`vec<IFoo>`，`vec<interface>` 仅在作为顶级参数时受支持）
- 句柄
- bitfield&lt;U&gt;
- vec&lt;U&gt;，其中 U 可以是此列表中的任何一项，接口除外（例如，`vec<vec<IFoo>>` 不受支持）
- U\[\]（有大小的 U 数组），其中 U 可以是此列表中的任何一项，接口除外

对于Java，vec&lt;T&gt;会生成ArrayList&lt;T&gt;，其中T为相对应的对象类型。例如 `vec<int32_t> => ArrayList<Integer>`。

例如，

```
vec<int32_t> result;
```

…会变为：

```
ArrayList<Integer> result = new ArrayList<Integer>;
```

对于C++，vec&lt;T&gt; 会映射为hidl_vec&lt;T&gt;，`hidl_vec<T>` 类模板是 `libhidlbase` 的一部分，可用于传递具备任意大小的任何 HIDL 类型的矢量。与之相当的具有固定大小的容器是 `hidl_array`。此外，您也可以使用 `hidl_vec::setToExternal()` 函数将 `hidl_vec<T>` 初始化为指向 `T` 类型的外部数据缓冲区。

除了在生成的 C++ 头文件中适当地发出/插入结构之外，您还可以使用 `vec<T>` 生成一些便利函数，用于转换到 `std::vector` 和 `T` 裸指针或从它们进行转换。如果您将 `vec<T>` 用作参数，则使用它的函数将过载（将生成两个原型），以接受并传递该参数的 HIDL 结构和 `std::vector<T>` 类型。

## <a id="t6"></a><a id="t6"></a>字符串

Java 中的 `String` 为 utf-8 或 utf-16，但在传输时会转换为 utf-8 作为常见的 HIDL 类型。另外，`String` 在传入 HIDL 时不能为空。

`C++ 中hidl_string` 类（`libhidlbase` 的一部分）可用于通过 HIDL 接口传递字符串，并在 `/system/libhidl/base/include/hidl/HidlSupport.h` 下进行定义。该类中的第一个存储位置是指向其字符缓冲区的指针。

`hidl_string` 知道如何使用 `operator=`、隐式类型转换和 `.c_str()` 函数转换到 `std::string and char*`（C 样式的字符串）以及如何从其进行转换。HIDL 字符串结构具有适当的复制构造函数和赋值运算符，可用于：

- 从 `std::string` 或 C 字符串加载 HIDL 字符串。
- 从 HIDL 字符串创建新的 `std::string`。

此外，HIDL 字符串还有转换构造函数，因此 C 字符串 (`char *`) 和 C++ 字符串 (`std::string`) 可用于采用 HIDL 字符串的方法。

这里简单列举下部分代码：

```
const char *c_str() const;    hidl_string &operator=(const hidl_string &);    hidl_string &operator=(const char *s);    hidl_string &operator=(const std::string &);    hidl_string &operator=(hidl_string &&other) noexcept;operator std::string() const;
```

## <a id="t7"></a><a id="t7"></a>pointer 类型

`pointer` 类型仅供 HIDL 内部使用。

## <a id="t8"></a><a id="t8"></a>bitfield&lt;T&gt;类型模板

bitfield&lt;T&gt;（其中 T 是用户定义的枚举）表明值是在 T 中定义的枚举值的按位“或”值。在生成的代码中，bitfield&lt;T&gt; 会显示为 T 的基础类型。例如：

```
typedef bitfield<Flag> Flags;setFlags(Flags flags) generates (bool success);
```

编译器会按照处理 `uint8_t` 的相同方式处理 Flag 类型。

## <a id="t9"></a><a id="t9"></a>内存（memory）

`memory` 类型用于表示 HIDL 中未映射的共享内存。只有 C++ 支持该类型。可以在接收端使用这种类型的值来初始化 `IMemory` 对象，从而映射内存并使其可用。

HIDL `memory` 类型会映射到 `libhidlbase` 中的 `hidl_memory` 类，该类表示未映射的共享内存。这是要在 HIDL 中共享内存而必须在进程之间传递的对象。要使用共享内存，需满足以下条件：

1.  获取 `IAllocator` 的实例（当前只有“ashmem”实例可用）并使用该实例分配共享内存。
2.  `IAllocator::allocate()` 返回 `hidl_memory` 对象，该对象可通过 HIDL RPC 传递，并能使用 `libhidlmemory`的 `mapMemory` 函数映射到某个进程。
3.  `mapMemory` 返回对可用于访问内存的 `sp<IMemory>` 对象的引用（`IMemory` 和 `IAllocator` 在 `android.hidl.memory@1.0` 中定义）。

`IAllocator` 的实例可用于分配内存：

```
#include <android/hidl/allocator/1.0/IAllocator.h>#include <android/hidl/memory/1.0/IMemory.h>#include <hidlmemory/mapping.h>using ::android::hidl::allocator::V1_0::IAllocator;using ::android::hidl::memory::V1_0::IMemory;using ::android::hardware::hidl_memory;  sp<IAllocator> ashmemAllocator = IAllocator::getService("ashmem");  ashmemAllocator->allocate(2048, [&](https://blog.csdn.net/shift_wwx/article/details/bool success, const hidl_memory& mem) {
```

对内存的实际更改必须通过 `IMemory` 对象完成（在创建 `mem` 的一端或在通过 HIDL RPC 接收更改的一端完成）。

```
sp<IMemory> memory = mapMemory(mem);void* data = memory->getPointer();
```

## <a id="t10"></a><a id="t10"></a>接口

`interface` 关键字有以下两种用途。

- 打开 .hal 文件中接口的定义。
- 可用作结构体/联合字段、方法参数和返回项中的特殊类型。该关键字被视为一般接口，与 `android.hidl.base@1.0::IBase` 同义。

例如，`IServiceManager` 具有以下方法：

```
get(string fqName, string name) generates (interface service);
```

该方法可按名称查找某个接口。此外，该方法与使用 `android.hidl.base@1.0::IBase` 替换接口完全一样。

接口只能以两种方式传递：作为顶级参数，或作为 `vec<IMyInterface>` 的成员。它们不能是嵌套式向量、结构体、数组或联合的成员。

存储接口的变量应该是强指针：`sp<IName>`。接受接口参数的 HIDL 函数会将原始指针转换为强指针，从而导致不可预料的行为（可能会意外清除指针）。为避免出现问题，请务必将 HIDL 接口存储为 `sp<>`。

下表说明 HIDL 基元与 C++ 数据类型之间的对应关系：

| **HIDL 类型** | **C++ 类型** | **头文件/库** |
| --- | --- | --- |
| enum | enum class |     |
| uint8\_t..uint64\_t | uint8\_t..uint64\_t | &lt;stdint.h&gt; |
| int8\_t..int64\_t | int8\_t..int64\_t | &lt;stdint.h&gt; |
| float | float |     |
| double | double |     |
| vec&lt;T&gt; | hidl_vec&lt;T&gt; | libhidlbase |
| T\[S1\]\[S2\]...\[SN\] | T\[S1\]\[S2\]...\[SN\] |     |
| string | hidl_string | libhidlbase |
| handle | hidl_handle | libhidlbase |
| opaque | uint64_t | &lt;stdint.h&gt; |
| struct | struct |     |
| union | union |     |
| fmq_sync | MQDescriptorSync | libhidlbase |
| fmq_unsync | MQDescriptorUnsync | libhidlbase |

相关文章：

[Android Treble 简介](https://blog.csdn.net/jingerppp/article/details/86513675)

[Android HIDL 详解](https://blog.csdn.net/jingerppp/article/details/86514997)

[Android HIDL 中 hidl-gen使用](https://blog.csdn.net/jingerppp/article/details/86525079)

[Android HIDL 编程规范](https://blog.csdn.net/jingerppp/article/details/86525761)

[Android HIDL 接口和软件包使用](https://blog.csdn.net/jingerppp/article/details/86526547)

[Android HIDL 实例](https://blog.csdn.net/jingerppp/article/details/86530600)

[Android HIDL 中的函数](https://blog.csdn.net/jingerppp/article/details/86531137)