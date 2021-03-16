Python ubuntu 安装及使用

Python ubuntu 安装及使用

# python 安装
为了彻底改变系统中默认使用的 python 版本，可以用 update-alternatives 命令。
首先查看一下目前系统中是否已经设置了 python 的 alternatives，用如下命令：
`update-alternatives --list python`
如果报错
`update-alternatives: error: no alternatives for python`
说明之前没有设置过。可以用如下方法设置
```
sudo update-alternatives --install /usr/bin/python python /usr/bin/python2.7 1
sudo update-alternatives --install /usr/bin/python python /usr/bin/python3.5 2
```
注意上述命令中最后的数值 1 和 2. 数值越大，优先级越高。
上述命令中 --install 后边的四个参数依次对应 <link> <name> <path> <priority>。通过这两个命令，就将 python 与两个可执行文件 （/usr/bin/python2.7 和 /usr/bin/python3.5）对应了起来，而且默认使用 /usr/bin/python3.5。

此时，再通过命令查看一下
`update-alternatives --list python`
会看到 python 的两个 alternatives
```
/usr/bin/python2.7
/usr/bin/python3.5
```
要设置 python 的版本，可以用命令

sudo update-alternatives --config python
会显示如下列表
![d7ac277a0de392ed25a235fe91d424e8.png](../_resources/44b1cea5773b4fbb82d11e46705d50ae.png)
这里就可以选择系统默认使用的 python 版本了。默认是优先级较高的 python3.5，如果选择 1， 则会切换到 python 2.7.

# pip 安装
1. python3
`sudo apt-get install python3-pip`

2. python2
`sudo apt-get install python-pip`

# pip 指定源
1. 永久修改：
```
vim ~/.pip/pip.conf
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
```
2. 临时修改
`pip install module_name -i https://pypi.tuna.tsinghua.edu.cn/simple`

# pip 下载指定版本
`sudo pip install pyasn1==0.4.8`

# 安装出错，错误如下
```
  ERROR: Command errored out with exit status 1:
   command: /usr/bin/python /usr/share/python-wheels/pep517-0.8.2-py2.py3-none-any.whl/pep517/_in_process.py get_requires_for_build_wheel /tmp/tmpvyomeuzu
       cwd: /tmp/pip-install-ge3ev15a/pyusb
```
安装代码中加入 `--no-use-pep517`
如：`sudo pip install pyusb -i https://pypi.tuna.tsinghua.edu.cn/simple --no-use-pep517`