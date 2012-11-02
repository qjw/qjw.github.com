---
layout: post
title: 封装Ipv4/6/Unix Socket的接口
category: network
---


        #ifndef SOCKET_INTERFACE__H_
        #define SOCKET_INTERFACE__H_

        #include <cassert>
        #include <cstring>
        #include <errno.h>

        #include "build_config.h"

        #if defined(OS_WIN)
        #pragma comment(lib,"Ws2_32.lib")
        #include <winsock2.h>
        #include <Ws2tcpip.h>
        #else
        #include <sys/socket.h>
        #include <fcntl.h>
        #include <unistd.h>
        #include <sys/ioctl.h>
        #include <net/if.h>
        #endif

        #if defined(OS_WIN)
        bool			NetworkInit(){
            WORD wVersionRequested;
            WSADATA wsaData;
            int err;

            /* Use the MAKEWORD(lowbyte, highbyte) macro declared in Windef.h */
            wVersionRequested = MAKEWORD(2, 2);

            err = WSAStartup(wVersionRequested, &wsaData);
            return err == 0;
        }

        void			NetworkFini(){
             WSACleanup();
        }
        #else
        inline bool		NetworkInit(){
            return true;
        }

        inline void		NetworkFini(){
        }
        #endif

        #if defined(OS_LINUX)
        //获得某个接口的ip地址
        bool			GetIfAddress(const char* ifname,bool ipv6,void* addr);

        inline bool	GetIfv4Address(const char* ifname,struct in_addr* addr){
                return GetIfAddress(ifname,false,(void* )addr);
        }
        inline bool	GetIfv6Address(const char* ifname,struct sin6_addr* addr){
                return GetIfAddress(ifname,true,(void* )addr);
        }
        #endif

        #if defined(OS_WIN)
        typedef SOCKET			SockFd;
        typedef int				socklen_t;
        //#define SOCKET_ERROR
        //#define INVALID_SOCKET
        #else
        typedef int				SockFd;
        //typedef socklen_t;
        #define SOCKET_ERROR (-1)
        #define INVALID_SOCKET  (SockFd)(~0)
        #endif

        class SocketBase{
        public:
            SocketBase():m_sock_fd_(INVALID_SOCKET){}

            virtual ~SocketBase(){}

            //Get/Set Fd
            inline SockFd			Fd() const{
                return this->m_sock_fd_;
            }
            inline void				SetFd(SockFd fd){
                this->m_sock_fd_ = fd;
            }

            //helper
            bool					Listen(int backlog){
                assert(INVALID_SOCKET != m_sock_fd_);
                return SOCKET_ERROR != ::listen(m_sock_fd_,backlog);
            }
            bool					GetSockopt(int level, int optname, char *optval, socklen_t *optlen){
                assert(INVALID_SOCKET != m_sock_fd_);
                assert(optval && optlen);
                return SOCKET_ERROR != ::getsockopt(m_sock_fd_,level,optname,optval,optlen);
            }
            bool					SetSockopt(int level, int optname, const char *optval, socklen_t optlen){
                assert(INVALID_SOCKET != m_sock_fd_);
                assert(optval);
                return SOCKET_ERROR != ::setsockopt(m_sock_fd_,level,optname,optval,optlen);
            }
            int						Recvfrom(char *buf,size_t len,int flags,struct sockaddr *from,socklen_t *fromlen){
                assert(INVALID_SOCKET != m_sock_fd_);
                assert(buf && len && from && fromlen);
                do{
                    int ret = ::recvfrom(m_sock_fd_,buf,len,flags,from,fromlen);

                    if(ret != SOCKET_ERROR)
                        return ret;

                    if(ret == SOCKET_ERROR && EINTR == errno)
                        continue;
                }while(0);
                return SOCKET_ERROR;
            }
            int						Sendto(const char *buf, size_t len, int flags, const struct sockaddr *to, socklen_t tolen){
                assert(INVALID_SOCKET != m_sock_fd_);
                assert(buf && len && to && tolen);
                do{
                    int ret = ::sendto(m_sock_fd_,buf,len,flags,to,tolen);

                    if(ret != SOCKET_ERROR)
                        return ret;

                    if(ret == SOCKET_ERROR && EINTR == errno)
                        continue;
                }while(0);
                return SOCKET_ERROR;
            }

            bool					Recv(char* buf,size_t len,size_t* readed){
                assert(INVALID_SOCKET != m_sock_fd_);
                assert(buf && len && readed);
                do{
                    int ret = ::recv(m_sock_fd_,buf,len,0);
                    if(ret >= 0){
                        *readed = (size_t)ret;
                        return true;
                    }
                    //被信号中断
                    if(ret < 0 && errno == EINTR)
                        continue;
                    return false;
                }while(true);
                return false;
            }

            bool					Send(char* buf,size_t len){
                assert(INVALID_SOCKET != m_sock_fd_);
                assert(buf && len);
                size_t total_ = 0;
                do{
                    int ret = ::send(m_sock_fd_,buf + total_,len - total_,0);
                    if(ret > 0){
                        total_ += (size_t)ret;
                        if(total_ >= len)
                            break;
                    }
                    //被信号中断
                    if(ret < 0 ){
                        if(errno == EINTR)
                            continue;
                        else
                            return false;
                    }
                }while(true);
                return true;
            }

            bool					SetReuseFlag(bool reuse){
                int flag = reuse?1:0;
                return this->SetSockopt(SOL_SOCKET,SO_REUSEADDR,(const char*)&flag,sizeof(flag));
            }

            bool					Shutdown(int how){
                if(INVALID_SOCKET != m_sock_fd_)
                    return 0 == ::shutdown(m_sock_fd_,how);
                else
                    return true;
            }

            void					Close(){

                if(INVALID_SOCKET != m_sock_fd_){
        #if defined(OS_WIN)
                    ::closesocket(m_sock_fd_);
        #else
                    ::close(m_sock_fd_);
        #endif
                    m_sock_fd_ = INVALID_SOCKET;
                }
            }

            bool					SetBlock(bool flag){
                assert(INVALID_SOCKET != m_sock_fd_);
        #if defined(OS_WIN)
                u_long ul = flag?0:1;
                return SOCKET_ERROR != ioctlsocket(m_sock_fd_, FIONBIO, &ul);
        #else
                int flags = fcntl(m_sock_fd_, F_GETFL, 0);
                if(flags >= 0){
                    if(!flag && fcntl(m_sock_fd_, F_SETFL, flags | O_NONBLOCK) >= 0)
                        return true;
                    if(flag && fcntl(m_sock_fd_, F_SETFL, flags & ~O_NONBLOCK) >= 0)
                        return true;
                }
                return false;
        #endif
            }

        #if defined(OS_LINUX)
            bool 					GetIfIpaddr(const char* ifname,char* buf,size_t bufsize);

            bool					GetIfreq(struct ifreq* ifr,const char* ifname,int request);
        #endif

            bool					SetSendTimeout(int ms){
                assert(INVALID_SOCKET != m_sock_fd_);
                return this->SetSockopt(SOL_SOCKET,SO_SNDTIMEO,(const char*)&ms,sizeof(ms));
            }

            bool					SetRecvTimeout(int ms){
                assert(INVALID_SOCKET != m_sock_fd_);
                return this->SetSockopt(SOL_SOCKET,SO_RCVTIMEO,(const char*)&ms,sizeof(ms));
            }
        private:
            //socket
            SockFd					m_sock_fd_;
        };


        template<typename ADDRT,bool TCP,int ID>
        class SocketBaseEx : public SocketBase{
        public:
            //地址类型
            typedef ADDRT			AddrType;

            //协议类型
            enum{
                //协议ID
                SinFamily = ID,
                //TCP
                TcpFlag = TCP,
            };

            bool					Socket(){
                assert(this->Fd() == INVALID_SOCKET);
                if(this->Fd() != INVALID_SOCKET)
                    this->Close();
                int type = TCP?SOCK_STREAM:SOCK_DGRAM;
                SockFd fd = ::socket(ID,type,0);
                if(INVALID_SOCKET == fd){
                    return false;
                }else{
                    this->SetFd(fd);
                    return true;
                }
            }

            SocketBaseEx(){
                memset(&m_my_addr_,0,sizeof(ADDRT));
                memset(&m_peer_addr_,0,sizeof(ADDRT));
            }

            const ADDRT*			MyAddr() const{
                return &(this->m_my_addr_);
            }
            const ADDRT*			PeerAddr() const{
                return &(this->m_peer_addr_);
            }

            socklen_t				AddrLen() const{
                return this->m_addr_len_;
            }

            bool					Bind(){
                assert(INVALID_SOCKET != Fd());
                return SOCKET_ERROR != ::bind(
                    Fd(),
                    reinterpret_cast<const struct sockaddr*>(&m_my_addr_),
                    sizeof(ADDRT));
            }

            SockFd					Accept(){
                assert(INVALID_SOCKET != Fd());
                this->m_addr_len_ = sizeof(ADDRT);
                return ::accept(
                    Fd(),
                    reinterpret_cast<struct sockaddr*>(&m_peer_addr_),
                    &m_addr_len_);
            }


            bool					Connect(int ms){
                assert(INVALID_SOCKET != Fd());

                if(ms > 0)
                {
                    if(!this->SetBlock(false))
                        return false;

                    bool ret = ConnectImp(ms);

                    if(!this->SetBlock(true))
                        return false;
                    return ret;
                }else{
                    return 0 == ::connect(Fd(),
                        reinterpret_cast<struct sockaddr*>(&m_peer_addr_),
                        sizeof(ADDRT));
                }

            }

            bool					Recvfrom(char *buf,size_t len,size_t* size){
                assert(size);
                this->m_addr_len_ = sizeof(ADDRT);
                int ret = SocketBase::Recvfrom(buf,len,0,
                    reinterpret_cast<struct sockaddr*>(&m_peer_addr_),
                    &m_addr_len_);
                if(ret == SOCKET_ERROR)
                    return false;
                else
                    *size = static_cast<size_t>(ret);
                return true;
            }


            bool					Sendto(const char *buf, size_t len){
                return this->Send2addr(buf,len,&m_peer_addr_);
            }

            bool					Send2addr(const char *buf, size_t len,const ADDRT* addr){
                assert(buf && len && addr);
        #if 1
                int ret = SocketBase::Sendto(buf,len,0,
                    reinterpret_cast<const struct sockaddr*>(addr),
                    sizeof(ADDRT));

                if(ret == SOCKET_ERROR)
                    return false;

                return (int)len <= ret;
        #else
                size_t total_ = 0;
                while(len){
                    int ret = SocketBase::Sendto(buf + total_,len,0,
                        reinterpret_cast<const struct sockaddr*>(addr),
                        sizeof(ADDRT));

                    if(ret == SOCKET_ERROR)
                        return false;

                    len -= static_cast<size_t>(ret);
                    total_ -= static_cast<size_t>(ret);
                }
                return true;
        #endif
            }

        private:
            bool					ConnectImp(int ms){

                if (::connect(Fd(),
                    reinterpret_cast<struct sockaddr*>(&m_peer_addr_),
                    sizeof(ADDRT)) == 0)
                    return true;
        #if defined(OS_WIN)
                if(WSAEWOULDBLOCK !=WSAGetLastError())
                    return false;
        #else
                if (errno != EINPROGRESS)
                    return false;
        #endif

                fd_set set;
                FD_ZERO(&set);
                FD_SET(Fd(), &set);

                struct timeval timeout;
                timeout.tv_sec = ms/1000;
                timeout.tv_usec = (ms%1000)*1000;
                int retval = select(Fd() + 1, NULL, &set, NULL, &timeout);
                if(retval > 0){
                    int error;
                    socklen_t size = sizeof(error);
                    if(GetSockopt(SOL_SOCKET, SO_ERROR, (char*)&error, &size) &&
                        error == 0)
                        return true;
                }
                return false;
            }

        protected:
            ADDRT*					MyAddr(){
                return &(this->m_my_addr_);
            }
            ADDRT*					PeerAddr(){
                return &(this->m_peer_addr_);
            }

        private:

            ADDRT					m_my_addr_;
            ADDRT					m_peer_addr_;
            socklen_t				m_addr_len_;
        };

        #endif


