解决：no matching key exchange method found. Their of

解决：no matching key exchange method found. Their offer: diffie-hellman-group1-sha1_u011983700的专栏-CSDN博客

<img width="36" height="32" src="../_resources/a35f0793f2da457db1c81b06d8b82332.png"/>

[juelianhuayao](https://me.csdn.net/u011983700) 2018-08-16 11:36:16 <img width="24" height="24" src="../_resources/af62769440de4cc29e160834d61b5961.png"/>11362 <a id="blog_detail_zk_collection"></a><img width="20" height="20" src=":/35b4eca2475c4d679456131709f6376d"/>收藏 1 

最后发布:2018-08-16 11:36:16首次发布:2018-08-16 11:36:16

第一种方法，在当前用户的.ssh目录下新建config文件，如当前用户为root

```
vi  ~/.ssh/config
```

```
KexAlgorithms +diffie-hellman-group1-sha1
```

此方法只对当前用户生效，使用其他用户是又会报错。

第二种方法，修改/etc/ssh/ssh_config文件，在最末尾加入即可。

```
KexAlgorithms +diffie-hellman-group1-sha1
```