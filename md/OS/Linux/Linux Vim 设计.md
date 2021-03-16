Linux Vim 设计

Linux Vim 设计

参考：https://github.com/yangyangwithgnu/use_vim_as_ide

1、1、使用他的.vimrc出错：Ultisnips requires py >= 2.7 or py3
    sudo apt install python-dev
     参考解决方式：https://github.com/SirVer/ultisnips/issues/707

2、操作：za，打开或关闭当前折叠；zm，关闭所有折叠；zr，打开所有折叠。（据我测试只有za、zr有用）效果如下：

![](https://github.com/yangyangwithgnu/use_vim_as_ide/raw/master/pics/%E4%BB%A3%E7%A0%81%E6%8A%98%E5%8F%A0.gif)

3、设定标签，方便跳转

`1、ctags -R --c++-kinds=+p+l+x+c+d+e+f+g+m+n+s+t+u+v --fields=+liaS --extra=+q --language-force=c++[[NEWLINE]]2、`ctags --list-kinds=c++

跳转
1、crtl + ]
2、下一个 :tnext 上一个 :tpre(修改过参考值)
crtl + t:返回上一个标签
crtl + o:返回光标上一个位置
crtl + i:返回光标下一个位置。（比如A->B->C, crtl+t或者crtl+o 到了B， 再crtl+i 到了C)。

`[[NEWLINE]]`