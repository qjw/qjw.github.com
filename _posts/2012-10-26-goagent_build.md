---
layout: post
title: 使用GAE、GoAgent搭建代理搭建代理
category: other
---

#开通GAE

使用Google Account登陆<https://appengine.google.com/>。创建App，使用默认设置即可。

1. 点击**Create Application**按钮，若第一次使用，可能会要求手机验证，填写手机号码注意加86前缀
1. **Application Identifier** 选择你自定义的名称，注意后面检测合法性
1. **Application Title**也随便你怎么写，不过我个人基于维护方便，一般和**Application Identifier**保持一致


#下载[GoAgent](https://github.com/phus/goagent/downloads)
点击 “Download as zip”，解压到本地即可

我在[这里](http://ishare.iask.sina.com.cn/f/24152878.html)下载，版本：**goagent 1.8.4 稳定版**

#Google Account应用程序专用验证码
见 <http://support.google.com/accounts/bin/answer.py?hl=zh-Hans&answer=185833>


#配置GoAgent
##gae appid修改

goagent下载解压后，找到 local 目录下 proxy.ini

修改 [gae] 段 appid= *** 部分 id为你第一步创建的id。**注意等号后面的空格**

找到 server/python 目录下的 app.yaml

修改 application: *** 的值，id为你第一步创建的id。


##上传python主程序到gae

点击server/upload.bat

按照提示设置

**要注意的是这里使用的密码是 第三步中生成的应用程序专用验证码而不是Google Account登录密码**

上传成功后查看 <https://appengine.google.com/> 你刚才创建的app已经有了版本号


#GoAgent代理的使用

打开主程序 local\goagent.exe，浏览器配置代理ip: 127.0.0.1 端口：8087

为了方便浏览器切换，推荐安装[SwitchySharp](https://chrome.google.com/webstore/detail/proxy-switchysharp/dpplabbmogkhghncfbfdeeokoefdjegm)插件。

#其他问题

我下载的版本，local/proxy.ini默认配置**google_cn**。但是似乎无法连接，修改成google_hk就行了

        path = /fetch.py
        profile = google_cn
        #profile = google_hk
        mulconn = 1

        
#参考
1. <http://www.6zou.net/tech/how-to-gae_goagent_proxy.html>
1. <http://www.itguoke.com/goagent.html>
1. <http://www.flakegx5.info/?p=270012>
1. <http://ishare.iask.sina.com.cn/f/24152878.html>
