---
layout: post
title:  grep/sed/awk匹配两个单词
category: bash
---
         
##grep
		qiuji_000@king-pc /tmp
		$ (echo aa;echo bb) | grep "\(aa\|bb\)"
		aa
		bb
		
		qiuji_000@king-pc /tmp
		$ (echo aa;echo bb) | grep -E "(aa|bb)"                                         
		aa
		bb

##sed
		qiuji_000@king-pc /tmp
		$ (echo aa;echo bb) | sed -n "/\(aa\|bb\)/p"
		aa
		bb

##awk
		qiuji_000@king-pc /tmp
		$ (echo aa;echo bb) | awk '/aa/||/bb/{ print $0 }'
		aa
		bb


##参考
1. <http://bbs.chinaunix.net/thread-1199427-1-1.html>
