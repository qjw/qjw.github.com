---
layout: post
title:  防止句柄被子进程继承
category: linux
---
	
在Linux下，句柄默认会被继承，这经常导致一些很难查的问题，可以通过fcntl函数禁止继承

	if (fcntl(fd, F_SETFD, fcntl(fd, F_GETFD) | FD_CLOEXEC) == -1)
	{
		this->Close();
		return false;
	}
	

较新的内核[socket](http://linux.die.net/man/2/socket)函数支持SOCK_CLOEXEC标志位（*以及SOCK_NONBLOCK*）

[open](http://linux.die.net/man/2/open)函数也支持O_CLOEXEC标志位
	

在windows下可以用[SetHandleInformation](http://msdn.microsoft.com/en-us/library/windows/desktop/ms724935\(v=vs.85\).aspx)

	SOCKET sock = socket(AF_INET, SOCK_STREAM, IPPROTO_TCP);
	SetHandleInformation((HANDLE)sock, HANDLE_FLAG_INHERIT, 0);
