---
layout: post
title:  UNIX域Socket抽象命名空间（abstract_namespace）
category: network
---

unix域 socket在*nix下是一种很受欢迎的IPC，不过有个小问题，它bind之后，会在文件系统中留下一个文件，然后再close之后文件却不会自动消失，这就导致下一次bind会失败，所以unix域socket的bind通常有个ulink凑热闹。

另一个问题是，这个文件很容易被其他程序不经意中删除，这导致很奇怪的问题，而很难发现。

下面要说的是如何在unix域socket中不使用文件系统路径，但保持接口的一致性（*这种方法目前不具备特别好的移植性*）。

简单地说，就是Linux在内存中维护了一个**虚拟**的"文件系统"，这个文件在close之后会**自动消失**，并且文件系统中看不到bind的文件，用**netstat -an**却有记录。

和普通的unix域socket大部分地方一致，除了地址的设置。地址结构如下

	struct sockaddr_un
	{
		__SOCKADDR_COMMON (sun_);
		char sun_path[108];     /* Path name.  */
	};
	
正常情况下，sun_path指定了需要bind（或者send/connect等）的路径，若使用**abstract_namespace**，那么sun_path[0]必须为**'\0'**(零字符)。同时在使用这个地址时，必须在addresslen参数中指定正确的长度，这个长度是**sizeof(sun_family) + 1 + strlen(path)**。其中path不包含后面的'\0'，Linux也不会去寻找这个零字符。

当接收（accept，recvfrom等）数据时，Linux也**不会**在sun_path后面附上一个'\0'，所以你必须根据**返回的长度**谨慎地处理字符串。

	#include "event.h"

	#include <stdlib.h>
	#include <string.h>
	#include <stdio.h>
	#include <sys/types.h>
	#include <sys/socket.h>
	#include <sys/un.h>

	#define SVR_PATH "/tmp/svr_path"
	#define CLI_PATH "/tmp/cli_path"

	struct sockaddr_un server_addr;
	int fd;
	int clifd;
	socklen_t addrlen;


	void            udp_cb(int fd, short event, void *argc)
	{
		char buf[65536];
		char addr_buf_[sizeof(struct sockaddr_un)];

		struct sockaddr_un* cli_addr_ = (struct sockaddr_un*)addr_buf_;
		socklen_t socklen = sizeof(addr_buf_);
		int ret_ = recvfrom(
				fd,
				(void*)buf,
				sizeof(buf),
				0,
				(struct sockaddr*)cli_addr_,
				&socklen);
		if(ret_ > 0)
		{
		    // 注意根据长度来设置零字符
			cli_addr_->sun_path[socklen - 2] = '\0';
			printf("recv '%s' from '%s'\n",buf,cli_addr_->sun_path + 1);
		}
	}

	void            udp_timer(int fd, short event, void *argc)
	{
		const char* msg_ = "hello world";
		printf("send msg %s\n",msg_);
		sendto(clifd,
				msg_,
				strlen(msg_) + 1,
				0,
				(const struct sockaddr*)(&server_addr),
				addrlen);
	}


	int main()
	{
		struct event_base *base = event_init();
		struct timeval tv1 = {500,0};
		event_base_loopexit(base,&tv1);

		fd = socket(AF_UNIX,SOCK_DGRAM,0);
		server_addr.sun_family = AF_UNIX;
		server_addr.sun_path[0]=0; 
		strcpy(server_addr.sun_path + 1,SVR_PATH); 
		// 注意计算长度
		addrlen = sizeof(server_addr.sun_family) + sizeof(SVR_PATH);
		bind(fd, (struct sockaddr*)&server_addr, addrlen);

		clifd = socket(AF_UNIX,SOCK_DGRAM,0);
		struct sockaddr_un client_addr;
		client_addr.sun_family = AF_UNIX;
		client_addr.sun_path[0]=0; 
		strcpy(client_addr.sun_path + 1,CLI_PATH); 
		// 注意计算长度
		socklen_t addrlen_ = sizeof(client_addr.sun_family) + sizeof(CLI_PATH);
		bind(clifd, (struct sockaddr*)&client_addr, addrlen_);

		struct event timer_event_;
		struct timeval tv = {1,0};
		event_set(&timer_event_,
				-1,
				EV_PERSIST,
				udp_timer,
				0);
		event_add(&timer_event_, &tv);

		/*libevent*/
		struct event svr_event_;
		event_set(
				&svr_event_,
				fd,
				EV_PERSIST | EV_READ,
				udp_cb,
				0);
		event_add(&svr_event_, NULL);

		event_base_dispatch(base);
		return 0;
	}

---

	~ <root@debian> 05:17:32 $ netstat -an
	unix  2      [ ]         DGRAM                    7260     @/tmp/cli_path
	unix  2      [ ]         DGRAM                    7259     @/tmp/svr_path
	
	
若使用netstat查询端口情况，通过**abstract_namespace**的方式创建的unix域socket bind之后，路径前有一个**@**

##参考
1. <http://blog.eduardofleury.com/archives/2007/09/13>
1. <http://blog.csdn.net/xnwyd/article/details/7359506>