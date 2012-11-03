---
layout: post
title: Windows 枚举进程
category: windows
---

###枚举进程
EnumProcesses枚举所有进程，获得进程**ID**（不是句柄）

        BOOL WINAPI EnumProcesses(
          _Out_  DWORD *pProcessIds,
          _In_   DWORD cb,//注意是直接，而不是元素数量
          _Out_  DWORD *pBytesReturned
        );

###从ID到HANDLE
OpenProcess通过ID获得句柄

        HANDLE WINAPI OpenProcess(
          _In_  DWORD dwDesiredAccess,
          _In_  BOOL bInheritHandle,
          _In_  DWORD dwProcessId
        );
        
**使用[GetProcessId](http://msdn.microsoft.com/zh-cn/library/windows/desktop/ms683215\(v=vs.85\).aspx)反转换**

###通过进程句柄获得模块句柄

        BOOL WINAPI EnumProcessModules(
          _In_   HANDLE hProcess,
          _Out_  HMODULE *lphModule, // 若不足也不会失败
          _In_   DWORD cb,
          _Out_  LPDWORD lpcbNeeded //注意是直接，而不是元素数量
        );
        
###[获得模块信息](http://msdn.microsoft.com/zh-cn/library/windows/desktop/ms684232\(v=vs.85\).aspx)
1. GetModuleBaseName 获得模块名称
1. GetModuleFileNameEx/GetModuleFileName 获得完成路径
1. GetModuleInformation 获得其他信息

###[Sample](http://msdn.microsoft.com/zh-cn/library/windows/desktop/ms682623\(v=vs.85\).aspx)

        #include <windows.h>
        #include <stdio.h>
        #include <tchar.h>
        #include <psapi.h>

        // To ensure correct resolution of symbols, add Psapi.lib to TARGETLIBS
        // and compile with -DPSAPI_VERSION=1

        void PrintProcessNameAndID( DWORD processID )
        {
            TCHAR szProcessName[MAX_PATH] = TEXT("<unknown>");

            // Get a handle to the process.

            HANDLE hProcess = OpenProcess( PROCESS_QUERY_INFORMATION |
                                           PROCESS_VM_READ,
                                           FALSE, processID );

            // Get the process name.

            if (NULL != hProcess )
            {
                HMODULE hMod;
                DWORD cbNeeded;

                if ( EnumProcessModules( hProcess, &hMod, sizeof(hMod), 
                     &cbNeeded) )
                {
                    GetModuleBaseName( hProcess, hMod, szProcessName, 
                                       sizeof(szProcessName)/sizeof(TCHAR) );
                }
            }

            // Print the process name and identifier.

            _tprintf( TEXT("%s  (PID: %u)\n"), szProcessName, processID );

            // Release the handle to the process.

            CloseHandle( hProcess );
        }

        int main( void )
        {
            // Get the list of process identifiers.

            DWORD aProcesses[1024], cbNeeded, cProcesses;
            unsigned int i;

            if ( !EnumProcesses( aProcesses, sizeof(aProcesses), &cbNeeded ) )
            {
                return 1;
            }


            // Calculate how many process identifiers were returned.

            cProcesses = cbNeeded / sizeof(DWORD);

            // Print the name and process identifier for each process.

            for ( i = 0; i < cProcesses; i++ )
            {
                if( aProcesses[i] != 0 )
                {
                    PrintProcessNameAndID( aProcesses[i] );
                }
            }

            return 0;
        }

###获得程序图标
SHGetFileInfo通过一个文件路径得到一个SHFILEINFO结构，SHFILEINFO.hIcon就是图标

        DWORD_PTR SHGetFileInfo(
          _In_     LPCTSTR pszPath,
          DWORD dwFileAttributes,
          _Inout_  SHFILEINFO *psfi,
          UINT cbFileInfo,
          UINT uFlags
        );

        typedef struct _SHFILEINFO {
          HICON hIcon;
          int   iIcon;
          DWORD dwAttributes;
          TCHAR szDisplayName[MAX_PATH];
          TCHAR szTypeName[80];
        } SHFILEINFO;
        
###获得文件版本号
        DWORD WINAPI GetFileVersionInfoSize(
          _In_       LPCTSTR lptstrFilename,
          _Out_opt_  LPDWORD lpdwHandle
        );

        BOOL WINAPI GetFileVersionInfo(
          _In_        LPCTSTR lptstrFilename,
          _Reserved_  DWORD dwHandle,
          _In_        DWORD dwLen,
          _Out_       LPVOID lpData
        );
        
###[Sample](http://blog.csdn.net/lanmanck/article/details/3901590)

        CString GetFileVersion(char*  FileName)
        {   
            int   iVerInfoSize;
            char   *pBuf;
            CString   asVer="";
            VS_FIXEDFILEINFO   *pVsInfo;
            unsigned   int   iFileInfoSize   =   sizeof(   VS_FIXEDFILEINFO   );
            
            
            iVerInfoSize   =   GetFileVersionInfoSize(FileName,NULL); 
            
            if(iVerInfoSize!= 0)
            {   
                pBuf   =   new   char[iVerInfoSize];
                if(GetFileVersionInfo(FileName,0,   iVerInfoSize,   pBuf   )   )   
                {   
                    if(VerQueryValue(pBuf,   "//",(void   **)&pVsInfo,&iFileInfoSize))   
                    {   
                        asVer.Format("%d.%d.%d.%d",
                            HIWORD(pVsInfo->dwFileVersionMS),
                            LOWORD(pVsInfo->dwFileVersionMS),
                            HIWORD(pVsInfo->dwFileVersionLS),
                            LOWORD(pVsInfo->dwFileVersionLS));
                    }   
                }   
                delete   pBuf;   
            }   
            return   asVer;   
        }  

###CreateToolhelp32Snapshot
枚举所有线程不仅仅只能用EnumProcesses，CreateToolhelp32Snapshot也可以用于枚举所有进程，两者的区别见[这里](http://stackoverflow.com/questions/4021307/enumprocesses-vs-createtoolhelp32snapshot)

---

I think they are pretty much the same in terms of performance (and results) as they both call the same underlying NT API, though CreateToolhelp32Snapshot() may have a slight overhead as it creates a section object and copies all the information to it whereas EnumProcesses()/EnumProcessModules() works directly with user-supplied buffers. The difference is probably negligible in real world performance, though.

I slightly prefer EnumProcesses() as it is (IMO) a simpler API to use, but CreateToolhelp32Snapshot() returns more information if you need it. The only downside to EnumProcesses() is that you are supposed to call it in a loop as you may not have allocated a large enough buffer; CreateToolhelp32Snapshot() takes care of the buffer management for you. In practice I just allocate a buffer on the stack large enough to hold 1024 process ids or module handles; so far I have not come across a system where either of these limits was even remotely close to being reached. Of course we said the same thing about MAX_PATH not so long ago and now we are running into problems with that...

---

###[Sample](http://msdn.microsoft.com/zh-cn/library/windows/desktop/ms686701\(v=vs.85\).aspx)

        #include <windows.h>
        #include <tlhelp32.h>
        #include <tchar.h>

        //  Forward declarations:
        BOOL GetProcessList( );
        BOOL ListProcessModules( DWORD dwPID );
        BOOL ListProcessThreads( DWORD dwOwnerPID );
        void printError( TCHAR* msg );

        int main( void )
        {
            GetProcessList( );
            return 0;
        }

        BOOL GetProcessList( )
        {
            HANDLE hProcessSnap;
            HANDLE hProcess;
            PROCESSENTRY32 pe32;
            DWORD dwPriorityClass;

            // Take a snapshot of all processes in the system.
            hProcessSnap = CreateToolhelp32Snapshot( TH32CS_SNAPPROCESS, 0 );
            if( hProcessSnap == INVALID_HANDLE_VALUE )
            {
                printError( TEXT("CreateToolhelp32Snapshot (of processes)") );
                return( FALSE );
            }

            // Set the size of the structure before using it.
            pe32.dwSize = sizeof( PROCESSENTRY32 );

            // Retrieve information about the first process,
            // and exit if unsuccessful
            if( !Process32First( hProcessSnap, &pe32 ) )
            {
                printError( TEXT("Process32First") ); // show cause of failure
                CloseHandle( hProcessSnap );          // clean the snapshot object
                return( FALSE );
            }

            // Now walk the snapshot of processes, and
            // display information about each process in turn
            do
            {
                _tprintf( TEXT("\n\n=====================================================" ));
                _tprintf( TEXT("\nPROCESS NAME:  %s"), pe32.szExeFile );
                _tprintf( TEXT("\n-------------------------------------------------------" ));

                // Retrieve the priority class.
                dwPriorityClass = 0;
                hProcess = OpenProcess( PROCESS_ALL_ACCESS, FALSE, pe32.th32ProcessID );
                if( hProcess == NULL )
                    printError( TEXT("OpenProcess") );
                else
                {
                    dwPriorityClass = GetPriorityClass( hProcess );
                    if( !dwPriorityClass )
                        printError( TEXT("GetPriorityClass") );
                    CloseHandle( hProcess );
                }

                _tprintf( TEXT("\n  Process ID        = 0x%08X"), pe32.th32ProcessID );
                _tprintf( TEXT("\n  Thread count      = %d"),   pe32.cntThreads );
                _tprintf( TEXT("\n  Parent process ID = 0x%08X"), pe32.th32ParentProcessID );
                _tprintf( TEXT("\n  Priority base     = %d"), pe32.pcPriClassBase );
                if( dwPriorityClass )
                    _tprintf( TEXT("\n  Priority class    = %d"), dwPriorityClass );

                // List the modules and threads associated with this process
                ListProcessModules( pe32.th32ProcessID );
                ListProcessThreads( pe32.th32ProcessID );

            } while( Process32Next( hProcessSnap, &pe32 ) );

            CloseHandle( hProcessSnap );
            return( TRUE );
        }


        BOOL ListProcessModules( DWORD dwPID )
        {
            HANDLE hModuleSnap = INVALID_HANDLE_VALUE;
            MODULEENTRY32 me32;

            // Take a snapshot of all modules in the specified process.
            hModuleSnap = CreateToolhelp32Snapshot( TH32CS_SNAPMODULE, dwPID );
            if( hModuleSnap == INVALID_HANDLE_VALUE )
            {
                printError( TEXT("CreateToolhelp32Snapshot (of modules)") );
                return( FALSE );
            }

            // Set the size of the structure before using it.
            me32.dwSize = sizeof( MODULEENTRY32 );

            // Retrieve information about the first module,
            // and exit if unsuccessful
            if( !Module32First( hModuleSnap, &me32 ) )
            {
                printError( TEXT("Module32First") );  // show cause of failure
                CloseHandle( hModuleSnap );           // clean the snapshot object
                return( FALSE );
            }

            // Now walk the module list of the process,
            // and display information about each module
            do
            {
                _tprintf( TEXT("\n\n     MODULE NAME:     %s"),   me32.szModule );
                _tprintf( TEXT("\n     Executable     = %s"),     me32.szExePath );
                _tprintf( TEXT("\n     Process ID     = 0x%08X"),         me32.th32ProcessID );
                _tprintf( TEXT("\n     Ref count (g)  = 0x%04X"),     me32.GlblcntUsage );
                _tprintf( TEXT("\n     Ref count (p)  = 0x%04X"),     me32.ProccntUsage );
                _tprintf( TEXT("\n     Base address   = 0x%08X"), (DWORD) me32.modBaseAddr );
                _tprintf( TEXT("\n     Base size      = %d"),             me32.modBaseSize );

            } while( Module32Next( hModuleSnap, &me32 ) );

            CloseHandle( hModuleSnap );
            return( TRUE );
        }

        BOOL ListProcessThreads( DWORD dwOwnerPID ) 
        { 
            HANDLE hThreadSnap = INVALID_HANDLE_VALUE; 
            THREADENTRY32 te32; 

            // Take a snapshot of all running threads  
            hThreadSnap = CreateToolhelp32Snapshot( TH32CS_SNAPTHREAD, 0 ); 
            if( hThreadSnap == INVALID_HANDLE_VALUE ) 
                return( FALSE ); 

            // Fill in the size of the structure before using it. 
            te32.dwSize = sizeof(THREADENTRY32); 

            // Retrieve information about the first thread,
            // and exit if unsuccessful
            if( !Thread32First( hThreadSnap, &te32 ) ) 
            {
                printError( TEXT("Thread32First") ); // show cause of failure
                CloseHandle( hThreadSnap );          // clean the snapshot object
                return( FALSE );
            }

            // Now walk the thread list of the system,
            // and display information about each thread
            // associated with the specified process
            do 
            { 
                if( te32.th32OwnerProcessID == dwOwnerPID )
                {
                    _tprintf( TEXT("\n\n     THREAD ID      = 0x%08X"), te32.th32ThreadID ); 
                    _tprintf( TEXT("\n     Base priority  = %d"), te32.tpBasePri ); 
                    _tprintf( TEXT("\n     Delta priority = %d"), te32.tpDeltaPri ); 
                    _tprintf( TEXT("\n"));
                }
            } while( Thread32Next(hThreadSnap, &te32 ) ); 

            CloseHandle( hThreadSnap );
            return( TRUE );
        }

        void printError( TCHAR* msg )
        {
            DWORD eNum;
            TCHAR sysMsg[256];
            TCHAR* p;

            eNum = GetLastError( );
            FormatMessage( FORMAT_MESSAGE_FROM_SYSTEM | FORMAT_MESSAGE_IGNORE_INSERTS,
                    NULL, eNum,
                    MAKELANGID(LANG_NEUTRAL, SUBLANG_DEFAULT), // Default language
                    sysMsg, 256, NULL );

            // Trim the end of the line and terminate it with a null
            p = sysMsg;
            while( ( *p > 31 ) || ( *p == 9 ) )
                ++p;
            do { *p-- = 0; } while( ( p >= sysMsg ) &&
                    ( ( *p == '.' ) || ( *p < 33 ) ) );

            // Display the message
            _tprintf( TEXT("\n  WARNING: %s failed with error %d (%s)"), msg, eNum, sysMsg );
        }



###参考
1. <http://blog.csdn.net/handsomerun/article/details/1114365>
1. <http://blog.csdn.net/lanmanck/article/details/3901590>
1. <http://safe.zol.com.cn/2005/0427/167328.shtm>
1. <http://stackoverflow.com/questions/4021307/enumprocesses-vs-createtoolhelp32snapshot>