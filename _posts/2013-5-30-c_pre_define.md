---
layout: post
title:  C语言前置声明
category: cpp
---

##C
	typedef struct struct_t struct_t;                                                                                                          
	typedef int (*func)(const struct_t* s); 
	struct struct_t
	{
		int i;
		func f;
	};

	int main()
	{
		struct struct_t ttt_;
		return 0;
	}

##C++

	class struct_t;
	typedef int (*func)(const struct_t* s);
	class struct_t
	{
		int i;
		func f;
	};

	int main()
	{
		struct_t ttt_;
		return 0;
	} 
	
	
gcc 4.4和msvc 2012测试OK
	
##参考
1. <http://www.cppblog.com/codejie/archive/2009/11/19/101382.html>
1. <http://www.embedded.com/electronics-blogs/programming-pointers/4024450/Tag-vs-Type-Names>