---

        /*
         * socket_interface.cpp
         *
         *  Created on: 2011-7-29
         *      Author: king
         */

        #include "socket_interface.h"

        #if defined(OS_LINUX)
        #include <ifaddrs.h>
        #include <arpa/inet.h>
        #include <cstring>
        #endif

        #if defined(OS_LINUX)

        bool		GetIfAddress(const char* ifname,bool ipv6,void* addr)
        {
            struct ifaddrs *ifaddr;
            if (getifaddrs(&ifaddr) == -1)
                return false;

            void* addPtr;
            bool ret = false;
            for (struct ifaddrs*ifa = ifaddr; ifa != NULL; ifa = ifa->ifa_next) {
                //是否合法
                if (ifa->ifa_addr == NULL ||
                        ifa->ifa_addr->sa_data == NULL ||
                        ifa->ifa_name == NULL)
                    continue;

                //判断类型
                addPtr = NULL;
                if (ifa->ifa_addr->sa_family == (ipv6?AF_INET6:AF_INET)) {
                    if(!ipv6)
                        addPtr = &((struct sockaddr_in *) ifa->ifa_addr)->sin_addr;
                    else
                        addPtr = &((struct sockaddr_in6 *) ifa->ifa_addr)->sin6_addr;
                } else {
                    continue;
                }

                //是否名称相等
                if(strcmp(ifname,ifa->ifa_name) != 0)
                    continue;

        #if 1
                if(!ipv6)
                    memcpy(addr,addPtr,sizeof(struct in_addr));
                else
                    memcpy(addr,addPtr,sizeof(struct in6_addr));

                ret = true;
                break;
        #else
                //转换地址
                if(inet_ntop(ifa->ifa_addr->sa_family,addPtr,buf,bufsize) == NULL){
                    continue;
                } else {
                    ret = true;
                    break;
                }
        #endif
            }

            freeifaddrs(ifaddr);
            return ret;
        }

        bool 			SocketBase::GetIfIpaddr(const char* ifname,char* buf,size_t bufsize){
            struct ifreq ifr;
            if(!GetIfreq(&ifr,ifname,SIOCGIFADDR))
                return false;
            struct sockaddr* addr = (struct sockaddr*)(&(ifr.ifr_addr));
            if(addr->sa_family == AF_INET){
                struct sockaddr_in* ptr_ = (struct   sockaddr_in*)(&(ifr.ifr_addr));
                return inet_ntop(ptr_->sin_family,&(ptr_->sin_addr),buf,bufsize);
            }
            return false;
        }

        bool			SocketBase::GetIfreq(struct ifreq* ifr,const char* ifname,int request){
                strncpy(ifr->ifr_name,ifname,IFNAMSIZ);
                return ioctl(m_sock_fd_, request, ifr) != -1;
        }

        #endif

