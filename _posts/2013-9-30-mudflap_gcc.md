---
layout: post
title: 使用mudflap检测内存错误
category: cpp
---

##在debian上安装

	aptitude install libmudflap0 libmudflap0-4.4-dev
	
##编译

	gcc x.c -fmudflap -lmudflap
	
##参考
1. <http://hi.baidu.com/mgqw864/item/e082042126bbed3094f62b8f>
1. <http://gcc.gnu.org/wiki/Mudflap_Pointer_Debugging>