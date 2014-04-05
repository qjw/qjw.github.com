---
layout: post
title:  打印Bash函数调用栈
category: bash
---
         
    #!/bin/bash

    func1()
    {
        local -r stack_size_="${#FUNCNAME[@]}"
        local i=0
        while [ "${i}" -lt "${stack_size_}" ];do
            echo "${FUNCNAME[${i}]}"
            i=$(($i + 1))
        done
    }

    func2()
    {
            func1
    }
    func3()
    {
            func2
    }
    func4()
    {
            func3
    }

    func4

----

    qjw@qjw-VirtualBox /tmp $ ./test.sh 
    func1
    func2
    func3
    func4
    main

##FUNCNAME

An array variable containing the names of all shell functions currently in the execution call stack. The element with index 0 is the name of any currently-executing shell function. The bottom-most element (the one with the highest index) is "main". This variable exists only when a shell function is executing. Assignments to FUNCNAME have no effect and return an error status. If FUNCNAME is unset, it loses its special properties, even if it is subsequently reset.

This variable can be used with BASH_LINENO and BASH_SOURCE. Each element of FUNCNAME has corresponding elements in BASH_LINENO and BASH_SOURCE to describe the call stack. For instance, ${FUNCNAME[$i]} was called from the file ${BASH_SOURCE[$i+1]} at line number ${BASH_LINENO[$i]}. The caller builtin displays the current call stack using this information.
    
<http://www.gnu.org/software/bash/manual/bashref.html#Bourne-Shell-Variables>