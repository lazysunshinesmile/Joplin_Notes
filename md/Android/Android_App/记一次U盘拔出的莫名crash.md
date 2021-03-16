记一次U盘拔出的莫名crash

记一次U盘拔出的莫名crash

1. 问题描述：
一个app的功能是在U盘插入时，会检测是否保存了ftp设置，若保存了，会上传相关文件。上传代码用的第三方的。在U盘插入，检测到文件，并开始上传之后，迅速拔出U盘，会导致app crash。查看日志，没有明显的Crash日志。

2. 解决过程
打印了全部日志，查看到如下的打印：
    ```
    06-20 11:37:04.386   380   385 W vold    : Found symlink /proc/9451/fd/84 referencing /storage/F083-54D4/Recording/4455_(34th_copy).mkv
    06-20 11:37:04.389   380   385 W vold    : Sending Interrupt to 9451
    ```
    这个日志就是说明Android系统杀死了，这个进程号，这个进程号就是莫名关闭的app的进程。
    
3. 原因：因为U盘拔出，而这个上传的进程仍然在使用这个文件。所以android系统杀死了这个进程。

4. 解决办法：上传的代码是一个整体，无法检测文件是否存在从而中断上传。在AndroidManifest.xml中，对上传服务配置android:process=":remote"，让他不在这个进程中。所以不会连累app。