---
layout: post
title: Posix文件工具函数
category: cpp
---

        #include <stdio.h>
        #include <direct.h>
        #include <io.h>
        #include <sys/stat.h>
        #include <sys/types.h>

        #define R_OK 4
        #define W_OK 2
        #define X_OK 1
        #define F_OK 0

        #else
        #include <unistd.h>
        #include <sys/stat.h>
        #include <sys/types.h>
        #endif
        namespace futil{
            /*
            * delete a name and possibly the file it refers to
            * On success, zero is returned. 
            * On error, -1 is returned, and errno is set appropriately.
            * _unlink
            * _wunlink
            * _tunlink
             */
            int unlink(const char *pathname){
                assert(pathname);
                if(pathname)
                    return ::unlink(pathname);
                else
                    return -1;
            }
            
            /*
            * create a temporary file
            * The tmpfile() function returns a stream descriptor, 
            * or NULL if a unique filename cannot be generated or 
            * the unique file cannot be opened. In the latter case,
            * errno is set to indicate the error.
             */
            FILE *tmpfile(void){
                return ::tmpfile();
            }
            
            /*
            * make a directory
            * Upon successful completion, mkdir() shall return 0. Otherwise, 
            * -1 shall be returned, no directory shall be created, 
            * and errno shall be set to indicate the error.
            * _tmkdir
            * _mkdir
            * _wmkdir
             */
            int mkdir(const char *path){
                assert(path);
                if(path)
                    return ::mkdir(path);
                else
                    return -1;
            }
            
            /*
            * delete a directory
            * On success, zero is returned. On error, -1 is returned, 
            * and errno is set appropriately.
             */
            int rmdir(const char *pathname){
                assert(pathname);
                if(pathname)
                    return ::rmdir(pathname);
                else
                    return -1;		
            }
            
            /*
            * change working directory
            * Upon successful completion, 0 shall be returned. 
            * Otherwise, -1 shall be returned, the current working directory
            * shall remain unchanged, and errno shall be set to indicate the error.
             */
            int chdir(const char *pathname){
                assert(pathname);
                if(pathname)
                    return ::chdir(pathname);
                else
                    return -1;	
            }
            
            /*
            * check real user's permissions for a file
            * On success (all requested permissions granted), zero is returned. 
            * On error (at least one bit in mode asked for a permission that is denied, 
            * or some other error occurred), -1 is returned, and errno is set appropriately.
            * _taccess		
            * _access	
            * _waccess
             */
            int access(const char *pathname, int mode){
                assert(pathname);
                if(pathname)
                    return ::access(pathname,mode);
                else
                    return -1;
            }

            inline bool filexist(const char* pathname){
                if(pathname)
                    return !access(pathname,F_OK);
                else
                    return false;
            }
            inline bool filereadable(const char* pathname){
                if(pathname)
                    return !access(pathname,R_OK);
                else
                    return false;
            }
            inline bool filewritable(const char* pathname){
                if(pathname)
                    return !access(pathname,W_OK);
                else
                    return false;
            }
            
            /*
            * change the name or location of a file
            * On success, zero is returned. On error, 
            * -1 is returned, and errno is set appropriately.
             */
            int rename(const char *oldpath, const char *newpath){
                assert(oldpath);
                assert(newpath);
                if(oldpath && newpath)
                    return ::rename(oldpath,newpath);
                else
                    return -1;
            }
            
            /*
            * stat, fstat, lstat - get file status
            * On success, zero is returned. On error, -1 is returned, 
            * and errno is set appropriately.
             */
            int stat2(const char *path, struct stat *buf){
                assert(path);
                assert(buf);
                if(path && buf)
                    return ::stat(path,buf);
                else
                    return -1;
            }
            
            inline bool filesize(const char* path,off_t* size){
                assert(size);
                struct stat stat_;
                if(size && !stat2(path,&stat_)){
                    *size = stat_.st_size;
                    return true;
                }else{
                    return false;
                }
            }
        }

