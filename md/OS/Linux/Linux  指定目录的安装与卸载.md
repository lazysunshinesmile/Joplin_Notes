Linux  指定目录的安装与卸载

软件安装卸载，分几种情况：
**A：RPM包：**

这种软件包就像windows的EXE安装文件一样，各种文件已经编译好，并打了包，哪个文件该放到哪个文件夹，都指定好了，安装非常方便，在图形界面里你只需要双击就能自动安装。

  如果指定Linux安装软件时所需要安装到的目录 为软件包指定安装目录：要加 -relocate 参数；下面的举例是把gaim-1.3.0-1.fc4.i386.rpm指定安装在 /opt/gaim 目录中：

[root@localhost RPMS]# rpm -ivh --relocate /=/opt/gaim gaim-1.3.0-1.fc4.i386.rpm

Preparing... ######### [100%]
1:gaim ####### [100%]
[root@localhost RPMS]# ls /opt/
gaim

为软件包指定安装目录：要加 -relocate 参数；下面的举例是把lynx-2.8.5-23.i386.rpm 指定安装在 /opt/lynx 目录中：

[root@localhost RPMS]# rpm -ivh --relocate /=/opt/lynx --badreloc lynx-2.8.5-23.i386.rpm

Preparing... ######### [100%]
1:lynx ######## [100%]
==如何卸载:
1、打开一个SHELL终端
2、因为Linux下的软件名都包括版本号，所以卸载前最好先确定这个软件的完整名称。
查找RPM包软件：rpm -qa ×××*

注意：×××指软件名称开头的几个字母，不要求写全，但别错，*就是通配符号“*”，即星号，如你想查找机子里安装的REALPLAYER软件，可以输入：rpm -qa realplay*

3、找到软件后，显示出来的是软件完整名称，如firefox-1.0.1-1.3.2
执行卸载命令：rpm -e firefox-1.0.1-1.3.2
===安装目录，执行命令查找：rpm -ql firefox-1.0.1-1.3.2

**B：tar.gz（bz或bz2等）结尾的源代码包**
这种软件包里面都是源程序，没有编译过，需要编译后才能安装，安装方法为:
1、打开一个SHELL，即终端
2、用CD 命令进入源代码压缩包所在的目录
3、根据压缩包类型解压缩文件(*代表压缩包名称)
tar -zxvf ****.tar.gz
tar -jxvf ****.tar.bz(或bz2)
4、用CD命令进入解压缩后的目录
5、输入编译文件命令：./configure（有的压缩包已经编译过，这一步可以省去）
6、然后是命令：make
7、再是安装文件命令：make install
8、安装完毕

====指定安装目录：注意make install命令过程中的安装目录，或者阅读安装目录里面的readme文件，当然最好的办法是在安装的过程中指定安装目录，即在./configure命令后面加参数--prefix=/**，可以通过./configure –help命令查看程序支持哪些参数。

如：./configure --prefix=/usr/local/aaaa，即把软件装在/usr/local/路径的aaaa这个目录里。一般的软件的默认安装目录在/usr/local或者/opt里，可以到那里去找找

===如何卸载：
1、打开一个SHELL，即终端
2、用CD 命令进入编译后的软件目录，即安装时的目录
3、执行反安装命令：make uninstall

**C：以bin结尾的安装包**，
这种包类似于RPM包，安装也比较简单
1、打开一个SHELL，即终端
2、用CD 命令进入源代码压缩包所在的目录
3、给文件加上可执行属性：chmod +x ******.bin（中间是字母x，小写）
4、执行命令：./******.bin(realplayer for linux就是这样的安装包)
===如何卸载：把安装时选择的安装目录删除就OK
===执行安装过程中可以指定安装目录，类似于Windows下安装。