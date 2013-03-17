---
layout: post
title:  使用bash求交集并集
category: bash
---

##并集
		qiuji_000@king-pc /tmp
		$ (echo a;echo b;echo a;echo c) | sort | uniq
		a
		b
		c

##交集
		qiuji_000@king-pc /tmp
		$ (echo a;echo b;echo a;echo c) | sort | uniq -d
		a

##去掉重复
		qiuji_000@king-pc /tmp
		$ (echo a;echo b;echo a;echo c) | sort | uniq -u
		b
		c


##参考
1. <http://hi.baidu.com/huangyunict/item/f1b7a5db7d26c24ddcf9bef7>
