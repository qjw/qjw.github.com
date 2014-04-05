---
layout: post
title: Lua操作二进制
category: lua
---

#下载Lunary
* <http://lunary.luaforge.net/>
* <http://lunary.luaforge.net/index.html#download>
* <http://hg.piratery.net/lunary/archive/tip.zip>

Lunary压缩包里面包含**serial.lua**，以及目录**serial**，目录下包含**optim.c**，**util.lua**（前者包含一些优化功能，一般可不用）
**serial.lua**依赖**bit**和**strcut**

#Lua BitOp
* <http://bitop.luajit.org/>
* <http://bitop.luajit.org/download.html>
* <http://bitop.luajit.org/download/LuaBitOp-1.0.2.zip>

这个库没有直接的DLL或者Lua文件可以直接用，需要执行编译，具体可以看压缩包里面的doc/install.html

##在Windows下编译
1. Open a "Visual Studio .NET Command Prompt"
2. change to the directory where msvcbuild.bat resides and run it:msvcbuild

不过在运行之前，有可能因为找不到Lua的头文件而失败，直接编辑这个Bat脚本
        @setlocal
        @rem Path to the Lua includes and the library file for the Lua DLL:
        @set LUA_INC=-I "C:\Program Files (x86)\Lua\5.1\include"
        @set LUA_LIB="C:\Program Files (x86)\Lua\5.1\lib\lua5.1.lib"
	
成功编译之后会生成bit.dll

#Lua Strcut
1. <http://www.inf.puc-rio.br/~roberto/struct/>
1. <http://www.inf.puc-rio.br/~roberto/struct/struct-0.2.tar.gz>

下载之后，就只有一个struct.c文件，拷贝BitOp的msvcbuild.bat，修改如下：

        @rem Script to build Lua BitOp with MSVC.
        
        @rem First change the paths to your Lua installation below.
        @rem Then open a "Visual Studio .NET Command Prompt", cd to this directory
        @rem and run this script. Afterwards copy the resulting bit.dll to
        @rem the directory where lua.exe is installed.
        
        @if not defined INCLUDE goto :FAIL
        
        @setlocal
        @rem Path to the Lua includes and the library file for the Lua DLL:
        @set LUA_INC=-I "C:/Program Files (x86)/Lua/5.1/include"
        @set LUA_LIB= "C:\Program Files (x86)/Lua/5.1/lib/lua51.lib"
        
        @set MYCOMPILE=cl /nologo /MD /O2 /W3 /c %LUA_INC%
        @set MYLINK=link /nologo
        @set MYMT=mt /nologo
        
        %MYCOMPILE% struct.c
        %MYLINK% /DLL /export:luaopen_struct /out:struct.dll struct.obj %LUA_LIB%
        if exist struct.dll.manifest^
          %MYMT% -manifest struct.dll.manifest -outputresource:struct.dll;2
        
        del *.obj *.exp *.manifest
        
        @goto :END
        :FAIL
        @echo You must open a "Visual Studio .NET Command Prompt" to run this script
        :END

编译之后会生成struct.dll


#Sample

        require 'serial'
        out=io.open("text.dat","wb")

            -- 0-255
            local uint8 = serial.serialize.uint8(0xab)
            out:write(uint8)
            -- -127-127
            local int8 = serial.serialize.sint8(0x12)
            out:write(int8)
        
            local uint16 = serial.serialize.uint16(0x1234,"le")
            out:write(uint16)
            local sint16 = serial.serialize.sint16(0x1234,"le")
            out:write(sint16)
        
            local uint32 = serial.serialize.uint32(0x12345678,"le")
            out:write(uint32)
            local sint32 = serial.serialize.sint32(0x12345678,"le")
            out:write(sint32)
        
            local struct_desc = {
                {'name', 'cstring'},
                {'value', 'uint32', 'le'},
            }
        
            local object = {
                name = "foo",
                value = 42,
            }
            local sobject = serial.serialize.struct(object,struct_desc)
            out:write(sobject)
        
        out:close()


