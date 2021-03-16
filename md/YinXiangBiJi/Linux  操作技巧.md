Linux  操作技巧

Linux  操作技巧

[TOC]

## 1、通过apt-get install安装软件后，软件的安装目录
### 1.通过apt-get install 命令安装了一些软件，但这些软件的源码以及那些安装完以后的文件放在哪个文件夹下面？
可以通过以下两种方式查看：
在terminal中输入命令：dpkg -L 软件名
eg：dpkg -L gcc
dpkg -L gcc-4.4
dpkg -L g++
dpkg -L g++-4.4
## 2、查找关键字：
grep /root/directory 'key word' -R

## 3、增加桌面快捷方式：
/usr/share/applications/...desktop

## 4、快速进入桌面快捷键
（1）System Settings -> Appearance -> Add show desktop icon to the launcher
（2）Ctrl + Win + D：快速显示桌面 / 恢复屏幕显示

## 5、查找包含某个命令的安装包
apt-cache search vim

## 6、创建连接
ln -s 源文件 目标(新生成的文件名)

## 7、文件目录：
/etc/hosts：ip

## 8、重启及关机
reboot
shutdown -r now 立刻重启 (root用户使用)
shutdown -r 10 过10分钟自动重启 (root用户使用)
shutdown -r 20:35 在时间为20:35时候重启 (root用户使用)
如果是通过shutdown命令设置重启的话，可以用shutdown -c命令取消重启
关机命令：
halt 立刻关机
poweroff 立刻关机
shutdown -h now 立刻关机 (root用户使用)
shutdown -h 10 10分钟后自动关机
如果是通过shutdown命令设置关机的话，可以用shutdown -c命令取消重启
## 9、命令行安装卸载deb包
sudo dpkg -i 软件包名.deb
卸载.deb文件，可以使用Adept，或输入：
sudo apt-get remove 软件包名称
## 10、查看已安装
dpkg -l |grep virtualbox