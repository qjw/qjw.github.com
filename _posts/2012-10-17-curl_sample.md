---
layout: post
title: Curl小牛试刀
category: network
---

        #include <string>
        #include <cassert>
        #include <sstream>
        extern "C" {
        #include "curl/curl.h"
        }

        class CurlApp
        {
        public:
            typedef size_t (*CB)(char*, size_t, size_t, void*);

        public:
            CurlApp():
                m_curl_handle_(NULL){
            }
            virtual ~CurlApp(){
                this->Fini();
            }

            bool	Init(){
                if( !(m_curl_handle_ = ::curl_easy_init()))
                    return false;
                else
                    return true;
            }
            void	Fini(){
                if( m_curl_handle_){
                    ::curl_easy_cleanup(m_curl_handle_);
                    m_curl_handle_ = NULL;
                }
            }

            bool	Empty() const{
                return NULL == m_curl_handle_;
            }

            CURLcode SetAgent(const char* agent){
                assert(m_curl_handle_);
                assert(agent);
                return ::curl_easy_setopt(m_curl_handle_, CURLOPT_USERAGENT,agent);
            }

            CURLcode SetUrl(const char* url){
                assert(m_curl_handle_);
                assert(url);
                return ::curl_easy_setopt(m_curl_handle_,CURLOPT_URL,url);
            }

            // size_t function( char *ptr, size_t size, size_t nmemb, void *userdata);
            CURLcode SetWriteCB(CB cb,void* param){
                assert(m_curl_handle_);
                assert(cb);
                CURLcode ret_ = ::curl_easy_setopt(
                    m_curl_handle_, 
                    CURLOPT_WRITEFUNCTION,
                    cb);
                if(CURLE_OK != ret_)
                    return ret_;
                return ::curl_easy_setopt(m_curl_handle_, 
                    CURLOPT_WRITEDATA, 
                    param);
            }
            CURLcode SetHeaderCB(CB cb,void* param){
                assert(m_curl_handle_);
                assert(cb);
                CURLcode ret_ = ::curl_easy_setopt(
                    m_curl_handle_, 
                    CURLOPT_HEADERFUNCTION,
                    cb);
                if(CURLE_OK != ret_)
                    return ret_;
                return ::curl_easy_setopt(m_curl_handle_, 
                    CURLOPT_WRITEHEADER, 
                    param);
            }
            

            // 0 表示关闭，>0 表示允许跳转的次数
            CURLcode SetRedirect(unsigned int cnt){
                assert(m_curl_handle_);
                int enable_ = (cnt>0?1:0);
                CURLcode ret_ = ::curl_easy_setopt(m_curl_handle_,
                    CURLOPT_FOLLOWLOCATION,
                    enable_);
                if(CURLE_OK != ret_)
                    return ret_;

                if(cnt > 0){
                    return ::curl_easy_setopt(m_curl_handle_,
                        CURLOPT_MAXREDIRS,
                        cnt);
                }else{
                    return CURLE_OK;
                }
            }

            // 单位毫秒
            CURLcode SetConnTimeout(unsigned int timeout){
                assert(m_curl_handle_);
                return ::curl_easy_setopt(m_curl_handle_,
                    CURLOPT_CONNECTTIMEOUT_MS,
                    timeout);
            }

            // 发起请求
            CURLcode SendRequest(){
                assert(m_curl_handle_);
                return ::curl_easy_perform(m_curl_handle_);
            }

            CURLcode SetRange(size_t offset){
                assert(m_curl_handle_);
                std::ostringstream stream_;
                stream_ << offset << "-";
                return curl_easy_setopt(m_curl_handle_, 
                    CURLOPT_RANGE, stream_.str().c_str());
            }

            CURLcode GetHttpCode(long* code){
                assert(m_curl_handle_);
                assert(code);
                return curl_easy_getinfo(m_curl_handle_,CURLINFO_RESPONSE_CODE,code);
            }
            
        private:
            CURL*			m_curl_handle_;
        };

        class Downloader{
        private:
            enum{RedirectLimit = 3};
            enum{ConnTimeout = 5000};
        public:
            CURLcode Download(const char* url,size_t offset){
                assert(url);
                assert(m_app_.Empty());
                if(!m_app_.Init())
                    return CURLE_FAILED_INIT;

                CURLcode ret_ = m_app_.SetUrl(url);
                if(CURLE_OK != ret_)
                    return ret_;

                ret_ = m_app_.SetRedirect(RedirectLimit);
                if(CURLE_OK != ret_)
                    return ret_;

                ret_ = m_app_.SetConnTimeout(ConnTimeout);
                if(CURLE_OK != ret_)
                    return ret_;

                ret_ = m_app_.SetWriteCB(RecvCB,this);
                if(CURLE_OK != ret_)
                    return ret_;
                ret_ = m_app_.SetHeaderCB(HeaderCB,this);
                if(CURLE_OK != ret_)
                    return ret_;

                if(offset > 0){
                    ret_ = m_app_.SetRange(offset);
                    if(CURLE_OK != ret_)
                        return ret_;
                }

                m_size_ = 0;
                ret_ =  m_app_.SendRequest();
                if(CURLE_OK != ret_)
                    return ret_;
                return CURLE_OK;
            }
        private:
            size_t RecvData(char *ptr,size_t size){
                ;
                m_size_ += size;
                return size;
            }

            static size_t RecvCB(char *ptr, 
                size_t size,
                size_t nmemb,
                void *userdata){
                ;
                if(userdata){
                    return reinterpret_cast<Downloader*>(userdata)->RecvData(
                    ptr,
                    size*nmemb);
                }else{
                    return size*nmemb;
                }
            }
            static size_t HeaderCB(char *ptr, 
                size_t size,
                size_t nmemb,
                void *userdata){

                if(userdata){
                    Downloader* ptr_ = 
                        reinterpret_cast<Downloader*>(userdata);

                    long code_;
                    CURLcode ret_ =  ptr_->m_app_.GetHttpCode(&code_);
                    if(CURLE_OK != ret_)
                        return ret_;

                    return CURLE_OK;
                }else{
                    return size*nmemb;
                }
            }
        private:
            CurlApp		m_app_;
            size_t		m_size_;
        };

        /*
        CURLOPT_PROGRESSFUNCTION

        Pass a pointer to a function that matches the following prototype: 
        int function(void *clientp, double dltotal, double dlnow, 
        double ultotal, double ulnow); . This function gets called by libcurl
        instead of its internal equivalent with a frequent interval during
        operation (roughly once per second or sooner) no matter if data is
        being transfered or not. Unknown/unused argument values passed to 
        the callback will be set to zero (like if you only download data, 
        the upload size will remain 0). Returning a non-zero value from this
        callback will cause libcurl to abort the transfer and 
        return CURLE_ABORTED_BY_CALLBACK.
        */

        int main(int argc,char** argv){
            
            Downloader download_;
            download_.Download("192.168.1.3/http.7z",441000);
            return 0;

            return 0;
        }
