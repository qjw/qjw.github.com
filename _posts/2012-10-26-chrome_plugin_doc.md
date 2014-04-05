---
layout: post
title: Chrome插件开发学习笔记
category: js
---

#[概述](http://open.chrome.360.cn/extension_dev/overview.html)
一个应用（扩展）其实是压缩在一起的一组文件，包括HTML，CSS，Javascript脚本，图片文件，还有其它任何需要的文件。应用（扩展）本质上来说就是web页面，它们可以使用所有的浏览器提供的API，从XMLHttpRequest到JSON到HTML5全都有。

应用（扩展）可以与Web页面交互，或者通过content script或cross-origin XMLHttpRequests与服务器交互。应用（扩展）还可以访问浏览器提供的内部功能，例如标签或书签等。

每个应用（扩展）都应该包含下面的文件：

1. 一个manifest文件
1. 一个或多个html文件（除非这个应用是一个皮肤）
1. 可选的一个或多个javascript文件
1. 可选的任何需要的其他文件，例如图片
1. 在开发应用（扩展）时，需要把这些文件都放到同一个目录下。发布应用（扩展）时，这个目录全部打包到一个应用（扩展）名是.crx的压缩文件中。如果使用Chrome Developer Dashboard,上传应用（扩展），可以自动生成.crx文件。

