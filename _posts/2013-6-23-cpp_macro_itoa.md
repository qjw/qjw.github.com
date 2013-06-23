---
layout: post
title:  使用宏将数字转换成字符串
category: cpp
---

需求如下

使用define定义一个端口，在另外一个地方需要使用这个端口的字符串


	#define ITOA2(x) #x
	#define ITOA(x) ITOA2(x)  // 这里将数字宏展开为实际的数字

	#define PORT 12345

	#include <iostream>
	int main()
	{
		std::cout << "port is:" ITOA(PORT) << std::endl;
		std::cout << "port is:" ITOA2(PORT) << std::endl;                                                                                      
	}
	
输出

	/tmp <root@debian> 02:42:52 $ ./a.out 
	port is:12345
	port is:PORT
	
###参考
1. <http://bbs.csdn.net/topics/200043144>