---
layout: post
title: 通过Chrome或Firedebug插件一段js
category: js
---


##动态插入jquery

        (function(){
            var scip = document.createElement("script");

            newatt=scip.setAttribute("src","http://code.jquery.com/jquery-1.7.1.min.js");
            newatt=scip.setAttribute("type","text/javascript");
            newatt=scip.setAttribute("charset","utf-8");
            
            x=document.getElementsByTagName("head")[0];
            x.appendChild(scip);
        })()


##修改让其浮动在右上
        $("#float").css({ top: "10px",right: "10px", width: "300", position: "fixed", background: "#99ff00" });

        
##参考
1. <http://www.cnblogs.com/regedit/archive/2008/03/11/1100170.html>
1. <http://code.google.com/p/jquery-scroll-follow/>