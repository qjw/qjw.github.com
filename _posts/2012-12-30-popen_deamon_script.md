---
layout: post
title:  popen调用脚本陷阱
category: bash
---
         
要执行一个脚本，并且获得它的输出(或者输入数据到stdin)，一般会使用popen，然而若调用的脚本中存在后台运行的脚本，例如

	#!/bin/bash
	# ...
	some command &
	# ...
	
那么可能会有些问题，popen的这个脚本可能会卡死，直到这个后台运行的脚本结束。具体原因是因为popen要等待stdout结束，所以解决办法就是

	#!/bin/bash
	# ...
	some command 1>&2 &
	# ...
	
若希望同时屏蔽stderr，

	#!/bin/bash
	# ...
	some command > /dev/null 2>&1 &
	# 下面的脚本将失效
	some command 2>&1 > /dev/null &
	# ...
	
##参考
1. <http://hi.baidu.com/hldgf/item/fc80aab25562b397194697ce>