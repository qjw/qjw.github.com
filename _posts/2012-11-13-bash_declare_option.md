---
layout: post
title:  Bash Declare妙用
category: linux
---

##参考
1. <http://www.yeeyan.org/articles/view/20180/38866>
1. <http://www.gnu.org/software/bash/manual/bashref.html#Bash-Builtins>

##Content

**declare**

 *declare [-aAfFilrtux] [-p] [name[=value] …]*
 
Declare variables and give them attributes. If no names are given, then display the values of variables instead.

The -p option will display the attributes and values of each name. When -p is used with name arguments, additional options are ignored.

When -p is supplied without name arguments, declare will display the attributes and values of all variables having the attributes specified by the additional options. If no other options are supplied with -p, declare will display the attributes and values of all shell variables. The -f option will restrict the display to shell functions.

The -F option inhibits the display of function definitions; only the function name and attributes are printed. If the extdebug shell option is enabled using shopt (see The Shopt Builtin), the source file name and line number where the function is defined are displayed as well. -F implies -f.

The -g option forces variables to be created or modified at the global scope, even when declare is executed in a shell function. It is ignored in all other cases.

The following options can be used to restrict output to variables with the specified attributes or to give variables attributes:

**-a**

Each name is an indexed array variable (see Arrays).

**-A**

Each name is an associative array variable (see Arrays).

**-f**

Use function names only.

**-i**

**The variable is to be treated as an integer; arithmetic evaluation (see Shell Arithmetic) is performed when the variable is assigned a value.**

**-l**

When the variable is assigned a value, all upper-case characters are converted to lower-case. The upper-case attribute is disabled.

**-r**

**Make names readonly. These names cannot then be assigned values by subsequent assignment statements or unset.**

**-t**

Give each name the trace attribute. Traced functions inherit the DEBUG and RETURN traps from the calling shell. The trace attribute has no special meaning for variables.

**-u**

When the variable is assigned a value, all lower-case characters are converted to upper-case. The lower-case attribute is disabled.

**-x**

**Mark each name for export to subsequent commands via the environment.**

Using ‘+’ instead of ‘-’ turns off the attribute instead, with the exceptions that ‘+a’ may not be used to destroy an array variable and ‘+r’ will not remove the readonly attribute. When used in a function, declare makes each name local, as with the local command, unless the ‘-g’ option is used. If a variable name is followed by =value, the value of the variable is set to value.

The return status is zero unless an invalid option is encountered, an attempt is made to define a function using ‘-f foo=bar’, an attempt is made to assign a value to a readonly variable, an attempt is made to assign a value to an array variable without using the compound assignment syntax (see Arrays), one of the names is not a valid shell variable name, an attempt is made to turn off readonly status for a readonly variable, an attempt is made to turn off array status for an array variable, or an attempt is made to display a non-existent function with -f.