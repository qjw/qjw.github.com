---
layout: post
title: 使用std::find_if根据某一字段进行精确查找
category: cpp
---

我们经常用std容器来存储各种对象，若需要查找，对于set或者map，可以利用自身的find函数来实现高效查找。对于其他容器，可以用std算法函数std::find来查找，例如

        std::find(v.begin(),v.end(),elem);
        // 这其中第三个参数必须是一个elem对象
        
很多时候，我们希望根据结构中的某个字段来进行查找，那怎么办呢？答案是std::find_if

        #include <string>
        #include <vector>
        #include <algorithm>
        #include <set>

        struct Element
        {
            Element(int id1):id(id1)
            {
            }
            int id;
            std::string name;
            std::string desc;
        };

        typedef std::vector<Element> Evector;
        typedef Evector::iterator Iter;

        struct Finder
        {
            int m_id_;
            Finder(int id):m_id_(id) {}
            bool operator()(const Element& m ) const
            {
                return m.id == m_id_;
            }
        };

        int _tmain(int argc, _TCHAR* argv[])
        {
            Evector evtor_;
            evtor_.push_back(Element(2));
            evtor_.push_back(Element(1));
            evtor_.push_back(Element(3));
            Iter iter_ = std::find_if(evtor_.begin(),evtor_.end(),Finder(1));
            
            return 0;
        }
