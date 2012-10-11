---
layout: default
title: MediaWiki常用插件
---
# MediaWiki常用插件

##语法高亮
####下载源码
svn co [http://chanye.07073.com/guonei/647395.html](http://chanye.07073.com/guonei/647395.html)
####官网
[http://www.mediawiki.org/wiki/Extension:SyntaxHighlight_GeSHi](http://www.mediawiki.org/wiki/Extension:SyntaxHighlight_GeSHi)
####配置
* require_once( "$IP/extensions/SyntaxHighlight_GeSHi/SyntaxHighlight_GeSHi.php");
* $wgSyntaxHighlightDefaultLang = "c";
* $wgSyntaxHighlightDefaultLang = "cpp";

##富文本编辑器
####下载源码
[http://svn.wikimedia.org/svnroot/mediawiki/trunk/extensions/FCKeditor](http://svn.wikimedia.org/svnroot/mediawiki/trunk/extensions/FCKeditor)
####官网
[http://www.mediawiki.org/wiki/Extension:FCKeditor_(Official)](http://www.mediawiki.org/wiki/Extension:FCKeditor_(Official))
####配置
require_once(“$IP/extensions/FCKeditor/FCKeditor.php”);

安装见[这里](http://mkv.cn/2223/install-fckeditor-for-mediawiki)