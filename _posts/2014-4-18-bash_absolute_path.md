---
layout: post
title: Bash下获取脚本绝对路径的几种方法
category: bash
---

	#!/bin/bash                                                                     
	var=$(readlink -f $0)                                                           
	echo $var                                                                       
		   
	# ${BASH_SOURCE[0]}                                                                     
	var=$(realpath $0)                                                              
	echo $var                                                                       
		                                                                        
	pushd `dirname $0` > /dev/null                                                  
	var=$(pwd)/$(basename $0)                                                       
	popd > /dev/null                                                                
	echo $var
