---
layout: post
title: android cocos2dx编译
category: other
---

打开COCOS2D根目录下create-android-project.bat文件（*最好用高级点的文本编辑器*），修改以下变量到合适的值（*加上一行cd /d %~dp0*）

	@echo off
	cd /d %~dp0
	setlocal
	set _CYGBIN=D:\cygwin\bin
	set _ANDROIDTOOLS=D:\android\adt-bundle-windows-x86_64-20131030\sdk\tools
	set _NDKROOT=D:\android\android-ndk-r9c
	
右键**以管理员运行**

输入包名，如com.android.fuck。然后输入工程名称，建议和fuck**保持一致**。

若成功，会在当前目录生成fuck目录，结构如下：

	$ tree -L 2
	.
	├── Classes
	│   ├── AppDelegate.cpp
	│   ├── AppDelegate.h
	│   ├── HelloWorldScene.cpp
	│   └── HelloWorldScene.h
	├── proj.android
	│   ├── AndroidManifest.xml
	│   ├── ant.properties
	│   ├── assets
	│   ├── bin
	│   ├── build.xml
	│   ├── build_native.sh
	│   ├── gen
	│   ├── jni
	│   ├── libs
	│   ├── local.properties
	│   ├── ndkgdb.sh
	│   ├── obj
	│   ├── proguard-project.txt
	│   ├── project.properties
	│   ├── res
	│   └── src
	└── Resources
		├── CloseNormal.png
		├── CloseSelected.png
		├── HelloWorld.png
		└── Icon.png

	11 directories, 16 files

cd到该目录下，运行$NDK/ndk-build编译。

若编译失败，则修改F:\cocos2d-x-2.1.4\MyGame\proj.android\jni\Application.mk，增加以下内容

	jni/../../../cocos2dx/platform/android/CCCommon.cpp: \
		In function 'void cocos2d::CCLog(char const*, ...)':  
	jni/../../../cocos2dx/platform/android/CCCommon.cpp:44:72: error: \
		format not a string literal and no format arguments [-Werror=format-security]  
	
	APP_CPPFLAGS += -Wno-error=format-security
	
见<http://blog.csdn.net/sgwhp/article/details/9663267>回复，或者<http://cocos2d-x.org/forums/6/topics/33525?r=33579#message-33579>

若无权限，使用文件管理器对错误输出对应的目录增加**完全控制**的权限，亦可在cygwin中**chmod 777 * **

若安装失败，

	ERROR: packaging of 'F:\client\tsDemo\proj.android\bin\resources.ap_' failed
	
	$ cd "F:\client\tsDemo\proj.android"
	$ ls -la assets
	$ chmod -R 755 assets
	
参考<http://www.9miao.com/thread-43656-1-1.html>

若无法运行，检查模拟器是否勾选**use host gpu**

##参考
1. <http://www.cnblogs.com/lhming/archive/2012/06/27/2566460.html>
1. <http://www.cnblogs.com/lhming/archive/2012/06/27/2566467.html>
1. <http://www.cnblogs.com/zhangzhifeng/archive/2011/12/26/2301866.html>
1. <http://blog.csdn.net/sgwhp/article/details/9663267>
1. <http://www.9miao.com/thread-43656-1-1.html>
1. <http://blog.csdn.net/teng_ontheway/article/details/9625649>