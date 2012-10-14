---
layout: post
title: google-code-prettify简介
category: jekyll
---


##官方网站：[google-code-prettify](http://code.google.com/p/google-code-prettify/)，[使用说明](http://google-code-prettify.googlecode.com/svn/trunk/README.html)
##详细说明
* 加载对应的js和css，例如
        <link href="prettify.css" type="text/css" rel="stylesheet" />
        <script type="text/javascript" src="prettify.js"></script>
* 将代码放置在一个pre结构中
        <pre class="prettyprint">您的代码</pre>
* google-code-prettify可以自动判断语言，不过你可以手动指定语言
        <pre class="prettyprint lang-html">
          The lang-* class specifies the language file extensions.
          File extensions supported by default include
            "bsh", "c", "cc", "cpp", "cs", "csh", "cyc", "cv", "htm", "html",
            "java", "js", "m", "mxml", "perl", "pl", "pm", "py", "rb", "sh",
            "xhtml", "xml", "xsl".
        </pre>
* 为了开启行号，可以用如下的方式
        <pre class="prettyprint linenums:4">// This is line 4.
            foo();
            bar();
            baz();
            boo();
            far();
            faz();
        <pre>
* 为了设定代码高亮的风格，可以修改prettify的css，具体参考[这里](http://google-code-prettify.googlecode.com/svn/trunk/styles/index.html)
* 为了方便，我将这些内容放到post.html里面
        <head>
            <link href="/js/google-code-prettify/prettify.css" type="text/css" rel="stylesheet"/>
            <script type="text/javascript" src="/js/jquery-1.7.1.min.js"></script>
            <script type="text/javascript" src="/js/google-code-prettify/prettify.js"></script>
            <script type="text/javascript">
            $(document).ready(function(){
                $('pre').addClass('prettyprint linenums:1') //添加Google code Hight需要的class
                
                // 导入Prettify的javascript
                prettyPrint()
            })
            </script>
        </head>
