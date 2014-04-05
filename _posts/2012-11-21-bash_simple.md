---
layout: post
title:  Bash脚本简单化
category: bash
---

通常一个完善的脚本都需要判断各种异常，所以很多人往往就是一堆

        #!/bin/bash
        if [ $var -gt $var2 ]
        then
            echo "do sth"
        fi
        
        command
        if [ $? -ne 0 ]
        then
            echo "do sth"
        fi
        

可以简单一点

        [ $var -gt $var2 ] || echo "do sth"
        command || echo "do sth"
        
进一步

        [ $var -gt $var2 ] && command || echo "do sth"
        
这基本能用，但是错误后，往往希望能够提示点什么，例如打日志

        Log()
        {
            local -r ret_="${?}"
            echo "log message"
            return "${ret_}"
        }
        [ $var -gt $var2 ] && command || Log "log sth"  || echo "do sth"
        
        
这能够解决日志的问题，但是若希望做更多的操作呢？子Shell是个不错的主意，不过看上去就不雅观了

        [ $var -gt $var2 ] && command || ( \
            local -r ret_="${?}";\
            command1;\
            command2;\
            return "${ret_}";\
            )  || echo "do sth"
