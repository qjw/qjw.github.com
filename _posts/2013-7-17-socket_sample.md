---
layout: post
title: Socket 工具程序
category: other
---

经常要测试socket程序，写几个工具来发送/接收报文

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

	#endif
	#include <stdio.h>
	#include <string.h>
	#include <stdlib.h> 
	#include <errno.h>
	#include <assert.h>

	#ifdef WIN32
	#pragma comment(lib,"Ws2_32.lib")
	typedef SOCKET socket_t;
	#else
	typedef int socket_t;
	#define  INVALID_SOCKET  -1
	#define closesocket close
	#endif

	int main(int argc,const char** argv)
	{

	#ifdef WIN32
		WSADATA wsaData = {0};
		WSAStartup(MAKEWORD(2, 2), &wsaData);
	#endif

		if(argc < 4)
		{
			printf("%s des_ip des_port [ src_ip src_port ] content\n",argv[0]);
			return -1;
		}
		socket_t fd_ = socket(AF_INET,SOCK_DGRAM,0);
		if(fd_ == INVALID_SOCKET)
		{
			printf("socket error '%d|%s'\n",errno,strerror(errno));
			return -1;
		}
		struct sockaddr_in addr_;
		addr_.sin_family = AF_INET;
		addr_.sin_addr.s_addr = inet_addr(argv[1]);
		addr_.sin_port = htons(atoi(argv[2]));

		if(!strcmp(argv[2],"255.255.255.255"))
		{
			int flag_ = 1;
			if(setsockopt(fd_, SOL_SOCKET,
						SO_BROADCAST, (char*)&flag_, sizeof(flag_)) < 0) {
				closesocket(fd_);
				return -1;
			}
		}

		const char* msg_ = argv[3];
		if(argc > 5)
		{
			msg_ = argv[5];
			struct sockaddr_in addr2_;
			addr2_.sin_family = AF_INET;
			addr2_.sin_addr.s_addr = inet_addr(argv[3]);
			addr2_.sin_port = htons(atoi(argv[4]));

			if(!!bind(fd_,(const struct sockaddr*)&addr2_,sizeof(addr2_)))
			{
				printf("bind error '%d|%s'\n",errno,strerror(errno));
				return -1;
			}
		}

		if(sendto(fd_,msg_,strlen(msg_)+1,
				0,
				(struct sockaddr*)&addr_,
				sizeof(addr_)) < 0)
		{
			printf("send error '%d|%s'\n",errno,strerror(errno));
		}
		return 0;
	}
	
	
