---
layout: post
title: deflate/inflate算法封装
category: cpp
---

        #ifndef MY_ZIP__H_
        #define MY_ZIP__H_

        #include "zlib.h"
        #include <cassert>
        #include <cstring>

        class ZipListener
        {
        public:
            virtual ~ZipListener(){}
            virtual void		OnZipdata(const char* data,size_t size,int magic) = 0;
            virtual void		OnZipEnd(int magic) = 0;
        };

        class ZipBase
        {
        protected:
            enum{
                BUF_SIZE = 4096,
            };

            ZipBase():m_stream_(NULL),m_listener_(NULL),m_magic_(0){}

            inline void			ResetInput(const char* buf,size_t size){
                m_stream_->avail_in = size;
                m_stream_->next_in = (Bytef*)buf;
            }
            inline void			ResetOutput(){
                m_stream_->next_out = (Bytef*)m_buf_;
                m_stream_->avail_out = sizeof(m_buf_);
            }
            inline void			Dispatch(){
                assert(m_stream_);
                if(m_stream_->avail_out < sizeof(m_buf_) && m_listener_){
                    this->m_listener_->OnZipdata(m_buf_,sizeof(m_buf_) - m_stream_->avail_out,m_magic_);
                }
            }
            inline void			Dispatch2(){
                assert(m_stream_);
                Dispatch();
                ResetOutput();
            }

            void				InitInside(ZipListener* listener,int magic){
                assert(!m_stream_ && magic);
                m_stream_ = new z_stream;
                memset(this->m_stream_,0,sizeof(z_stream));

                this->m_listener_ = listener;
                this->m_magic_ = magic;
            }

            inline void			CleanInside(){
                assert(m_stream_);
                delete m_stream_;
                m_stream_ = NULL;
                m_listener_ = NULL;
                m_magic_ = 0;
            }

            z_stream*			m_stream_;
            ZipListener*		m_listener_;
            int					m_magic_;
            char				m_buf_[BUF_SIZE];
        };


        class ZipDeflate : public ZipBase{
        private:
            bool				Clean(bool notify){
                assert(m_stream_);
                int ret = deflateEnd(m_stream_);
                if(Z_OK == ret && notify && m_listener_)
                    m_listener_->OnZipEnd(m_magic_);
                this->CleanInside();
                return Z_OK == ret;
            }
        public:
            ~ZipDeflate(){
                if(m_stream_)
                    Clean(false);
            }

            inline bool			Init(ZipListener* listener,int magic){
                InitInside(listener,magic);
                if(Z_OK == deflateInit(m_stream_, Z_DEFAULT_COMPRESSION))
                    return true;
                else{
                    CleanInside();
                    return false;
                }
            }

            bool				Append(const char* buf,size_t size){
                assert(m_stream_);
                ResetInput(buf,size);
                ResetOutput();
                int ret;
                do{
                    ret = deflate(m_stream_, Z_NO_FLUSH);
                    if(Z_OK != ret){
                        Clean(false);
                        return false;
                    }

                    if(0 == m_stream_->avail_in){
                        Dispatch();
                        return true;
                    }
                    if(0 == m_stream_->avail_out){
                        Dispatch2();
                    }
                }while(true);
            }

            bool				Fini(){
                assert(m_stream_);
                ResetOutput();
                int ret;
                do{
                    ret = deflate(m_stream_, Z_FINISH);
                    if(Z_STREAM_END == ret){
                        Dispatch();
                        return Clean(true);
                    }else if(Z_OK == ret){
                        if(0 == m_stream_->avail_out){
                            Dispatch2();
                        }
                        continue;
                    }else{
                        Clean(false);
                        return false;
                    }
                }while(true);
            }
        };


        class ZipInflate : public ZipBase{
        private:
            bool				Clean(bool notify){
                assert(m_stream_);
                int ret = inflateEnd(m_stream_);
                if(Z_OK == ret && notify && m_listener_)
                    m_listener_->OnZipEnd(m_magic_);
                this->CleanInside();
                return Z_OK == ret;
            }
        public:
            ~ZipInflate(){
                if(m_stream_)
                    Clean(false);
            }

            bool				Init(ZipListener* listener,int magic){
                InitInside(listener,magic);
                if(Z_OK == inflateInit(m_stream_))
                    return true;
                else{
                    CleanInside();
                    return false;
                }
            }

            /*
                1.解压结束
                0.解压成功，但是尚未结束
                -1.解压失败
            */
            int					Append(const char* buf,size_t size){
                ResetInput(buf,size);
                ResetOutput();
                int ret;
                do{
                    ret = inflate(m_stream_, Z_NO_FLUSH);
                    if(Z_STREAM_END == ret){
                        //已经处理完毕
                        //dispatch
                        Dispatch();
                        //关闭，并通知回调
                        return Clean(true)?1:-1;
                    }
                    else if(Z_OK == ret){
                        //如果输入已经处理完毕
                        if(0 == m_stream_->avail_in){
                            Dispatch();
                            return true;
                        }
                        
                        //如果输出已经处理完毕，则dispatch，并重设输出
                        if(0 == m_stream_->avail_out){
                            Dispatch2();
                        }

                        //继续inflate
                    }
                    else{
                        //出现错误
                        Clean(false);
                        return false;
                    }
                }while(true);
            }
        };

        #endif

        
