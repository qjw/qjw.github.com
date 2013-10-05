---
layout: post
title: Cocos2d-x linux下编译
category: bash
---

由于windows 8下,intel那破显卡跟不上,所以用virtualbox装了个xubuntu跑.

	OpenGL 1.5 or higher is required(your version is 1.1.00). Please upgrade the driver of your video card.

直接从<http://cocos2d.cocoachina.com/download>下载源码.解压.

为了编译成功,首先必须安装依赖的库,在根目录下有个文件**install-deps-linux.sh **.在ubuntu直接运行即可.

	apt-get install build-essential 
	apt-get install libgl1-mesa-dev libglu1-mesa-dev 
	apt-get install freeglut3-dev
	apt-get install libx11-dev
	apt-get install libxmu-dev
	apt-get install libgl2ps-dev
	apt-get install libglfw-dev
	apt-get install libzip-dev
	apt-get install libcurl4-gnutls-dev
	apt-get install libfontconfig1-dev
	apt-get install libsqlite3-dev
	apt-get install libglew*-dev
	
若找不到**libcurl.so**,自己建个软连接即可.

	/usr/lib/i386-linux-gnu/libcurl.so -> libcurl-gnutls.so
	
完了编译,直接**make all**就行了.若需要编译debug模式,使用**make all DEBUG=1**

编译后的程序藏得很省，例如HelloCpp如下：

	cocos2d-x-2.1.4/samples/Cpp/HelloCpp/proj.linux/bin/debug/HelloCpp
	
我把HelloCpp的源码，makefile和Resource单独拷贝出来，编译成功，但运行总是找不到资源。找了需求，发现一个坑爹的玩意：

	bool CCFileUtilsLinux::init()
	{
		// get application path
		char fullpath[256] = {0};
		ssize_t length = readlink("/proc/self/exe", fullpath, sizeof(fullpath)-1);

		if (length <= 0) {
			return false;
		}

		fullpath[length] = '\0';
		std::string appPath = fullpath;
		m_strDefaultResRootPath = appPath.substr(0, appPath.find_last_of("/"));
		m_strDefaultResRootPath += "/../../../Resources/";
		
		
尼玛，写个程序，还必须放到下三层的子目录运行（**/../../../Resources/**）。