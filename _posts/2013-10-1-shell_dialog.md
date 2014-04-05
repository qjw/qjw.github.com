---
layout: post
title: shell dialog简介
category: bash
---

##手册

	* Display dialog boxes from shell scripts *

	Usage: dialog <options> { --and-widget <options> }
	where options are "common" options, followed by "box" options

	Special options:
	  [--create-rc "file"]
	Common options:
	  [--ascii-lines] [--aspect <ratio>] [--backtitle <backtitle>]
	  [--begin <y> <x>] [--cancel-label <str>] [--clear] [--colors]
	  [--column-separator <str>] [--cr-wrap] [--date-format <str>]
	  [--default-item <str>] [--defaultno] [--exit-label <str>]
	  [--extra-button] [--extra-label <str>] [--help-button]
	  [--help-label <str>] [--help-status] [--ignore] [--input-fd <fd>]
	  [--insecure] [--item-help] [--keep-tite] [--keep-window]
	  [--max-input <n>] [--no-cancel] [--no-collapse] [--no-kill]
	  [--no-label <str>] [--no-lines] [--no-ok] [--no-shadow] [--nook]
	  [--ok-label <str>] [--output-fd <fd>] [--output-separator <str>]
	  [--print-maxsize] [--print-size] [--print-version] [--quoted]
	  [--scrollbar] [--separate-output] [--separate-widget <str>] [--shadow]
	  [--single-quoted] [--size-err] [--sleep <secs>] [--stderr] [--stdout]
	  [--tab-correct] [--tab-len <n>] [--time-format <str>] [--timeout <secs>]
	  [--title <title>] [--trace <file>] [--trim] [--version] [--visit-items]
	  [--yes-label <str>]
	Box options:
	  --calendar     <text> <height> <width> <day> <month> <year>
	  --checklist    <text> <height> <width> <list height> <tag1> <item1> <status1>...
	  --dselect      <directory> <height> <width>
	  --editbox      <file> <height> <width>
	  --form         <text> <height> <width> <form height> <label1> \
			<l_y1> <l_x1> <item1> <i_y1> <i_x1> <flen1> <ilen1>...
	  --fselect      <filepath> <height> <width>
	  --gauge        <text> <height> <width> [<percent>]
	  --infobox      <text> <height> <width>
	  --inputbox     <text> <height> <width> [<init>]
	  --inputmenu    <text> <height> <width> <menu height> <tag1> <item1>...
	  --menu         <text> <height> <width> <menu height> <tag1> <item1>...
	  --mixedform    <text> <height> <width> <form height> <label1> <l_y1> \
			<l_x1> <item1> <i_y1> <i_x1> <flen1> <ilen1> <itype>...
	  --mixedgauge   <text> <height> <width> <percent> <tag1> <item1>...
	  --msgbox       <text> <height> <width>
	  --passwordbox  <text> <height> <width> [<init>]
	  --passwordform <text> <height> <width> <form height> <label1> <l_y1> \
			<l_x1> <item1> <i_y1> <i_x1> <flen1> <ilen1>...
	  --pause        <text> <height> <width> <seconds>
	  --progressbox  <height> <width>
	  --radiolist    <text> <height> <width> <list height> <tag1> <item1> <status1>...
	  --tailbox      <file> <height> <width>
	  --tailboxbg    <file> <height> <width>
	  --textbox      <file> <height> <width>
	  --timebox      <text> <height> <width> <hour> <minute> <second>
	  --yesno        <text> <height> <width>
	  

##常用选项
1. --backtitle "welcome" : 背景标题
2. --title : 窗口标题
3. --no-shadow/--shadow : 是否显示窗口阴影
1. --colors : 支持字体颜色
1. --defaultno ：默认NO(yesno默认是yes)
1. --ok-label : 设置ok按钮的字符
1. --yes-label/--no-lable : 设置yes/no按钮的字符
1. --exit-label ：设置退出按钮的字符
1. --insecure : 密码框时,用*代替,而不是啥都没有

解释内含在对话框的”\Z”的顺序属性。他告诉对话框设置颜色或者视频属性:
0到7是ANSI码在curses中分别指定为:黑色，红色，绿色，黄色，蓝色，紫红色，蓝绿色和白色。
粗体用’b’设置，重设用‘B’。背面用’r’设置，重设用’R’。下划线用’u’设置，重设用’U’。所做出的改动将会累积起来。例如，”\Zb\Z1’”表示文本显示红色。恢复正常的设置用”\Zn”

##常用窗口

	#/bin/bash
	# 消息窗口(带ok按钮)
	dialog --backtitle "welcome" --title "hello world" --msgbox "content" 20 100
	
	# 确认窗口(带yes/no按钮),根据返回值判断点击那个按钮,yes返回0
	dialog --backtitle "welcome" --title "hello world" --yesno "content" 20 100
	
	#解析颜色
	dialog --backtitle "welcome" --title "hello world" --colors --msgbox "\Z2content\Zu" 20 100 
	
	# 输出结果需要文件重定向,根据返回值判断点击那个按钮,yes返回0
	dialog --inputbox "tips:" 10 30 "default string" 2> /tmp/input
	
	# 直接读取某个文件显示(带exit按钮)
	dialog --textbox /etc/fstab  200 0
	
	# 菜单
	# 输出结果需要文件重定向,根据返回值判断点击那个按钮,yes返回0
	#menu         <text> <height> <width> <menu height>
	dialog --menu "menu test" 10 100 10  key1 value1 key2 value2 2> /tmp/input
	
	#单选框(on/off)
	dialog --radiolist "radiolist" 10 100 10 a1 a2 on b1 b2 off 2> /tmp/input
	
	#复选框
	dialog --checklist "checklist" 10 100 10 a1 a2 on b1 b2 off
	
	# 密码框
	dialog --insecure --passwordbox pwd 10 100 default 2> /tmp/input
	
	#进度条
	fun()
	{
		for((i=0;i<=100;i++))                                                                                                                  
		do  
			echo XXX
			echo "do sth with percent '${i}' "
			echo XXX
			echo "${i}"
			sleep 0.1 
		done
	}

	fun | dialog --gauge "title" 10 100 0
	
	#表单
	dialog --form "input" 10 100 4 \ 
		ip 1 1 "192.168.1.1" 1 10 50 50 \
		mask 2 1 "255.255.255.0" 2 10 50 50 \                                                                                                  
		gateway 3 1 "192.168.1.1" 3 10 50 50
	  
##参考
1. <http://blog.csdn.net/zhuying_linux/article/details/5465290>
1. <http://bbs.chinaunix.net/thread-1776555-1-1.html>
1. <http://www.ibm.com/developerworks/cn/aix/library/au-spunix_guiscripting/>
1. <http://molinux.blog.51cto.com/2536040/466001>
1. <http://blog.chinaunix.net/uid-24921475-id-2547158.html>