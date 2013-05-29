---
layout: post
title: 简单的日志接口
category: cpp
---

#C
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
        
        
#CPP

        #include <tchar.h>
        #include <Windows.h>
        #include <sstream>

        class Message
        {
        public:
            ~Message()
            {
                OutputDebugString(m_stream_.str().c_str());
            }

        #if defined _UNICODE
            std::wostringstream m_stream_;
        #else
            std::ostringstream m_stream_;
        #endif
        };

        #define LogError() \
            Message().m_stream_ << \
                _T("E") << \
                _T("|") << \
                _T(__FILE__) << \
                _T("|") << \
                __LINE__ << \
                _T("|")
        #define LogInfo() \
            Message().m_stream_ << \
                _T("I") << \
                _T("|") << \
                _T(__FILE__) << \
                _T("|") << \
                __LINE__ << \
                _T("|")

        int _tmain(int argc, _TCHAR* argv[])
        {
            LogError() << _T("fjasldfadf") << 12345;
            return 0;
        }

##其他
若以下写法编译不过，可以用后者替代

	WriteLog(_T("E|%s|%d|")##_T(VAR)##_T("\n"),_T(__FILE__),__LINE__,__VA_ARGS__)	
	WriteLog(_T("E|%s|%d|") _T(VAR) _T("\n"),_T(__FILE__),__LINE__,__VA_ARGS__)
	
即使用**空格**做取代连接符**##**

	#define CHECK(expr,ret) \
		do{ int ret_ = 0; \
			if((ret_ = (expr)) != ret) \
				printf("expr \"" #expr "\" run fail with ret '%d',expect '%d'\n",ret_,ret); \
		}while(0)
