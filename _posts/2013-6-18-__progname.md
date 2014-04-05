---
layout: post
title:  使用__progname变量获取当前程序的名称
category: cpp
---

该方法不具移植性，在Linux可使用。它指向**程序文件**，而不是程序文件绝对/相对路径。

	#include <cstdio>
	extern const char* __progname;
	int main()
	{
		printf("%s\n",__progname);
		return 0;
	}

##参考
1. <http://stackoverflow.com/questions/273691/using-progname-instead-of-argv0>