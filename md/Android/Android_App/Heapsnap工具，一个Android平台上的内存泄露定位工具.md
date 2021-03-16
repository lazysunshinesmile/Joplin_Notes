Heapsnap工具，一个Android平台上的内存泄露定位工具

Heapsnap工具，一个Android平台上的内存泄露定位工具


Heapsnap工具是一个Android平台上的内存泄露定位工具, 可以很方便集成到项目中, 并能够打印出每一个native内存的调用堆栈, 根据官方文档,支持Android 4.4及以上系统(但是在我们目前的C03/3212项目上似乎无法使用, 有兴趣的同学可以研究一下失败的原因)

项目主页
https://github.com/albuer/heapsnap


根据说明,将预编译库libheapsnap.so作为动态库链接到相关的模块中,
LOCAL_SHARED_LIBRARIES := libheapsnap

并开启malloc debug功能:

setprop libc.debug.malloc 1


然后运行时执行 "kill -21 [PID]" 就能dump出当前动态内存的分配镜像.

具体使用方法详见github项目主页.

也可以考虑将libheapsnap.so作为框架的os工具集成进来.
