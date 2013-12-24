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
	
---

	<application android:label="@string/app_name" android:icon="@drawable/icon"> 
	<application android:label="@string/app_name" android:icon="@drawable/ic_launcher"> 
	
	$ /usr/bin/find . -name ic_launcher.png
	./proj.android/res/drawable-hdpi/ic_launcher.png
	./proj.android/res/drawable-ldpi/ic_launcher.png
	./proj.android/res/drawable-mdpi/ic_launcher.png
	./proj.android/res/drawable-xhdpi/ic_launcher.png
	
---

	Cannot find module with tag 'CocosDenshion/android' in import path
	
	$(call import-add-path, /cygdrive/f/cocos2d-x-2.1.4) \
	$(call import-add-path, /cygdrive/f/cocos2d-x-2.1.4/cocos2dx/platform/third_party/android/prebuilt) \
	# 这里两点注意，1. 反斜杠，2. cygwin路径


见<http://blog.csdn.net/sgwhp/article/details/9663267>回复，或者<http://cocos2d-x.org/forums/6/topics/33525?r=33579#message-33579>

若无权限，使用文件管理器对错误输出对应的目录增加**完全控制**的权限，亦可在cygwin中**chmod 777 * **

若安装失败，

	ERROR: packaging of 'F:\client\tsDemo\proj.android\bin\resources.ap_' failed
	
	$ cd "F:\client\tsDemo\proj.android"
	$ ls -la assets
	$ chmod -R 755 assets #或者
	$ find . | xargs -i chmod 777 {}
	
参考<http://www.9miao.com/thread-43656-1-1.html>

若无法运行，检查模拟器是否勾选**use host gpu**

或者无法获取图片导致断言或堆栈

	Get data from file(assets/*) failed
	
手动调用**build_native.sh**生成资源

##参考
1. <http://www.cnblogs.com/lhming/archive/2012/06/27/2566460.html>
1. <http://www.cnblogs.com/lhming/archive/2012/06/27/2566467.html>
1. <http://www.cnblogs.com/zhangzhifeng/archive/2011/12/26/2301866.html>
1. <http://blog.csdn.net/sgwhp/article/details/9663267>
1. <http://www.9miao.com/thread-43656-1-1.html>
1. <http://blog.csdn.net/teng_ontheway/article/details/9625649>
1. <http://blog.csdn.net/wangbofei/article/details/7951362>
1. <http://stackoverflow.com/questions/12417748/cocos2dx-android-get-data-from-fileassets-failed>
1. <http://hi.baidu.com/vvlee/item/bedc32e63d5c6e3b86d9dee8>