---
layout: post
title: Github/Bitbucket用ssh/pop/push
category: www
---

```
ssh-keygen -t rsa
ssh-add id_rsa
```

```
king@king:~$ ls ~/.ssh/
id_rsa  id_rsa.pub  known_hosts
```
	
打开github，右上角【setting】-【SSH and GPG keys】，点击右上角【NEW SSH keys】，将id_rsa.pub添加进去

打开Bitbucket，【Bitbucket设置】，左列的【安全】-【SSH密钥】，同样添加id_rsa.pub。

接下来就可以直接无密码clone，push，pop了，记得使用git开头的路径而不是http。

##参考
1. <http://blog.csdn.net/xsckernel/article/details/8563993>
