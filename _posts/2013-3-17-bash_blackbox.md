---
layout: post
title:  屏蔽或挂钩特定的命令
category: bash
---

在Linux下，我们经常会调用一些脚本，有的时候基于安全性考虑希望屏蔽某些命令，或者做些特殊的调试或者审计，对某些命令进行挂钩。

假如yy.sh脚本里面使用了reboot脚本，而xx.sh调用它，但是希望屏蔽yy.sh里面的reboot的脚本，那如何处理呢。

	#!/bin/bash
	reboot()
	{
		true;
	}
	export -f reboot
	yy.sh

若希望审计，则可以

	#!/bin/bash
	reboot()
	{
		# do something else
		/bin/reboot
	}
	export -f reboot
	yy.sh