---


        /*
         * socket_unix.h
         *
         *  Created on: 2011-6-29
         *      Author: king
         */

        #ifndef SOCKET_UNIX_H_
        #define SOCKET_UNIX_H_

        #include "socket_interface.h"

        #if defined(OS_LINUX)
        #include <sys/un.h>
        #include <unistd.h>

        template<bool TCP>
        class SocketUnix : public SocketBaseEx<struct sockaddr_un,TCP,AF_UNIX>{
            typedef SocketBaseEx<struct sockaddr_un,TCP,AF_UNIX> BaseClass;
        public:

            inline void				SetMyAddr(const char* path){
                return this->SetAddr(this->MyAddr(),path);
            }
            inline void				SetPeerAddr(const char* path){
                return this->SetAddr(this->PeerAddr(),path);
            }

            //virtual
            bool						Bind(){
                unlink(this->MyAddr()->sun_path);
                return BaseClass::Bind();
            }

        private:
            void					SetAddr(struct sockaddr_un* addr,const char* path){
                assert(path);
                addr->sun_family = AF_UNIX;
                strncpy(addr->sun_path, path,sizeof(addr->sun_path));
            }
        };

        typedef SocketUnix<true>		SocketUnixTcp;
        typedef SocketUnix<false>		SocketUnixUdp;

        #endif


        #endif /* SOCKET_UNIX_H_ */


