---
layout: default
title: MediaWiki常用插件
---
# MediaWiki常用插件

#语法高亮
##下载源码
svn co http://svn.wikimedia.org/svnroot/mediawiki/trunk/extensions/SyntaxHighlight_GeSHi
##官网
http://www.mediawiki.org/wiki/Extension:SyntaxHighlight_GeSHi
##配置
require_once( "$IP/extensions/SyntaxHighlight_GeSHi/SyntaxHighlight_GeSHi.php");
$wgSyntaxHighlightDefaultLang = "c";
$wgSyntaxHighlightDefaultLang = "cpp";

#富文本编辑器
##下载源码
http://svn.wikimedia.org/svnroot/mediawiki/trunk/extensions/FCKeditor
#官网
http://www.mediawiki.org/wiki/Extension:FCKeditor_(Official)
#配置
require_once(“$IP/extensions/FCKeditor/FCKeditor.php”);

* from http://mkv.cn/2223/install-fckeditor-for-mediawiki
* markdown在线编辑器 http://dillinger.io/