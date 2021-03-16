JVM与DVM的关系

[<img width="40" height="40" src="../../_resources/7f5c2a10513c4117964e6060e80b1eb0.jpg"/>](https://www.jianshu.com/u/5856b99e969b)

2017.02.23 12:45:06字數 612閱讀 789

### 【】名词解释  

JVM是Java Virtual Machine的缩写，叫做 java虚拟机。

DVM是Dalvik Virtual Machine的缩写，叫做 Dalvik虚拟机。

### 【】二者的关系

DVM是针对JVM而言的，JVM是Oricle公司的产品，担心版权问题，既然java是开源的，索性就研究了JVM，写出了DVM。

DVM是针对移动设备而生的虚拟机。

### 【】二者的区别

#### 一、基于的架构不一样

java是基于栈的架构，栈是内存上面一段连续的存储空间。

Android是基于寄存器的架构，寄存器是cpu上的一块存储空间。

所以cpu直接访问自己上面的一块空间的数据的效率肯定要大于访问内存上面的数据。

#### 二、执行的字节码文件不一样

JVM：    .java-->.class-->.jar

DVM：    .java-->.class-->.dex

![](https://upload-images.jianshu.io/upload_images/4624988-f971f0ca9a9bca30.png?imageMogr2/auto-orient/strip|imageView2/2/w/554/format/webp)

.jar文件里面包含多个.class文件，每个.class文件里面包含了该类的头信息（如编译版本）、常量池、类信息、域、方法、属性等等，当JVM加载该.jar文件的时候，会加载里面的所有的.class文件，这样会很慢，而移动设备的内存本来就很小，不可能像JVM这样加载，所以它使用的不是.jar文件，而是.apk文件，该文件里面只包含了一个.dex文件，这个.dex文件里面将所有的.class里面所包含的信息全部整合在一起了，这样再加载就很快了。.class文件存在很多的冗余信息，dex工具会去除冗余信息，并把所有的.class文件整合到.dex文件中。减少了I/O操作。  

#### 三、运行环境的区别

DVM： 每个应用启动都运行一个单独的DVM，每个DVM单独占用一个Linux进程。

JVM： 只能运行一个实例，即所有的应用都运行在同一个JVM中。

* * *

【】声明：

本文是在参考其他人资料的基础上做的一个归纳，多部分是复制过来的。也加入了一部分自己的理解。如有理解不到位的，希望读者可以指正。  

参考资料：[www.cnblogs.com/Chenshuai7/p/5277807.html](https://link.jianshu.com/?t=http://www.cnblogs.com/Chenshuai7/p/5277807.html)   

"小禮物走一走，來簡書關注我"

还没有人赞赏，支持一下

[<img width="48" height="48" src="../../_resources/7f5c2a10513c4117964e6060e80b1eb0.jpg"/>](https://www.jianshu.com/u/5856b99e969b)

總資產3 (約0.28元)共写了5153字获得4个赞共0个粉丝

### 推薦閱讀[更多精彩內容](https://www.jianshu.com/)

- dumpsys命令功能很强大，能dump系统服务的各种状态，非常有必要熟悉该命令的用法以及含义。 一、 概述\[ht...
    
- 前言 回顾一下自己这段时间的经历，三月份的时候，疫情原因公司通知了裁员，我匆匆忙忙地出去面了几家，但最终都没有拿到...
    
    [![](https://upload-images.jianshu.io/upload_images/25052971-77367e6a084a36eb?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)](https://www.jianshu.com/p/95dacea50488)
- 作者：RicardoMJiang 在下2017届毕业生，一名普通的211硕士，目前从事android开发工作已经3...
    
    [![](https://upload-images.jianshu.io/upload_images/23089205-3003cea199f48f82?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)](https://www.jianshu.com/p/3680f00aaa8f)
- 根据上篇文章了解，对应ClassLoader初始化时，会将对应的dex加载到内存。接下来再继续看Class的加载、...
    
    [![](https://upload-images.jianshu.io/upload_images/2828107-b81b534379aa1491.png?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)](https://www.jianshu.com/p/a29b3d6d2e9c)
- 年底了，xjjdog决定来一篇实用的硬核文章。本篇文章多达38道面试题，照顾到了JVM的方方面面，都是常见的题目。...
    
    [![](https://upload-images.jianshu.io/upload_images/25454423-8aed9a23acb793a2?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)](https://www.jianshu.com/p/14018c414d8b)