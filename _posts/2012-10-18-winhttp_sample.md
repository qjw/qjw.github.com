---
layout: post
title: Winhttp练笔
category: network
---


        #define USE_WINHTTP

        #include <cassert>
        #include <string>
        #include <sstream>
        #include <tchar.h>
        #ifdef USE_WINHTTP
        #include <windows.h>
        #include <winhttp.h>
        #pragma comment(lib, "winhttp.lib")
        #else
        #include <wininet.h>
        #pragma comment(lib, "wininet.lib")
        #endif

        static void FormatWinHttpMessage(DWORD errCode,std::wstring* str){
            HMODULE hModule = ::GetModuleHandleW(L"winhttp.dll");
            if(hModule){
                LPTSTR pBuffer = NULL;
                DWORD systemLocale = MAKELANGID(LANG_NEUTRAL, SUBLANG_NEUTRAL);
                if(!!FormatMessageW(FORMAT_MESSAGE_ALLOCATE_BUFFER |
                    FORMAT_MESSAGE_IGNORE_INSERTS |
                    FORMAT_MESSAGE_FROM_HMODULE,
                    hModule,
                    errCode,
                    systemLocale,
                    (LPTSTR)&pBuffer,
                    0,
                    NULL))
                {
                    if(str)
                        *str = pBuffer;
                    LocalFree(pBuffer);
                }
            }
        }

        class WinHttpConnection;
        class WinHttpRequest;
        class WinHttpSession;
        class WinHttpHandle
        {
        public:
            explicit WinHttpHandle() :
                m_handle_(0)
            {}

            virtual ~WinHttpHandle()
            {
                this->Close();
            }

            bool Attach(HINTERNET handle)
            {
                assert(NULL == m_handle_);
                m_handle_ = handle;
                return NULL != m_handle_;
            }

            HINTERNET Detach()
            {
                HANDLE handle = m_handle_;
                m_handle_ = NULL;
                return handle;
            }

            void Close()
            {
                if (NULL != m_handle_)
                {
                    if(!::WinHttpCloseHandle(m_handle_))
                    {
                        assert(0);
                        ;
                    }
                    m_handle_ = NULL;
                }
            }

            DWORD SetOption(DWORD option,
                              const void* value,
                              DWORD length)
            {
                assert(NULL != m_handle_);
                if (!!::WinHttpSetOption(m_handle_,
                                        option,
                                        const_cast<void*>(value),
                                        length))
                {
                    return 0;
                }
                return ::GetLastError();
            }

            HRESULT QueryOption(DWORD option,
                                void* value,
                                DWORD& length) const
            {
                // WINHTTP_OPTION_URL 必须转换成wchar_t
                // 注意返回值 ERROR_NOT_ENOUGH_MEMORY
                assert(NULL != m_handle_);
                if (!!::WinHttpQueryOption(m_handle_,
                                          option,
                                          value,
                                          &length))
                {
                    return 0;
                }
                return ::GetLastError();
            }

            bool		Empty() const{
                return NULL == m_handle_;
            }
        protected:
            HINTERNET m_handle_;
        };

        class WinHttpSession : public WinHttpHandle
        {
            friend class WinHttpConnection;
        public:
            DWORD Init()
            {
                return InitEx(0,
                    WINHTTP_ACCESS_TYPE_DEFAULT_PROXY,
                    WINHTTP_NO_PROXY_NAME,
                    WINHTTP_NO_PROXY_BYPASS,
                    0);
            }

            
            DWORD InitEx(const wchar_t* usr_agent,
                DWORD access_type,
                const wchar_t* proxy_name,
                const wchar_t* proxy_pwd,
                DWORD flag)
            {
                if (Attach(::WinHttpOpen(usr_agent, // no agent string
                                          access_type,
                                          proxy_name,
                                          proxy_pwd,
                                          flag)))
                {
                    return 0;
                }
                return ::GetLastError();
            }
        };

        class WinHttpConnection : public WinHttpHandle
        {
            friend class WinHttpRequest;
        public:
            DWORD Init(const wchar_t* server_name,
                               INTERNET_PORT port,
                               WinHttpSession& session)
            {
                if (Attach(::WinHttpConnect(session.m_handle_,
                                             server_name,
                                             port,
                                             0))) // reserved
                {
                    return 0;
                }
                return ::GetLastError();
            }
        };


        class WinHttpRequest : public WinHttpHandle
        {
        public:
            DWORD Init(const wchar_t* verb,
                const wchar_t* path,
                WinHttpConnection& connection)
            {
                return InitEx(verb,
                    path,
                    0,
                    WINHTTP_NO_REFERER,
                    WINHTTP_DEFAULT_ACCEPT_TYPES,
                    0,
                    connection);
            }

            
            DWORD InitEx(const wchar_t* verb,
                const wchar_t* object_name,
                const wchar_t* version,
                const wchar_t* referrer,
                const wchar_t** accept_types,
                DWORD flag,
                WinHttpConnection& connection)
            {
                if (Attach(::WinHttpOpenRequest(connection.m_handle_,
                                             verb,
                                             object_name,
                                             version,
                                             referrer,
                                             accept_types,
                                             flag))) 
                {
                    return 0;
                }
                return ::GetLastError();
            }

            DWORD SendRequest(){
                assert(m_handle_);
                if(!!::WinHttpSendRequest(m_handle_,
                    WINHTTP_NO_ADDITIONAL_HEADERS,//WinHttpAddRequestHeaders
                    0, // headers length
                    WINHTTP_NO_REQUEST_DATA,
                    0, // request data length
                    0, // total length
                    0))
                {
                    return 0;
                }
                return ::GetLastError();
            }

            DWORD  ReceiveResponse(){
                assert(m_handle_);
                if(!!::WinHttpReceiveResponse(m_handle_,0))
                    return 0;
                else
                    return ::GetLastError();
            }

            // 注意：
            // WINHTTP_QUERY_STATUS_TEXT 返回unicode字符，需要做如下转换才能用
            //	wchar_t* ptr_ = reinterpret_cast<wchar_t*>(buf);
            // 注意返回值 ERROR_NOT_ENOUGH_MEMORY
            DWORD QueryHeaders(DWORD flag,
                char* buf,
                DWORD* buflen){
                assert(m_handle_);
                assert(buf);
                assert(buflen);
                if(!!::WinHttpQueryHeaders(m_handle_,
                    flag,
                    WINHTTP_HEADER_NAME_BY_INDEX,
                    reinterpret_cast<LPVOID>(buf),
                    buflen,
                    WINHTTP_NO_HEADER_INDEX))
                {
                    return 0;
                }
                return ::GetLastError();
            }
            // 查询一个DWORD数据
            DWORD QueryDWordHeaders(DWORD flag,
                DWORD* value){
                assert(value);
                DWORD len_ = sizeof(*value);
                return QueryHeaders(flag | WINHTTP_QUERY_FLAG_NUMBER,
                    reinterpret_cast<char*>(value),
                    &len_);
            }
            // 获得http code
            DWORD QueryHttpCode(DWORD* value){
                return QueryDWordHeaders(WINHTTP_QUERY_STATUS_CODE,value);
            }
            // 获得content 长度
            DWORD QueryContentLength(DWORD* value){
                return QueryDWordHeaders(WINHTTP_QUERY_CONTENT_LENGTH,value);
            }

            //WinHttpAddRequestHeaders

            DWORD QueryDataAvailable(DWORD* value){
                assert(m_handle_);
                assert(value);
                if(!!::WinHttpQueryDataAvailable(m_handle_,value)){
                    return 0;
                }
                return ::GetLastError();
            }

            DWORD ReadData(char* buf,
                DWORD buflen,
                DWORD* readed){
                assert(m_handle_);
                assert(buf);
                assert(buflen);
                if(!!::WinHttpReadData(m_handle_,
                    buf,
                    buflen,
                    readed)){
                    return 0;
                }
                return ::GetLastError();
            }

            DWORD AddRequestHeaders(const wchar_t* headers,
                DWORD length,
                DWORD modifiers){
                assert(m_handle_);
                assert(headers);
                if(!!::WinHttpAddRequestHeaders(m_handle_,
                    headers,
                    length,
                    modifiers)){
                    return 0;
                }
                return ::GetLastError();
            }
            DWORD		AddRangeHeader(DWORD offset){
                std::wostringstream strbuf_;
                strbuf_ << L"Range:bytes=" << offset << L"-";
                std::wstring header_ = strbuf_.str();
                return this->AddRequestHeaders(header_.c_str(),
                    -1,
                    WINHTTP_ADDREQ_FLAG_ADD | WINHTTP_ADDREQ_FLAG_REPLACE);
            }

            static DWORD CrackUrl(URL_COMPONENTS* url_comp,const wchar_t* url){
                assert(url_comp);
                assert(url);

                ZeroMemory(url_comp, sizeof(*url_comp));
                url_comp->dwStructSize = sizeof(*url_comp);
                url_comp->dwSchemeLength    = (DWORD)-1;
                url_comp->dwHostNameLength  = (DWORD)-1;
                url_comp->dwUrlPathLength   = (DWORD)-1;
                url_comp->dwExtraInfoLength = (DWORD)-1;

                if (!!::WinHttpCrackUrl(url,(DWORD)wcslen(url), 0, url_comp)){
                    return 0;
                }
                return ::GetLastError();
            }
        };

        class Downloader {
        private:
            enum{DldBuflen = 4096};
        public:
            DWORD InitFromUrl(const wchar_t* url){
                URL_COMPONENTS url_comp_;
                DWORD ret_ = 0;
                ret_ = WinHttpRequest::CrackUrl(&url_comp_,url);
                if(!!ret_)
                    return ret_;
                std::wstring host_(url_comp_.lpszHostName);
                std::wstring path_(url_comp_.lpszUrlPath);
                host_ = host_.substr(0,host_.size() - path_.size());

                ret_ = Init(host_.c_str(),url_comp_.nPort);
                if(!!ret_){
                    return ret_;
                }
                m_path_ = path_;
                return 0;
            }

            DWORD Init(const wchar_t* host,
                INTERNET_PORT port){
                assert(m_session_.Empty());
                assert(m_connection_.Empty());
                DWORD ret_ = m_session_.Init();
                if(!!ret_){
                    return ret_;
                }
                ret_ = m_connection_.Init(host,port,m_session_);
                if(!!ret_){
                    m_session_.Close();
                    return ret_;
                }
                return 0;
            }

            void		Close(){
                m_session_.Close();
                m_connection_.Close();
            }

            DWORD		DownloadEx(const wchar_t* path,DWORD offset){
                assert(!m_session_.Empty());
                assert(!m_connection_.Empty());
                assert(path || !m_path_.empty());
                if(!path)
                    path = m_path_.c_str();

                WinHttpRequest request_;
                DWORD ret_ = request_.Init(L"get",path,m_connection_);
                if(!!ret_)
                    return ret_;

                if(offset > 0)
                {
                    ret_ = request_.AddRangeHeader(offset);
                    if(!!ret_)
                        return ret_;
                }

                ret_ = request_.SendRequest();
                if(!!ret_)
                    return ret_;
                ret_ = request_.ReceiveResponse();
                if(!!ret_)
                    return ret_;

                // 判断http code
                DWORD http_code_;
                ret_ = request_.QueryHttpCode(&http_code_);
                if(!!ret_)
                    return ret_;
                if(HTTP_STATUS_OK != http_code_ &&
                    HTTP_STATUS_PARTIAL_CONTENT != http_code_)
                    return 0;

                return DowndloadData(request_);
            }
            DWORD		Download(const wchar_t* path){
                return DownloadEx(path,0);
            }

        private:
            DWORD		DowndloadData(WinHttpRequest& request){

                char buf_[DldBuflen];
                DWORD available_,ret_,readed_,total_;
                total_ = 0;
                do{
                    ret_ = request.QueryDataAvailable(&available_);
                    if(!!ret_)
                        return false;
                    if(!available_)
                        break;
                    ret_ = request.ReadData(buf_,sizeof(buf_),&readed_);
                    if(!!ret_)
                        return false;
                    total_ += readed_;
                }while(true);
                return 0;
            }
        private:
            WinHttpSession		m_session_;
            WinHttpConnection	m_connection_;
            std::wstring		m_path_;
        };


        #endif


        #include "myhttp.h"
        #include <iostream>
        #include <cstdio>

        int main(){
            /*
            WinHttpSession session_;
            assert(!session_.Init());

            //http://ttplayer.qianqian.com/download/ttpsetup_593-44059078.exe
            //http://msdn.microsoft.com/en-us/library/windows/desktop/aa384070(v=vs.85).aspx
            WinHttpConnection connection_;
            assert(!connection_.Init(L"msdn.microsoft.com",80,session_));

            WinHttpRequest request_;
            assert(!request_.Init(
                L"get",
                L"/en-us/library/windows/desktop/aa384070(v=vs.85).aspx",
                connection_));
            assert(!request_.SendRequest());
            assert(!request_.ReceiveResponse());
            DWORD code_;
            assert(!request_.QueryHttpCode(&code_));
            assert(!request_.QueryContentLength(&code_));

            char buf[1024];
            DWORD buflen = 1024;
            DWORD ret_ = request_.QueryHeaders(WINHTTP_QUERY_HOST,buf,&buflen);
            wchar_t* ptr_ = reinterpret_cast<wchar_t*>(buf);

            buflen = 1024;
            ret_ = request_.QueryOption(WINHTTP_OPTION_URL,buf,buflen);

            std::wstring errstr_;
            FormatWinHttpMessage(ret_,&errstr_);
            */
            // 6223320

            //http://169.254.161.130/hfs.exe


            Downloader download_;
            download_.InitFromUrl(L"http://169.254.161.130/hfs.exe");
            //assert(!download_.Init(L"169.254.161.130",80));
            assert(!download_.Download(NULL));
            return 0;
        }

