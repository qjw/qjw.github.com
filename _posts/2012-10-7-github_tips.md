---
layout: default
title: 你好，世界 master
---
#搭建Github Page注意点


* 对于*username.github.com*的账户，那么新建名为*username.github.com*的Repository将自动转换为Page页面，并且支持*username.github.com*直接访问
* 对于上面的Repository，不要在_config.yml文件中设置baseurl属性
* 所有的文件建议用utf8编码，但是需要注意使用*无BOM格式*编码
* 对于其他名字的Repository，需要手动配置才能开启Page功能
* 对于其他名字的Repositor，必须明确的设置baseurl属性，路径为Repository的名称
* jekyll --server --auto 或者 jekyll --server --no-auto结合起来做本地调试