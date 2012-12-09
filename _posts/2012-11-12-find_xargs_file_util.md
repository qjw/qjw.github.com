---
layout: post
title:  两个有意思的小脚本
category: bash
---

####批量改名
rename也可以实现，以下脚本将所有xml文件后缀改成bin

        find . -name "*.xml" | sed s/\.xml/\./g | xargs -i mv {}xml {}bin
        
        find . -name "*.bin" -exec echo {} \; -exec rename 's/\.bin$/\.xml/' {} \;
        
        find . -name "*.bin" | while read file;do rename 's/\.bin$/\.xml/' $file; echo $file; done;
####将所有文件改成同名目录

        find . -mindepth 1 -type f | xargs -i -t rm {} 2>&1 | sed s/^rm//g | xargs -i -t mkdir {}