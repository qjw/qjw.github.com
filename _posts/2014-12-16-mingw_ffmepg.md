---
layout: post
title: mingw交叉编译ffmepg
category: linux
---

## 安装依赖库和工具

	apt-get install gcc-mingw-w64-i686 g++-mingw-w64-i686 mingw-w64-tools
	# for x64
	#apt-get install gcc-mingw-w64-x86-64 g++-mingw-w64-x86-64 mingw-w64-tools
	apt-get install lua5.1 libtool automake autoconf autopoint make 
	apt-get install gettext pkg-config git subversion cmake cvs zip p7zip-full nsis bzip2

从[这里](https://packages.debian.org/experimental/mingw-w64-i686-dev)下载mingw-w64-i686-dev.deb。以及可能的依赖项，见[这里](https://packages.debian.org/experimental/mingw-w64-common)mingw-w64-common.deb。dpkg -i *.deb 安装之。

其他依赖，参见<http://www.linuxfromscratch.org/blfs/view/svn/multimedia/ffmpeg.html>

	apt-get install yasm

##编译

下载源码<http://ffmpeg.org/releases/ffmpeg-2.5.tar.bz2>，解压（tar jvfx ffmpeg-2.5.tar.bz2）。

	./configure \
	--enable-cross-compile \
	--arch=x86 \
	--target-os=mingw32 \
	--cross-prefix=i686-w64-mingw32-
	make -j8
	
####精简

	./configure \
	--enable-cross-compile \
	--arch=x86 \
	--target-os=mingw32 \
	--cross-prefix=i686-w64-mingw32- \
	--enable-static \
	--disable-shared \
	--enable-small \
	--disable-debug \
	--disable-symver \
	--disable-runtime-cpudetect \
	--disable-ffserver \
	--disable-ffprobe \
	--disable-ffplay \
	--disable-programs \
	--disable-doc \
	--disable-htmlpages \
	--disable-manpages  \
	--disable-podpages  \
	--disable-txtpages \
	--enable-memalign-hack \
	--disable-encoders \
	--disable-decoders \
	--enable-decoder=h264
	make -j8
	
##替换vlc libavcodec

参考<http://www.linuxfromscratch.org/blfs/view/svn/multimedia/vlc.html>找到vlc依赖的ffmepg版本，下载源码，用上述的方法编译ffmepg。

拷贝到contrib目录，重新编译vlc。

	cd /home/king/ffmpeg-2.5
	find . -name "*.a" -type f | xargs  -i cp {} /home/king/vlc/contrib/i686-w64-mingw32/lib/
	cd /home/king/vlc/win32
	make clean;make -j8

	
##参考

1. <http://www.chenshengjian.com/archives/64>
1. <http://www.linuxfromscratch.org/blfs/view/svn/multimedia/vlc.html>
1. <http://blog.csdn.net/jdsjlzx/article/details/7333360>
