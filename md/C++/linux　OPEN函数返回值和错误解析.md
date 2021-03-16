linux　OPEN函数返回值和错误解析

linux　OPEN函数返回值和错误解析

[<img width="40" height="40" src=":/e5d931ede4fd4436bf6bc212a411f25f"/>](https://www.jianshu.com/u/f5f8bb96c487)

2016.06.27 15:01:46字數 1,308閱讀 10,573

经常在调用linux 系统api 的时候会出现一些错误，比方说使用open() write() creat()之类的函数有些时候会返回-1，也就是调用失败，这个时候往往需要知道失败的原因。这个时候使用errno这个全局变量就相当有用了。  
在程序代码中包含 #include &lt;errno.h&gt;,然后每次程序调用失败的时候，系统会自动用用错误代码填充errno这个全局变量，这样你只需要读errno这个全局变量就可以获得失败原因了。  
例如：  
&lt;code&gt;  
#include &lt;stdio.h&gt;  
#include &lt;string.h&gt;  
#include &lt;errno.h&gt;  
int main(void)  
{  
int fd;  
extern int errno;

if((fd = open("/dev/dsp",O_WRONLY)) < 0)  
{  
printf("errno=%d\
",errno);  
}

exit(0);  
}  
&lt;/code&gt;  
如果dsp设备忙的话errno值将是16。

errno.h中定义的错误代码值如下：

查 看错误代码errno是调试程序的一个重要方法。当linuc C api函数发生异常时,一般会将errno变量(需include errno.h)赋一个整数值,不同的值表示不同的含义,可以通过查看该值推测出错的原因。在实际编程中用这一招解决了不少原本看来莫名其妙的问题。比较 麻烦的是每次都要去linux源代码里面查找错误代码的含义，现在把它贴出来，以后需要查时就来这里看了。  
以下来自linux 2.4.20-18的内核代码中的/usr/include/asm/errno.h  
&lt;code&gt;  
#ifndef \_I386\_ERRNO_H  
#define \_I386\_ERRNO_H  
#define EPERM 1 /* Operation not permitted */  
#define ENOENT 2 /* No such file or directory */  
#define ESRCH 3 /* No such process */  
#define EINTR 4 /* Interrupted system call */  
#define EIO 5 /* I/O error */  
#define ENXIO 6 /* No such device or address */  
#define E2BIG 7 /* Arg list too long */  
#define ENOEXEC 8 /* Exec format error */  
#define EBADF 9 /* Bad file number */  
#define ECHILD 10 /* No child processes */  
#define EAGAIN 11 /* Try again */  
#define ENOMEM 12 /* Out of memory */  
#define EACCES 13 /* Permission denied */  
#define EFAULT 14 /* Bad address */  
#define ENOTBLK 15 /* Block device required */  
#define EBUSY 16 /* Device or resource busy */  
#define EEXIST 17 /* File exists */  
#define EXDEV 18 /* Cross-device link */  
#define ENODEV 19 /* No such device */  
#define ENOTDIR 20 /* Not a directory */  
#define EISDIR 21 /* Is a directory */  
#define EINVAL 22 /* Invalid argument */  
#define ENFILE 23 /* File table overflow */  
#define EMFILE 24 /* Too many open files */  
#define ENOTTY 25 /* Not a typewriter */  
#define ETXTBSY 26 /* Text file busy */  
#define EFBIG 27 /* File too large */  
#define ENOSPC 28 /* No space left on device */  
#define ESPIPE 29 /* Illegal seek */  
#define EROFS 30 /* Read-only file system */  
#define EMLINK 31 /* Too many links */  
#define EPIPE 32 /* Broken pipe */  
#define EDOM 33 /* Math argument out of domain of func */  
#define ERANGE 34 /* Math result not representable */  
#define EDEADLK 35 /* Resource deadlock would occur */  
#define ENAMETOOLONG 36 /* File name too long */  
#define ENOLCK 37 /* No record locks available */  
#define ENOSYS 38 /* Function not implemented */  
#define ENOTEMPTY 39 /* Directory not empty */  
#define ELOOP 40 /* Too many symbolic links encountered */  
#define EWOULDBLOCK EAGAIN /* Operation would block */  
#define ENOMSG 42 /* No message of desired type */  
#define EIDRM 43 /* Identifier removed */  
#define ECHRNG 44 /* Channel number out of range */  
#define EL2NSYNC 45 /* Level 2 not synchronized */  
#define EL3HLT 46 /* Level 3 halted */  
#define EL3RST 47 /* Level 3 reset */  
#define ELNRNG 48 /* Link number out of range */  
#define EUNATCH 49 /* Protocol driver not attached */  
#define ENOCSI 50 /* No CSI structure available */  
#define EL2HLT 51 /* Level 2 halted */  
#define EBADE 52 /* Invalid exchange */  
#define EBADR 53 /* Invalid request descriptor */  
#define EXFULL 54 /* Exchange full */  
#define ENOANO 55 /* No anode */  
#define EBADRQC 56 /* Invalid request code */  
#define EBADSLT 57 /* Invalid slot */  
#define EDEADLOCK EDEADLK  
#define EBFONT 59 /* Bad font file format */  
#define ENOSTR 60 /* Device not a stream */  
#define ENODATA 61 /* No data available */  
#define ETIME 62 /* Timer expired */  
#define ENOSR 63 /* Out of streams resources */  
#define ENONET 64 /* Machine is not on the network */  
#define ENOPKG 65 /* Package not installed */  
#define EREMOTE 66 /* Object is remote */  
#define ENOLINK 67 /* Link has been severed */  
#define EADV 68 /* Advertise error */  
#define ESRMNT 69 /* Srmount error */  
#define ECOMM 70 /* Communication error on send */  
#define EPROTO 71 /* Protocol error */  
#define EMULTIHOP 72 /* Multihop attempted */  
#define EDOTDOT 73 /* RFS specific error */  
#define EBADMSG 74 /* Not a data message */  
#define EOVERFLOW 75 /* Value too large for defined data type */  
#define ENOTUNIQ 76 /* Name not unique on network */  
#define EBADFD 77 /* File descriptor in bad state */  
#define EREMCHG 78 /* Remote address changed */  
#define ELIBACC 79 /* Can not access a needed shared library */  
#define ELIBBAD 80 /* Accessing a corrupted shared library */  
#define ELIBSCN 81 /* .lib section in a.out corrupted */  
#define ELIBMAX 82 /* Attempting to link in too many shared libraries */  
#define ELIBEXEC 83 /* Cannot exec a shared library directly */  
#define EILSEQ 84 /* Illegal byte sequence */  
#define ERESTART 85 /* Interrupted system call should be restarted */  
#define ESTRPIPE 86 /* Streams pipe error */  
#define EUSERS 87 /* Too many users */  
#define ENOTSOCK 88 /* Socket operation on non-socket */  
#define EDESTADDRREQ 89 /* Destination address required */  
#define EMSGSIZE 90 /* Message too long */  
#define EPROTOTYPE 91 /* Protocol wrong type for socket */  
#define ENOPROTOOPT 92 /* Protocol not available */  
#define EPROTONOSUPPORT 93 /* Protocol not supported */  
#define ESOCKTNOSUPPORT 94 /* Socket type not supported */  
#define EOPNOTSUPP 95 /* Operation not supported on transport endpoint */  
#define EPFNOSUPPORT 96 /* Protocol family not supported */  
#define EAFNOSUPPORT 97 /* Address family not supported by protocol */  
#define EADDRINUSE 98 /* Address already in use */  
#define EADDRNOTAVAIL 99 /* Cannot assign requested address */  
#define ENETDOWN 100 /* Network is down */  
#define ENETUNREACH 101 /* Network is unreachable */  
#define ENETRESET 102 /* Network dropped connection because of reset */  
#define ECONNABORTED 103 /* Software caused connection abort */  
#define ECONNRESET 104 /* Connection reset by peer */  
#define ENOBUFS 105 /* No buffer space available */  
#define EISCONN 106 /* Transport endpoint is already connected */  
#define ENOTCONN 107 /* Transport endpoint is not connected */  
#define ESHUTDOWN 108 /* Cannot send after transport endpoint shutdown */  
#define ETOOMANYREFS 109 /* Too many references: cannot splice */  
#define ETIMEDOUT 110 /* Connection timed out */  
#define ECONNREFUSED 111 /* Connection refused */  
#define EHOSTDOWN 112 /* Host is down */  
#define EHOSTUNREACH 113 /* No route to host */  
#define EALREADY 114 /* Operation already in progress */  
#define EINPROGRESS 115 /* Operation now in progress */  
#define ESTALE 116 /* Stale NFS file handle */  
#define EUCLEAN 117 /* Structure needs cleaning */  
#define ENOTNAM 118 /* Not a XENIX named type file */  
#define ENAVAIL 119 /* No XENIX semaphores available */  
#define EISNAM 120 /* Is a named type file */  
#define EREMOTEIO 121 /* Remote I/O error */  
#define EDQUOT 122 /* Quota exceeded */  
#define ENOMEDIUM 123 /* No medium found */  
#define EMEDIUMTYPE 124 /* Wrong medium type */  
#endif  
&lt;/code&gt;  
同时也可以使用strerror（）来自己翻译  
如：  
&lt;code&gt;  
#include &lt;stdio.h&gt;  
#include &lt;string.h&gt;  
#include &lt;errno.h&gt;  
int main(void)  
{  
int fd;  
extern int errno;

if((fd = open("/dev/dsp",O_WRONLY)) < 0)  
{  
printf("errno=%d\
",errno);  
char * mesg = strerror(errno);  
printf("Mesg:%s\
",mesg);  
}

exit(0);  
}  
&lt;/code&gt;  
dsp设备忙的话将输出如下：  
errno=16  
Mesg:Device or resource busy

更多精彩內容下載簡書APP

"小禮物走一走，來簡書關注我"

还没有人赞赏，支持一下

[<img width="46" height="46" src=":/e5d931ede4fd4436bf6bc212a411f25f"/>](https://www.jianshu.com/u/f5f8bb96c487)

總資產1 (約0.05元)共写了4007字获得2个赞共2个粉丝

### 推薦閱讀[更多精彩內容](https://www.jianshu.com/)

- NAME dnsmasq - A lightweight DHCP and caching DNS server....
    
- PLEASE READ THE FOLLOWING APPLE DEVELOPER PROGRAM LICENSE...
    
    [![](https://upload.jianshu.io/users/upload_avatars/1389150/26db72c5-e584-4325-8d28-c20624c93065.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/48/h/48/format/webp)念念不忘的](https://www.jianshu.com/u/4eb15a4a68c8)閱讀 10,909評論 5贊 6
    
- 1、TCP状态linux查看tcp的状态命令：1）、netstat -nat 查看TCP各个状态的数量2）、lso...
    
    [![](https://upload.jianshu.io/users/upload_avatars/3269883/5bbac787-bb0a-4a9d-af08-4677b8c4e8ce.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/48/h/48/format/webp)北辰青](https://www.jianshu.com/u/067c538d5004)閱讀 5,919評論 0贊 11
    
- 这是一个平凡小镇的平凡的傍晚。不远处的山腰上，剩下的余晖与霪雨后的湿气形成一条稀薄的雾霭，交错绵延的山丘像几笔湛蓝...
    
    [![](https://upload-images.jianshu.io/upload_images/3887963-866de3f66d18e95f.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)](https://www.jianshu.com/p/fb9f4cb4af35)
- 2017年10月21日 晴、多云 下午4点钟，小叔叔来接我们回家。回到家里，已经有姑姑来了！故...
    
    [![](https://upload.jianshu.io/users/upload_avatars/6849493/0e2bf894-9984-48a0-b305-66b645fb8a0f.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/48/h/48/format/webp)何雨桐](https://www.jianshu.com/u/121cfbaaf855)閱讀 28評論 1贊 0