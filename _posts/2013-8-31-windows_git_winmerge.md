---
layout: post
title: 在Windows下配合winmerge使用git
category: other
---

##Windows

	#!/bin/sh
	echo Launching WinMergeU.exe: $1 $2
	"$PROGRAMFILES/WinMerge/WinMergeU.exe" -e -ub -dl "Base" -dr "Mine" "$1" "$2"

---

	git config --replace --global diff.tool winmerge
	git config --replace --global difftool.winmerge.cmd "winmerge.sh \"$LOCAL\" \"$REMOTE\""
	git config --replace --global difftool.prompt false
	git config --replace --global diff.external winmerge.sh

##Linux

	#!/usr/bin/python

	import sys
	import os

	os.system('meld "%s" "%s"' % (sys.argv[2], sys.argv[5]))

---

	apt-get install meld
	git config --global diff.external meld
	git config --global diff.external diff.py

	
##参考
1. <http://stackoverflow.com/questions/1881594/use-winmerge-inside-of-git-to-file-diff>
1. <http://winmerge.org/>
1. <https://code.google.com/p/msysgit/>