很多应用（不包括WebApp）会以[browser action](http://open.chrome.360.cn/extension_dev/browserAction.html)或[page action](http://open.chrome.360.cn/extension_dev/pageAction.html)的形式在浏览器界面上展现出来。每个应用（扩展）最多可以有一个browser action或page action。**当应用（扩展）的图标是否显示出来是取决于单个的页面时，应当选择page action**；当其它情况时可以选择browser action。

#[Manifest](http://open.chrome.360.cn/extension_dev/manifest.html)
每一个扩展、可安装的WebApp、皮肤，都有一个JSON格式的manifest文件，叫manifest.json，里面提供了重要的信息 。

##字段说明
下面的JSON示例了manifest支持的字段，每个字段都有连接指向专有的说明。必须的字段只有：name和version。

        {
            // 必须的字段
            "name": "My Extension",
            "version": "versionString",
            "manifest_version": 2,
            // 建议提供的字段
            "description": "A plain text description",
            "icons": { ... },
            "default_locale": "en",
            // 多选一，或者都不提供
            "browser_action": {...},
            "page_action": {...},
            "theme": {...},
            "app": {...},
            // 根据需要提供
            "background": {...},
            "chrome_url_overrides": {...},
            "content_scripts": [...],
            "content_security_policy": "policyString",
            "file_browser_handlers": [...],
            "homepage_url": "http://path/to/homepage",
            "incognito": "spanning" or "split",
            "intents": {...}
            "key": "publicKey",
            "minimum_chrome_version": "versionString",
            "nacl_modules": [...],
            "offline_enabled": true,
            "omnibox": { "keyword": "aString" },
            "options_page": "aFile.html",
            "permissions": [...],
            "plugins": [...],
            "requirements": {...},
            "update_url": "http://path/to/updateInfo.xml",
            "web_accessible_resources": [...]
        }  

#[BrowserAction](http://open.chrome.360.cn/extension_dev/browserAction.html)
用 browser actions 可以在chrome主工具条的地址栏右侧增加一个图标。作为这个图标的延展，一个browser action图标还可以有tooltip、badge和popup。**如果你想创建一个不总是可见的图标, 可以使用page action来代替browser action.**

##Manifest
在extension manifest中用下面的方式注册你的browser action:

        {
            "name": "My extension",
            //...
            "browser_action": {
                "default_icon": "images/icon19.png", // optional 
                "default_title": "Google Mail",      // optional; shown in tooltip 
                "default_popup": "popup.html"        // optional 
            },
            //...
        }

**browserAction可以弹出下拉菜单，或者在图标点击的时候触发一个事件**

#[PageActions](http://open.chrome.360.cn/extension_dev/pageAction.html)
使用page actions把图标放置到地址栏里。page actions定义需要处理的页面的事件，但是它们不是适用于所有页面的。**想让扩展图标总是可见，则使用browser action。**
        
##Manifest
在 extension manifest 中用下面的方式注册你的page action：

        {
            "name": "My extension",
            //...
            "page_action": {
                "default_icon": "icons/foo.png", // optional 
                "default_title": "Do action",    // optional; shown in tooltip 
                "default_popup": "popup.html"    // optional 
            },
            //...
        }
        
同browser actions一样，page actions 可以有图标、提示信息、 弹出窗口。但没有badge，也因此，作为辅助，page actions可以有显示和消失两种状态。阅读browser action UI. 可以找到图标、提示信息、 弹出窗口相关信息。

使用方法 show() 和 hide() 可以显示和隐藏page action。缺省情况下page action是隐藏的。当要显示时，需要指定图标所在的标签页，图标显示后会一直可见，直到该标签页关闭或开始显示不同的URL （如：用户点击了一个连接）

**PageActions可以弹出下拉菜单，或者在图标点击的时候触发一个事件。那么什么时候调用show()函数让这个隐藏的图标显示出来呢?见[Event](http://open.chrome.360.cn/extension_dev/events.html)**

#[BackgroundPage](http://open.chrome.360.cn/extension_dev/background_pages.html)
扩展常常用一个单独的长时间运行的脚本来管理一些任务或者状态。 Background pages to the rescue.

如同 architecture overview 的解释。背景页是一个运行在扩展进程中的HTML页面。它在你的扩展的整个生命周期都存在，同时，在同一时间只有一个实例处于活动状态。

在一个有背景页的典型扩展中，用户界面（比如，浏览器行为或者页面行为和任何选项页）是由沉默视图实现的。当视图需要一些状态，它从背景页获取该状态。当背景页发现了状态改变，它会通知视图进行更新。

##Manifest
请在扩展清单中注册背景页。一般，背景页不需要任何HTML，仅仅需要js文件，比如：
        {
            "name": "My extension",
            //...
            "background": {
                "scripts": ["background.js"]
            },
            //...
        }
        
浏览器的扩展系统会自动根据上面scripts字段指定的所有js文件自动生成背景页。如果您的确需要自己的背景页，可以使用page字段，比如：

        {
            "name": "My extension",
            //...
            "background": {
                "page": "background.html"
            },
            //...
        }

可以使用chrome.extension.getBackgroundPage()获得background page,

#[ContentScripts](http://open.chrome.360.cn/extension_dev/content_scripts.html)
Content scripts是在Web页面内运行的javascript脚本。通过使用标准的DOM，它们可以获取浏览器所访问页面的详细信息，并可以修改这些信息。

##Manifest
如果content scipt的代码总是需要注入，可以在extension manifest中的 content_scipt字段注册它。如下面的例子：

        {
            "name": "My extension",
            //...
            "content_scripts": [
            {
              "matches": ["http://www.google.com/*"],
              "css": ["mystyles.css"],
              "js": ["jquery.js", "myscript.js"]
            }
            ],
            //...
        }
        
如果只是在某些情况下需要注入，可以使用[permission](http://open.chrome.360.cn/extension_dev/manifest.html#permissions)字段，详见[Programmatic injection](http://open.chrome.360.cn/extension_dev/content_scripts.html#pi)。

        {
            "name": "My extension",
            //...
            "permissions": [
            "tabs", "http://www.google.com/*"
            ],
            //...
        }

##编程式注入
如果不需要将javascript 和css注入到每一个匹配的网页里面，可以通过程序来控制代码的注入。 例如， 可以只在用户点击了一个browser action图标后才注入脚本。

如果要将代码注入页面，扩展必须具有cross-origin 权限， 还必须可以使用chrome.tabs模块。 可以通过在manifest文件的permissions字段里声明来取得这些权限。

一旦设置好了权限，就可以通过调用[executeScript](http://open.chrome.360.cn/extension_dev/tabs.html#method-executeScript)()来注入javascript脚本。如果要注入css，可以调用[insertCSS](http://open.chrome.360.cn/extension_dev/tabs.html#method-insertCSS)()。

        /* in background.html */
        chrome.browserAction.onClicked.addListener(function(tab) {
          chrome.tabs.executeScript(null,
                                   {code:"document.body.bgColor='red'"});
        });

        /* in manifest.json */
        "permissions": [
          "tabs", "http://*/*"
        ],
        
当浏览器显示一个http网页并且用户点击了扩展的browser action按钮后，扩展会将页面的bgcolor属性设置为红色。 如果这个页面没有用css设置它的背景颜色的话， 它会变成红色。

一般来说，可以将代码放在文件里面而不是像上面那个例子那样直接注入。 可以这样写：

        chrome.tabs.executeScript(null, {file: "content_script.js"});
        
##执行环境
Content script是在一个特殊环境中运行的，这个环境成为isolated world（隔离环境）。它们可以访问所注入页面的DOM,但是不能访问里面的任何javascript变量和函数。 对每个content script来说，就像除了它自己之外再没有其它脚本在运行。 反过来也是成立的： 页面里的javascript也不能访问content script中的任何变量和函数。

例如，这个简单的页面：

        hello.html
        ==========
        <html>
          <button id="mybutton">click me</button>
          <script>
            var greeting = "hello, ";
            var button = document.getElementById("mybutton");
            button.person_name = "Bob";
            button.addEventListener(
                "click", 
                function() {
                    alert(greeting + button.person_name + ".");
                },
                false);
          </script>
        </html>

现在，将下面这个脚本注入hello.html：

        contentscript.js
        ================
        var greeting = "hola, ";
        var button = document.getElementById("mybutton");
        button.person_name = "Roberto";
        button.addEventListener(
            "click", 
            function() {
              alert(greeting + button.person_name + ".");
            }, 
            false);
            
然后，如果按下按钮， 可以同时看到两个问候。隔离环境使得content script可以修改它的javascript环境而不必担心会与这个页面上的其它content script冲突。 例如，一个content script可以包含JQuery v1而页面可以包含JQuery v2， 它们之间不会产生冲突。

另一个重要的优点是隔离环境可以将页面上的脚本与扩展中的脚本完全隔离开。这使得开发者可以在content script中提供更多的功能，而不让web页面利用它们。

#[LocalStorage](http://www.cnblogs.com/xiaowei0705/archive/2011/04/19/2021372.html)

首先自然是检测浏览器是否支持本地存储。在HTML5中，本地存储是一个window的属性，包括localStorage和sessionStorage，从名字应该可以很清楚的辨认二者的区别，前者是一直存在本地的，后者只是伴随着session，窗口一旦关闭就没了。二者用法完全相同，这里以localStorage为例。

        if(window.localStorage){
            alert('This browser supports localStorage');
        }else{
            alert('This browser does NOT support localStorage');
        }

存储数据的方法就是直接给window.localStorage添加一个属性，例如：window.localStorage.a 或者 window.localStorage["a"]。它的读取、写、删除操作方法很简单，是以键值对的方式存在的，如下：

        localStorage.a = 3;//设置a为"3"
        localStorage["a"] = "sfsf";//设置a为"sfsf"，覆盖上面的值
        localStorage.setItem("b","isaac");//设置b为"isaac"
        var a1 = localStorage["a"];//获取a的值
        var a2 = localStorage.a;//获取a的值
        var b = localStorage.getItem("b");//获取b的值
        localStorage.removeItem("c");//清除c的值

#[选项页](http://open.chrome.360.cn/extension_dev/options.html)
##manifest

        {
          "name": "My extension",
          //...
          "options_page": "options.html",
          //...
        }

##打开选项页
浏览器地址栏输入"chrome://chrome/extensions/"，然后每一个插件都包含一个"选项"的超链接，点击即可打开该插件的选项页面

        function help(){
            window.open("help.html","_self");
        }

        function option(){
            window.open("options.html");
            window.close();
        }
        
        
##传递信息
1. 实现配置好的信息，可以存储到localStorage，然后在background page里面读取
1. 动态配置的，可以先getBackgroundPage获得background page的实例，然后直接调用它内部的js函数更新即可。

#载入
1. 访问如下URL进入扩展管理页面: **chrome://chrome/extensions/**
1. 如果开发人员模式边上有 + , 点击 + 以展开**开发人员模式**。
1. 点击**载入正在开发的扩展程序** 选择那个安装目录即可。

#消息
## [向某个tab发送消息（content script）](http://developer.chrome.com/extensions/tabs.html)
Sends a single message to the content script(s) in the specified tab, with an optional callback to run when a response is sent back. The [chrome.extension.onMessage](http://developer.chrome.com/extensions/extension.html#event-onMessage) event is fired in each content script running in the specified tab for the current extension.

        chrome.tabs.sendMessage(integer tabId, any message, function responseCallback)
        // Sample
        chrome.pageAction.onClicked.addListener(function(tab) {
            chrome.tabs.getSelected(null, function(tab){
                chrome.tabs.executeScript(tab.id, {file: "js/jquery-1.7.1.js"},function(){
                    chrome.tabs.executeScript(tab.id, {file: "insert.js"},function(){
                        chrome.tabs.sendMessage(tab.id,{request:"fuck"});
                    });
                });
            })
        }); 
        
为了让content script能够收到消息，需要在为content script注入js时，注册一个回调函数，当收到消息时回调会自动触发

Fired when a message is sent from either an extension process or a content script.
        
        chrome.extension.onMessage.addListener(function(any message, MessageSender sender, function sendResponse) {...});
        // Sample
        chrome.extension.onMessage.addListener(function(request, sender, sendResponse){
            $("title").text(request.request);
        }); 
        
##向其他模块发送消息
Sends a single message to other listeners within the extension. Similar to chrome.extension.connect, but only sends a single message with an optional response. The chrome.extension.onMessage event is fired in each extension page of the extension. Note that extensions cannot send messages to content scripts using this method. To send messages to content scripts, use chrome.tabs.sendMessage().

        chrome.extension.sendMessage(string extensionId, any message, function responseCallback)
        
同样是需要注册chrome.extension.onMessage来接收消息。

##Connect
Connects to the content script(s) in the specified tab. The chrome.extension.onConnect event is fired in each content script running in the specified tab for the current extension. 

        extension.Port chrome.tabs.connect(integer tabId, object connectInfo)
        
这个函数会返回一个extension.Port对象
        
对方需要注册消息

Fired when a connection is made from either an extension process or a content script.

        chrome.extension.onConnect.addListener(function(Port port) {...});
        
这个回调包含一个参数，参数同样是extension.Port对象
        
当连接建立好之后，就可以利用这个Port对象进行**长连接**的通信了，类似于TCP，而sendMessage类似于UDP。

        Port
            ( object )
                An object which allows two way communication with other pages.
        Properties of Port
            postMessage ( function )
            sender ( optional MessageSender )
                This property will only be present on ports passed to onConnect/onConnectExternal listeners.
            onDisconnect ( events.Event )
            onMessage ( events.Event )
            name ( string )
        
#参考
1. <http://code.google.com/chrome/extensions/overview.html>
1. <http://open.chrome.360.cn/extension_dev/overview.html>
1. <http://maotto.com/archives/548?1351262742>
1. <http://www.chromi.org/archives/13915>
1. <http://www.cnblogs.com/xiaowei0705/archive/2011/04/19/2021372.html>
1. <http://blog.csdn.net/dojotoolkit/article/details/6624972>
