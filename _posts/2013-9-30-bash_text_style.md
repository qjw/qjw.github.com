---
layout: post
title: BASH 输出彩色字符
category: bash
---

##输出彩色

	declare -r g_red="\e[0;31;44m"
	declare -r g_green="\e[0;32m"
	declare -r g_blue="\e[0;34m"
	declare -r g_end="\e[0m"

	echo -e "${g_red}read${g_end}"
	echo -e "${g_green}green${g_end}"
	echo -e "${g_blue}blue${g_end}"  

we used the ANSI escape sequence **\e[attribute code;text color code*m*** to display a blue text

####Attribute codes:
00=none 01=bold 04=underscore 05=blink 07=reverse 08=concealed 

####Text color codes:
30=black 31=red 32=green 33=yellow 34=blue 35=magenta 36=cyan 37=white

####Background color codes:
40=black 41=red 42=green 43=yellow 44=blue 45=magenta 46=cyan 47=white


	Black       0;30     Dark Gray     1;30
	Blue        0;34     Light Blue    1;34
	Green       0;32     Light Green   1;32
	Cyan        0;36     Light Cyan    1;36
	Red         0;31     Light Red     1;31
	Purple      0;35     Light Purple  1;35
	Brown       0;33     Yellow        1;33
	Light Gray  0;37     White         1;37
	
	
##参考
1. <http://www.tldp.org/HOWTO/Bash-Prompt-HOWTO/x329.html>
1. <http://www.csc.uvic.ca/~sae/seng265/fall04/tips/s265s047-tips/bash-using-colors.html>