---
layout: post
title: 修改Windows线程的名称
category: windows
---


        #pragma once

        #define MS_VC_EXCEPTION 0x406d1388
        #pragma warning(disable: 6312)
        #pragma warning(disable: 6322)

        typedef struct tagTHREADNAME_INFO
        {
            DWORD dwType;        // must be 0x1000
            LPCSTR szName;       // pointer to name (in same addr space)
            DWORD dwThreadID;    // thread ID (-1 caller thread)
            DWORD dwFlags;       // reserved for future use, most be zero
        } THREADNAME_INFO;

        inline
        void SetThreadName(DWORD dwThreadID, LPCSTR szThreadName)
        {
        #ifdef _DEBUG
            THREADNAME_INFO info;
            info.dwType = 0x1000;
            info.szName = szThreadName;
            info.dwThreadID = dwThreadID;
            info.dwFlags = 0;

            __try
            {
                RaiseException(MS_VC_EXCEPTION, 0, sizeof(info) / sizeof(DWORD), (DWORD *)&info);
            }
            __except (EXCEPTION_CONTINUE_EXECUTION)
            {
            }
        #else
            dwThreadID;
            szThreadName;
        #endif
        }

#参考
1. <http://stackoverflow.com/questions/478875/how-to-change-the-name-of-a-thread>
2. <http://msdn.microsoft.com/en-us/library/xcb2z8hs.aspx>

