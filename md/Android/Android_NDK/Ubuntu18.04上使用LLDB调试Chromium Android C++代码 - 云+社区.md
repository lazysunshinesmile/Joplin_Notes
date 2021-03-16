Ubuntu18.04上使用LLDB调试Chromium Android C++代码 - 云+社区

Ubuntu18.04上使用LLDB调试Chromium Android C++代码 - 云+社区

## 关于lldb/gdb调试命令区别和清单, 见 [https://lldb.llvm.org/use/map.html](https://lldb.llvm.org/use/map.html)

## [Ubuntu18.04上使用LLDB调试Chromium Android C++代码。](https://www.cnblogs.com/huangguanyuan/p/9651108.html)

Chromium Android源代码庞大且复杂。 在调试器LLDB下能帮助我们更好的理解代码流程。 介绍使用LLDB调试器调试android上chromium的C++代码。

\[1\] 编译Chromium时，记得修改编译选项：-g -O0, 使得编译器做更少的优化，便于调试。 使用attach方式调试(因此进程在启动过程中暂时不能调试)。 查看进程PID：adb shell ps -A | grep chrome 启动lldb-server：adb shell /system/bin/lldb-server platform --server --listen unix-abstract:///data/local/tmp/debug.sock 确保手机目录/system/bin/存在lldb-server。lldb-server可以从Android SDK目录中找到。

\[2\] ubuntu18.04 安装LLDB。 apt install lldb 2.1启动lldb： lldb 2.2列出lldb支持平台：platform list 2.3选择LLDB使用平台：platform select remote-android 2.4连接lldb-server： platform connect unix-abstract-connect:///data/local/tmp/debug.sock 2.5列举手机进程： platform process list 2.6ATTACH PID： attach 16767 #PID=16767 就是要调试的进程PID 2.7查看： platform status

\[3\] 在LLDB中，设置源码路径：因为调试时源码路径与编译时源码路径可能不相同，因此需要设置。否则LLDB将无法设置断点和找到源代码。 settings set target.source-map /home/huangguanyuan/chromium\_android/src /home/huangguanyuan/chromium\_android/src settings show settings list

\[4\] 在LLDB中，加载符号表。 target symbols add /home/u1804/chromium\_android/src/out/default64d/lib.unstripped/libnet.cr.so target symbols add /home/u1804/chromium\_android/src/out/default64d/lib.unstripped/libchrome.cr.so target symbols add /home/u1804/chromium\_android/src/out/default64d/lib.unstripped/libcontent.cr.so target symbols add /home/u1804/chromium\_android/src/out/default64d/lib.unstripped/libbase.cr.so

\[5\] 在LLDB中，查找符号、设置断点、查看断点、启用/禁用断点、查看call stack、单步调试、查看变量。 详细请参考：http://lldb.llvm.org/lldb-gdb.html image lookup --name TCPSocketPosix::ReadIfReady breakpoint set -f tcp\_socket\_posix.cc -l 413 breakpoint disable 5 breakpoint enable 5 bt c n print frame variable

本文参与[腾讯云自媒体分享计划](https://cloud.tencent.com/developer/support-plan)，欢迎正在阅读的你也加入，一起分享。