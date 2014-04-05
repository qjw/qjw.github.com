---
layout: post
title:  使用socket发送多块不连续的内存
category: network
---

我们常使用send/write/sendto发送socket数据，而使用recv/read/recvfrom接受数据。这些函数都要求传入的buf是连续的。但经常有些需求，例如在客户发送的报文前填写一些内容，但是又必须对客户透明（*要客户预留XX空间显然很撮，况且预留的大小还可能变化*）

一种很撮的办法就是在里面新建一个更大的buf，填充我们的数据之后，再将客户传入的数据拷贝到后面，最后发送这段数据。

对于send/write/recv/read等可以自动推导出发送/接收方的函数，我们可以使用[readv/writev](http://linux.die.net/man/2/writev)来处理两段不连续的数据。

	ssize_t writev(int fd, const struct iovec *iov, int iovcnt);
	struct iovec {
		void  *iov_base;    /* Starting address */
		size_t iov_len;     /* Number of bytes to transfer */
	};

如上，我们可以建一个iovec数组，最后通过writev一并发送出去。

对于recvfrom/sendto等每次发送/接收方都可能变化的函数，我们可以使用[sendmsg](http://linux.die.net/man/2/sendmsg)/[recvmsg](http://linux.die.net/man/2/recvmsg)。

	ssize_t sendmsg(int sockfd, const struct msghdr *msg, int flags);
	struct msghdr {
		void         *msg_name;       /* optional address */
		socklen_t     msg_namelen;    /* size of address */
		struct iovec *msg_iov;        /* scatter/gather array */
		size_t        msg_iovlen;     /* # elements in msg_iov */
		void         *msg_control;    /* ancillary data, see below */
		size_t        msg_controllen; /* ancillary data buffer len */
		int           msg_flags;      /* flags on received message */
	};
	
如上，我们同样可以构建一个iovec数组，并附送到msghdr.msg_iov(msg_iovlen指定数量)。并且将地址附送到msghdr.msg_name(msg_namelen指定地址长度)，然后一并发送出去
	
##参考
1. <http://www.cnblogs.com/lucasfeng/archive/2012/10/03/2710871.html>