---


        #ifndef SOCKET_IPV4__H_
        #define SOCKET_IPV4__H_

        #include "socket_interface.h"

        #if defined(OS_LINUX)
        #include <netinet/in.h>
        #include <arpa/inet.h>
        #endif

        template<bool TCP>
        class SocketIpv4 : public SocketBaseEx<struct sockaddr_in,TCP,AF_INET>{
        public:
            bool						InitMultiBroadcast(const char* ifname,bool loop,const char* grpip)
            {
                assert(grpip);

                //是否换回
                unsigned char loop_flag = loop?1:0;
                if(!this->SetSockopt(IPPROTO_IP,IP_MULTICAST_LOOP,
                        (const char *)&loop_flag,sizeof(loop_flag)))
                    return false;

                //设置接口
                struct in_addr addr;
                if(ifname)
                {
                    if(!GetIfv4Address(ifname,&addr))
                        return false;
                }else
                {
                    addr.s_addr = INADDR_ANY;
                }
                if(!this->SetSockopt(IPPROTO_IP,IP_MULTICAST_IF,
                        (const char *)&addr,sizeof(struct in_addr)))
                    return false;

                //设置组播结构
                struct ip_mreq mreq;
                memcpy(&mreq.imr_interface,&addr,sizeof(struct in_addr));
                if(1 != inet_pton(AF_INET, grpip, &(mreq.imr_multiaddr)))
                    return false;

                if(!this->SetSockopt(IPPROTO_IP,IP_ADD_MEMBERSHIP,
                        (const char *)&mreq,sizeof(mreq)))
                    return false;
                return true;
            }


            inline bool				SetMyAddr(unsigned short port,const char* addr){
                return this->SetAddr(this->MyAddr(),port,addr);
            }
            inline bool				SetPeerAddr(unsigned short port,const char* addr){
                return this->SetAddr(this->PeerAddr(),port,addr);
            }

        private:
            bool						SetAddr(struct sockaddr_in* addr,unsigned short port,const char* ip){
                addr->sin_family	=   AF_INET;
                addr->sin_port		=   htons(port);
                if(!ip){
                    addr->sin_addr.s_addr   =   INADDR_ANY;
                    return true;
                }else{
                    return 1 == inet_pton(AF_INET, ip, &(addr->sin_addr));
                }
            }
        };

        typedef SocketIpv4<true>		SocketIpv4Tcp;
        typedef SocketIpv4<false>	SocketIpv4Udp;

        #endif

