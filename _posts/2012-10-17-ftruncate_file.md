---
layout: post
title: 修改文件大小
category: cpp
---

        bool setFileLength(FILE* file, unsigned int len)
        {
        #ifdef _WIN32
            fseek(file, len, SEEK_SET);
            int fd = _fileno(file);
            HANDLE hfile = (HANDLE)_get_osfhandle(fd);
            return !!SetEndOfFile(hfile);
        #else
            int fd = fileno(file);
            return ftruncate(fd, len) == 0;
        #endif
        }
