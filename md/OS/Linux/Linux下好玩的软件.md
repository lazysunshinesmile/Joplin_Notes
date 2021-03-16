Linux下好玩的软件

Linux下好玩的软件

[TOC]
# android studio 创建桌面快捷方式
Tools -> Create desktop entry
![b466ddeba85f5a46b604123acfc7657d.png](../../_resources/8c094bef48364dd294dacd6a38605daa.png)

# sqlitebrowser
`sudo apt-get install sqlitebrowser`

# Joplin 笔记
[Joplin](../../工具使用/Joplin/Joplin.md)

# Charles

1. wget -q -O - https://www.charlesproxy.com/packages/apt/PublicKey | sudo apt-key add -

2. sudo sh -c 'echo deb https://www.charlesproxy.com/packages/apt/ charles-proxy main > /etc/apt/sources.list.d/charles.list'

3. sudo apt update

4. sudo apt install charles-proxy

5. 注册码
 Registered Name: https://zhile.io
 License Key: 48891cf209c6d32bf4

6. 证书安装
     - 保存charlesRoot.cer(选择的是cer文件)
    运行charles软件，菜单Help->SSL Proxying->Save Charles Root Certificate…
    保存名为CharlesRoot.cer
    
     - 将cer转为pem格式的crt文件
    openssl x509 -inform der -in CharlesRoot.cer -outform pem -out CharlesRoot.crt
       
     - 安装crt文件并更新ca-certificates   
         ```shell
         sudo cp CharlesRoot.crt /usr/share/ca-certificates
         sudo dpkg-reconfigure ca-certificates //选择ask,勾选CharlesRoot.crt并确认
         ```

7. Charles配置

    - 开启http代理
      运行charles软件，菜单Proxy->Proxy Setting->标签Proxies下勾选Enable transparent HTTP proxying
        ![9127e135aa61dc7cc0530ab2bf1e104c.png](:/c0cee6ee61e8445294dc5e91450de435)
    
    
    - 允许访问http和https协议
       在proxy>ssl proxy settings>Enable SSL Proxying, 然后add， host和 port都填 *（如果出现抓https的包是unknown的话，一般是由于这个没设置），至此，服务端配置结束
       ![25343fe6c052691635ade6c0cdd12dca.png](../../_resources/599bd1f436df4d0fbb417c973b01c191.png)


8. 手机移动端与charles所在电脑在同一个局域网下
使设备跟charles所在电脑连在同个局域网，
    - 手机
    如果手机使用wifi，找到该wifi的高级设置，代理选择手动，代理服务器主机名填charles所在电脑的IP，可通过Help -> SSL Proxying -> Install Charles Root Certificate on a Mobile Device or Remote Browser查看，服务端口填8888
    - 若是纯系统的：
        ```
        export http_proxy=http://127.0.0.1:8888
        export https_proxy=http://127.0.0.1:8888
        ```
    
    - adb shell 控制代理：
    `adb shell settings put global http_proxy 代理IP地址:端口号`
    如：
    `adb shell settings put global http_proxy 127.0.0.1:8888`
    移除代理：
        ```
        adb shell settings delete global http_proxy
        adb shell settings delete global global_http_proxy_host
        adb shell settings delete global global_http_proxy_port
        ```
    

9. 客户端下载证书（先设置手机代理IP，再在这IP下下载证书）
    - 手机
    浏览器中输入 `chls.pro/ssl` 来安装证书
    若不是手机端，无法加入代理，使用上面的export，然后在shell中执行`wget chls.pro/ssl`
         > ps. 会报安装失败，到系统设置->安全->从设备内在或sd卡安装证书来安装
         *注：若手机是iphone，安装完证书后，需要到设置-> 通用 -> 关于本机 -> 证书信任设置,打开证书信任*

    - android 原生设备：
        1. 先用上面的命令设置代理
        2.  adb push ~/Downloads/charles-proxy-ssl-proxying-certificate.pem /sdcard/Download。证书可以先用手机下载出来，然后发送给pc端。
        3. am start -n com.android.settings/.Settings 打开设置，安全->从设备内在或sd卡安装证书来安装

