Android C_C++打印日志，和堆栈方式

Android C/C++打印日志，和堆栈方式

# 加入日志打印
## Android.bp 或者Android.mk加入日志库

```Android.bp
	shared_libs: [
        ....
        "liblog"
    ]
```
```Android.mk
LOCAL_SHARED_LIBRARIES += liblog

```

## 对应c文件
tag与include的顺序不要颠倒，否则没有tag打印，因为log.h已经定义了tag。
```c
#ifndef LOG_TAG
#define LOG_TAG "log_tag_xiangsun"
#endif

#include <log/log.h>
```

## 调用
```c
	ALOGD("1, this is my log, name=%s", name);
```

# 加入堆栈日志打印

C打印堆栈，其实就是调用C++的方法，所以，这里只说明C的方式，C++也就是，不用extern "C"声明

## Android.bp 或者Android.mk加入堆栈库
```Android.bp
    shared_libs: [
        ...
        "liblog",
        "libutilscallstack"
    ],
	srcs: [
		...,
        "callstack.cpp"
	]
```
## callstack.cpp 
```cpp
#include <utils/CallStack.h>
#include <android/log.h>

extern "C" void dumping_callstack(const char* msg,const char* prefix, const char* tag)
{
         ALOGD("xiangsun---:%s", msg);
         android::CallStack cs;
         cs.update();
         cs.log(tag, ANDROID_LOG_ERROR, prefix);

}
```

## 对应C文件

```c
#ifndef LOG_TAG
#define LOG_TAG "pvalue_validation_xiangsun"
#endif
#include <log/log.h>
extern void dumping_callstack(const char* msg,const char* prefix,const char* tag);
int method {
	
	dumping_callstack("msg", "prefix", LOG_TAG);
}

```

