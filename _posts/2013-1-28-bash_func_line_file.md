---
layout: post
title:  bash中打印行号，函数和文件名
category: bash
---

1. 在任意函数中**${0}**总是打印当前的脚本路径,*可能是相对路径*
1. 可以使用**${LINENO}**打印当前的行号
1. 可以使用**${FUNCNAME[0]}**打印当前的函数名称
1. 遍历数组变量**${FUNCNAME}**可以打印整个调用栈,*${FUNCNAME[1]}就是调用当前函数的函数，以此类推*
1. 在Bash里面,**${BASH_LINENO[0]}**可以获得调用它的函数所在地行，即匹配**${FUNCNAME[1]}**调用当前函数时的行号，此变量同样是个数组，配合**${FUNCNAME}**可打印整个调用栈和行号。
1. 在Bash里面,**${BASH_SOURCE}**和**${FUNCNAME}**是一致的。


##参考
1. <http://stackoverflow.com/questions/3055755/equivalent-of-file-line-in-bash>
1. <http://www.ibm.com/developerworks/cn/linux/l-cn-shell-debug/>
2. <http://www.gnu.org/software/bash/manual/bashref.html>