---
layout: post
title:  bash两种比较符的区别
category: bash
---
         
1. [[用"&&"而不是"-a"表示逻辑"与"，用"||"而不是"-o"表示逻辑"或"

        #!/bin/bash
        if [[ true && false ]];then
                echo "[["
        fi
        if [ true -a false ];then
                echo "[["
        fi
         
2. [ ]为shell命令，所以在其中的表达式应是它的命令行参数，所以串比较操作符">" 与"<"必须转义，否则就变成IO改向操作符了。[[中"<"与">"不需转义。两者支持的运算符，例如-gt == > < -f -e -n等都一致

        qjw@qjw-VirtualBox /tmp $ type [
        [ is a shell builtin
        qjw@qjw-VirtualBox /tmp $ type [[
        [[ is a shell keyword
        qjw@qjw-VirtualBox /tmp $ which [[
        qjw@qjw-VirtualBox /tmp $ which [
        /usr/bin/[

3. [[]]能用正则，而[]不行

当使用"=" 或者"=="时，右边匹配"通配符"，若使用"=~"时，右边匹配"正则表达式"，无论"通配符"还是"正则表达式"都**不要**用双引号'"'括起来。

        #!/bin/bash
        [ "test.php" == *.php ] && echo true || echo false 
        [[ "test.php" == *.php ]] && echo true || echo false 

4. [[ ... ]]进行算术扩展，而[ ... ]不做

        $ [[ 99+1 -eq 100 ]]&&echo true||echo false 
        true 
        $ [ 99+1 -eq 100 ]&&echo true||echo false 
        bash: [: 99+1: integer expression expected 
        false 
        $ [ $((99+1)) -eq 100 ]&&echo true||echo false 
        true 

##参考
1. <http://www.xclinux.cn/?p=808>
1. <http://zengrong.net/post/1563.htm>