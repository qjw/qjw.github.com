---
layout: post
title:  从socket句柄获得family和type
category: network
---

##Linux

	#include <cstdio>
	#include <sys/socket.h>
	#include <sys/types.h> 

	unsigned short get_socket_family(int fd)
	{
	   struct sockaddr sa;
	   size_t len = sizeof(sa);
	   getsockname(fd, &sa, &len);   
	   return sa.sa_family;
	}

	int main()
	{
	#define FD_CNT 7
		int fd[FD_CNT]; 
		int fd_family[FD_CNT][2] = 
		{
			{ AF_UNIX,SOCK_STREAM},
			{ AF_UNIX,SOCK_DGRAM},
			{ AF_UNIX,SOCK_RAW},
			{ AF_INET,SOCK_STREAM},
			{ AF_INET,SOCK_DGRAM},
			/*{ AF_INET,SOCK_RAW},*/
			{ AF_INET6,SOCK_STREAM},
			{ AF_INET6,SOCK_DGRAM},
			/*{ AF_INET6,SOCK_RAW},*/
		};

		int fd_family2[FD_CNT][2];
		for(int i = 0;i < FD_CNT;i++)
		{
			fd[i] = socket(fd_family[i][0],fd_family[i][1],0);

			fd_family2[i][0] = get_socket_family(fd[i]);

			int type;
			socklen_t length = sizeof( int );
			getsockopt(fd[i], SOL_SOCKET, SO_TYPE, &type, &length );
			fd_family2[i][1] = type;

		}
		for(int i = 0;i < FD_CNT;i++)
		{
			printf("%d,[%d,%d],[%d,%d]\n",
					fd[i],
					fd_family[i][0],fd_family[i][1],
					fd_family2[i][0],fd_family2[i][1]);
		}
		return 0;
	}

##Windows

	#include <winsock2.h>
	#include <ws2tcpip.h>
	#include <stdio.h>
	#include <string.h>
	#include <stdlib.h>   // Needed for _wtoi

	// link with Ws2_32.lib
	#pragma comment(lib,"Ws2_32.lib")

	unsigned short get_socket_family(SOCKET fd)
	{
		WSAPROTOCOL_INFO proto;
		WSADuplicateSocket(fd, GetCurrentProcessId(), &proto);
		return proto.iAddressFamily;
	}

	int main()
	{
			// Initialize Winsock
		WSADATA wsaData = {0};
		WSAStartup(MAKEWORD(2, 2), &wsaData);


	#define FD_CNT 4
		SOCKET fd[FD_CNT]; 
		int fd_family[FD_CNT][2] = 
		{
			{ AF_INET,SOCK_STREAM},
			{ AF_INET,SOCK_DGRAM},
			/*{ AF_INET,SOCK_RAW},*/
			{ AF_INET6,SOCK_STREAM},
			{ AF_INET6,SOCK_DGRAM},
			/*{ AF_INET6,SOCK_RAW},*/
		};

		int fd_family2[FD_CNT][2];
		for(int i = 0;i < FD_CNT;i++)
		{
			fd[i] = socket(fd_family[i][0],fd_family[i][1],0);

			fd_family2[i][0] = get_socket_family(fd[i]);

			int type;
			socklen_t length = sizeof( type );
			getsockopt(fd[i], SOL_SOCKET, SO_TYPE, (char*)&type, &length );
			fd_family2[i][1] = type;

		}
		for(int i = 0;i < FD_CNT;i++)
		{
			printf("%d,[%d,%d],[%d,%d]\n",
					fd[i],
					fd_family[i][0],fd_family[i][1],
					fd_family2[i][0],fd_family2[i][1]);
		}
		WSACleanup();
		return 0;
	}
	
##参考
1. <http://stackoverflow.com/questions/3217650/how-can-i-find-the-socket-type-from-the-socket-descriptor>
1. <http://stackoverflow.com/questions/6646861/determine-address-family-of-an-unbound-socket>
1. <http://bbs.csdn.net/topics/320001187>