---
layout: post
title:  find -exec a shell function
category: bash
---


find命令有一个exec选项，他可以在找到一个匹配的文件之后自动执行某条命令，例如find ... -exec rm {} \;

但是你若传一个shell函数，却被提示找不到符号，怎么办呢？Bash有一个内建命令export，见<http://www.gnu.org/software/bash/manual/bashref.html>

	export
		export [-fn] [-p] [name[=value]]
		Mark each name to be passed to child processes in the environment. 
		If the -f option is supplied, the names refer to shell functions; 
		otherwise the names refer to shell variables. The -n option means 
		to no longer mark each name for export. If no names are supplied, 
		or if the -p option is given, a list of exported names is displayed.
		The -p option displays output in a form that may be reused as input. 
		If a variable name is followed by =value, the value of the variable 
		is set to value.

		The return status is zero unless an invalid option is supplied, 
		one of the names is not a valid shell variable name, 
		or -f is supplied with a name that is not a shell function.

其中**-f选项**可以导出一个shell函数，导出之后，find就能找到了

##参考
1. <http://stackoverflow.com/questions/4321456/find-exec-a-shell-function>
2. <http://www.gnu.org/software/bash/manual/bashref.html>