git配置多个帐号

git配置多个帐号

### 背景

公司最近将代码由svn迁移到了gitlab来托管，我个人平时也经常使用Github，可是公司邮箱与我的Github使用的邮箱是不同的，SSH Key是由邮箱来生成的，这样子就造成了冲突，本文记录了解决此类问题的方案。

- 公司GitLab服务邮箱 **[xiangsun@grandstream.cn](mailto:xiangsun@grandstream.cn)**
- 个人GitHub服务邮箱 **[lazysunshinesmile@gmail.com](mailto:lazysunshinesmile@gmail.com)**

### 解决方案

使用 git config工具，对不同的域名采用不同的认证密钥。

#### git config介绍

Git 提供了一个叫做 git config 的工具，专门用来配置或读取相应的工作环境变量。而正是由这些环境变量，决定了 Git 在各个环节的具体工作方式和行为。这些变量可以存放在以下三个不同的地方：

- /etc/gitconfig文件：系统中对所有用户都普遍适用的配置。若使用git config时用--system` 选项，读写的就是这个文件。
- ~/.gitconfig 文件：用户目录下的配置文件只适用于该用户。若使用 git config 时用 --global 选项，读写的就是这个文件。
- 当前项目的 Git 目录中的配置文件（也就是工作目录中的 .git/config 文件）：这里的配置仅仅针对当前项目有效。每一个级别的配置都会覆盖上层的相同配置，所以 .git/config 里的配置会覆盖 /etc/gitconfig 中的同名变量。

**由于经常使用GitHub看一些开源项目，相对公司的GitLab项目来说还是多一些的，所以将GitHub的配置到~/.gitconfig 文件，将公司项目配置到各自的项目工作目录。**

#### 配置用户名、邮箱

- 公司的
```
sun@Sun:~/.ssh$ git config --global user.email xiangsun@grandstream.cn
sun@Sun:~/.ssh$ git config --global user.name xiangsun
sun@Sun:~/.ssh$ git config --get user.name
xiangsun
sun@Sun:~/.ssh$ git config --get user.email
xiangsun@grandstream.cn
sun@Sun:~/.ssh$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/sun/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/sun/.ssh/id_rsa.
Your public key has been saved in /home/sun/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:Hj/XjfWEQcn1jq9cRJqOsKsS6X9cd/u+qiyo3/etDBs sun@Sun
The key's randomart image is:
+---[RSA 2048]----+
|             ..o.|
|             .o .|
|              . o|
|               O |
|       .S .   = =|
|      o. o o.+.Bo|
|     . .o.+Eo.+.*|
|      o...+==..+ |
|     .o+o+++o==+=|
+----[SHA256]-----+
sun@Sun:~/.ssh$ ls
id_rsa  id_rsa.pub
```



#### 配置config文件

touch ~/.ssh/config



#### 设置公钥

以github为例：
1. 登录[https://github.com/](https://link.jianshu.com?t=https://github.com/)
2. 点击右上角个人头像图标，然后点击Settings选项
3. 选择SSH and GPG keys选项，然后点击右上角New SSH key按钮

4. Title填上自定义的名字方便区分即可，然后打开~/.ssh/id_rsa.pub文件，将文件的所有内容copy粘贴到key对应的文本域即可

> grandstream
> https://192.168.120.246/#/settings/ssh-keys 中填入id_rsa.pub的内容。

#### 验证SSH连接

打开Terminal，输入一下命令来校验：
ssh -T git@github.com
若看到如下信息，则代表连接成功了：

Hi zhouyang-cn! You've successfully authenticated, but GitHub does not provide shell access.

若出现问题，可以参考[github的文档](https://link.jianshu.com?t=https://help.github.com/articles/testing-your-ssh-connection/)

#### 如果在已有的项目中 `git pull -r` 出现`Unable to negotiate with 192.168.120.246 port 29418: no matching key exchange method found. Their offer: diffie-hellman-group1-sha1
fatal: Could not read from remote repository.`
修改`～/.ssh/config`,增加如下内容：
```
Host *
KexAlgorithms +diffie-hellman-group1-sha1
```


### 在AS中的Problems and solutions（未验证）

经过上面的配置后，在as里进行代码修改、commit和push操作时，会出现push失败的问题，错误信息如下：
Failed with error: fatal: Could not read from remote repository
通过google搜索后找出问题产生的原因：
由于AS默认使用的是Built-in（自带的）进行SSH校验，而我们上面设置的是mac本机的SSH校验。所以解决方案如下：
settings >> Version Control >> Git >> SSH executable 改为 native

[参考]

https://qdct01.baidupcs.com/file/f96bae1db12c0d6d565e6401a5661717?bkt=p3-1400f96bae1db12c0d6d565e6401a5661717de4765fb0000000015d6&fid=1848182417-250528-815914623972887&time=1556588980&sign=FDTAXGERLQBHSKfW-DCb740ccc5511e5e8fedcff06b081203-tThXwY8d4AMVY%2FXAsAC0k9yJpzc%3D&to=90&size=5590&sta_dx=5590&sta_cs=0&sta_ft=md&sta_ct=0&sta_mt=0&fm2=MH%2CYangquan%2CAnywhere%2C%2Czhejiang%2Cct&ctime=1556588955&mtime=1556588955&resv0=cdnback&resv1=0&vuk=1848182417&iv=0&htype=&newver=1&newfm=1&secfm=1&flow_ver=3&pkey=1400f96bae1db12c0d6d565e6401a5661717de4765fb0000000015d6&sl=68616270&expires=8h&rt=pr&r=107615788&mlogid=2769443147704973906&vbdid=1590203453&fin=git%E9%85%8D%E7%BD%AE%E5%A4%9A%E4%B8%AA%E5%B8%90%E5%8F%B7.md&fn=git%E9%85%8D%E7%BD%AE%E5%A4%9A%E4%B8%AA%E5%B8%90%E5%8F%B7.md&rtype=1&dp-logid=2769443147704973906&dp-callid=0.1.1&hps=1&tsl=200&csl=200&csign=c4RJukU1fw7WR6CQLtPBtFPzEIo%3D&so=0&ut=6&uter=4&serv=0&uc=4285954612&ti=8525e99dbc6685b111e32ff0ede44cd9da84939beb542e6f&by=themis