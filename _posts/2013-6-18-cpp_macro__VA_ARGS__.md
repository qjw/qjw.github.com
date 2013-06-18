---
layout: post
title:  使用可变参数宏提高代码整洁性
category: cpp
---

写一个宏来判断条件是否符合，若不符合则中断函数，同时兼容**返回void**的函数。在VS2012和Linux运行正常。

	#include <cstdio>

	#define CHECK(expr,...) \
		do{ \
			if(!(expr)){ \
				printf("check [%s] fail\n",#expr); \
				return __VA_ARGS__; \
			}\
		}while(0)

	void func1()
	{
		CHECK(1 - 1);
	}

	int func2()
	{
		func1();
		CHECK(2 - 2,1);
	}


	int main()
	{
		func2();
		return 0;
	}		