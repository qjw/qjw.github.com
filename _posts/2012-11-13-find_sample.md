---
layout: post
title:  find的一些用法
category: bash
---

    #!/bin/sh

    # find 由选项，测试，动作组成，一般按照这个顺序写参数
    # 我们常用的maxdepth和mindepth都是选项，所以若放在某个测试后面find会给出警告

    # 可以有多个测试，默认使用-a（与），你手动指定-a效果是一样的
    # find默认使用-print动作，你也可以手动指定exec
    # 对于-exec，后面必须使用\;并且必须与前面的内容保持至少一个空格
    find . -mindepth 2 -path "*/b/*" -a -name "run*" -exec ls -l {} \;
    #-rwxrwxr-x 1 qjw qjw 121 Nov 17 10:22 ./b/run.sh

    # 一般都是用-mindepth 1排除当前目录
    # 使用-type 限定文件类型
    # 使用-name 限定名称
    # 使用-path 限定路径

    # 我们可以用()来限定优先级，-o -a用来用来表示与或
    # 注意动作也受到关系运算符的影响
    find . -mindepth 2 \( -path "*/b/*" -a -type f  -o -path "*/a/*" \) \
        -exec ls -l {} \;
    #-rwxrwxr-x 1 qjw qjw 201 Nov 17 10:22 ./b/test.sh
    #-rwxrwxr-x 1 qjw qjw 121 Nov 17 10:22 ./b/run.sh
    #-rwxrwxr-x 1 qjw qjw 201 Nov 17 10:22 ./a/test.sh
    #-rwxrwxr-x 1 qjw qjw 121 Nov 17 10:22 ./a/run.sh

    # 这一句就变成了，只有-o 后面的测试才执行exec动作了
    find . -mindepth 2  -path "*/b/*" -a -type f  -o -path "*/a/*" \
        -exec ls -l {} \;
    #-rwxrwxr-x 1 qjw qjw 201 Nov 17 10:22 ./a/test.sh
    #-rwxrwxr-x 1 qjw qjw 121 Nov 17 10:22 ./a/run.sh

    # 括号可以嵌套
    find . -mindepth 2 \( -path "*/b/*" -a \( -type f  -o -path "*/a/*" \) \) \
        -exec ls -l {} \;
    #-rwxrwxr-x 1 qjw qjw 201 Nov 17 10:22 ./b/test.sh
    #-rwxrwxr-x 1 qjw qjw 121 Nov 17 10:22 ./b/run.sh

    # 动作同样可以有多个
    find . -mindepth 2 \( -path "*/b/*" -a -type f  -o -path "*/a/*" \) \
        -exec ls -l {} \; -exec ls -l {} \; -print

    # 某些动作若担心误操作，可以先确认（-ok)
    find . -mindepth 2 \( -path "*/b/*" -a -type f  -o -path "*/a/*" \) \
        -ok ls -l {} \;

    # 排除目录
    # 若满足前面条件，那么 -prune将不进入该目录，也就是返回false，-o后面会被执行
    find . \( -path "*/b/*"  \) -prune -o -exec echo {} \;

    # 排除后继续过滤
    find . \( -path */b/* -o -path "*/a/*" \) -prune -o \
        -type f -name "*.sh" -print

