Linux系统调用之open(), close() - 乔部落格

Linux系统调用之open(), close() - 乔部落格

`open`函数可以打开或创建一个文件。

|     |     |
| --- | --- |
| 1<br><br>2<br><br>3<br><br>4<br><br>5<br><br>6<br><br>7<br><br>8 | `#include <sys/types.h>`<br><br>`#include <sys/stat.h>`<br><br>`#include <fcntl.h>`<br><br>`int` `open(``const` `char` `*pathname,` `int` `flags);`<br><br>`int` `open(``const` `char` `*pathname,` `int` `flags, mode_t mode);`<br><br>`返回值：成功返回新分配的文件描述符，出错返回-1并设置``errno` |

`pathname`参数是要打开或创建的文件名，和`fopen`一样，`pathname`既可以是相对路径也可以是绝对路径。

`flags`参数有一系列常数值可供选择，可以同时选择多个常数用按位或运算符连接起来，所以这些常数的宏定义都以`O_`开头，表示or。

必选项：以下三个常数中必须指定一个，且仅允许指定一个。

- `O_RDONLY` 只读打开
- `O_WRONLY` 只写打开
- `O_RDWR` 可读可写打开

以下可选项可以同时指定0个或多个，和必选项按位或起来作为`flags`参数。

- `O_APPEND` 表示追加。如果文件已有内容，这次打开文件所写的数据附加到文件的末尾而不覆盖原来的内容。
- `O_CREAT` 若此文件不存在则创建它。使用此选项时需要提供第三个参数`mode`，表示该文件的访问权限。
- `O_EXCL` 如果同时指定了`O_CREAT`，并且文件已存在，则出错返回。
- `O_TRUNC` 如果文件已存在，并且以只写或可读可写方式打开，则将其长度截断（Truncate）为0字节。
- `O_NONBLOCK` 对于设备文件，以`O_NONBLOCK`方式打开可以做非阻塞I/O（Nonblock I/O）。

第三个参数`mode`指定文件权限，可以用八进制数表示，比如0644表示`-rw-r--r--`，也可以用`S_IRUSR`、`S_IWUSR`等宏定义按位或起来表示，详见`open(2)`的Man Page。要注意的是，文件权限由`open`的`mode`参数和当前进程的`umask`掩码共同决定。

`close`函数关闭一个已打开的文件：

|     |     |
| --- | --- |
| 1<br><br>2<br><br>3<br><br>4 | `#include <unistd.h>`<br><br>`int` `close(``int` `fd);`<br><br>`返回值：成功返回0，出错返回-1并设置``errno` |

参数fd是要关闭的文件描述符。需要说明的是，当一个进程终止时，内核对该进程所有尚未关闭的文件描述符  
调用close关闭，所以即使用户程序不调用close，在终止时内核也会自动关闭它打开的所有文件。但是对于一  
个长年累月运行的程序（比如网络服务器），打开的文件描述符一定要记得关闭，否则随着打开的文件越来越  
多，会占用大量文件描述符和系统资源。  
给个例子：

|     |     |
| --- | --- |
| 1<br><br>2<br><br>3<br><br>4<br><br>5<br><br>6<br><br>7<br><br>8<br><br>9<br><br>10<br><br>11<br><br>12<br><br>13<br><br>14 | `#include <error.h>`<br><br>`#include <sys/types.h>`<br><br>`#include <sys/stat.h>`<br><br>`#include <fcntl.h>`<br><br>`int` `main(``int` `argc,` `const` `char` `*argv[])`<br><br>`{`<br><br>`int` `fd = open(``"mytest"``, O_RDWR \| O_CREAT, 0666);`<br><br>`if` `(fd == -1)`<br><br>`perror``(``"open file mytest"``);`<br><br>`close(fd);`<br><br>`return` `0;`<br><br>`}` |

|     |     |
| --- | --- |
| 1<br><br>2<br><br>3<br><br>4<br><br>5<br><br>6<br><br>7<br><br>8<br><br>9<br><br>10<br><br>11<br><br>12<br><br>13<br><br>14<br><br>15<br><br>16<br><br>17<br><br>18<br><br>19<br><br>20<br><br>21 | `编译：$cc` `test``.c`<br><br>`执行：$.``/a``.out`<br><br>`查看结果：$``umask`<br><br>`0002`<br><br>`$ll`<br><br>`总计 12`<br><br>`-rwxrwxr-x 1 joseph joseph 5085 12-17 10:35 a.out`<br><br>`-rw-rw-r– 1 joseph joseph    0 12-17 10:33 mytest`<br><br>`-rw-rw-r– 1 joseph joseph  263 12-17 10:35` `test``.c`<br><br>`以上例子以mode 0666创建一个文件mytest,而``umask``是0002，`<br><br>`所以创建的文件权限是0666-0002=0664(-rw-rw-r–)。` |