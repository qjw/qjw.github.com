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
	
##PC文件

一个简单的例子

	prefix=/usr/local
	exec_prefix=${prefix}
	includedir=${prefix}/include
	libdir=${exec_prefix}/lib

	Name: foo
	Description: The foo library
	Version: 1.0.0
	Cflags: -I${includedir}/foo
	Libs: -L${libdir} -lfoo
	
常用的字段，摘录自[这里](http://blog.csdn.net/exbob/article/details/6991037)

1. Name：一个人们可读的链接库或软件包的名称，这不影响pkg-config的使用，它用的是.pc文件的名称。
1. Description：关于软件包的简单描述。
1. URL：一个URL，可以在那里获得更多的信息，并且下载这个软件包。
1. Version：软件包的版本。
1. Requires：这个软件包所需的包的列表。这些包的版本可能用一写运算符来指定：=、>、<、>=、<=。
1. Requires.private：这个软件包所需的私有包的列表，不会暴露给应用。版本的指定规则与Requires相同。
1. Conflicts：可选，描述了会与这个软件包产生冲突的包。版本的指定规则与Requires相同。这个域会提供同一个包的多个实例，例如：Conflicts: bar < 1.2.3, bar >= 1.3.0。
1. Cflags：为这个软件包指定编译器选项，以及pkg-config不支持的必要的库。如果所需的库支持pkg-config，应该将它们添加到Requires和Requires.private。
1. Libs：为这个软件包指定的链接选项，以及pkg-config不支持的必要的库。与Cflags的规则相同。
1. Libs.private：这个软件包所需的私有库的链接选项，不会暴露给应用。规则与Cflags相同。

##Requires.private

假设一种场景，foo依赖bar，分别包含foo.so和bar.so。当上层应用使用foo时，实际只需要连接foo即可，动态库的机制可以确保bar被foo自动依赖。而避免上层应用直接依赖bar的情形。

然后在静态链接时，情况就不一样了。若不连接bar，那么链接foo时将报一大堆找不到符号的错误。

这就是Requires.private存在的意义。参考官方sample

foo.pc:

	prefix=/usr
	exec_prefix=${prefix}
	includedir=${prefix}/include
	libdir=${exec_prefix}/lib
	Name: foo
	Description: The foo library
	Version: 1.0.0
	Cflags: -I${includedir}/foo
	Libs: -L${libdir} -lfoo
	
bar.pc:
	
	prefix=/usr
	exec_prefix=${prefix}
	includedir=${prefix}/include
	libdir=${exec_prefix}/lib
	Name: bar
	Description: The bar library
	Version: 2.1.2
	Requires.private: foo >= 0.7
	Cflags: -I${includedir}
	Libs: -L${libdir} -lbar
	
下面没有输出-L指明库的路径，是因为pkg-config会读取连接器的默认链接路径，若在默认路径下，就智能地不输出-L选项。

可以发现，若动态链接，链接选项不会依赖bar，而静态链接则会包含所有（直接、间接）的依赖。
	
	$ pkg-config --libs foo
	-lfoo
	$ pkg-config --libs bar
	-lbar
	$ pkg-config --libs --static bar
	-lbar -lfoo


##参考
1. <http://www.mike.org.cn/articles/description-configure-pkg-config-pkg_config_path-of-the-relations-between/>
1. <http://blog.csdn.net/exbob/article/details/6991037>
1. <http://people.freedesktop.org/~dbn/pkg-config-guide.html>
1. <http://www.chenjunlu.com/2011/03/understanding-pkg-config-tool/>
