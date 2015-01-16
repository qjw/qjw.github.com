---
layout: post
title: Linux编译原生vlc
category: linux
---

##下载源码

	#!/bin/bash
	#安装依赖工具
	apt-get install git libtool build-essential pkg-config autoconf
	apt-get install subversion yasm cvs cmake libpcsclite-dev
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

##live555 bug
在live555的初始化代码中，有一段很挫的获取本机ip的代码，错误提示如下：

	live555 This computer has an invalid IP address: 0.0.0.0
	
修改代码 vlc/contrib/native/live555/groupsock/GroupsockHelper.cpp

	netAddressBits GetSelfIP()
	{
		int newSocket = socket(AF_INET, SOCK_DGRAM, 0);
		if (newSocket < 0)
		{
			printf("unable to create datagram socket22222\n");
			return INADDR_ANY;
		}
		struct sockaddr_in fromAddr;
		fromAddr.sin_family=AF_INET;
		fromAddr.sin_port = htons(80);
		fromAddr.sin_addr.s_addr= inet_addr("8.8.8.8");
		if(connect(newSocket,(struct sockaddr*)&fromAddr,sizeof(fromAddr)) == -1)
		{
				printf("%d %d|%s\n",__LINE__,errno,strerror(errno));
				closeSocket(newSocket);
				return INADDR_ANY;
		}

		struct sockaddr_in myAddr;
		socklen_t myaddr_len_ = sizeof(myAddr);
		if(getsockname(newSocket,(struct sockaddr*)&myAddr,&myaddr_len_) == -1)
		{
			printf("%d %d|%s\n",__LINE__,errno,strerror(errno));
			closeSocket(newSocket);
			return INADDR_ANY;
		}
		closeSocket(newSocket);
		return myAddr.sin_addr.s_addr;
	}

	netAddressBits ourIPAddress(UsageEnvironment& env) {
	#if 1
		netAddressBits res_ = GetSelfIP();
		if(res_ == INADDR_ANY)
			return inet_addr("127.0.0.1");
		else
			return res_;
	#else
		static netAddressBits ourAddress = 0;
		int sock = -1;
		struct in_addr testAddr;

		if (ReceivingInterfaceAddr != INADDR_ANY) {
			// Hack: If we were told to receive on a specific interface address, then
			// define this to be our ip address:
			ourAddress = ReceivingInterfaceAddr;
		}

		if (ourAddress == 0) {
			// We need to find our source address
			struct sockaddr_in fromAddr;
			fromAddr.sin_addr.s_addr = 0;

			// Get our address by sending a (0-TTL) multicast packet,
			// receiving it, and looking at the source address used.
			// (This is kinda bogus, but it provides the best guarantee
			// that other nodes will think our address is the same as we do.)
			do {
				loopbackWorks = 0; // until we learn otherwise

				testAddr.s_addr = our_inet_addr("228.67.43.91"); // arbitrary
				Port testPort(15947); // ditto

				sock = setupDatagramSocket(env, testPort);
				if (sock < 0)
					break;

				if (!socketJoinGroup(env, sock, testAddr.s_addr))
					break;

				unsigned char testString[] = "hostIdTest";
				unsigned testStringLength = sizeof testString;

				if (!writeSocket(env, sock, testAddr, testPort, 0, testString,
						testStringLength))
					break;

				// Block until the socket is readable (with a 5-second timeout):
				fd_set rd_set;
				FD_ZERO(&rd_set);
				FD_SET((unsigned )sock, &rd_set);
				const unsigned numFds = sock + 1;
				struct timeval timeout;
				timeout.tv_sec = 5;
				timeout.tv_usec = 0;
				int result = select(numFds, &rd_set, NULL, NULL, &timeout);
				if (result <= 0)
					break;

				unsigned char readBuffer[20];
				int bytesRead = readSocket(env, sock, readBuffer, sizeof readBuffer,
						fromAddr);
				if (bytesRead != (int) testStringLength
						|| strncmp((char*) readBuffer, (char*) testString,
								testStringLength) != 0) {
					break;
				}

				// We use this packet's source address, if it's good:
				loopbackWorks = !badAddressForUs(fromAddr.sin_addr.s_addr);
			} while (0);

			if (sock >= 0) {
				socketLeaveGroup(env, sock, testAddr.s_addr);
				closeSocket(sock);
			}

			if (!loopbackWorks)
				do {
					// We couldn't find our address using multicast loopback,
					// so try instead to look it up directly - by first getting our host name, and then resolving this host name
					char hostname[100];
					hostname[0] = '\0';
					int result = gethostname(hostname, sizeof hostname);
					if (result != 0 || hostname[0] == '\0') {
						env.setResultErrMsg("initial gethostname() failed");
						break;
					}

					// Try to resolve "hostname" to an IP address:
					NetAddressList addresses(hostname);
					NetAddressList::Iterator iter(addresses);
					NetAddress const* address;

					// Take the first address that's not bad:
					netAddressBits addr = 0;
					while ((address = iter.nextAddress()) != NULL) {
						netAddressBits a = *(netAddressBits*) (address->data());
						if (!badAddressForUs(a)) {
							addr = a;
							break;
						}
					}

					// Assign the address that we found to "fromAddr" (as if the 'loopback' method had worked), to simplify the code below:
					fromAddr.sin_addr.s_addr = addr;
				} while (0);

			// Make sure we have a good address:
			netAddressBits from = fromAddr.sin_addr.s_addr;
			if (badAddressForUs(from)) {
				char tmp[100];
				sprintf(tmp, "This computer has an invalid IP address: %s",
						AddressString(from).val());
				env.setResultMsg(tmp);
				from = 0;
			}

			ourAddress = from;

			// Use our newly-discovered IP address, and the current time,
			// to initialize the random number generator's seed:
			struct timeval timeNow;
			gettimeofday(&timeNow, NULL);
			unsigned seed = ourAddress ^ timeNow.tv_sec ^ timeNow.tv_usec;
			our_srandom(seed);
		}
		return ourAddress;
	#endif
	}

##参考
1. <https://wiki.videolan.org/UnixCompile/>
1. <http://jeremiah.blog.51cto.com/539865/275791>
