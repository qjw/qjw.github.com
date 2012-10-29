---
layout: post
title: 简单的日志接口
category: cpp
---

        #include <tchar.h>
        #include <Windows.h>
        #include <stdarg.h>
        #include <cstring>

        static void WriteLog(const TCHAR * format, ...)
        {
            TCHAR buf_[4096];
            va_list arg_ptr;
            va_start(arg_ptr, format);
            if(_vsntprintf(buf_,
                sizeof(buf_)/sizeof(buf_[0]),
                format,
                arg_ptr) > 0)
            {
                OutputDebugString(buf_);
            }
            va_end(arg_ptr);
        }

        #define LogError(VAR,...) \
            WriteLog(_T("E|%s|%d|")##_T(VAR)##_T("\n"),_T(__FILE__),__LINE__,__VA_ARGS__)
        #define LogInfo(VAR,...) \
            WriteLog(_T("I|%s|%d|")##_T(VAR)##_T("\n"),_T(__FILE__),__LINE__,__VA_ARGS__)
        #define LogDebug(VAR,...) \
            WriteLog(_T("D|%s|%d|")##_T(VAR)##_T("\n"),_T(__FILE__),__LINE__,__VA_ARGS__)

        // 第一个参数不用_T()，或者L
        int _tmain(int argc, _TCHAR* argv[])
        {
            LogError("fjasldfadf");
            LogError("fjasldfadf %s %d",_T("fadfad"),1234);
            return 0;
        }