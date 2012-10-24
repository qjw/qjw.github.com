---
layout: post
title: 内存碎片序列化
category: cpp
---

我们经常有各种字段需要序列化，对于文本，可以使用[std::ostringstream](http://www.cplusplus.com/reference/iostream/ostringstream/),或者[snprintf](http://linux.die.net/man/3/snprintf)系列函数。

对于二进制流，可以使用[ostrstream](http://msdn.microsoft.com/en-us/library/2c1zt7yk\(v=vs.80\).aspx)，不过这个函数似乎被废弃

这里yy一个简单的，**注意它不会将所有的内存重新拷贝，而是将它们组织起来，但是对外看起来就像序列化一样**

        #pragma once

        #include <vector>

        class mem
        {
        private:

            class MemoryPtr{
            public:
                MemoryPtr(const char* ptr,size_t size,size_t offset):
                    m_ptr_(ptr),
                    m_size_(size),
                    m_offset_(offset){}
                    
                inline size_t	offset() const{
                    return m_offset_;
                }
                
                inline size_t	size() const{
                    return m_size_;
                }
                
                inline const char*	buf() const{
                    return m_ptr_;
                }
            private:
                const char*		m_ptr_;
                size_t			m_size_;
                size_t			m_offset_;
            };
            

            typedef std::vector<MemoryPtr*> Vector;
            typedef Vector::iterator	Iter;
            
            
        public:

            mem(void):
                m_size_(0),
                m_last_index_(0){}
            
            bool			Memcpy(char* buf,size_t offset,size_t size)
            {
                if(offset + size <= m_size_)
                {
                    if( offset < m_container_[m_last_index_]->offset() ||
                        offset >= m_container_[m_last_index_]->offset() + m_container_[m_last_index_]->size())
                    {
                        m_last_index_ = IndexPos(offset);
                    }
                    
                    size_t tmp = offset - m_container_[m_last_index_]->offset();
                    
                    size_t total = std::min(size,m_container_[m_last_index_]->size() - tmp);
                    size -= total;
                    memcpy(buf,m_container_[m_last_index_]->buf() + tmp,total);
                    
                    
                    size_t index = m_last_index_ + 1;
                    while(size > 0)
                    {
                        tmp = std::min(size,m_container_[index]->size());
                        memcpy(buf + total,m_container_[index]->buf(),tmp);
                        total += tmp;
                        size -= tmp;
                        index ++;
                    }
                    
                    return true;
                }else{
                    return false;
                }
            }
            
            void			Append(const char* ptr,size_t size)
            {
                m_container_.push_back(new MemoryPtr(ptr,size,m_size_));
                m_size_ += size;
            }
            
            void			Clear()
            {
                for(Iter iter = m_container_.begin();
                    iter != m_container_.end();
                    ++iter)
                {
                    delete *iter;	
                }
                m_container_.clear();
            }
            
        private:
            size_t			IndexPos(size_t pos){
                int low = 0;
                int high = (int)m_container_.size() - 1;
                int mid;
                int midVal;
                while (low <= high) {
                    mid = (low + high) / 2;
                    midVal = (int)(m_container_[mid]->offset());
                    if (midVal < pos){
                        low = mid + 1;
                    }
                    else if (midVal > pos){
                        high = mid - 1;
                    }
                    else{
                        return (size_t)mid;
                    }
                }
                return (size_t)(low-1);
            }

        private:
            Vector			m_container_;
            size_t			m_size_;
            size_t			m_last_index_;
        };

---

        #include "mem.h"

        #include <cstdio>
        int main(int argc,const char** argv)
        {
            mem m;
            char buf1[] = "qiujinwu";
            char buf2[] = "lilili";
            char buf3[] = "kingqiu";
            m.Append(buf1,sizeof(buf1));
            m.Append(buf2,sizeof(buf2));
            m.Append(buf3,sizeof(buf3));
            
            char buf[11];
            m.Memcpy(buf,10,sizeof(buf));
            m.Memcpy(buf,10,sizeof(buf));
            return 0;
        }

