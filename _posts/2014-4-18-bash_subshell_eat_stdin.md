---
layout: post
title: 子shell吃掉所有输入导致的bug
category: bash
---

	/tmp <king@king> test.sh 2>/dev/null
	abcd
	efgh
	ijkl
	------------------------
	abcd
	------------------------
	abcd
	efgh
	ijkl

---

	#!/bin/bash
	some_command()
	{
		while read line
		do
			true
		done
		echo "${@}"
	}

	# 默认全部输出
	cat data.txt |
	while read line
	do
		echo "${line}"
	done

	echo "------------------------"

	# read line的输入被some_command全部吃掉了，导致就一行
	cat data.txt |
	while read line
	do
		some_command "${line}"
	done

	echo "------------------------"

	# 在调用吃stdin的子shell前，先关闭stdin
	cat data.txt |
	while read line
	do
		some_command "${line}" <&-
	done

---

	/tmp <king@king> 12:27:15 $ cat data.txt 
	abcd
	efgh
	ijkl

##参考
1. <http://stackoverflow.com/questions/8376166/execute-a-command-on-remote-hosts-via-ssh-from-inside-a-bash-script>

