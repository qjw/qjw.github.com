---
layout: post
title: string 工具函数
category: cpp
---


        #include <string>
        #include <algorithm>
        #include <iostream>

        // 大小写转换
        void func1(){
            std::string str("abcDEFG");
            std::transform(str.begin(), str.end(), str.begin(), tolower);
            std::cout << str << std::endl;
            
            str = "abcDEFG";
            std::transform(str.begin(), str.end(), str.begin(), toupper);
            std::cout << str << std::endl;
        }

        // 去掉前后的空白字符
        void trim(){
            std::string str(" abcDEFG ");
            str.erase(0, str.find_first_not_of(" \t\n\r"));
            std::cout << str << "|" << std::endl;
            
            str = " abcDEFG ";
            str.erase(str.find_last_not_of(" \t\n\r") + 1);
            std::cout << str << "|" << std::endl;
            
            str = " abcDEFG ";
            str.erase(0, str.find_first_not_of(" \t\n\r"));
            str.erase(str.find_last_not_of(" \t\n\r") + 1);
            std::cout << str << "|" << std::endl;
        }

        // 以XX开头
        bool begin_with(const std::string& str,const std::string& substr){
            return str.find(substr) == 0;
        }
        // 以XX结尾
        bool end_with(const std::string& str,const std::string& substr){
            return str.rfind(substr) == (str.length() - substr.length());;
        }

        #include <sstream>
        // string 转换成其他类型
        template<class T> 
        T parse_string(const std::string& str) {
            T value;
            std::istringstream iss(str);
            iss >> value;
            return value;
        }
        // tostring
        template<class T> 
        std::string to_string(const T& value) {
            std::ostringstream oss;
            oss << value;
            return oss.str();
        }

