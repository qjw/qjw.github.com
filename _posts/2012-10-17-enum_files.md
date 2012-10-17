---
layout: post
title: 枚举目录
category: cpp
---

        #include <windows.h>
        typedef void (*FileFound)(const std::string& filename,bool dir);
        DWORD IterFiles(const std::string& path,FileFound cb,bool subdir)
        {
            WIN32_FIND_DATA findFileData;
            std::string filePath = path;
            filePath += "\\*.*";
            HANDLE hFind = FindFirstFile(filePath.c_str(), &findFileData);
            if (hFind != INVALID_HANDLE_VALUE)
            {
                do 
                {
                    if (strcmp(findFileData.cFileName, ".") == 0 ||
                        strcmp(findFileData.cFileName, "..") == 0)
                    {
                        continue;
                    }
                    
                    std::string subfile(path);
                    subfile += "\\";
                    subfile += findFileData.cFileName;
                    if(cb){
                        cb(subfile,
                            findFileData.dwFileAttributes &
                            FILE_ATTRIBUTE_DIRECTORY);
                    }

                    if (subdir &&
                        findFileData.dwFileAttributes & FILE_ATTRIBUTE_DIRECTORY)
                    {
                        DWORD dwStatus = IterFiles(subfile,cb,subdir);
                        if(!dwStatus)
                            return dwStatus;
                    }
                } while (FindNextFile(hFind, &findFileData));
                return 0;
            }
            return ::GetLastError();
        }

        void MyFileFound(const std::string& filename,bool dir){
            std::cout << filename << " " << dir << std::endl;
        }

        int main(int argc, char **argv)
        {
            IterFiles("C:/TDDownload",MyFileFound,false);
            printf("hello world\n");
            return 0;
        }

