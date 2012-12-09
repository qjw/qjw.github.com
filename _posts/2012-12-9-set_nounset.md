---
layout: post
title:  Bash访问未定义变量报错
category: bash
---
         
        #!/bin/sh
        # XXX命令不存在
        set -o nounset

----

    -o option-name
     Set the option corresponding to option-name:
       nounset
       Same as -u.

    -u
     Treat unset variables and parameters other than the special 
     parameters ‘@’ or ‘*’ as an error when performing parameter expansion. 
     An error message will be written to the standard error, 
     and a non-interactive shell will exit.


1. <http://www.gnu.org/software/bash/manual/bashref.html#The-Set-Builtin>
2. <http://xshell.net/shell/bash.html>