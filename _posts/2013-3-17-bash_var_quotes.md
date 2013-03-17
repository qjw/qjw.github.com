---
layout: post
title:  Bash使用变量时，缺少双引号导致的bug
category: bash
---

当我们使用一个变量时，一般可以$var,${var},"${var}","$var",在正常情况下，加不加大括号区别不大，除非变量和后面的内容会混淆，见下面的例子

		echo "$var1" # 输出变量var1
		echo "${var}1" # 输出变量var
		echo "${11}" # 输出变量11
		echo "$11" # 输出变量1

不过，加上大括号总是个好习惯。

当变量用双引号括起来时，里面的内容总是作为一个总体对待，无论里面是不是换行或者其他，见下面的例子。

		qiuji_000@king-pc /tmp
		$ var="`(echo a;echo b)`";echo $var
		a b

		qiuji_000@king-pc /tmp
		$ var="`(echo a;echo b)`";echo "$var"
		a
		b

可以看到，当不用双引号时，换行符被无视了，这就导致类似以下的代码逻辑出现严重的错误

		echo $var | while read line;do echo line;done

##参考
1. <http://www.igigo.net/archives/128>