---

	#ifdef WIN32
	#define _CRT_SECURE_NO_WARNINGS
	#include <winsock2.h>
	#include <ws2tcpip.h>
	#include <Mswsock.h>
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

	#endif
	#include <stdio.h>
	#include <string.h>
	#include <stdlib.h> 
	#include <errno.h>
	#include <assert.h>

	#ifdef WIN32
	#pragma comment(lib,"Ws2_32.lib")
	typedef SOCKET socket_t;
	#else
	typedef int socket_t;
	#define  INVALID_SOCKET  -1
	#define closesocket close
	#endif


	int main(int argc,const char** argv)
	{
	#ifdef WIN32
		WSADATA wsaData = {0};
		WSAStartup(MAKEWORD(2, 2), &wsaData);
	#endif

		if(argc < 2)
		{
			printf("%s port\n",argv[0]);
			return -1;
		}
		socket_t fd_ = socket(AF_INET,SOCK_DGRAM,0);
		if(fd_ < 0)
		{
			printf("socket error '%d|%s'\n",errno,strerror(errno));
			return -1;
		}

		int flag_ = 1;
		if(setsockopt(fd_, IPPROTO_IP, IP_PKTINFO, (char*)&flag_,
					sizeof(flag_)) < 0)
		{
			printf("setsockopt error '%d|%s'\n",errno,strerror(errno));
			return -1;
		}

	#ifndef WIN32
		struct ifaddrs * ifaddr_ = NULL;
		if(!!getifaddrs(&ifaddr_))
		{
			printf("getifaddrs error '%d|%s'\n",errno,strerror(errno));
			return -1;
		}

		while (ifaddr_ != NULL) {
			if (ifaddr_->ifa_addr &&
					ifaddr_->ifa_name &&
					ifaddr_->ifa_addr->sa_family == AF_INET) 
			{ 
				struct ifreq ifstruct_;
				memset(&ifstruct_,0,sizeof(ifstruct_));
				int ret_ = snprintf(ifstruct_.ifr_name,
						sizeof(ifstruct_.ifr_name),
						"%s",
						ifaddr_->ifa_name);
				if (ret_ <= 0 ||
						ret_ >= sizeof(ifstruct_.ifr_name) ||
						ioctl(fd_, SIOCGIFINDEX, &ifstruct_) < 0)
				{
					printf("snprintf or ioctl error '%d|%s'\n",errno,strerror(errno));
					return -1;
				}

				struct sockaddr_in* addrptr_ =
					(struct sockaddr_in*)(ifaddr_->ifa_addr);
				printf("eth '%s' with ip '%s' ifindex '%d'\n",
						ifaddr_->ifa_name,
						inet_ntoa(addrptr_->sin_addr),
						ifstruct_.ifr_ifindex);
			}
			ifaddr_ = ifaddr_->ifa_next;
		}
		freeifaddrs(ifaddr_);
	#else
		GUID guid_wsa_recvmsg_ = WSAID_WSARECVMSG;
		LPFN_WSARECVMSG lpfn_wsarecvmsg_ = NULL;
		DWORD outsize_ = 0;
		WSAIoctl(
			fd_,
			SIO_GET_EXTENSION_FUNCTION_POINTER,
			&guid_wsa_recvmsg_,
			sizeof(guid_wsa_recvmsg_),
			&lpfn_wsarecvmsg_,
			sizeof(lpfn_wsarecvmsg_),
			&outsize_,
			NULL,
			NULL
			);
		if (lpfn_wsarecvmsg_ == NULL)
		{
			return -1;
		}
	#endif

		struct sockaddr_in addr_;
		addr_.sin_family = AF_INET;
		addr_.sin_port = htons(atoi(argv[1]));
		addr_.sin_addr.s_addr = inet_addr("0.0.0.0");

		if(!!bind(fd_,(const struct sockaddr*)&addr_,sizeof(addr_)))
		{
			printf("bind error '%d|%s'\n",errno,strerror(errno));
			return -1;
		}

		struct sockaddr_in addr2_;
		char buf_[65536];
		char cmbuf_[0x100];
		do
		{
	#ifndef WIN32
			struct iovec msg_iov_ =
			{
				.iov_base = buf_,
				.iov_len = sizeof(buf_)
			};
			struct msghdr mh_ = {
				.msg_name = &addr2_,
				.msg_namelen = sizeof(addr2_),
				.msg_iov = &msg_iov_,
				.msg_iovlen = 1,
				.msg_control = cmbuf_,
				.msg_controllen = sizeof(cmbuf_),
			};
			int ret_ = recvmsg(fd_,&mh_,0);
	#else		
			WSABUF wsa_bufdata_;
			WSAMSG wsa_msg_;
			wsa_bufdata_.buf = buf_;
			wsa_bufdata_.len = sizeof(buf_);
			wsa_msg_.name = (sockaddr *)&addr2_;
			wsa_msg_.namelen = sizeof(addr2_);
			wsa_msg_.lpBuffers = &wsa_bufdata_;
			wsa_msg_.dwBufferCount = 1;
			wsa_msg_.Control.buf = cmbuf_;
			wsa_msg_.Control.len = sizeof(cmbuf_);
			wsa_msg_.dwFlags = 0;
			
			DWORD recvd_cnt_ = 0;
			if (0 != lpfn_wsarecvmsg_(fd_, &wsa_msg_, &recvd_cnt_, NULL, NULL))
			{
				printf("lpfn_wsarecvmsg_ fail '%d|%s'\n",errno,strerror(errno));
				continue;
			}
			int ret_ = (int)recvd_cnt_;
	#endif
			if(ret_ > 0)
			{
				struct sockaddr_in toaddr_;
				socklen_t toaddr_len_ = sizeof(toaddr_);
				if(!!getsockname(fd_,
							(struct sockaddr*)(&toaddr_),
							&toaddr_len_))
				{
					printf("getsockname error '%d|%s'\n",errno,strerror(errno));
					return -1;
				}
	#ifndef WIN32
				struct cmsghdr *cmsg_ = CMSG_FIRSTHDR(&mh_);
				struct in_pktinfo *pi_ = NULL;
				for (;cmsg_ != NULL;cmsg_ = CMSG_NXTHDR(&mh_, cmsg_))
				{
					if (cmsg_->cmsg_level == IPPROTO_IP &&
							cmsg_->cmsg_type == IP_PKTINFO)
					{
						pi_ = (struct in_pktinfo *)(CMSG_DATA(cmsg_));
						break;;
					}
				}
				if(!pi_)
				{
					printf("invalid in_pktinfo '%d|%s'\n",errno,strerror(errno));
					continue;
				}
	#else			
				WSACMSGHDR *pcmsg_hdr_ = WSA_CMSG_FIRSTHDR(&wsa_msg_);
				if(!pcmsg_hdr_ || 
					IP_PKTINFO != pcmsg_hdr_->cmsg_type
					|| !(WSA_CMSG_DATA(pcmsg_hdr_)))
				{
					printf("WSA_CMSG_FIRSTHDR fail '%d|%s'\n",errno,strerror(errno));
					continue;
				}
				IN_PKTINFO *pi_ = (IN_PKTINFO *)WSA_CMSG_DATA(pcmsg_hdr_);
	#endif
				toaddr_.sin_addr = pi_->ipi_addr; 
				printf("recv from [%d|%s] to ",                
					ntohs(addr2_.sin_port),                                        
					inet_ntoa(addr2_.sin_addr)); 
				printf("[%d|%s|%d] '%d' | '%s'\n",                                      
					ntohs(toaddr_.sin_port),                                       
					inet_ntoa(pi_->ipi_addr),                                   
					pi_->ipi_ifindex,                                              
					ret_,
					buf_); 
			}
			else if(ret_ == 0)
			{
				printf("empty recv error '%d|%s'\n",errno,strerror(errno));
			}
			else
			{
				printf("recv error '%d|%s'\n",errno,strerror(errno));
			}
		}while(1);
		return 0;
	}
	
