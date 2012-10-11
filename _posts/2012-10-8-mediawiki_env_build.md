---
layout: default
title: 搭建一个简单的Mediawiki环境
---
#搭建一个简单的Mediawiki环境

* 下载，安装WampServer2.2a-x32.exe
* 下载，安装mediawiki-1.18.4.tar.gz(解压到C:\wamp\www\wiki 目录)
* 浏览器访问http://localhost/wiki/
* LocalSettings.php not found.Please set up the wiki first.点击超链接
* 按照向导生成LocalSettings.php，并保存到wiki目录
* 替换图片C:\wamp\www\wiki\skins\common\images\wiki.png
* 修改MediaWiki:Sidebar和首页
* mysqldump -u root -p1 -h localhost --opt wiki --default-character-set=gbk > rswiki-backup.sql
* mysql -uroot -p1 -hlocalhost -e "create database if not exists database"
* mysql -uroot -p1 -hlocalhost wiki < rswiki-backup.sql
* 建议将## $wgServer           = "http://localhost"; 注释，因为跨主机访问会失败（或者改成对应的IP）
* $wgScriptPath       = "/wiki";  这个需要根据放置的路径而修改。若放在根目录，那么就必须为空，而不是/
* 文件上传页面Special:Upload，需要将$wgEnableUploads		= true;
* special:filelist  已经上传的文件列表
* special:recentchanges  最近更新的页面
* 对于指定到图片的超链接，wiki默认会将图片显示出来。你可以使用如下的方式显示图片[http://www.qjw.com/logo.png http://www.qjw.com/logo.png]若需要禁用外部图片显示，可以将后面的超链接描述修改成和超链接不一致，例如[http://www.qjw.com/logo.png logo.png]