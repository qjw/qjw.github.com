---
layout: post
title:  使用UNIX域协议时，处理源地址的陷阱
category: network
---

在socket编程中，客户端无需明确的bind一个地址（当然也可以这样做）。对于IPv4和IPv6，系统会自动分配一个合适的地址和端口。

使用**UNIX**域协议时，具体的行为有所不同，若没有bind，**recvfrom**的addrlen总是返回**0**，此时src_addr指向的内存是**随机**的。

而**accept**的addrlen返回**2**，src_addr的**sa_family**字段设置了正确的**AF_UNIX**，path是空的。（*不排除其他版本有其他的行为*）

不过下面的判断总是安全的

	if(2 >= *addrlen || '\0' == src_addr->sun_path[0])
	{
	    // 空地址