---


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
	#include <netinet/ip.h>
	#include <netinet/udp.h>


	#endif
	#include <stdio.h>
	#include <string.h>
	#include <stdlib.h> 
	#include <errno.h>
	#include <assert.h>

	#ifdef WIN32
	#pragma comment(lib,"Ws2_32.lib")
	typedef SOCKET socket_t;
	typedef char flag_t;
	#else
	typedef int socket_t;
	#define  INVALID_SOCKET  -1
	#define closesocket close
	typedef int flag_t;
	#endif

	#define PCKT_LEN 8192

	unsigned short csum(unsigned short *addr, int nleft)
	{
		unsigned sum = 0;
		while (nleft > 1) {
			sum += *addr++;
			nleft -= 2;
		}

		if (nleft == 1) {
			if (0)
				sum += *(uint8_t*)addr;
			else
				sum += *(uint8_t*)addr << 8;
		}

		sum = (sum >> 16) + (sum & 0xffff);   
		sum += (sum >> 16);      
		return (uint16_t)~sum;
	}

	int main(int argc,const char** argv)
	{

	#ifdef WIN32
		WSADATA wsaData = {0};
		WSAStartup(MAKEWORD(2, 2), &wsaData);
	#endif

		if(argc < 8)
		{
			printf("%s ethname mac des_ip des_port src_ip src_port content\n",argv[0]);
			return -1;
		}
		socket_t fd_ = socket(AF_PACKET, SOCK_DGRAM, htons(ETH_P_IP));
		if(fd_ == INVALID_SOCKET)
		{
			printf("socket error '%d|%s'\n",errno,strerror(errno));
			return -1;
		}

		struct ifreq ifstruct;
		memset(&ifstruct, 0, sizeof(ifstruct));
		strcpy(ifstruct.ifr_name, argv[1]);
		if (ioctl(fd_, SIOCGIFINDEX, &ifstruct) < 0) 
		{
			printf("ioctl error '%d|%s'\n",errno,strerror(errno));
			return -1;
		}

		struct sockaddr_ll addr_ll_;
		memset(&addr_ll_, 0, sizeof(addr_ll_));
		addr_ll_.sll_family = AF_PACKET;
		addr_ll_.sll_protocol = htons(ETH_P_IP);
		addr_ll_.sll_ifindex = ifstruct.ifr_ifindex;
		unsigned char mac[6];
		sscanf(argv[2],
				"%hhx:%hhx:%hhx:%hhx:%hhx:%hhx",
				&mac[0], &mac[1], &mac[2],
				&mac[3], &mac[4], &mac[5]);
		addr_ll_.sll_halen = 6;
		memcpy(addr_ll_.sll_addr, mac, 6);

		char buffer[PCKT_LEN];
		// Our own headers' structures
		struct iphdr*ip = (struct iphdr*) buffer;
		struct udphdr* udp = (struct udphdr*) (buffer
				+ sizeof(struct iphdr));
		char * payload = (buffer + sizeof(struct iphdr)
				+ sizeof(struct udphdr));
		memset(buffer, 0, PCKT_LEN);
		int contentlen_ = strlen(argv[7]);
		snprintf(payload, contentlen_+1, "%s", argv[7]);

		ip->ihl = 5;
		ip->version = 4;
		ip->tos = 16; 
		ip->id = htons(54321);
		ip->ttl = 255; 
		ip->protocol = IPPROTO_UDP; 
		ip->saddr = inet_addr(argv[5]);
		ip->daddr = inet_addr(argv[3]);
		udp->source = htons(atoi(argv[6]));
		udp->dest = htons(atoi(argv[4]));
		udp->len = htons(sizeof(struct udphdr) + contentlen_ + 1);
		udp->check = 0;
		unsigned short len_ = sizeof(struct iphdr) +
			sizeof(struct udphdr) + contentlen_ + 1;
		ip->tot_len = htons(len_);
		ip->check = csum((unsigned short *) buffer, sizeof(struct iphdr));

		if(sendto(fd_, buffer, len_, 0, 
					(struct sockaddr *)&addr_ll_, sizeof(addr_ll_)) < 0)
		{
			printf("sendto error '%d|%s'\n",errno,strerror(errno));
		}
		return 0;
	}

##参考
1. <http://stackoverflow.com/questions/5281409/get-destination-address-of-a-received-udp-packet>
1. <http://blog.csdn.net/my3439955/article/details/8905893>

