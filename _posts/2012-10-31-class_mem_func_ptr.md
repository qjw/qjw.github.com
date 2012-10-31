---
layout: post
title: 类成员函数指针小用
category: cpp
---

        class Entry
        {
        public:
            typedef void (Entry::*TestT)(int i);
            void test1(int i)
            {
                printf("test1 %d\n",i);
            }
            void test2(int i)
            {
                printf("test2 %d\n",i);
            }
            void test3(int i)
            {
                printf("test3 %d\n",i);
            }
        };

        class EntryMng
        {
        public:
            void test1(int i)
            {
                test_imp(i,&Entry::test1);
            }
            void test2(int i)
            {
                test_imp(i,&Entry::test2);
            }
            void test3(int i)
            {
                test_imp(i,&Entry::test3);
            }

        private:
            void test_imp(int i,Entry::TestT func)
            {
                // do other sth
                //(m_entry_ptr_->*func)(i);
                (m_entry_.*func)(i);
                // do other sth
            }

        private:
            Entry  m_entry_;
        };

        int _tmain(int argc, _TCHAR* argv[])
        {
            EntryMng mng_;
            mng_.test1(1);
            mng_.test2(2);
            mng_.test3(3);
            return 0;
        }