---


        /*
         * socket_ipv6.h
         *
         *  Created on: 2011-7-29
         *      Author: king
         */

        #ifndef SOCKET_IPV6_H_
        #define SOCKET_IPV6_H_

        #include "socket_interface.h"

        #if defined(OS_LINUX)
        #include <netinet/in.h>
        #endif

        template<bool TCP>
        class SocketIpv6 : public SocketBaseEx<struct sockaddr_in6,TCP,AF_INET6>{
        public:
            bool						InitMultiBroadcast(const char* ifname,bool loop,const char* grpip)
            {
                assert(grpip);

                //是否换回,注意这里必须是uint，ipv4是uchar
                unsigned int loop_flag = loop?1:0;
                if(!this->SetSockopt(IPPROTO_IPV6, IPV6_MULTICAST_LOOP,
                        (const char*)&loop_flag,sizeof(loop_flag)))
                    return false;

                //设置接口
                unsigned if_index = 0;
                if(ifname){
                    if_index = if_nametoindex(ifname);
                    if(!if_index)
                        return false;
                }
                if(!this->SetSockopt(IPPROTO_IPV6,IPV6_MULTICAST_IF,
                        (const char*)&if_index,sizeof(if_index)))
                    return false;

                struct ipv6_mreq imr6;
                imr6.ipv6mr_interface = if_index;
                if(1 != inet_pton(AF_INET6, grpip, &(imr6.ipv6mr_multiaddr)))
                    return false;

                if(!this->SetSockopt(IPPROTO_IPV6,IPV6_JOIN_GROUP,
                        (const char*)&imr6,sizeof(imr6)))
                    return false;
                return true;
            }

            inline bool				SetMyAddr(unsigned short port,const char* addr){
                return this->SetAddr(this->MyAddr(),port,addr);
            }
            inline bool				SetPeerAddr(unsigned short port,const char* addr){
                return this->SetAddr(this->PeerAddr(),port,addr);
            }

        private:
            bool						SetAddr(struct sockaddr_in6* addr,unsigned short port,const char* ip){
                addr->sin6_family	=   AF_INET6;
                addr->sin6_port		=   htons(port);
                if(!ip){
                    addr->sin6_addr   =   in6addr_any;
                    return true;
                }else{
                    return 1 == inet_pton(AF_INET6, ip, &(addr->sin6_addr));
                }
            }
        };

        typedef SocketIpv6<true>		SocketIpv6Tcp;
        typedef SocketIpv6<false>	SocketIpv6Udp;

        #endif /* SOCKET_IPV6_H_ */

