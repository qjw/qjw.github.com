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

在程序中，笔者将gettext用宏"_"重定向。代码中使用_("fuck")来表示字符串。若是英文，或者找不到匹配的翻译，就直接使用这个名称，否则使用翻译。

编译，使用mingw

	#!/bin/bash
	i686-w64-mingw32-g++ main.cpp \
	/home/king/vlc/contrib/i686-w64-mingw32/lib/libintl.a \
	/home/king/vlc/contrib/i686-w64-mingw32/lib/libiconv.a \
	-I/home/king/vlc-2.1.5/contrib/i686-w64-mingw32/include/

##生成翻译

	#!/bin/bash
	mkdir po
	cd po
	# 查找所有的源码，cpp处可修改。
	find .. -name "*.cpp" > POTFILEE.in
	# 查找所有的源码，并找出所有需要翻译的字段
	# 若有多个id，只会显示一个。
	# 若有多个文件，也全部会放入zh_CN.po中
	# ID支持空格
	xgettext -f POTFILEE.in -d zh_CN --keyword=_ --keyword=N_ --from-code=UTF-8

例如zh_CN.po。将最后的msgstr替换成需要的翻译。

	# SOME DESCRIPTIVE TITLE.
	# Copyright (C) YEAR THE PACKAGE'S COPYRIGHT HOLDER
	# This file is distributed under the same license as the PACKAGE package.
	# FIRST AUTHOR <EMAIL@ADDRESS>, YEAR.
	#
	#, fuzzy
	msgid ""
	msgstr ""
	"Project-Id-Version: PACKAGE VERSION\n"
	"Report-Msgid-Bugs-To: \n"
	"POT-Creation-Date: 2014-12-10 20:39-0500\n"
	"PO-Revision-Date: YEAR-MO-DA HO:MI+ZONE\n"
	"Last-Translator: FULL NAME <EMAIL@ADDRESS>\n"
	"Language-Team: LANGUAGE <LL@li.org>\n"
	"Language: \n"
	"MIME-Version: 1.0\n"
	"Content-Type: text/plain; charset=CHARSET\n"
	"Content-Transfer-Encoding: 8bit\n"

	#: ../main.cpp:10
	msgid "fuck"
	msgstr ""

完了将文本描述的翻译编译成对程序友好的”半二进制格式“

	mkdir -p zh_CN/LC_MESSAGES/
	msgfmt -o zh_CN/LC_MESSAGES/myapp.mo zh_CN.po

注意**zh_CN/LC_MESSAGES/**这个路径是必须的。zh_CN表示locale，而myapp表示模块名称，见c代码中**textdomain**函数。这样可以让多个程序公用一个目录下的翻译。而互不冲突干扰。

##gettext常用组件

安装 apt-get install gettext

1. gettext : 进行translate，根据LC_MESSAGES环境变量的值和TEXTDOMAIN查找字串对应的本地表示。
1. xgettext : 从程序中抽取调用gettext进行本地化的字符串，生成一份.po结尾的配置文件。
1. msgfmt : 将配置好的本地化配置文件进行转换成gettext使用的格式。
1. msgmerge:合并已有的两份.po文件
1. msgunfmt:从msgfmt生成的.mo文件生成.po配置文件

##Windows
zh_CN.po使用utf8编码。正常情况，若程序全部用mingw来编写，包括界面（用gtk/qt/wxwdiget)。那也无视。但是Windows下总是有些地方要用gb编码，

笔者使用gb2312编码来写zh_CN.po发现msgfmt转换也没有错误，在Windows cmd也正常，中文正常显示。

##Merge
假如又添加了一些新的待翻译词汇，那么久需要更新po，并添加对应的翻译。最后再编译成mo

首先用xgettext重新生成完整的po文件，这个文件是没有任何翻译的（空白）。

然后 msgmerge -o zh_CN.po zh_CN.po zh_CN_new.po进行合并。注意老的必须是目标文件，反了不行。

生成新的po文件里面，msgmerge会尝试查找类似的id来匹配。并且标注**fuzzy**。翻译人员可以查找该字段进行翻译，并删除该字段。

	#: ../main.cpp:15
	msgid "fuck you"
	msgstr "ri ni"

	#: ../main.cpp:16
	#, fuzzy
	msgid "fuck you1"
	msgstr "ri ni"

	#: ../fuck.cpp:15
	msgid "fuck"
	msgstr "ri"


##参考
1. <http://blog.csdn.net/cnlamo/article/details/39085123>
1. <http://socol.iteye.com/blog/871899>

