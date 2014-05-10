---
layout: post
title: 当socket句柄被继承后close句柄未关闭导致的bug
category: network
---

Linux下socket句柄默认（应该是什么句柄都这样）会被子进程继承，包括accept的tcp句柄。

当accept一个句柄并创建子进程，子进程未结束前调用close关闭句柄，试图让客户端的tcp连接自动断开。而实际并未断开。原因是句柄被子进程继承。

看看close和shutdown的区别。

1. **close 关闭本进程的socket id，但链接还是开着的，用这个socket id的其它进程还能用这个链接，能读或写这个socket id**
2. **shutdown--则破坏了socket 链接，读的时候可能侦探到EOF结束符，写的时候可能会收到一个SIGPIPE信号**

所以这问题可以双管齐下：

1. **使用accept4替换accept，让句柄自动不可继承，或者accept之后调用fcntl手动让其不可继承**
2. **在close之前，现shutdown这条连接**


##sample

	#ifdef WIN32
	#define _CRT_SECURE_NO_WARNINGS
	#include <winsock2.h>
	#include <ws2tcpip.h>
	#else
	#include <sys/socket.h>
	#include <sys/ioctl.h>
	#include <net/if.h>
	#include <netinet/in.h>
	#include <arpa/inet.h>
	#include <netpacket/packet.h>
	#include <netinet/in.h>
	#include <linux/if_ether.h>
	#include <ifaddrs.h>
	#include <unistd.h>
	#endif
	#include <stdio.h>
	#include <string.h>
	#include <stdlib.h>
	#include <errno.h>
	#include <assert.h>

	#ifdef WIN32
	#pragma comment(lib,"Ws2_32.lib")
	typedef SOCKET socket_t;
	typedef int socklen_t;
	#else
	typedef int socket_t;
	#define  INVALID_SOCKET  -1
	#define closesocket close
	#endif

	void svr(socket_t fd)
	{
	    struct sockaddr_in addr_;
	    addr_.sin_family = AF_INET;
	    addr_.sin_addr.s_addr = inet_addr("0.0.0.0");
	    addr_.sin_port = htons(12345);

	    int flag_=1;
	    if (setsockopt(fd, SOL_SOCKET, SO_REUSEADDR,
		        (const void *)&flag_ , sizeof(int)) < 0) 
	    {
		printf("setsockopt error '%d|%s'\n",errno,strerror(errno));
		return;
	    }

	    if(!!bind(fd,(const struct sockaddr*)&addr_,sizeof(addr_)))
	    {
		printf("bind error '%d|%s'\n",errno,strerror(errno));
		return;
	    }
	    listen(fd,SOMAXCONN);

	    struct sockaddr_in newaddr_;
	    socklen_t addrlen_ = sizeof(newaddr_);
	    const char* msg_ = "test";
	    do{
		//int newfd_ = accept(fd,(struct sockaddr*)&newaddr_,&addrlen_);
		// 使用accept4让新fd自动不可继承
		int newfd_ = accept4(fd,(struct sockaddr*)&newaddr_,&addrlen_,SOCK_CLOEXEC);
		if(newfd_ > 0)
		{
		    send(newfd_,msg_,strlen(msg_)+1,0);
		    printf("new tcp fd '%d'\n",newfd_);
	#ifdef WIN32
		    shutdown(newfd_,SD_BOTH);
	#else
		    shutdown(newfd_,SHUT_RDWR);
	#endif
		    closesocket(newfd_);
		}
	    }while(true);
	}

	void cli(socket_t fd,const char* ip)
	{
	    assert(ip);
	    struct sockaddr_in addr_;
	    addr_.sin_family = AF_INET;
	    addr_.sin_addr.s_addr = inet_addr(ip);
	    addr_.sin_port = htons(12345);

	    if(connect(fd,(const struct sockaddr*)&addr_,sizeof(addr_)) != 0)
	    {
	#ifdef WIN32
		printf("connect error '%d'\n",WSAGetLastError());
	#else
		printf("connect error '%d|%s'\n",errno,strerror(errno));
	#endif
		return;
	    }
	    char buf_[1024];
	    do{
		memset(buf_,0,sizeof(buf_));
		int cnt_ = recv(fd,buf_,sizeof(buf_),0);
		if(cnt_ == 0)
		{
		    printf("fd '%d'closed\n",fd);
		    break;
		}
		else if(cnt_ > 0)
		{
		    printf("recv content '%s'\n",buf_);
		}
		else
		{
	#ifdef WIN32
		    printf("recv error '%d'\n",WSAGetLastError());
	#else
		    printf("recv error '%d|%s'\n",errno,strerror(errno));
	#endif
		    break;
		}
	    }while(true);
	}

	int main(int argc,const char** argv)
	{

	#ifdef WIN32
	    WSADATA wsaData = {0};
	    WSAStartup(MAKEWORD(2, 2), &wsaData);
	#endif

	    socket_t fd_ = socket(AF_INET,SOCK_STREAM,0);
	    if(fd_ == INVALID_SOCKET)
	    {
		printf("socket error '%d|%s'\n",errno,strerror(errno));
		return -1;
	    }

	    if(argc > 1)
	    {
		cli(fd_,argv[1]);
	    }
	    else
	    {
		svr(fd_);
	    }
	}

