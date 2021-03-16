sepolicy 中unlabeled 修改_私房菜之 --学--无--止--境---CSDN博客

文章出处：[http://blog.csdn.net/shift_wwx/article/details/77500458](http://blog.csdn.net/shift_wwx/article/details/77500458)

请转载的朋友标明出处~~

最近需要在平台上添加一个persist 分区，需要添加sepolicy，但是不管怎么修改，发现分区最终 ls -Z 出来一直是：u:object\_r:unlabeled:s0，而不是想要的persist\_file 属性。

修改如下：

file.te 中：

```
type persist_file, file_type;
```

file_contexts中：

```
/persist(/.*)?       u:object_r:persist_file:s0
```

可是为什么没有生效了？

查看了init 中的code：

```
int main(int argc, char** argv) {        mount("tmpfs", "/dev", "tmpfs", MS_NOSUID, "mode=0755");        mkdir("/dev/socket", 0755);        mount("devpts", "/dev/pts", "devpts", 0, NULL);#define MAKE_STR(x) __STRING(x)        mount("proc", "/proc", "proc", 0, "hidepid=2,gid=" MAKE_STR(AID_READPROC));        mount("sysfs", "/sys", "sysfs", 0, NULL);    klog_set_level(KLOG_NOTICE_LEVEL);    NOTICE("init %s started!\n", is_first_stage ? "first stage" : "second stage");        close(open("/dev/.booting", O_WRONLY | O_CREAT | O_CLOEXEC, 0000));        process_kernel_cmdline();        export_kernel_boot_props();    selinux_initialize(is_first_stage);if (restorecon("/init") == -1) {            ERROR("restorecon failed: %s\n", strerror(errno));char* args[] = { path, const_cast<char*>("--second-stage"), nullptr };if (execv(path, args) == -1) {            ERROR("execv(\"%s\") failed: %s\n", path, strerror(errno));    NOTICE("Running restorecon...\n");    restorecon("/dev/socket");    restorecon("/dev/__properties__");    restorecon("/property_contexts");    restorecon_recursive("/sys");    parser.ParseConfig("/init.rc");    selinux_initialize(is_first_stage);if (restorecon("/init") == -1) {            ERROR("restorecon failed: %s\n", strerror(errno));char* args[] = { path, const_cast<char*>("--second-stage"), nullptr };if (execv(path, args) == -1) {            ERROR("execv(\"%s\") failed: %s\n", path, strerror(errno));    NOTICE("Running restorecon...\n");    restorecon("/dev/socket");    restorecon("/dev/__properties__");    restorecon("/property_contexts");    restorecon_recursive("/sys");    parser.ParseConfig("/init.rc");
```

可以看到在init.rc 解析之前做了selinux 的load，在load 之后都会对一些分区做restorecon的操作，这个应该是说在init.rc 解析之前先做了selinux 的load，也就是file_contexts等load，但是此时并没有persist 分区，所以sepolicy 并没有生效。

看到这，大概就知道之前为什么一直不行了，因为需要restore的操作，所以，最终修改如下：（在init.rc 或者init.*.rc 中：）

```
restorecon_recursive /persist
```

对于，restorecon 和 restorecon_recursive 的区别，可以看下code，就是一个递归的效果。