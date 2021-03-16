Android.mk使用文档

Android.mk使用文档

每个模块描述应遵守以下基本流程：

1. 使用 CLEAR_VARS 变量初始化或取消定义与模块相关的变量。
2. 为用于描述模块的变量赋值。
3. 使用 BUILD_XXX 变量设置 NDK 构建系统，以便为模块使用适当的构建脚本。

1. LOCAL_PATH
LOCAL_PATH := $(call my-dir)
指定当前文件的路径，必须在开头就指定他。CLEAR_VARS 指向的脚本不会清除此变量。因此即使定义多个模块也只需要定义该变量一次。

2. CLEAR_VARS
清除与模块相关的变量。

3. LOCAL_MODULE

定义当前的模块名称。在所有模块名称中唯一，不包含任何空格。在CLEAR_VARS 之后，其他脚本之前。定义他不需要添加lib前缀和.so或者.a后缀。构建系统自动添加。

LOCAL_MODULE := "foo" --> libfoo.so
如果不想用Libxxx.so的形式。可以用 LOCAL_MODULE_FILENAME 定义名称。
如：LOCAL_MODULE_FILENAME := newfoo 会生成newfoo.so。文件的扩展和路径不会改。

4. LOCAL_SRC_FILES

包含了所有生成模块的源文件列表。不需要按照依赖关系排序。构建系统会自动计算。可以使用相对路径（想对于LOCAL_PATH）和绝对路径。建议避免使用绝对路径。路径使用`/`，不能使用`\`.

5. LOCAL_CPP_EXTENSION
指明除了cpp以外的文件扩展名。如：
LOCAL_CPP_EXTENSION := .cxx .cpp .cc
可以指定多个扩展名。这样就会编译后缀为.cxx .cpp .cc 的文件。上文指定时，必须带有.

6. LOCAL_CPP_FEATURES
可以使用此可选变量指明您的代码依赖于特定 C++ 功能。例如，要指示您的代码使用 RTTI（运行时类型信息），请编写：
LOCAL_CPP_FEATURES := rtti

您还可为此变量指定多个值，顺序不重要。例如：
LOCAL_CPP_FEATURES := rtti features

7. LOCAL_C_INCLUDES
指定源文件的 include 的搜索路径。

8. LOCAL_CFLAGS
制定额外的宏定义或者编译选项。如：
LOCAL_CFLAGS += -I<path>
指定了include路径，但用LOCAL_C_INCLUDES比较好。

9. LOCAL_STATIC_LIBRARIES

指明当前模块依赖的静态库模块列表。如果该模块是共享库或可执行文件，此变量将强制这些库链接到生成的二进制文件。如果该模块是静态库，此变量只是只是，依赖当前模块的模块也会依赖列出的库。

10. LOCAL_SHARED_LIBRARIES
指明当前模块依赖的共享库模块列表。

11. LOCAL_WHOLE_STATIC_LIBRARIES

将提及的模块视为整个存档。会把后面提及的模块里面的每一个对象都作为静态库中的变量。而不是仅仅把当前模块需要用到的作为静态库中的变量。当多个静态库之间具有循环相依关系时，此变量很有用。

12. LOCAL_LDLIBS
动态库中使用。如下使用，其中-l表示链接到特定系统库：
LOCAL_LDLIBS := -lz
会链接到/system/lib/libz.so。

13. LOCAL_LDFLAGS
动态库中使用，选择其他的链接器。以下示例在 ARM/X86 GCC 4.6+ 上使用 ld.bfd 链接器，该系统上的默认链接器是 ld.gold
LOCAL_LDFLAGS += -fuse-ld=bfd
如果为静态库定义此变量，构建系统会忽略它，ndk-build会显示一则警告。

14. LOCAL_ALLOW_UNDEFINED_SYMBOLS

默认情况下，如果构建系统在构建共享库时，遇到未定义的引用，会引发“未定义的符号”错误。如果想停用此检查，可以将该变量设为true。请注意，此设置可能导致共享哭在运行时加载。

如果静态库使用该变量，同上。

15. LOCAL_ARM_MODE
如果
LOCAL_SRC_FILES := foo.c bar.c.arm
LOCAL_ARM_MODE := arm
则在ARM模式下编译bar.c，但会构建foo.c。

16. LOCAL_ARM_NEON
此变量仅在您针对 armeabi-v7a ABI 时才重要。
LOCAL_SRC_FILES = foo.c.neon bar.c zoo.c.arm.neon
构建系统编译支持 thumb 和 neon 的 foo.c、支持 thumb 的 bar.c，以及支持 ARM 和 NEON 的 zoo.c。

# 完整示例：
```
LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)

