---
layout: post
title: bash中eval和双引号的一些坑
category: bash
---

## 变量展开

	#!/bin/bash    
	    
	val="-argc \"a b c\""                                                        
		                                                                     
	fuck()
	{
	    echo "${1} | ${2}"                                                       
	}                                                                            

	fuck "${val}"
	fuck ${val} 
	eval fuck ${val}      
	eval fuck "${val}" 
	
---

	/tmp <king@king-HP-Compaq-6530b-FP587PA-AB2> 09:43:38 $ ./test.sh 
	-argc "a b c" | 
	-argc | "a
	-argc | a b c
	-argc | a b c
	
##多行变量

	#!/bin/bash
	var="`cat /proc/meminfo | head -n 2`"                                        
	echo $var                                                                    
	echo ${var}                                                                  
	echo "$var"                                                                  
	echo "${var}" 
	
---

	MemTotal: 3946796 kB MemFree: 566252 kB
	MemTotal: 3946796 kB MemFree: 566252 kB
	MemTotal:        3946796 kB
	MemFree:          566252 kB
	MemTotal:        3946796 kB
	MemFree:          566252 kB
	
	
##缩进

	#!/bin/bash                                                                  
		                                                                     
	var="a    b    c"                                                            
	echo $var                                                                    
	echo "$var"  
	
---

	a b c
	a    b    c
	
	
##参考
1. <http://www.cnblogs.com/fhefh/archive/2011/04/17/2019103.html>




