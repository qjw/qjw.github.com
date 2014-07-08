---
layout: post
title: Linux下获取网卡，PCI映射
category: linux
---

	#!/bin/bash                                                                        
	find /sys/bus/pci/devices/ -mindepth 1 -maxdepth 1 -type l |                       
	while read file                                                                    
	do                                                                                 
	    newpath="`realpath "${file}"`"                                                 
	    find "${newpath}" -name address                                                
	done |                                                                             
	sort |                                                                             
	uniq |                                                                             
	while read file                                                                    
	do                                                                                 
	    mac="`cat "${file}"`"                                                          
		                                                                           
	    file="${file%/*}"                                                              
	    name="${file##*/}"                                                             
		                                                                           
	    pci="${file%/*}"                                                               
	    pci="${pci%/*}"                                                                
	    pci="${pci##*/}"
	    echo "${pci} ${name} ${mac}"                                                   
	done

##参考
1. <http://wangxu.me/site/node/39>

