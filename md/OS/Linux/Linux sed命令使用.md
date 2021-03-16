Linux sed命令使用

Linux sed命令使用


[Linux 命令搜索](https://wangchujiang.com/linux-command/c/sed.html)

将文件`${1}`中的 `${line_num}`这一行的值改为`${TO_DELETE_LINE_FLAG}`
`sed -i ${line_num}c${TO_DELETE_LINE_FLAG} ${1}`

