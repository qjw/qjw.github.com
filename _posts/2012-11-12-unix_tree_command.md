---
layout: post
title:  unix下的tree命令
category: bash
---

**该命令可以打印某个目录的树状结构（文本描述）**

**正常情况下，它使用字母排序，建议使用--dirsfirst选项让目录和文件分开排序**
        
        ~> tree -d /proc/self/
        /proc/self/
        |-- attr
        |-- cwd -> /proc
        |-- fd
        |   `-- 3 -> /proc/15589/fd
        |-- fdinfo
        |-- net
        |   |-- dev_snmp6
        |   |-- netfilter
        |   |-- rpc
        |   |   |-- auth.rpcsec.context
        |   |   |-- auth.rpcsec.init
        |   |   |-- auth.unix.gid
        |   |   |-- auth.unix.ip
        |   |   |-- nfs4.idtoname
        |   |   |-- nfs4.nametoid
        |   |   |-- nfsd.export
        |   |   `-- nfsd.fh
        |   `-- stat
        |-- root -> /
        `-- task
            `-- 15589
                |-- attr
                |-- cwd -> /proc
                |-- fd
                | `-- 3 -> /proc/15589/task/15589/fd
                |-- fdinfo
                `-- root -> /

        27 directories
        
#参考
1. <http://linux.die.net/man/1/tree>
1. <http://en.wikipedia.org/wiki/Tree_(Unix)>
1. <http://yjplxq.blog.51cto.com/4081353/958888>