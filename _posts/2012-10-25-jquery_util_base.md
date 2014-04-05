---
layout: post
title: Jquery的一些常用小技巧
category: js
---

#创建DOM元素

        $("<p>Hi there!</p>")

        
#jquery判断对象是否存在
用jquery判断一个对象是否存在不能用

        if($ ("#id")){
        }else{
        }

jquery不管对象存不存在都会返回object。应该用

        if($ ("#id").length>0){
        }else{
        }
        
or

        if($ ("#id")[0]){ 
        } else {
        }

or 

        if(document.getElementById("id")){
        } else {
        }

#字符串转换成json对象

        function func(data){
            var dataObj=eval("("+data+")");//转换为json对象 
            alert(dataObj.member);
        }
        
#清空子元素

        $("#id").empty();
        
        
#遍历所有结果

        $("button").click(function(){
            $("li").each(function(){
                alert($(this).text())
            });
        });

#显示&隐藏

        $("#id").hide();
        $("#id").show();
        

#删除函数绑定

        $("#id").off("click");
        
#禁止选择

        $("#id").attr('unselectable','on').css('MozUserSelect','none');
        
#参考
1. <http://book.51cto.com/art/200904/119229.htm>
1. <http://blog.5d.cn/user36/SmallTalker/200906/520875.html>
1. <http://www.jb51.net/article/19366.htm>
1. <http://topic.csdn.net/u/20100521/17/c559305f-4e84-4b78-90a7-05b98e65017c.html>
1. <http://www.w3school.com.cn/jquery/traversing_each.asp>
1. <http://stackoverflow.com/questions/2700000/how-to-disable-text-selection-using-jquery>

