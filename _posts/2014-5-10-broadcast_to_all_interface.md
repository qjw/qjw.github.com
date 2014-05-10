---
layout: post
title: 广播数据到所有网口
category: network
---

	#ifdef WIN32
	#define _CRT_SECURE_NO_WARNINGS
	#include <winsock2.h>
	#include <ws2tcpip.h>
	#include <Iphlpapi.h>
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
	#pragma comment(lib,"Iphlpapi.lib")
	typedef SOCKET socket_t;
	typedef u_long IP_t;
	#else
	typedef int socket_t;
	typedef in_addr_t IP_t;
	#define  INVALID_SOCKET  -1
	#define closesocket close
	#endif

	#include <iostream>
	#include <vector>
	typedef std::vector<IP_t> IPVector;
	typedef IPVector::const_iterator IPIter;


	#ifdef WIN32
	bool GetIP(IPVector& ips)
	{
	    char* buf_ = new char[sizeof(IP_ADAPTER_INFO)];
	    PIP_ADAPTER_INFO pIpAdapterInfo = reinterpret_cast<IP_ADAPTER_INFO*>(buf_);
	    unsigned long stSize = sizeof(IP_ADAPTER_INFO);
	    int nRel = GetAdaptersInfo(pIpAdapterInfo,&stSize);
	    // 处理Buf内存不足的情况
	    if (ERROR_BUFFER_OVERFLOW == nRel)
	    {
		delete [] pIpAdapterInfo;
		buf_ = new char[stSize];
		pIpAdapterInfo = reinterpret_cast<IP_ADAPTER_INFO*>(buf_);
		nRel=GetAdaptersInfo(pIpAdapterInfo,&stSize);    
	    }
	    if (ERROR_SUCCESS != nRel)
	    {
		delete [] buf_;
		return false;
	    }
	    while (pIpAdapterInfo)
	    {
		// 过滤以太网
		if(pIpAdapterInfo->Type == MIB_IF_TYPE_ETHERNET)
		{
		    pIpAdapterInfo = pIpAdapterInfo->Next;
		    continue;
		}
		std::cout << "name:" << pIpAdapterInfo->Description << std::endl;
		std::cout << "description:" << pIpAdapterInfo->AdapterName << std::endl;

		// 一个网卡可能有多个IP地址
		IP_ADDR_STRING *pIpAddrString =&(pIpAdapterInfo->IpAddressList);
		do
		{
		    IP_t ip_,mask_,broadcastip_;
		    if(inet_pton(AF_INET,pIpAddrString->IpAddress.String,&ip_) != 1 ||
		            inet_pton(AF_INET,pIpAddrString->IpMask.String,&mask_) != 1 ||
		            ip_ == 0 ||
		            mask_ == 0)
		    {
		        pIpAddrString=pIpAddrString->Next;
		        continue;
		    }
		    broadcastip_ = ip_ | (~mask_);

		    char ipbuf_[64];
		    inet_ntop(AF_INET,&broadcastip_,ipbuf_,sizeof(ipbuf_));

		    std::cout << "ip:" << pIpAddrString->IpAddress.String << std::endl;
		    std::cout << "mask:" << pIpAddrString->IpMask.String << std::endl;
		    std::cout << "gateway:" << pIpAdapterInfo->GatewayList.IpAddress.String << std::endl;
		    std::cout << "broadcast:" << ipbuf_ << std::endl;

		    ips.push_back(broadcastip_);

		    pIpAddrString=pIpAddrString->Next;
		} while (pIpAddrString);

		pIpAdapterInfo = pIpAdapterInfo->Next;
	    }
	    delete [] buf_;
	    return true;
	}
	#else
	bool GetIP(IPVector& ips)
	{
	    struct ifaddrs * ifaddr_ = NULL;
	    if(!!getifaddrs(&ifaddr_))
	    {
		printf("getifaddrs error '%d|%s'\n",errno,strerror(errno));
		return -1;
	    }

	    while (ifaddr_ != NULL) 
	    {
		if (ifaddr_->ifa_addr &&
		        ifaddr_->ifa_name &&
		        ifaddr_->ifa_addr->sa_family == AF_INET) 
		{
		    struct sockaddr_in* addrptr_ =
		        (struct sockaddr_in*)(ifaddr_->ifa_addr);
		    struct sockaddr_in* maskptr_ =
		        (struct sockaddr_in*)(ifaddr_->ifa_netmask);
		    printf("eth '%s' with ip '%s' \n",
		            ifaddr_->ifa_name,
		            inet_ntoa(addrptr_->sin_addr));
		    printf("eth '%s' with mask '%s' \n",
		            ifaddr_->ifa_name,
		            inet_ntoa(maskptr_->sin_addr));

		    IP_t broadcastip_ = addrptr_->sin_addr.s_addr  | (~(maskptr_->sin_addr.s_addr));

		    char ipbuf_[64];
		    inet_ntop(AF_INET,&broadcastip_,ipbuf_,sizeof(ipbuf_));
		    printf("eth '%s' with broadcast ip '%s' \n",
		            ifaddr_->ifa_name,
		            ipbuf_);
		    ips.push_back(broadcastip_);
		}
		ifaddr_ = ifaddr_->ifa_next;
	    }
	    freeifaddrs(ifaddr_);
	}
	#endif

	int main(int argc,const char** argv)
	{

	#ifdef WIN32
	    WSADATA wsaData = {0};
	    WSAStartup(MAKEWORD(2, 2), &wsaData);
	#endif

	    IPVector ips_;
	    if(!GetIP(ips_))
	    {
		return 1;
	    }

	    return 0;
	    socket_t fd_ = socket(AF_INET,SOCK_DGRAM,0);
	    if(fd_ == INVALID_SOCKET)
	    {
		printf("socket error '%d|%s'\n",errno,strerror(errno));
		return -1;
	    }

	    int flag_ = 1;
	    if(setsockopt(fd_, SOL_SOCKET,
		        SO_BROADCAST, (char*)&flag_, sizeof(flag_)) < 0) {
		closesocket(fd_);
		return -1;
	    }

	    const char* msg_ = "test";
	    struct sockaddr_in addr_;
	    addr_.sin_family = AF_INET;
	    addr_.sin_port = htons(12345);
	    for(IPIter iter_ = ips_.begin();
		    iter_ != ips_.end();
		    ++ iter_)
	    {
		addr_.sin_addr.s_addr = *iter_;
		if(sendto(fd_,msg_,strlen(msg_)+1,
		            0,
		            (struct sockaddr*)&addr_,
		            sizeof(addr_)) < 0)
		{
	#ifdef WIN32
		    printf("socket error '%d'\n",WSAGetLastError(errno));
	#else
		    printf("socket error '%d|%s'\n",errno,strerror(errno));
	#endif
		}
	    }
	    return 0;
	}

##参考
1. <http://www.linuxsir.org/bbs/thread16872.html>
1. <http://qjw.qiujinwu.com/blog/2013/07/17/socket_sample/>
