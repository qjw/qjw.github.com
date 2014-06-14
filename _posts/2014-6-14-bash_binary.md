---
layout: post
title: 使用bash处理二进制
category: bash
---

##Sample

	#!/bin/bash                                                                     
		                                                                        
	char_print()                                                                    
	{                                                                               
	    echo -ne $(printf '\\x%02x' "$1")                                           
	}                                                                               
		                                                                        
	short_print()                                                                   
	{                                                                               
	    local a b tmp t                                                             
		                                                                        
	    tmp="${1}"                                                                  
	    t=$((a=tmp/256)); t=$((tmp-=a*256))                                         
	    t=$((b=tmp))                                                                
	    echo -ne $(printf '\\x%02x\\x%02x' $a $b)                                   
	}                                                                               
		                                                                        
	char_print "$((0x12))"                                                          
	char_print "$((0x34))"                                                          
	char_print "$((0x56))"                                                          
	char_print "$((0x78))"                                                          
		                                                                        
	short_print "$((0x90ab))"                                                       
	short_print "$((0xcdef))"

---
	/tmp <king@king-HP-Compaq-6530b-FP587PA-AB2> 10:52:38 $ ./x.sh > a
	/tmp <king@king-HP-Compaq-6530b-FP587PA-AB2> 10:55:22 $ hexdump -C a 
	00000000  12 34 56 78 90 ab cd ef                           |.4Vx....|
	00000008


若要更新一个二进制文件，可以使用dd命令notrunc

	dd if=/tmp/a of=/tmp/b conv=notrunc

##参考
1. <http://ar.newsmth.net/thread-129a6c92b6aa170.html>
1. <http://blog.sina.com.cn/s/blog_6a6a80aa0101352n.html>

