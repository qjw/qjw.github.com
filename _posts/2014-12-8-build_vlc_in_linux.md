---
layout: post
title: Linux编译原生vlc
category: linux
---

##下载源码

	#!/bin/bash
	#安装依赖工具
	apt-get install git libtool build-essential pkg-config autoconf
	apt-get install subversion yasm cvs cmake
	#下载源码
	git clone git://git.videolan.org/vlc.git
	cd vlc
	./bootstrap

也可以直接下载源码包，经测试，会快很多。

##安装依赖库

	# 注意不是apt-get install build-dep,另外这个命令会去墙外的地址下东西，代理是必须的
	apt-get build-dep vlc
	cd contrib
	mkdir native
	cd native
	../bootstrap
	make

##编译

运行命令**./configure --help**，根据你的功能需要，配置选项。例如

	./configure --disable-qt \
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
		--disable-chromecast
	make
	
若希望放入正式的环境中，需要添加--libdir=/usr/lib/  以便vlc从/usr/lib/vlc/plugins中去查找插件，具体可以configure之后，查看src/Makefile中的定义

	/**
	 * Enumerates all dynamic plug-ins that can be found.
	 *
	 * This function will recursively browse the default plug-ins directory and any
	 * directory listed in the VLC_PLUGIN_PATH environment variable.
	 * For performance reasons, a cache is normally used so that plug-in shared
	 * objects do not need to loaded and linked into the process.
	 */
	static void AllocateAllPlugins (vlc_object_t *p_this)
	{
		/* Contruct the special search path for system that have a relocatable
		 * executable. Set it to <vlc path>/plugins. */
		char *vlcpath = config_GetLibDir ();
		if (likely(vlcpath != NULL)
		 && likely(asprintf (&paths, "%s" DIR_SEP "plugins", vlcpath) != -1))
		{
			AllocatePluginPath (p_this, paths, mode);
			free( paths );
		}
		
---

	/**
	 * Determines the architecture-dependent data directory
	 *
	 * @return a string (always succeeds).
	 */
	char *config_GetLibDir (void)
	{
		return strdup (PKGLIBDIR);
	}
	
---

	libdir = /usr/lib
	vlclibdir = ${libdir}/${PKGDIR}
	AM_CPPFLAGS = $(INCICONV) $(IDN_CFLAGS) -DMODULE_STRING=\"core\" \
		-DLOCALEDIR=\"$(localedir)\" -DPKGDATADIR=\"$(vlcdatadir)\" \
		-DPKGLIBDIR=\"$(vlclibdir)\" $(am__append_1) $(am__append_2)
	PKGDIR = vlc

编译结束，会生成vlc、vlcstatic等可执行文件

##参考
1. <https://wiki.videolan.org/UnixCompile/>
