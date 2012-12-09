---
layout: post
title:  获得wget的进度
category: bash
---

        #!/bin/bash

        arg="arg"
        wget --progress=dot "$1" 2>&1 | \
                grep --line-buffered "%" |\
                sed -u -e "s,\.,,g" |\
                sed -u -e "s,\%,,g" |\
                awk '{printf("'"${arg}"'%5s\n",$2) | "xargs -i ./a.out {} \n" }'
                
        # 上面的方案会导致arg和progress作为一个参数存在
        wget --progress=dot "$1" 2>&1 | \
                grep --line-buffered "%" |\
                sed -u -e "s,\.,,g" |\
                sed -u -e "s,\%,,g" |\
                awk '{printf($2) | "xargs -i ./a.out '"${app_id_}"' {} \n" }'
                
---    

        #include <iostream>

        int main(int argc,const char** argv)
        {
                if(argc > 1)
                {
                        std::cout << "the arg is :" << argv[1] << std::endl;
                }
                return 0;
        }
        
---
        qjw@qjw-VirtualBox ~/sh $ ./test.sh $url 
        the arg is :arg   10
        the arg is :arg   21
        the arg is :arg   31
        the arg is :arg   42
        the arg is :arg   53
        the arg is :arg   63
        the arg is :arg   74
        the arg is :arg   84
        the arg is :arg   95
        the arg is :arg  100
        
**这里发现一个问题，管道经过awk之后，数据就丢了，所以那些命令放在awk里面**

#参考
1. <http://stackoverflow.com/questions/4686464/howto-show-wget-progress-bar-only>
1. <http://imtx.me/archives/436.html>
1. <http://leeon.me/upload/other/awk.html>
1. <http://bbs.chinaunix.net/thread-1919867-1-1.html>