---
layout: post
title: Tolua++入门
category: lua
---

#源码组织
##头文件
1. tolua++.h

##tolua++.lib 或者 tolua++.dll
1. tolua_event.c
1. tolua_event.h
1. tolua_is.c
1. tolua_map.c
1. tolua_push.c
1. tolua_to.c
 
##tolua++.exe
1. tolua.c
1. toluabind.c
1. toluabind.h
1. toluabind_default.c
1. toluabind_default.h

---
* 头文件需要被其他使用tolua++的程序引用
* lib/dll需要被其他使用tolua++的程序链接
* exe将pkg文件转换成c/c++文件
* 编译lib/dll/exe需要连接lua库和lua头文件

#导出Dll
dll中必须导出一个luaopen_$(库名称)的函数，例如下面的代码导出一个名为sd的库

        extern "C"{
        #include "lua.h"
        }
        #include "tolua++.h"
        ///////////////////////////////////////////////////////////////
        // 导出dll需要
        // 这个函数是Lua库的初始化函数，由tolua++自动生成
        TOLUA_API int tolua_sd_open (lua_State* tolua_S);

        extern "C" __declspec(dllexport) 
        int luaopen_sd(lua_State *tolua_S){
            return tolua_sd_open(tolua_S);
        }

#Exe初始化lua库
        extern "C"{
        #include "lua.h"
        #include "lualib.h"
        #include "lauxlib.h"
        }
        #include "tolua++.h"
        // 这个函数是Lua库的初始化函数，有tolua++自动生成
        TOLUA_API int tolua_sd_open (lua_State* tolua_S);

        int main(){
            lua_State* L = lua_open();
            if(!L)
                return 1;
            luaL_openlibs(L);

            // 在这里初始化 需要导出的库
            if(!tolua_sd_open(L))
                return 1;

            // 在C/C++ 内部调用lua脚本
            luaL_dofile(L,"sd.lua");
            lua_close(L);

            return 0;
        }

#参考
* [tolua++参考手册(翻译一)tolua++使用](http://blog.csdn.net/obsona/article/details/3478518)
* [tolua++的使用](http://blog.csdn.net/summerhust/article/details/6451135)
* [面向对象的Lua编程(三) ---- tolua++使用入门](http://bbs.ogdev.net/TopicContent.aspx?BoardID=2&TopicID=1594)


