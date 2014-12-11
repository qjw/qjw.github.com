---
layout: post
title: Linux gettext国际化
category: linux
---

##Sample

	#include <cstdio>
	#include <cstdlib>
	#include <unistd.h>

	#include <libintl.h>
	#include <locale.h>

	#define _(txt) gettext(txt)

	int main()
	{
		setlocale(LC_ALL,"");
		// 设置名称 以及查找目录
		bindtextdomain("myapp","./po");
		// 设置名称
		textdomain("myapp");
		printf("hello world %s\n",_("fuck"));
		system("pause");
		return 0;
	}
	
编译，使用mingw

	#!/bin/bash
	i686-w64-mingw32-g++ main.cpp \
	/home/king/vlc/contrib/i686-w64-mingw32/lib/libintl.a \
	/home/king/vlc/contrib/i686-w64-mingw32/lib/libiconv.a \
	-I/home/king/vlc-2.1.5/contrib/i686-w64-mingw32/include/

在程序中，笔者将gettext用宏"_"重定向。代码中使用_("fuck")来表示字符串。若是英文，或者找不到匹配的翻译，就直接使用这个名称，否则使用翻译。
	
##解释

