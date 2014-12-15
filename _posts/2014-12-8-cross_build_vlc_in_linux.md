---
layout: post
title: Linux交叉编译vlc
category: linux
---

## 安装依赖库和工具

	apt-get install gcc-mingw-w64-i686 g++-mingw-w64-i686 mingw-w64-tools
	# for x64
	#apt-get install gcc-mingw-w64-x86-64 g++-mingw-w64-x86-64 mingw-w64-tools
	apt-get install lua5.1 libtool automake autoconf autopoint make 
	apt-get install gettext pkg-config git subversion cmake cvs zip p7zip-full nsis bzip2

从[这里](https://packages.debian.org/experimental/mingw-w64-i686-dev)下载mingw-w64-i686-dev.deb。以及可能的依赖项，见[这里](https://packages.debian.org/experimental/mingw-w64-common)mingw-w64-common.deb。dpkg -i *.deb 安装之。

##下载源码

	git clone git://git.videolan.org/vlc.git
	cd vlc
	mkdir -p contrib/win32
	cd contrib/win32
	../bootstrap --host=i686-w64-mingw32
	make prebuilt
	
源码包在<http://download.videolan.org/pub/videolan/vlc/2.1.5/>下载，版本号可变。
	
在make prebuilt会下载一个很大的压缩包，可以实现下载好，放入**contrib/win32**目录。名称根据cpu体系名称分别为**[vlc-contrib-i586-mingw32msvc-latest.tar.bz2](https://get.videolan.org/contrib/i586-mingw32msvc/)**和**[vlc-contrib-i686-w64-mingw32-latest.tar.bz2](https://get.videolan.org/contrib/i686-w64-mingw32/)**

x64需要删除文件 rm -f ../i686-w64-mingw32/bin/moc ../i686-w64-mingw32/bin/uic ../i686-w64-mingw32/bin/rcc

注意：**若使用prebuilt库，合适的版本很重要，若编译各种奇奇怪怪的问题，可以考虑换一个prebuilt压缩包**

##编译
跳到根目录

	./bootstrap
	mkdir win32 && cd win32
	export PKG_CONFIG_LIBDIR=$HOME/vlc/contrib/i686-w64-mingw32/lib/pkgconfig
	../configure --help  #确认选项
	../configure --host=i686-w64-mingw32 \
		--disable-sout \
		--enable-static=false \
		--disable-nls \
		--disable-debug \
		--disable-gprof \
		--disable-cprof \
		--disable-coverage \
		--disable-lua \
		--disable-httpd \
		--disable-dc1394 \
		--disable-dv1394 \
		--disable-dvdread \
		--disable-dvdnav \
		--disable-bluray \
		--disable-smbclient \
		--disable-sftp \
		--disable-vcd \
		--disable-vcdx \
		--disable-screen \
		--disable-vnc \
		--disable-freerdp \
		--disable-chromaprint \
		--disable-chromecast \
		--disable-taglib \
		--disable-upnp \
		--disable-qt
	make
	
vlc的界面是qt写的，那货用linux交叉编译，死活编译不过，也不知道问题在哪里

##精简dll尺寸

为了减少尺寸，首先应该裁剪一些不需要的功能，这样就可以砍掉大部分plugin的dll。

其次，在configure时，默认的编译选项是"**-g -O2**"。我们可以在configure之前设置环境变量CFLAGS、CXXFLAGS，取消-g选项。

	export CFLAGS="-O2"
	export CXXFLAGS="-O2"
	
不过笔者亲测，发现dll还是很多，若使用官方prebuilt的库。比如libavcodec就有40多M，若-g甚至到了50M。尝试很多办法，都没有讲大小缩下去。最后发现官方prebuilt的库都包含完整调试信息。所以直接干掉就行了。

	find vlc/win32 -name "*.dll" -type f | xargs -i strip --strip-all {}


##参考
1. <https://wiki.videolan.org/Win32Compile/>	
1. <https://wiki.videolan.org/Win32Compile_Under_Fedora>
1. <https://forum.videolan.org/viewtopic.php?f=14&t=106846>
1. <http://blog.chinaunix.net/uid-24774106-id-3526766.html>
