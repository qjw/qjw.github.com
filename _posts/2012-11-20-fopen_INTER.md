---
layout: post
title:  fread/fwrite信号中断处理
category: linux
---
 
我们知道在Linux下几乎所有的IO操作都有可能被信号中断，errno=EINTR。这些API如read,write等。

在更上层，我们可以使用ANSI C的标准文件接口fopen/fwrite/fread/fgets等，那么这些函数会不会被信号中断呢？（*不是不中断，而是内部有没有做自动重启*）

在[linux die](http://linux.die.net/man/3/fread)、[cplusplus](http://www.cplusplus.com/reference/cstdio/fwrite/)和[opengroup](http://pubs.opengroup.org/onlinepubs/009695399/functions/fwrite.html)上并没有明确这一点，而且基于跨平台考虑，我们一般会使用[ferror](http://linux.die.net/man/3/ferror)

于是自己写个Sample验证，发现在Ubuntu 12 Server Version**会被中断，所以保险起见，在ANSI C fXXX函数还是处理下信号为妙**。