10. Ubuntu 电脑：
在网络中设置代理：
![c207a6776be5f352ded7f658a8252184.png](../../_resources/b408f39190cc47168024ef077b1aa4e7.png)

- 抓包
 在charles软件中Proxy –> Start Recording
 
# scrcpy
[参考](https://www.zhyong.cn/posts/99d/)
    能获取到连接的android设备，并操作。
    - 安装
    `sudo snap install scrcpy`
    - 使用
        - 使用这个软件携带的adb连接：
        `/snap/bin/scrcpy.adb connect 192.168.124.96`
        > `/snap/bin/scrcpy.adb`这个命令可用于所有的本来的`adb`命令
        - 开始获取手机屏幕
        `/snap/bin/scrcpy`
    - 关闭手机屏幕
    ```
    scrcpy --turn-screen-off
    scrcpy -S
    ```

# ftp 安装
[参考](http://blog.csdn.net/qq_33279781/article/details/73607466)
    
    
# 录屏
    ```
    sudo apt-get install recordmydesktop
    recordmydesktop
    ```

# deepin 截屏
     - 百度网盘>工具>deepin 截图>Deepin_Screenshot_4.0.10.4-x86_64.AppImage
     - github搜索下载[Deepin_Screenshot_4.0.10.4-x86_64.AppImage](https://github.com/ZetaoYang/deepin-app-package/releases/download/v4.0.10.4/Deepin_Screenshot_4.0.10.4-x86_64.AppImage)

# flameshot截图
[apt install flameshot](https://github.com/flameshot-org/flameshot)

# apache
    > 一般自带的。加用户名密码鉴定步骤：
    
    - 修改http.conf文件
    设置指定目录的配置，例如设定test目录的配置，增加验证。
    ```
    <Directory "/www/test">
        Options Indexes
        AllowOverride AuthConfig
        Order allow,deny
        Allow from all
    </Directory>
    ```
    
    - 在指定目录下增加.htaccess文件
    文件内容如下：
    ```
    AuthName "sys"
    AuthType Basic
    AuthUserFile /www/test/.htpasswd
    require user admin
    ```
    > 第一行，验证提示信息。
    > 第二行，验证类别。
    > 第三行，密码文件所在路径，绝对路径。
    > 第四行，指定登录用户名。指定admin用户登录。
    
    - 增加.htpasswd文件
    `/usr/local/apache2/bin/htpasswd -c /www/test/.htpasswd admin`
    输入上面命令，htpasswd目录写自己的目录，最后一个参数即为登录账号名，会提示输入两次密码。
    
    - 重启apache
    `systemctl reload apache2`
    如出现如下错误:
    `AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 127.0.1.1. Set the 'ServerName' directive globally to suppress this message`
    解决办法：
    1. `echo "ServerName localhost" | sudo tee /etc/apache2/conf-available/fqdn.conf`
    2. `sudo a2enconf fqdn`
    3. `systemctl reload apache2`
       
# 查看实时网络
1. 生成文件
    ```
    vim /bin/net_oberver.sh
    
    #!/bin/bash
     
    ethn=$1
     
    while true
    do
     RX_pre=$(cat /proc/net/dev | grep $ethn | sed 's/:/ /g' | awk '{print $2}')
     TX_pre=$(cat /proc/net/dev | grep $ethn | sed 's/:/ /g' | awk '{print $10}')
     sleep 1
     RX_next=$(cat /proc/net/dev | grep $ethn | sed 's/:/ /g' | awk '{print $2}')
     TX_next=$(cat /proc/net/dev | grep $ethn | sed 's/:/ /g' | awk '{print $10}')
     
     clear
     echo -e "\t RX `date +%k:%M:%S` TX"
     
     RX=$((${RX_next}-${RX_pre}))
     TX=$((${TX_next}-${TX_pre}))
     
     if [[ $RX -lt 1024 ]];then
     RX="${RX}B/s"
     elif [[ $RX -gt 1048576 ]];then
     RX=$(echo $RX | awk '{print $1/1048576 "MB/s"}')
     else
     RX=$(echo $RX | awk '{print $1/1024 "KB/s"}')
     fi
     
     if [[ $TX -lt 1024 ]];then
     TX="${TX}B/s"
     elif [[ $TX -gt 1048576 ]];then
     TX=$(echo $TX | awk '{print $1/1048576 "MB/s"}')
     else
     TX=$(echo $TX | awk '{print $1/1024 "KB/s"}')
     fi
     
     echo -e "$ethn \t $RX $TX "
     
    done
    ``` 
2. 使用
`net_oberver.sh 接口`,例如`net_oberver.sh enp1s0`

# 视频播放器
> 选择视频播放器，右键视频文件，选择应用。
1. vlc 
    `sudo apt install vlc`
2. flowblade
    
    
# 音乐播放器
1. `Listen 1`
[官网](http://listen1.github.io/listen1/)
2. 下载：
- AppImage: [链接](https://github-production-release-asset-2e65be.s3.amazonaws.com/59187489/d17ce780-c308-11ea-9767-70e8730a8776?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20200724%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20200724T090807Z&X-Amz-Expires=300&X-Amz-Signature=f15435bf3e692a8a0497fa35db3e6e28a4ce992e5ac4cb9ab7fce8213b869a2c&X-Amz-SignedHeaders=host&actor_id=18102357&repo_id=59187489&response-content-disposition=attachment%3B%20filename%3DListen1_2.10.0_linux_x86_64.AppImage&response-content-type=application%2Foctet-stream)
- Deb [链接](https://github-production-release-asset-2e65be.s3.amazonaws.com/59187489/ebb6c580-c308-11ea-9f54-02673fca2530?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20200724%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20200724T092058Z&X-Amz-Expires=300&X-Amz-Signature=10ee42168b1b3143250cf1345d3915cb1dfca200072b286801b9339b3f6235c7&X-Amz-SignedHeaders=host&actor_id=18102357&repo_id=59187489&response-content-disposition=attachment%3B%20filename%3DListen1_2.10.0_linux_amd64.deb&response-content-type=application%2Foctet-stream)
3.安装
- 下载的AppImage格式的，修改为可执行文件即加上可执行权限。可以现在`/bin`目录创建指令运行这个AppImage,然后在`/usr/share/applications`创建桌面应用。
- 下载的Deb格式，直接安装。
    
    
# 清理系统垃圾(不是很好)
bleachbit
```
sudo add-apt-repository ppa:n-muench/programs-ppa
sudo apt-get update
sudo apt-get install bleachbit -y
sudo apt-get purge bleachbit
sudo add-apt-repository -r ppa:n-muench/programs-ppa
```

# 下载
```
sudo apt-get install aria2
#简单使用：只需要加上下载链接即可
aria2c ${link}
#分段下载
aria2c -s 2 ${link}
```

# Ubuntu Cleaner
```
$ sudo add-apt-repository ppa:gerardpuig/ppa
$ sudo apt-get update
$ sudo apt-get install ubuntu-cleaner
```

# Ubuntu 下Android模拟器的安装与卸载()
1. 安装：
```
$ sudo add-apt-repository ppa:morphis/anbox-support
$ sudo apt update
$ sudo apt install anbox-modules-dkms
$ sudo modprobe ashmem_linux
$ sudo modprobe binder_linux
$ sudo snap install --devmode --beta anbox
$ snap refresh --beta --devmode anbox
$ snap info anbox
```
2. 使用
```
anbox launch 
# 或者搜索anbox启动
```
3. 卸载
```
$ snap remove anbox
$ sudo apt install ppa-purge
$ sudo ppa-purge ppa:morphis/anbox-support
```


# git对比工具
```
sudo apt-get install meld
meld to_compare.c
```

# ubuntu 文件夹映射
1）sshfs文件系统的安装
通过ubuntu源直接安装，使用命令：
sudo apt-get install sshfs

2）挂载远程文件夹
挂载命令：
sshfs xxx@172.16.0.79:remote_dir local_dir

3）取消挂载
fusermount -u local_dir
    
