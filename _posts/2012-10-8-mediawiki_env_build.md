---
layout: default
title: 搭建一个简单的Mediawiki环境
---
#搭建一个简单的Mediawiki环境

* 下载，安装WampServer2.2a-x32.exe
* 下载，安装mediawiki-1.18.4.tar.gz(解压到C:\wamp\www\wiki 目录)
* 浏览器访问http://localhost/wiki/
* LocalSettings.php not found.Please set up the wiki first.点击超链接
* 替换图片C:\wamp\www\wiki\skins\common\images\wiki.png
* 修改MediaWiki:Sidebar和首页
* 备份数据库mysqldump -u root -p1 -h localhost --opt ${DB_NAME} > rswiki-backup.sql