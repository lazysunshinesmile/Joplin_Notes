安装nfs服务器

安装nfs服务器

1. 安装
`sudo apt install nfs-kernel-server`
2. 配置共享目录
    1. 新建目录
        `sudo mkdir /mnt/testdir`
    2. 修改配置文件
        `sudo vi /etc/exports`
        最底部加入：
        `/mnt/testdir *(rw,sync,no_subtree_check,no_root_squash)`
        >其中/mnt/testdir表示将要共享出去的目录；
        >后边的星号（*）表示所有用户都可连接这台服务器，如果想限定某个网段的用户可以写成类似192.168.1.*这样的形式，如果想限定只有某个用户可以访问可以直接写这个用户的IP；
        > 括号中的部分，rw代表客户端将来可读可写，如果写成ro就是只读
        > sync表示同步？表示所有数据在请求时写入共享，这个不太理解；
        > no_subtree_check表示不检测父目录的权限；
        > no_root_squash表示不能降低root用户的权限；
3. 重启
`sudo /etc/init.d/nfs-kernel-server restart`
4. 更改支持的版本
`sudo vim /etc/default/nfs-kernel-server`   
最后末尾添加：
`RPCNFSDOPTS="--nfs-version 2,3,4 --debug --syslog"`
第三步重启