---

        #ifndef CLIENT_INTERFACE__H_
        #define CLIENT_INTERFACE__H_

        #include <cstdio>

        template<typename T>
        class SockClientS_T : public T{
        public:
            ~SockClientS_T(){
                this->Close();
            }

            bool			InitClient(bool bind){
                //bool ret = this->SetPeerAddr(12345,"127.0.0.1")

                //if bind == true
                //bool ret = this->SetMyAddr(12345,"127.0.0.1")
                if(!this->Socket()){
                    perror("");
                    return false;
                }

                if(bind){
                    if(!this->SetReuseFlag(true) || !this->Bind()){
                        perror("");
                        return false;
                    }
                }

                return true;
            }
        };

        template<typename T>
        class SockClientL_T : public T{
        public:
            ~SockClientL_T(){
                this->Close();
            }

            bool			InitClient(bool bind,int ms){

                if(!this->Socket()){
                    perror("");
                    return false;
                }

                if(bind){
                    //if bind == true
                    //bool ret = this->SetMyAddr(12345,"127.0.0.1")
                    if(!this->SetReuseFlag(true) || !this->Bind()){
                        perror("");
                        return false;
                    }
                }

                //bool ret = this->SetPeerAddr(12345,"127.0.0.1")
                if(!this->Connect(ms)){
                    perror("");
                    return false;
                }

                return true;
            }
        };

        #include "socket_ipv4.h"
        typedef SockClientS_T<SocketIpv4Udp>	SockClientS_Ipv4;
        typedef SockClientL_T<SocketIpv4Tcp>	SockClientL_Ipv4;

        #if defined(OS_LINUX)
        #include "socket_unix.h"
        typedef SockClientS_T<SocketUnixUdp>	SockClientS_Unix;
        typedef SockClientL_T<SocketUnixTcp>	SockClientL_Unix;
        #endif

        #include "socket_ipv6.h"
        typedef SockClientS_T<SocketIpv6Udp>	SockClientS_Ipv6;
        typedef SockClientL_T<SocketIpv6Tcp>	SockClientL_Ipv6;

        #endif

