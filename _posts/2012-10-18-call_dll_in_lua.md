---
layout: post
title: Lua直接调用Dll中的函数
category: lua
---

#下载[Lua For Windows](http://luaforwindows.googlecode.com/files/LuaForWindows_v5.1.4-45.exe)

安装之后，目录"Lua\5.1\lua\"目录包含alien.lua和alien目录

或者直接下载[LuaRocks](http://alien.luaforge.net)

或者见

1. <http://luarocks.org/en/Download>
1. <http://luarocks.org/releases/>
1. <http://luarocks.org/releases/luarocks-2.0.tar.gz>

 LuaRocks is a pure Lua application with no library dependencies. Its current release depends on some helper tools, but you shouldn't worry about them, as they are shipped by default on most Unix systems: LuaRocks requires as a basic Unix toolchain, zip, unzip and either wget (default on most Linux systems) or curl (default on Mac OSX). For Windows, these helper tools are provided by the UnxUtils project; the all-in-one Windows package already includes them, as a convenience.


#调用Windows API实例
1. <http://lua.iteye.com/blog/451757>
1. <http://www.hakkaku.net/articles/20090615-459>

#Sample
        -- http://www.hakkaku.net/articles/20090615-459
        -- http://blog.csdn.net/kowity/article/details/7256376
        require "alien"
        
        MessageBox = alien.User32.MessageBoxA
        MessageBox:types{ret = 'long', abi = 'stdcall', 'long', 'string', 'string', 'long' }
        MessageBox(0, "itle for test","LUA call windows api", 0)
        
        ---------------------------------------------------------------
        
        local ExpandEnvironmentStrings = alien.Kernel32.ExpandEnvironmentStringsA
        ExpandEnvironmentStrings:types{ ret = "long", abi = 'stdcall', "string", "pointer", "long" }
        
        local buffer = alien.buffer(512)
        ExpandEnvironmentStrings("%USERPROFILE%", buffer, 512)
        print(tostring(buffer))
        
        ---------------------------------------------------------------
        
        EnumWindows = alien.user32.EnumWindows
        EnumWindows:types {"callback", "pointer", abi="stdcall"}
        
        GetClassName = alien.user32.GetClassNameA
        GetClassName:types {"long", "pointer", "int", abi="stdcall" }
        
        local buf = alien.buffer(512)
        
        local function enum_func(hwnd, p)
          GetClassName(hwnd, buf, 511)
          print (hwnd..":"..tostring(buf))
          return 1
        end
        
        local callback_func = alien.callback(
                enum_func,
                {"int", "pointer", abi="stdcall"})
        
        EnumWindows(callback_func, nil)



