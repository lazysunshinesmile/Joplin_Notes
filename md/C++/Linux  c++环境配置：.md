Linux  c++环境配置：

Linux  c++环境配置：

﻿1.安装

**(1).gcc
ubuntu下自带gcc编译器。可以通过“gcc -v”命令来查看是否安装。**
(2).g++
安装g++编译器，可以通过命令“sudo apt-get install build-essential”实现。
执行完后，就完成了gcc,g++,make的安装。build-essential是一整套工具，gcc，libc等等。
通过“g++ -v”可以查看g++是否安装成功。

注：“sudo apt-get install build-essential –fix-missing”，这个命令是修补安装build-essential，即已安装了部分build-essential，但没有安装完全，此时可以使用该命令继续安装build-essential。

(3).gfortran
输入命令：sudo apt-get install gfortran
通过命令“gfortran -v”，可以查看gfortran是否安装成功。