---

        #ifndef SERVER_INTERFACE__H_
        #define SERVER_INTERFACE__H_

        #include <cstdio>

        template <typename T>
        class SvrSListener_T{
        public:
            typedef typename T::AddrType		AddrType;

            virtual void	OnContentRecvd(const char* buf,size_t size,const AddrType& clt_addr) = 0;
            virtual void	OnServerClosed(SockFd fd,const AddrType& svr_addr) = 0;
        };

        template<typename T>
        class SockServerS_T : public T{
            enum{
                DefaultInitBufSize = 4096
            };

        public:
            typedef SvrSListener_T<T>	Listener;

            SockServerS_T(Listener*	listener,size_t bufsize = DefaultInitBufSize):
                m_listener_(listener),
                m_bufsize_(bufsize){
                m_buffer_ = new char[bufsize];
                assert(m_buffer_);
                m_cur_size_ = bufsize;
            }

            ~SockServerS_T(){
                this->Close();
            }

            bool					InitServer(){
                //bool ret = this->SetMyAddr(12345,"127.0.0.1")

                if(!this->Socket()){
                    perror("");
                    return false;
                }

                if(!this->SetReuseFlag(true) || !this->Bind()){
                    perror("");
                    return false;
                }

                return true;
            }

            void					StartServer(){
                size_t readed_;
                while(true){
                    if(this->Recvfrom(m_buffer_,m_cur_size_,&readed_)){
                        if(m_listener_)
                            m_listener_->OnContentRecvd(m_buffer_,readed_,*(this->PeerAddr()));

                        if(readed_ == m_cur_size_)
                            ResizeBuffer();
                    }else{
                        break;
                    }
                }

                if(m_listener_)
                    m_listener_->OnServerClosed(this->Fd(),*(this->MyAddr()));
            }
        private:
            void					ResizeBuffer(){
                if(m_buffer_)
                    delete [] m_buffer_;
                m_cur_size_ += m_bufsize_;
                m_buffer_ = new char[m_cur_size_];
                assert(m_buffer_);
            }

        private:
            Listener*				m_listener_;
            size_t					m_bufsize_;
            char*					m_buffer_;
            size_t					m_cur_size_;
        };


        template <typename T>
        class SvrLListener_T{
        public:
            typedef typename T::AddrType		AddrType;

            virtual void	OnConnAccept(SockFd fd,const AddrType& clt_addr) = 0;
            virtual void	OnServerClosed(SockFd fd,const AddrType& svr_addr) = 0;
        };


        template<typename T>
        class SockServerL_T : public T{
        public:
            typedef SvrLListener_T<T>	Listener;

            SockServerL_T(Listener*	listener):
                m_listener_(listener){}

            ~SockServerL_T(){
                this->Close();
            }

            inline void				ResetListener(Listener*	listener){
                this->m_listener_ = listener;
            }

            bool					InitServer(int backlog){

                if(!this->Socket()){
                    perror("");
                    return false;
                }

                //bool ret = this->SetMyAddr(12345,"127.0.0.1")
                if(!this->SetReuseFlag(true) || !this->Bind()){
                    perror("");
                    return false;
                }

                if(!this->Listen(backlog)){
                    perror("");
                    return false;
                }

                return true;
            }

            void					StartServer(){
                SockFd fd;
                do{
                    fd = this->Accept();
                    if(fd == SOCKET_ERROR){
                        perror("");
                        break;
                    }

                    if(m_listener_)
                        m_listener_->OnConnAccept(fd,*(this->PeerAddr()));
                }while(true);
                if(m_listener_)
                    m_listener_->OnServerClosed(this->Fd(),*(this->MyAddr()));
            }
        private:
            Listener*				m_listener_;
        };

        template <typename T>
        class ConnListener_T : public SvrSListener_T<T>{};

        template<typename T>
        class SockConnL_T : public SocketBase{
        private:
            enum{
                BUFSIZE = 4096
            };
        public:
            typedef typename T::AddrType	AddrType;
            typedef ConnListener_T<T> 			Listener;


            SockConnL_T(Listener* listener,const AddrType& addr,SockFd fd):
                m_listener_(listener){
                this->SetFd(fd);
                memcpy(&m_addr_,&addr,sizeof(AddrType));
            }


            void						StartConn(){
                char buf[BUFSIZE];
                size_t readed_;
                while(true)
                {
                    if(this->Recv(buf,sizeof(buf),&readed_)){
                        if(readed_ && m_listener_)
                            m_listener_->OnContentRecvd(buf,readed_,m_addr_);
                    }else{
                        break;
                    }
                }
                if(m_listener_)
                    m_listener_->OnServerClosed(this->Fd(),m_addr_);
            }
        private:
            AddrType 					m_addr_;
            Listener*					m_listener_;
        };

        #include "socket_ipv4.h"
        typedef SockServerS_T<SocketIpv4Udp>	SockServerS_Ipv4;
        typedef SockServerL_T<SocketIpv4Tcp>	SockServerL_Ipv4;
        typedef SockConnL_T<SocketIpv4Tcp>		SockConnL_Ipv4;

        typedef SvrSListener_T<SocketIpv4Udp>		SvrSListener_Ipv4;
        typedef SvrLListener_T<SocketIpv4Tcp>		SvrLListener_Ipv4;
        typedef ConnListener_T<SocketIpv4Tcp>		ConnListener_Ipv4;

        #if defined(OS_LINUX)

        #include "socket_unix.h"
        typedef SockServerS_T<SocketUnixUdp>	SockServerS_Unix;
        typedef SockServerL_T<SocketUnixTcp>	SockServerL_Unix;
        typedef SockConnL_T<SocketUnixTcp>		SockConnL_Unix;

        typedef SvrSListener_T<SocketUnixUdp>		SvrSListener_Unix;
        typedef SvrLListener_T<SocketUnixTcp>		SvrLListener_Unix;
        typedef ConnListener_T<SocketUnixTcp>		ConnListener_Unix;
        #endif

        #include "socket_ipv6.h"
        typedef SockServerS_T<SocketIpv6Udp>	SockServerS_Ipv6;
        typedef SockServerL_T<SocketIpv6Tcp>	SockServerL_Ipv6;
        typedef SockConnL_T<SocketIpv6Tcp>		SockConnL_Ipv6;

        typedef SvrSListener_T<SocketIpv6Udp>		SvrSListener_Ipv6;
        typedef SvrLListener_T<SocketIpv6Tcp>		SvrLListener_Ipv6;
        typedef ConnListener_T<SocketIpv6Tcp>		ConnListener_Ipv6;

        #endif
