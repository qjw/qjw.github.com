---
layout: post
title:  获得正在执行的程序路径
category: bash
---
         
##Ansi
若使用main(int argc,const char**argv)的方式，那么argv[0]就是程序自身的名称

##Linux
通过函数readlink("/proc/self/exe", buf, buf_size)或readlink("/proc/pid/exe", buf, buf_size)来获取应用程序的绝对路径。readlink() does not append a null byte to buf。所以在后面追加一个null字符。

##Windows
通过函数GetModuleFileName获得当前程序的路径

##参考
1. <http://linux.die.net/man/1/readlink>
1. <http://msdn.microsoft.com/en-us/library/windows/desktop/ms683197\(v=vs.85\).aspx>