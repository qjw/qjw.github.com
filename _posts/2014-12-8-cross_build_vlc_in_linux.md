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

##编译ActiveX插件

最新的vlc（2.\*.\*)已经将浏览器插件作为一个独立的工程实现，所以为了编译，先单独下载源码。安装一些依赖的库

	git clone git://git.videolan.org/npapi-vlc.git
	apt-get install libwine-dev
	
	cd npapi-vlc
	./autogen.sh
	
	# 在vlc编译环境目录下查找libvlc.pc所在目录
	export PKG_CONFIG_LIBDIR=/home/king/vlc/win32/_win32/lib/pkgconfig/
	# 写死，我不需要编译firefox和chrome的插件
	export FETCH_NPAPI_FALSE="#"
	# 根据vlc编译环境来设置
	export LIBVLC_CFLAGS="-I/home/king/vlc/include/ -O2"
	# 同样查找libvlc.dll来设置
	export LIBVLC_LIBS="-L/home/king/vlc/win32/lib/.libs/ -lvlc"
	# 只编译activex。npapi需要去google code下载一些东西，那玩意被强了。
	./configure --host=i686-w64-mingw32 --disable-npapi
	make -j8
	
#### 常见问题

	checking for LIBVLC... yes
	Package libvlc was not found in the pkg-config search path.
	Perhaps you should add the directory containing `libvlc.pc'
	to the PKG_CONFIG_PATH environment variable
	No package 'libvlc' found

	king@debian:~/vlc/lib$ find /home/king/vlc -name libvlc.pc
	/home/king/vlc/win32/lib/libvlc.pc
	/home/king/vlc/win32/_win32/lib/pkgconfig/libvlc.pc
	
---

	checking that generated files are newer than configure... done
	configure: error: conditional "FETCH_NPAPI" was never defined.
	Usually this means the macro was only invoked conditionally.

	export FETCH_NPAPI_FALSE="#"
	
---

	king@debian:~/npapi-vlc$ find . -name "*.dll"
	./activex/.libs/axvlc.dll


##本朝特色自主研发

#### 替换插件图标
	king@debian:~/npapi-vlc$ find . -name "*.bmp" -o -name "*.ico"
	./share/pixmaps/win32/fullscreen.bmp
	./share/pixmaps/win32/defullscreen.bmp
	./share/pixmaps/win32/volume-muted.bmp
	./share/pixmaps/win32/vlc.ico
	./share/pixmaps/win32/volume.bmp
	./share/pixmaps/win32/pause.bmp
	./share/pixmaps/win32/play.bmp
	
经查，这些默认的bmp是灰度的，若改成彩色，可以将其存储为24色RGB，并且不要包含色彩空间信息

然后修改代码（*有多处类似的修改*）

	--- a/activex/plugin.cpp
	+++ b/activex/plugin.cpp
	@@ -228,23 +228,23 @@ VLCPlugin::VLCPlugin(VLCPluginClass *p_class, LPUNKNOWN pUnkOuter) :
	 {
		 _ViewRC.hDeFullscreenBitmap =
			 LoadImage(DllGetModule(), MAKEINTRESOURCE(3),
	-                  IMAGE_BITMAP, 0, 0, LR_LOADMAP3DCOLORS);
	+                  IMAGE_BITMAP, 0, 0, LR_CREATEDIBSECTION);
	
		 _ViewRC.hBackgroundIcon =
			 (HICON) LoadImage(DllGetModule(), MAKEINTRESOURCE(8),
	-                          IMAGE_ICON, 0, 0, LR_DEFAULTSIZE);
	+                          IMAGE_ICON, 0, 0, LR_DEFAULTCOLOR);

将中央的图标可以支持任意尺寸，（*注意：若ico有多张图片，那么GetIconInfo获取的大小不准确*）

	--- a/common/win32_fullscreen.cpp
	+++ b/common/win32_fullscreen.cpp
	@@ -655,9 +655,13 @@ LRESULT VLCHolderWnd::WindowProc(UINT uMsg, WPARAM wParam, LPARAM lParam)
				 HDC hDC = BeginPaint(hWnd(), &PaintStruct);
				 RECT rect;
				 GetClientRect(hWnd(), &rect);
	-            int IconX = ((rect.right - rect.left) - GetSystemMetrics(SM_CXICON))/2;
	-            int IconY = ((rect.bottom - rect.top) - GetSystemMetrics(SM_CYICON))/2;
	-            DrawIcon(hDC, IconX, IconY, RC().hBackgroundIcon);
	+
	+            ICONINFO icon_info_;
	+            ::GetIconInfo(RC().hBackgroundIcon,&icon_info_);
	+
	+            int IconX = ((rect.right - rect.left) - icon_info_.xHotspot)/2;
	+            int IconY = ((rect.bottom - rect.top) - icon_info_.yHotspot)/2;
	+            DrawIconEx(hDC, IconX, IconY, RC().hBackgroundIcon,icon_info_.xHotspot,icon_info_.yHotspot,0,NULL,DI_NORMAL);
				 EndPaint(hWnd(), &PaintStruct);
				 break;	

####版本号

	--- a/configure.ac
	+++ b/configure.ac
	@@ -2,19 +2,19 @@ dnl Autoconf settings for vlc
	 
	 AC_COPYRIGHT([Copyright 2002-2014 VLC authors and VideoLAN])
	 
	-AC_INIT(vlc, 3.0.0-git)
	+AC_INIT(King, 1.0.0)
	 VERSION_MAJOR=3
	 VERSION_MINOR=0
	 VERSION_REVISION=0
	 VERSION_EXTRA=0
	-VERSION_DEV=git
	+VERSION_DEV=0
	 
	-PKGDIR="vlc"
	+PKGDIR="King"
	 AC_SUBST(PKGDIR)
	 
	 CONFIGURE_LINE="`echo "$0 $ac_configure_args" | sed -e 's/\\\/\\\\\\\/g'`"
	-CODENAME="Vetinari"
	-COPYRIGHT_YEARS="1996-2014"
	+CODENAME="Test"
	+COPYRIGHT_YEARS="2013-2014"
	 
	 AC_CONFIG_SRCDIR(src/libvlc.c)
	 AC_CONFIG_AUX_DIR(autotools)
	
##参考
1. <https://wiki.videolan.org/Win32Compile/>	
1. <https://wiki.videolan.org/Win32Compile_Under_Fedora>
1. <https://forum.videolan.org/viewtopic.php?f=14&t=106846>
1. <http://blog.chinaunix.net/uid-24774106-id-3526766.html>
1. <https://forum.videolan.org/viewtopic.php?f=16&t=112839>
1. <http://askubuntu.com/questions/114216/cannot-find-vlc-web-plugin-while-compiling-vlc-2-0-from-source>