---

        //测试用例

        #include "my_zip.h"
        #include <cassert>
        #include <cstdio>


        #if 1
        class XXX : public ZipListener{
        public:
            XXX(ZipInflate* inflate,FILE* fp):
              count_(0),count2_(0){
                  this->inflate_ = inflate;
                  this->fp_ = fp;
              }

            virtual void		OnZipdata(const char* data,size_t size,int magic){
                if(magic == 1){
                    count_ += size;
                    inflate_->Append(data,size);
                }else if(magic == 2){
                    count2_ += size;
                    fwrite(data,1,size,fp_);
                }
            }
            virtual void		OnZipEnd(int magic) {
            }

            int		count_;
            int		count2_;
            ZipInflate* inflate_;
            FILE*		fp_;
        };

        int main(int argc,const char** argv){
            if(argc < 3)
                return 1;

            FILE* ifp_ = fopen(argv[1],"rb");
            FILE* ofp_ = fopen(argv[2],"wb");
            if(!ifp_ || !ofp_ )
                return 2;


            ZipDeflate deflate_;
            ZipInflate inflate_;
            XXX xxxx_(&inflate_,ofp_);
            assert(deflate_.Init(&xxxx_,1));
            assert(inflate_.Init(&xxxx_,2));

            char buf[4096] = {0};
            size_t readed_;
            do{
                readed_ = fread(buf,1,sizeof(buf),ifp_);
                if(!readed_)
                    break;
                assert(deflate_.Append(buf,readed_));
            }while(true);
            assert(deflate_.Fini());

            return 0;
        }

        #else
        class XXX : public ZipListener{
        public:
            XXX(ZipInflate* inflate):
              count_(0),count2_(0){
                  this->inflate_ = inflate;
              }

            virtual void		OnZipdata(const char* data,size_t size,int magic){
                if(magic == 1){
                    count_ += size;
                    inflate_->Append(data,size);
                }else if(magic == 2){
                    count2_ += size;
                }
            }
            virtual void		OnZipEnd(int magic) {
            }

            int		count_;
            int		count2_;
            ZipInflate* inflate_;
        };

        int main(){
            char buf[1000] = {0};

            ZipDeflate deflate_;
            ZipInflate inflate_;
            XXX xxxx_(&inflate_);
            assert(deflate_.Init(&xxxx_,1));
            assert(inflate_.Init(&xxxx_,2));

            assert(deflate_.Append(buf,sizeof(buf)));
            assert(deflate_.Fini());

            return 0;
        }

        #endif
