---
layout: post
title: sed处理多行数据
category: bash
---

####多行合并成一行

	root@qjw-VirtualBox:~# seq 0 9 | sed 'N;s/\n/ /'
	0 1
	2 3
	4 5
	6 7
	8 9

	root@qjw-VirtualBox:~# seq 0 17 | sed -e '1~3N;s/\n/ /' -e 'N;s/\n/ /'
	0 1 2
	3 4 5
	6 7 8
	9 10 11
	12 13 14
	15 16 17

####每隔N行处理

	root@qjw-VirtualBox:~# seq 10 19 | sed '1~2 s/1/x/'
	x0
	11
	x2
	13
	x4
	15
	x6
	17
	x8
	19

	root@qjw-VirtualBox:~# seq 10 19 | sed -e '1~3 s/1/x/' -e '2~3 s/1/x/'
	x0
	x1
	12
	x3
	x4
	15
	x6
	x7
	18
	x9

####去掉重复

	#!/bin/bash
	next_flag=0
	last_line=""
	cat tmp |
	while read line
	do
		if [ "${next_flag}" -eq 0 ]
		then
			if echo "${line}" | grep "aaa:" > /dev/null
			then
				last_line="${line}"
				next_flag=1
			fi
		else
			if echo "${line}" | grep "bbb:" > /dev/null
			then
				echo "${last_line}"
				echo "${line}"
				next_flag=0
			else
				last_line="${line}"
			fi
		fi
	done

---

	root@qjw-VirtualBox:/tmp/test# ./tmp.sh 
	aaa:2
	bbb:3
	aaa:6
	bbb:7
	aaa:10
	bbb:11
	root@qjw-VirtualBox:/tmp/test# cat tmp
	aaa:0
	aaa:1
	aaa:2
	bbb:3
	bbb:4
	bbb:5
	aaa:6
	bbb:7
	bbb:8
	aaa:9
	aaa:10
	bbb:11
	aaa:12

##参考
1. <http://my.oschina.net/shelllife/blog/118337>