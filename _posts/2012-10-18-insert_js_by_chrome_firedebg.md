---
layout: post
title: 通过Chrome或Firedebug插件一段js
category: js
---


        (function(){
            var scip = document.createElement("script");

            newatt=scip.setAttribute("src","http://code.jquery.com/jquery-1.7.1.min.js");
            newatt=scip.setAttribute("type","text/javascript");
            newatt=scip.setAttribute("charset","utf-8");
            
            x=document.getElementsByTagName("head")[0];
            x.appendChild(scip);
        })()

