---
layout: post
title:  xmlstarlet笔记
category: bash
---

##Help

    qjw@qjw-VirtualBox ~/xml $ xmlstarlet --help
    XMLStarlet Toolkit: Command line utilities for XML
    Usage: xmlstarlet [<options>] <command> [<cmd-options>]
    where <command> is one of:
    XMLStarlet Toolkit: Command line utilities for XML
    Usage: xmlstarlet [<options>] <command> [<cmd-options>]
    where <command> is one of:
      ed    (or edit)      - Edit/Update XML document(s)
      sel   (or select)    - Select data or query XML document(s) (XPATH, etc)
      tr    (or transform) - Transform XML document(s) using XSLT
      val   (or validate)  - Validate XML document(s) (well-formed/DTD/XSD/RelaxNG)
      fo    (or format)    - Format XML document(s)
      el    (or elements)  - Display element structure of XML document
      c14n  (or canonic)   - XML canonicalization
      ls    (or list)      - List directory as XML
      esc   (or escape)    - Escape special XML characters
      unesc (or unescape)  - Unescape special XML characters
      pyx   (or xmln)      - Convert XML into PYX format (based on ESIS - ISO 8879)
      p2x   (or depyx)     - Convert PYX into XML

    qjw@qjw-VirtualBox ~/xml $ xmlstarlet ls
    <dir>
    <f p="rw-rw-r--" a="20121125T074458Z" m="20121125T074458Z" s="0"    n="d"/>
    <d p="rwxrwxr-x" a="20121125T074502Z" m="20121125T074502Z" s="4096" n="f"/>
    </dir>

####安装
    #!/bin/bash
    apt-get install xmlstarlet


##校验
    #!/bin/bash
    # 使用xsd文件校验
    xmlstarlet val -s xml.xsd xml.xml
    
##查询
    <dir>
    <f p="rw-rw-r--" a="20121125T074458Z" m="20121125T074458Z" s="0"    n="d">d</f>
    <d p="rwxrwxr-x" a="20121125T074502Z" m="20121125T074502Z" s="4096" n="g">g</d>
    <f p="rw-rw-r--" a="20121125T074458Z" m="20121125T074458Z" s="0"    n="b">b</f>
    <f p="rw-rw-r--" a="20121125T074458Z" m="20121125T074458Z" s="0"    n="a">a</f>
    <f p="rw-rw-r--" a="20121125T074458Z" m="20121125T074458Z" s="0"    n="c">c</f>
    <d p="rwxrwxr-x" a="20121125T074502Z" m="20121125T074502Z" s="4096" n="h">h</d>
    <d p="rwxrwxr-x" a="20121125T074502Z" m="20121125T074502Z" s="4096" n="e">e</d>
    <f p="rw-rw-r--" a="20121125T074646Z" m="20121125T074646Z" s="0"    n="xml.xml">xml.xml</f>
    <d p="rwxrwxr-x" a="20121125T074502Z" m="20121125T074502Z" s="4096" n="f">f</d>
    </dir>

####Sample（结果有删减）
    #!/bin/bash
    # 统计节点"/dir/d"的数量
    xmlstarlet sel -t -v "count(/dir/d)" xml.xml
    
    # 枚举节点"/dir/f"的所有"xml格式"的内容
    # 其中"-n"表示在结果后面插入一个换行，否则所有内容都跑到一行
    xmlstarlet sel -t -m "/dir/f" -c . -n xml.xml
    #<f p="rw-rw-r--" a="20121125T074458Z" m="20121125T074458Z" s="0" n="d">d</f>
    #<f p="rw-rw-r--" a="20121125T074646Z" m="20121125T074646Z" s="0" n="xml.xml">xml.xml</f>
    # 枚举节点"/dir/f"的第二条记录的"xml格式"的内容
    xmlstarlet sel -t -m "/dir/f[2]" -c . -n xml.xml
    #<f p="rw-rw-r--" a="20121125T074458Z" m="20121125T074458Z" s="0" n="b">b</f>

    # 枚举节点"/dir/f"的所有value
    xmlstarlet sel -t -m "/dir/f" -v . -n xml.xml
    #d
    #xml.xml
    
    # 枚举所有节点"/dir/d"的"a"属性的"值"
    xmlstarlet sel -t -m "/dir/d" -v "@a" -n xml.xml
    #20121125T074502Z
    #20121125T074502Z
    
    # 枚举节点"/dir/d"的所有节点中"属性n"为"g"的节点的p属性值
    xmlstarlet sel -t -m "/dir/d[@n='g']" -v "@p" xml.xml
    #rwxrwxr-x
    # 枚举节点"/dir/d"的所有节点中"属性n"为"g"的节点的value
    xmlstarlet sel -t -m "/dir/d[@n='g']" -v . xml.xml
    #g
    xmlstarlet sel -t -m "/dir/f[.='xml.xml']" -v @a xml.xml
    #20121125T074646Z

####拼装
    <xml>
      <table>
        <rec id="1">
          <numField>123</numField>
          <stringField>String Value</stringField>
        </rec>
        <rec id="2">
          <numField>346</numField>
          <stringField>Text Value</stringField>
        </rec>
        <rec id="3">
          <numField>-23</numField>
          <stringField>stringValue</stringField>
        </rec>
      </table>
    </xml>
---
    #!/bin/bash
    xmlstarlet sel -T -t -m /xml/table/rec -v "concat(@id,'|',numField,'|',stringField)" -n a.xml 
    #1|123|String Value
    #2|346|Text Value
    #3|-23|stringValue
    
    # 排序
    xmlstarlet sel -T -t -m /xml/table/rec -s D:N:- "@id" -v "concat(@id,'|',numField,'|',stringField)" -n a.xml 
    #3|-23|stringValue
    #2|346|Text Value
    #1|123|String Value
    
##参考
1. <http://www.ibm.com/developerworks/cn/xml/x-starlet.html>
1. <http://xmlstar.sourceforge.net/doc/UG/xmlstarlet-ug.html>
1. <http://www.w3schools.com/xpath/xpath_syntax.asp>