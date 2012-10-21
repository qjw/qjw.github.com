---
layout: post
title: 获得socket连接使用的IP地址
category: network
---

###见[getsockname/Windows](http://msdn.microsoft.com/en-us/library/windows/desktop/ms738543\(v=vs.85\).aspx)或[getsockname/Linux](http://linux.die.net/man/2/getsockname)

        #include <winsock2.h>

        #pragma comment(lib, "Ws2_32.lib")


        int main(){
            WSADATA wsaData;
            int Ret;
            if ((Ret = WSAStartup(MAKEWORD(2,2), &wsaData)) != 0)
                return 1;

            SOCKET fd_ = socket(AF_INET,SOCK_DGRAM,0);
            if(fd_ == INVALID_SOCKET)
                return 1;

            struct sockaddr_in sa_dest;
            memset(&sa_dest,0,sizeof(sa_dest));
            sa_dest.sin_family=AF_INET;
            sa_dest.sin_port=htons(80);
            sa_dest.sin_addr.s_addr = inet_addr("127.0.0.3");

            if(connect(fd_,(struct sockaddr*)&sa_dest,sizeof(sa_dest)) == SOCKET_ERROR)
                return 1;

            struct sockaddr_in self_addr;
            int len_ = sizeof(self_addr);
            if(getsockname(fd_,(struct sockaddr*)&self_addr,&len_) == SOCKET_ERROR )
                return 1;

            char* xx = inet_ntoa(self_addr.sin_addr);

            closesocket(fd_);
            if (WSACleanup() == SOCKET_ERROR)
                return 1;
            return 0;
        }
