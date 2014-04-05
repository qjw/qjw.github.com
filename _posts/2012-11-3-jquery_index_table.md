---
layout: post
title: jQuery获取table下的子节点误区
category: js
---

###参考: 
1. <http://akunamotata.iteye.com/blog/478437>
2. <http://www.css88.com/jqapi-1.7/>

**Html代码**

        <table>
          <tr>
            <td>111</td>
            <td>222</td>
            <td>333</td>
          </tr>
        </table>

**如果想获取第一个td，jQuery怎么写：**

        $('table>tr>td').eq(0).text();  
        
**结果一定为null，为什么呢？因为table下一个子节点是tbody。所以存在一个误区，table下的子节点是tr。可以怎么写：**

        $('table tr>td').eq(0).text();  
