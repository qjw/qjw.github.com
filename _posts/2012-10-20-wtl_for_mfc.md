---
layout: post
title: WTL匹配MFC的接口
category: wtl
---

#资源接口
MFC里面有一个函数AfxGetInstanceHandle() 来获得具体的资源实例

WTL可以使用

        CComModule::GetModuleInstance()

具体是
        CAppModule _Module;
        _Module.GetModuleInstance()


