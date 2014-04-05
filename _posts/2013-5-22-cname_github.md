---
layout: post
title:  给github博客绑定域名
category: www
---
	
###获得二级域名的IP

使用命令"dig qjw.github.com",参考这里[这里](http://jser.me/p/a1186)。在Linux下，使用apt-get install dnsutils安装dig工具。

###创建CNAME文件
在根目录创建CNAME，内容为绑定的域名，例如qjw.qiujinwu.com

###更新DNS
如果绑定的是顶级域名，则DNS要新建一条A记录，指向204.232.175.78。如果绑定的是二级域名，则DNS要新建一条CNAME记录，指向username.github.com（请将username换成你的用户名）。此外，别忘了将_config.yml文件中的baseurl改成根目录"/"。

	
###参考
1. <http://jser.me/p/a1186>
1. <http://www.ruanyifeng.com/blog/2012/08/blogging_with_jekyll.html>