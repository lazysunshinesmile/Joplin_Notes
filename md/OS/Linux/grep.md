grep

grep

1、指定文件内搜索
grep "string" filename

2、指定多个文件内搜索
grep "string" filename*

3、不区分大小写 -i
grep -i "string" filename

4、使用正则表达式
grep "string.*" filename

其中“.”不能少
正则表达式遵循的几个重复的操作：
.?  最多匹配一次
.* 匹配零次或者任意多次
.+ 匹配一次以上
.{n} 匹配n次
.{n,} 最少匹配n次
.{,m} 最多匹配m次
.{n,m} 匹配n到m次

5、以整个单词为一个单位查询，-w
grep -w "is" filename

其中his，this等单词不会被搜索出来

6、输出查询的单词行，及前后的行。-A（after）,-B(before),-C(after and before)
grep -A 3 "is" filename
输出本行及后三行。

7、递归查找 -r
grep -r "string" directory

8、不包含某个字符 -v
grep  -v "string" filename

9、不包含多个字符  -v -e
grep -v -e "string1" -e "string2" filename

10、计算(不)包含字符的行数
grep -c "string" filename
grep -v -c "string" filename

11、只显示符合要求的字符串 -o
grep -o "str*ng" filename
string
strong

12、显示该字符的具体位置 -b
grep -o -b "str*ng" filename
2:12

13、显示行号 -n
grep -n "string" filename

14、ubuntu下多标签终端使用
（1）打开一个terminal
     ctrl+alt+t

（2）在terminal中打开多个标签
     ctrl+shift+t

（3）在多个标签中切换的方法

          方法1（直达）

               alt+1 alt+2 alt+3.......

          方法二（上下切换）

               ctrl + pageUp
               ctrl + pageDown.

（4）关闭一个terminal快捷键（标签或者窗口）
     ctrl+ d
     
15、正则表达式
grep -E "kkk|ccc"

16、打印上下文
grep -C 5 foo file 显示file文件里匹配foo字串那行以及上下5行
grep -B 5 foo file 显示foo及前5行
grep -A 5 foo file 显示foo及后5行

查看grep版本号的方法是
grep -V

16. 除去某个文件夹 
`grep -rn "nvramExchangeProperty" --exclude-dir out .`

17. 仅查找指定文件
`grep -r "report.partition.stats" --include \*.java --exclude-dir tez-ui`