LOCAL_USE_AAPT2 := true

LOCAL_MODULE_TAGS := optional

LOCAL_MANIFEST_FILE := app/src/main/AndroidManifest.xml
LOCAL_RESOURCE_DIR := $(LOCAL_PATH)/app/src/main/res
#LOCAL_SRC_FILES := $(call all-java-files-under, /app/src/main/java)
LOCAL_AAPT_FLAGS := \
    --auto-add-overlay \
    --extra-packages com.gs.apidemo
LOCAL_PRIVATE_PLATFORM_APIS := true
LOCAL_STATIC_JAVA_LIBRARIES := api.common api.dbus api.phone api.account api.settings api.device api.upgrade api.nvram api.ir

LOCAL_JAVA_LIBRARIES := gsframework

LOCAL_SRC_FILES := $(call all-java-files-under, app/src/main)
LOCAL_PACKAGE_NAME := AM3X00ApiDemo


include $(BUILD_PACKAGE)
# This finds and builds ext as well.
include $(call all-makefiles-under,$(LOCAL_PATH))

```

## android.bp 示例：
1. jni c 代码打包：
```
cc_binary {
    srcs: ["ir.c"],
    name: "ir_send",
    //LOCAL_MODULE_TAGS := optional
}
```
2. jni 引用上面的c的包
```
java_library_static {

    name: "api.ir",

    srcs: ["java/**/*.java"] ,

    libs: ["gsframework"],
    static_libs: [
        "api.common",
        "api.account",
    ],
    required: ["ir_send"],
}
```











以下部分纯属拷贝，尚未细细品味。https://blog.csdn.net/xiaole0313/article/details/73065831
LOCAL_DISABLE_NO_EXECUTE

Android NDK r4 添加了对“NX 位”安全功能的支持。此支持默认启用，但您也可通过将此变量设置为 true 将其停用。 如果没有必要的原因，我们不建议停用。

此功能不会修改 ABI，并且仅在针对 ARMv6+ CPU 设备的内核上启用。 启用此功能的机器代码在运行较早 CPU 架构的设备上将不加修改而直接运行。

如需了解详细信息，请参阅 Wikipedia：NX 位和 GNU 栈快速入门。

LOCAL_DISABLE_RELRO

默认情况下，NDK 编译具有只读重定位和 GOT 保护的代码。 此变量指示运行时链接器在重定位后将某些内存区域标记为只读，增加了某些安全漏洞利用（例如 GOT 覆盖）的难度。 请注意，这些保护仅在 Android API 级别 16 和更高版本上有效。在较低的 API 级别上，该代码仍会运行，但没有内存保护。

此变量默认启用，但您也可通过将其值设置为 true 来停用它。 如果没有必要的原因，我们不建议停用。

如需了解详细信息，请参阅 RELRO：重定位只读和 RedHat Enterprise Linux 中的安全增强功能（第 6 节）。

LOCAL_DISABLE_FORMAT_STRING_CHECKS
默认情况下，构建系统编译具有格式字符串保护的代码。如果 printf 样式的函数中使用非常量的格式字符串，这样会强制编译器出错。

此保护默认启用，但您也可通过将此变量的值设置为 true 将其停用。 如果没有必要的原因，我们不建议停用。

LOCAL_EXPORT_CFLAGS

此变量用于记录一组 C/C++ 编译器标志，这将标志将添加到通过 LOCAL_STATIC_LIBRARIES 或 LOCAL_SHARED_LIBRARIES 变量使用它们的任何其他模块的 LOCAL_CFLAGS 定义。

例如，假设有以下模块对：foo 和 bar，分别依赖于 foo：

include $(CLEAR_VARS)
LOCAL_MODULE := foo
LOCAL_SRC_FILES := foo/foo.c
LOCAL_EXPORT_CFLAGS := -DFOO=1
include $(BUILD_STATIC_LIBRARY)

include $(CLEAR_VARS)
LOCAL_MODULE := bar
LOCAL_SRC_FILES := bar.c
LOCAL_CFLAGS := -DBAR=2
LOCAL_STATIC_LIBRARIES := foo
include $(BUILD_SHARED_LIBRARY)

在这里，构建系统在构建 bar.c 时会向编译器传递标志 -DFOO=1 和 -DBAR=2。 它还会在模块的 LOCAL_CFLAGS 前面加上导出的标志，以便您轻松替换它们。

此外，模块之间的关系也是可传递的：如果 zoo 依赖于 bar，后者又依赖于 foo，则 zoo 也会继承从 foo 导出的所有标志。

最后，构建系统在本地构建时不使用导出的标志（即，构建要导出其标志的模块）。 因此，在上面的示例中，构建 foo/foo.c 时不会将 -DFOO=1 传递到编译器。 要在本地构建，请改用 LOCAL_CFLAGS。

LOCAL_EXPORT_CPPFLAGS
此变量与 LOCAL_EXPORT_CFLAGS 相同，但仅适用于 C++ 标志。

LOCAL_EXPORT_C_INCLUDES
此变量与 LOCAL_EXPORT_CFLAGS 相同，但适用于 C include 路径。例如，当 bar.c 需要包含模块 foo 中的标头时很有用。

LOCAL_EXPORT_LDFLAGS
此变量与 LOCAL_EXPORT_CFLAGS 相同，但适用于链接器标志。

LOCAL_EXPORT_LDLIBS
此变量与 LOCAL_EXPORT_CFLAGS 相同，用于指示构建系统将特定系统库的名称传递到编译器。 在您指定的每个库名称前面附加 -l。

请注意，构建系统会将导入的链接器标志附加到模块的 LOCAL_LDLIBS 变量值。 其原因在于 Unix 链接器运行的方式。

当模块 foo 是静态库并且具有依赖于系统库的代码时，此变量通常很有用。 然后您可以使用 LOCAL_EXPORT_LDLIBS 导出相依关系。 例如：

include $(CLEAR_VARS)
LOCAL_MODULE := foo
LOCAL_SRC_FILES := foo/foo.c
LOCAL_EXPORT_LDLIBS := -llog
include $(BUILD_STATIC_LIBRARY)

include $(CLEAR_VARS)
LOCAL_MODULE := bar
LOCAL_SRC_FILES := bar.c
LOCAL_STATIC_LIBRARIES := foo
include $(BUILD_SHARED_LIBRARY)

在此示例中，构建系统在构建 libbar.so 时，将在链接器命令的末尾放置 -llog。 这样会告知链接器，由于 libbar.so 依赖于 foo，因此它也依赖于系统日志记录库。

LOCAL_SHORT_COMMANDS
当您的模块有很多源文件和/或相依的静态或共享库时，将此变量设置为 true。 这样会强制构建系统对包含中间对象文件或链接库的存档使用 @ 语法。

此功能在 Windows 上可能很有用，其中命令行最多只接受 8191 个字符，这对于复杂的项目可能太少。 它还会影响个别源文件的编译，而且将几乎所有编译器标志放在列表文件内。

请注意，true 以外的任何值都将恢复到默认行为。 您也可在 Application.mk 文件中定义 APP_SHORT_COMMANDS，以强制对项目中的所有模块实施此行为。

不建议默认启用此功能，因为它会减慢构建的速度。

LOCAL_THIN_ARCHIVE
构建静态库时将此变量设置为 true。这样会生成一个瘦存档 ，即一个库文件，其中不含对象文件，而只包含它通常要包含的实际对象的文件路径。

这对于减小构建输出的大小非常有用。缺点是：这样的库无法移至不同的位置（其中的所有路径都是相对的）。

有效值为 true、false 或空白。可通过 APP_THIN_ARCHIVE 变量在 Application.mk 文件中设置默认值。

注：对于非静态库模块或预构建的静态库模块会忽略此变量。

LOCAL_FILTER_ASM
将此变量定义为构建系统要用于过滤从您为 LOCAL_SRC_FILES 指定的文件提取或生成的汇编文件的 shell 命令。

定义此变量会导致发生以下情况：

构建系统从任何 C 或 C++ 源文件生成临时汇编文件，而不是将它们编译到对象文件。

构建系统在任何临时汇编文件以及 LOCAL_SRC_FILES 中所列任何汇编文件的 LOCAL_FILTER_ASM 中执行 shell 命令，因此会生成另一个临时汇编文件。

构建系统将这些过滤的汇编文件编译到对象文件中。
例如：

LOCAL_SRC_FILES := foo.c bar.S
LOCAL_FILTER_ASM :=

foo.c --1--> $OBJS_DIR/foo.S.original --2--> $OBJS_DIR/foo.S --3--> $OBJS_DIR/foo.o

bar.S --2--> $OBJS_DIR/bar.S --3--> $OBJS_DIR/bar.o

“1”对应编译器，“2”对应过滤器，“3”对应汇编程序。过滤器必须是采用输入文件名称作为其第一个参数、输出文件名称作为第二个参数的独立 shell 命令。 例如：

myasmfilter $OBJS_DIR/foo.S.original $OBJS_DIR/foo.S
myasmfilter bar.S $OBJS_DIR/bar.S

本节说明 NDK 提供的 GNU Make 函数宏。使用 $(call <function>) 对它们估值；它们返回文本信息。

my-dir

此宏返回最后包含的 makefile 的路径，通常是当前 Android.mk 的目录。my-dir 可用于在 Android.mk 文件的开头定义 LOCAL_PATH。 例如：

LOCAL_PATH := $(call my-dir)

由于 GNU Make 运行的方式，此宏实际返回的内容是构建系统在解析构建脚本时包含在最后一个 makefile 的路径。 因此，在包含另一个文件后不应调用 my-dir。

例如，考虑以下示例：
```
LOCAL_PATH := $(call my-dir)

# ... declare one module

include $(LOCAL_PATH)/foo/`Android.mk`

LOCAL_PATH := $(call my-dir)

# ... declare another module
```
这里的问题在于，对 my-dir 的第二次调用将 LOCAL_PATH 定义为 $PATH/foo，而不是 $PATH，因为这是其最近 include 指向的位置。

在 Android.mk 文件中的任何其他内容后放置额外 include 可避免此问题。 例如：
```

LOCAL_PATH := $(call my-dir)

# ... declare one module

LOCAL_PATH := $(call my-dir)

# ... declare another module

# extra includes at the end of the Android.mk file

include $(LOCAL_PATH)/foo/Android.mk

如果以这种方式构建文件不可行，请将第一个 my-dir 调用的值保存到另一个变量中。 例如：

MY_LOCAL_PATH := $(call my-dir)

LOCAL_PATH := $(MY_LOCAL_PATH)

# ... declare one module

include $(LOCAL_PATH)/foo/`Android.mk`

LOCAL_PATH := $(MY_LOCAL_PATH)

# ... declare another module
```