---
layout: post
title:  可变参数及其陷阱
category: cpp
---

说到可变参数，最常见的就是printf系列。在可变参数中一个重要的问题就是如何知道我需要多少个参数

对于printf系列，通过前面一个模式字符串来描述，例如%d表示一个整形数字。

而fcntl则通过cmd来确定，例如当cmd为F_SETFD时，那么后面需要一个int，若为F_GETFD，则无需参数
	
下面这个例子更直接，通过第一个参数n指定参数个数，总之任何方法都行。
	
	/* va_arg example */
	#include <stdio.h>      /* printf */
	#include <stdarg.h>     /* va_list, va_start, va_arg, va_end */

	int FindMax (int n, ...)
	{
		int i,val,largest;
		va_list vl;
		va_start(vl,n);
		largest=va_arg(vl,int);
		for (i=1;i<n;i++)
		{
			val=va_arg(vl,int);
			largest=(largest>val)?largest:val;
		}
		va_end(vl);
		return largest;
	}

	int main ()
	{
		int m;
		m= FindMax (7,702,422,631,834,892,104,772);
		printf ("The largest value is: %d\n",m);
		return 0;
	}

不过需要多少参数只是我们的期望，**调用时实际输入了多少，却不得而知**，例如上面的例子，我调用FindMax时，FindMax(7,1,2)也可以运行，但程序还是会老老实实的寻找7个可变参数，这有可能导致非法内存访问。
	
对于printf类似的函数，可以使用**__attribute__((format**加以检测。
	
#参考
1. <http://www.cplusplus.com/reference/cstdarg/va_arg/>
