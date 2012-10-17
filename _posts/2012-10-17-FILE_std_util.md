---
layout: post
title: std FILE工具函数
category: cpp
---


        #include <stdio.h>
        #include <errno.h>
        #include <cassert>

        class CFILE{
        public:
            CFILE():m_fp_(NULL){}
            ~CFILE(){
                if(m_fp_){
                    ::fclose(m_fp_);
                }
            }
            
            /*
            * Open file
            * If the file has been successfully opened the function will return a pointer 
            * to a FILE object that is used to identify the stream on all further operations 
            * involving it. Otherwise, a null pointer is returned.
             */
            bool	fopen(const char * filename, const char * mode){
                FILE* fp_ = ::fopen(filename,mode);
                if(!fp_){
                    return false;
                }
                if(m_fp_){
                    fclose();
                }
                m_fp_ = fp_;
                return true;
            }
            
            /*
            * Close file
            * If the stream is successfully closed, a zero value is returned.
            * On failure, EOF is returned.
             */
            int		fclose(){
                if(m_fp_){
                    FILE* fp_ = m_fp_;
                    m_fp_ = NULL;
                    return ::fclose(fp_);
                }else{
                    return 0;
                }
            }
            
            /*
            * Check End-of-File indicator
            * A non-zero value is returned in the case that the End-of-File indicator 
            * associated with the stream is set.Otherwise, a zero value is returned.
             */
            int 	feof(){
                assert(m_fp_);
                if(m_fp_)
                    return ::feof(m_fp_);
                else
                    return 0;
            }
            
            /*
            * Check error indicator
            * If the error indicator associated with the stream was set, 
            * the function returns a nonzero value.Otherwise, it returns a zero value.
             */
            int 	ferror(){
                assert(m_fp_);
                if(m_fp_)
                    return ::ferror(m_fp_);
                else
                    return -1; 
            }
             
            /*
            * Flush stream
            * A zero value indicates success.If an error occurs, 
            * EOF is returned and the error indicator is set (see feof).
             */
            int 	fflush(){
                assert(m_fp_);
                if(m_fp_)
                    return ::fflush(m_fp_);
                else
                    return EOF;   
            }
              
            /*
            * Get character from stream
            * The character read is returned as an int value.
            * If the End-of-File is reached or a reading error happens, 
            * the function returns EOF and the corresponding error or 
            * eof indicator is set. You can use either ferror or feof to 
            * determine whether an error happened or the End-Of-File was reached.
             */
            int 	fgetc(){
                assert(m_fp_);
                if(m_fp_)
                    return ::fgetc(m_fp_);
                else
                    return EOF;   
            }

            /*
            * Get current position in stream
            * The function return a zero value on success, 
            * and a non-zero value in case of error.
             */
            int	fgetpos(fpos_t * position){
                assert(m_fp_);
                assert(position);
                if(m_fp_)
                    return ::fgetpos(m_fp_,position);
                else
                    return -1;    
            }

            char* fgets(char * str, int num){
                assert(m_fp_);
                assert(str);
                assert(num);
                if(m_fp_)
                    return ::fgets(str,num,m_fp_);
                else
                    return NULL;   
            }

            /*
            * Write character to stream
            * If there are no errors, the same character that has been written is returned.
            * If an error occurs, EOF is returned and the error indicator is set (see ferror).
             */
            int 	fputc(int character){
                assert(m_fp_);
                if(m_fp_)
                    return ::fputc(character,m_fp_);
                else
                    return EOF;   	  
            }
            
            /*
            * Write string to stream
            * On success, a non-negative value is returned.
            * On error, the function returns EOF.
             */
            int fputs(const char * str){
                assert(m_fp_);
                assert(str);
                if(m_fp_)
                    return ::fputs(str,m_fp_);
                else
                    return EOF;  	
            }
            
            /*
            * Read block of data from stream
            * The total number of elements successfully read is returned 
            * as a size_t object, which is an integral data type.
            * If this number differs from the count parameter, 
            * either an error occured or the End Of File was reached.
            * You can use either ferror or feof to check whether an error 
            * happened or the End-of-File was reached.
             */
            size_t fread(char * ptr, size_t size, size_t count){
                assert(m_fp_);
                assert(ptr);
                assert(size && count);
                if(m_fp_)
                    return ::fread(ptr,size,count,m_fp_);
                else
                    return 0;  	
            }
            
            /*
            * Reposition stream position indicator
            * SEEK_SET	Beginning of file
            * SEEK_CUR	Current position of the file pointer
            * SEEK_END	End of file
            * If successful, the function returns a zero value.
            * Otherwise, it returns nonzero value.
             */
            int fseek(long int offset, int origin ){	
                assert(m_fp_);
                if(m_fp_)
                    return ::fseek(m_fp_,offset,origin);
                else
                    return -1;  	
            }
            
            /*
            * Set position indicator of stream
            * If successful, the function returns a zero value.
            * Otherwise, it returns a nonzero value and sets the global variable 
            * errno to a positive value, which can be interpreted with perror.
             */
            int fsetpos(const fpos_t * pos ){
                assert(m_fp_);
                assert(pos);
                if(m_fp_)
                    return ::fsetpos(m_fp_,pos);
                else
                    return -1;  
            }
            
            /*
            * Get current position in stream
            * On success, the current value of the position indicator is returned.
            * If an error occurs, -1L is returned, and the global variable errno 
            * is set to a positive value. This value can be interpreted by perror.
             */
            long int ftell (){
                assert(m_fp_);
                if(m_fp_)
                    return ::ftell(m_fp_);
                else
                    return -1;  	
            }
            
            /*
            * Write block of data to stream
            * The total number of elements successfully written is returned as a 
            * size_t object, which is an integral data type.
            * If this number differs from the count parameter, it indicates an error.
             */
            size_t fwrite ( const char * ptr, size_t size, size_t count){
                assert(m_fp_);
                assert(ptr);
                assert(size && count);
                if(m_fp_)
                    return ::fwrite(ptr,size,count,m_fp_);
                else
                    return 0;  
            }
            
            ///////////////////////////////////////////////
            FILE* get(){
                return this->m_fp_;
            }
            void	attach(FILE* fp){
                if(this->m_fp_){
                    this->fclose();
                }
                this->m_fp_ = fp;
            }
            FILE*	dettach(){
                FILE* fp_ = m_fp_;
                m_fp_ = NULL;
                return fp_;
            }
            
            bool freadex( char *buf, size_t items,size_t* readed){
                assert(m_fp_);
                assert(readed);
                assert(items);
                assert(buf);
                if(!m_fp_ || !readed || !items || !buf){
                    return false;
                }

                size_t count = 0;
                *readed = 0;
                do{
                    count = fread( buf, 1, items);
                    if(ferror() && errno != EINTR )
                        return false;
                    
                    assert(items >= count);
                    *readed += count;
                    
                    if(!items)
                        return true;
                    if(feof())
                        return true;
                        
                    buf += count;
                    items -= count;
                }while(true);
                return false;
            }
            
            bool 	fwriteex( const char *buf,size_t items)
            {
                assert(m_fp_);
                assert(items);
                assert(buf);
                if(!m_fp_ || !items || !buf){
                    return false;
                }
                
                size_t count;
                do{
                    count = fwrite( buf, 1, items);
                    if(ferror() && errno != EINTR )
                        return false;
                        
                    assert(items >= count);
                    if(!items)
                        return true;
                    buf += count;
                    items -= count;
                }while(true);
                return false;
            }
        private:
            FILE*	m_fp_;
        };

