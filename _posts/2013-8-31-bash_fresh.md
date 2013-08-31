---
layout: post
title: Bash中直接刷新
category: bash
---

##回车

	for ((i=0;i<10000;i++))
	do
		j=$((i%4))
		if [ "${j}" -eq 0 ]
		then
			echo -ne "\r${i}-"
		elif [ "${j}" -eq 1 ]
		then
			echo -ne "\r${i}\\"
		elif [ "${j}" -eq 2 ]
		then
			echo -ne "\r${i}|"
		else
			echo -ne "\r${i}/"
		fi
		sleep 0.2
	done
	
##退格

	echo -ne ">0%"
	for ((i=1;i<100;i++))
	do
		if [ "${i}" -le 10 ]
		then
			echo -ne "\b\b\b->${i}%"
		else
			echo -ne "\b\b\b\b->${i}%"
		fi
		sleep 0.3
	done
	
##tput

	clear
	for ((i=1;i<10000;i++))
	do
		tput cup 0
		echo "aa ${i}"
		echo "bb ${i}-${i}"
		sleep 1
	done
	
##参考
1. <http://stackoverflow.com/questions/1881594/use-winmerge-inside-of-git-to-file-diff>
1. <http://winmerge.org/>
1. <https://code.google.com/p/msysgit/>
