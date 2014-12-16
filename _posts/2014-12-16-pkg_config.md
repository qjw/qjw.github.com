---
layout: post
title: PkgConfig工具
category: linux
---

相信用过qt、wxwidget或者其他一些比较复杂的工具库时，都用过pkg-config.

那么pkg-config都做了些什么呢？摘录[这里](http://www.mike.org.cn/articles/description-configure-pkg-config-pkg_config_path-of-the-relations-between/)一段。

1. 检查库的版本号。如果所需库的版本不满足要求，打印出错误信息，避免连接错误版本的库文件。
1. 获得编译预处理参数，如宏定义，头文件的路径。
1. 获得编译参数，如库及其依赖的其他库的位置，文件名及其他一些连接参数。
1. 自动加入所依赖的其他库的设置。

##简单实用

####安装
	apt-get install pkg-config

最简单的，我们用它来动态获取库的编译、链接参数。例如如下是QT core的编译、链接参数。
	
	king@debian:~$ pkg-config --list-all | grep -i qt
	Qt5Widgets           Qt5 Widgets - Qt Widgets module
	QtCore               Qtcore - Qtcore Librarydule
	Qt5Network           Qt5 Network - Qt Network module
	Qt5Gui               Qt5 Gui - Qt Gui module
	Qt5Core              Qt5 Core - Qt Core module
	QtGui                Qtgui - Qtgui Library
	
	king@debian:~$ pkg-config --libs --cflags QtCore
	-DQT_SHARED -I/usr/include/qt4 -I/usr/include/qt4/QtCore -lQtCore 
	
	king@debian:~$ pkg-config --libs QtCore
	-lQtCore 
	
	king@debian:~$ pkg-config --cflags QtCore
	-DQT_SHARED -I/usr/include/qt4 -I/usr/include/qt4/QtCore 
	
不过pkg-config并不能凭空生成这些，它会在某些目录查找**[模块名].pc**的文件。默认路径一般是/usr/lib/pkg-config

	king@debian:~$ find / -name pkgconfig -type d 2>/dev/null
	/usr/share/pkgconfig
	/usr/lib/i386-linux-gnu/pkgconfig
	/usr/lib/pkgconfig
	
	king@debian:~$ find /usr/lib/i386-linux-gnu/pkgconfig/ -name "*.pc" | grep -i qt
	/usr/lib/i386-linux-gnu/pkgconfig/QtScriptTools.pc
	/usr/lib/i386-linux-gnu/pkgconfig/QtWebKit.pc
	/usr/lib/i386-linux-gnu/pkgconfig/QtTest.pc
	/usr/lib/i386-linux-gnu/pkgconfig/QtCore.pc
	
若你新增的\*.pc不在上述目录，可以修改环境变量**PKG_CONFIG_PATH**。例如编译vlc就要求

	export PKG_CONFIG_LIBDIR=$HOME/vlc/contrib/i686-w64-mingw32/lib/pkgconfig

##参考
1. <http://www.mike.org.cn/articles/description-configure-pkg-config-pkg_config_path-of-the-relations-between/>
1. <http://blog.csdn.net/exbob/article/details/6991037>
1. <http://people.freedesktop.org/~dbn/pkg-config-guide.html>
