ubuntu无法添加PPA的解决办法

ubuntu无法添加PPA的解决办法

<img width="36" height="32" src=":/f18c3a174c1c47bf8334eea416dfd763"/>

[木盏](https://me.csdn.net/leviopku) 2019-09-20 14:50:06 <img width="24" height="24" src=":/5458cf4cd75947e58b81fb1b29a61a22"/>9703 <a id="blog_detail_zk_collection"></a><img width="20" height="20" src="../../_resources/a4a6fdfbf6be4b7092af90fc959a91c4.png"/>收藏 10 

最后发布:2019-09-20 14:50:06首次发布:2019-09-20 14:43:16

版权声明：本文为博主原创文章，遵循 [CC 4.0 BY-SA](http://creativecommons.org/licenses/by-sa/4.0/) 版权协议，转载请附上原文出处链接和本声明。

使用apt-get安装软件经常需要先添加PPA，这样才能下载到apt-cache以外的软件。

然而，我在输入"sudo add-apt-repository ppa:xxx"以后，会遇到两个问题：**1\. 长时间没有响应； 2，添加失败(ERROR: user or team does not exist)**

以上主要由两种原因导致：1\. CA证书损坏；2. 没有绕过代理；

我们先重装一遍CA证书：

```
sudo apt-get install --reinstall ca-certificates
```

如果还不行，我们就绕过代理，加一个"**-E**"：

```
sudo -E add-apt-repository --update ppa:ubuntu-toolchain-r/test
```

我采用上述2步，解决了困扰我好几个月的问题~

非常感谢这篇文章：[https://blog.bflyer.com/2016/03/27/Fix-Cannot-Add-PPA-Error-In-Ubuntu-14-04-Linux-Mint-17/](https://blog.bflyer.com/2016/03/27/Fix-Cannot-Add-PPA-Error-In-Ubuntu-14-04-Linux-Mint-17/)