---
layout: post
title: 内存萃取接口
category: cpp
---


        /*
        *
        * Copyright (c) 1997
        * Silicon Graphics Computer Systems, Inc.
        *
        * Permission to use, copy, modify, distribute and sell this software
        * and its documentation for any purpose is hereby granted without fee,
        * provided that the above copyright notice appear in all copies and
        * that both that copyright notice and this permission notice appear
        * in supporting documentation.  Silicon Graphics makes no
        * representations about the suitability of this software for any
        * purpose.  It is provided "as is" without express or implied warranty.
        */

        #ifndef __TYPE_TRAITS_H
        #define __TYPE_TRAITS_H

        // #ifndef __STL_CONFIG_H
        // #include <stl_config.h>
        // #endif

        /*
        This header file provides a framework for allowing compile time dispatch
        based on type attributes. This is useful when writing template code.
        For example, when making a copy of an array of an unknown type, it helps
        to know if the type has a trivial copy constructor or not, to help decide
        if a memcpy can be used.

        The class template __type_traits provides a series of typedefs each of
        which is either __true_type or __false_type. The argument to
        __type_traits can be any type. The typedefs within this template will
        attain their correct values by one of these means:
        1. The general instantiation contain conservative values which work
        for all types.
        2. Specializations may be declared to make distinctions between types.
        3. Some compilers (such as the Silicon Graphics N32 and N64 compilers)
        will automatically provide the appropriate specializations for all
        types.

        EXAMPLE:

        //Copy an array of elements which have non-trivial copy constructors
        template <class T> void copy(T* source, T* destination, int n, __false_type);
        //Copy an array of elements which have trivial copy constructors. Use memcpy.
        template <class T> void copy(T* source, T* destination, int n, __true_type);

        //Copy an array of any type by using the most efficient copy mechanism
        template <class T> inline void copy(T* source,T* destination,int n) {
        copy(source, destination, n,
        typename __type_traits<T>::has_trivial_copy_constructor());
        }
        */


        struct __true_type {
        };

        struct __false_type {
        };

        template <class _Tp>
        struct __type_traits {
            typedef __true_type     this_dummy_member_must_be_first;
            /* Do not remove this member. It informs a compiler which
            automatically specializes __type_traits that this
            __type_traits template is special. It just makes sure that
            things work if an implementation is using a template
            called __type_traits for something unrelated. */

            /* The following restrictions should be observed for the sake of
            compilers which automatically produce type specific specializations
            of this class:
            - You may reorder the members below if you wish
            - You may remove any of the members below if you wish
            - You must not rename members without making the corresponding
            name change in the compiler
            - Members you add will be treated like regular members unless
            you add the appropriate support in the compiler. */


            typedef __false_type    has_trivial_default_constructor;
            typedef __false_type    has_trivial_copy_constructor;
            typedef __false_type    has_trivial_assignment_operator;
            typedef __false_type    has_trivial_destructor;
            typedef __false_type    is_POD_type;
        };



        // Provide some specializations.  This is harmless for compilers that
        //  have built-in __types_traits support, and essential for compilers
        //  that don't.

        #ifndef __STL_NO_BOOL

        template<> struct __type_traits<bool> {
            typedef __true_type    has_trivial_default_constructor;
            typedef __true_type    has_trivial_copy_constructor;
            typedef __true_type    has_trivial_assignment_operator;
            typedef __true_type    has_trivial_destructor;
            typedef __true_type    is_POD_type;
        };

        #endif /* __STL_NO_BOOL */

        template<> struct __type_traits<char> {
            typedef __true_type    has_trivial_default_constructor;
            typedef __true_type    has_trivial_copy_constructor;
            typedef __true_type    has_trivial_assignment_operator;
            typedef __true_type    has_trivial_destructor;
            typedef __true_type    is_POD_type;
        };

        template<> struct __type_traits<signed char> {
            typedef __true_type    has_trivial_default_constructor;
            typedef __true_type    has_trivial_copy_constructor;
            typedef __true_type    has_trivial_assignment_operator;
            typedef __true_type    has_trivial_destructor;
            typedef __true_type    is_POD_type;
        };

        template<> struct __type_traits<unsigned char> {
            typedef __true_type    has_trivial_default_constructor;
            typedef __true_type    has_trivial_copy_constructor;
            typedef __true_type    has_trivial_assignment_operator;
            typedef __true_type    has_trivial_destructor;
            typedef __true_type    is_POD_type;
        };

        #ifdef __STL_HAS_WCHAR_T

        template<> struct __type_traits<wchar_t> {
            typedef __true_type    has_trivial_default_constructor;
            typedef __true_type    has_trivial_copy_constructor;
            typedef __true_type    has_trivial_assignment_operator;
            typedef __true_type    has_trivial_destructor;
            typedef __true_type    is_POD_type;
        };

        #endif /* __STL_HAS_WCHAR_T */

        template<> struct __type_traits<short> {
            typedef __true_type    has_trivial_default_constructor;
            typedef __true_type    has_trivial_copy_constructor;
            typedef __true_type    has_trivial_assignment_operator;
            typedef __true_type    has_trivial_destructor;
            typedef __true_type    is_POD_type;
        };

        template<> struct __type_traits<unsigned short> {
            typedef __true_type    has_trivial_default_constructor;
            typedef __true_type    has_trivial_copy_constructor;
            typedef __true_type    has_trivial_assignment_operator;
            typedef __true_type    has_trivial_destructor;
            typedef __true_type    is_POD_type;
        };

        template<> struct __type_traits<int> {
            typedef __true_type    has_trivial_default_constructor;
            typedef __true_type    has_trivial_copy_constructor;
            typedef __true_type    has_trivial_assignment_operator;
            typedef __true_type    has_trivial_destructor;
            typedef __true_type    is_POD_type;
        };

        template<> struct __type_traits<unsigned int> {
            typedef __true_type    has_trivial_default_constructor;
            typedef __true_type    has_trivial_copy_constructor;
            typedef __true_type    has_trivial_assignment_operator;
            typedef __true_type    has_trivial_destructor;
            typedef __true_type    is_POD_type;
        };

        template<> struct __type_traits<long> {
            typedef __true_type    has_trivial_default_constructor;
            typedef __true_type    has_trivial_copy_constructor;
            typedef __true_type    has_trivial_assignment_operator;
            typedef __true_type    has_trivial_destructor;
            typedef __true_type    is_POD_type;
        };

        template<> struct __type_traits<unsigned long> {
            typedef __true_type    has_trivial_default_constructor;
            typedef __true_type    has_trivial_copy_constructor;
            typedef __true_type    has_trivial_assignment_operator;
            typedef __true_type    has_trivial_destructor;
            typedef __true_type    is_POD_type;
        };

        #ifdef __STL_LONG_LONG

        template<> struct __type_traits<long long> {
            typedef __true_type    has_trivial_default_constructor;
            typedef __true_type    has_trivial_copy_constructor;
            typedef __true_type    has_trivial_assignment_operator;
            typedef __true_type    has_trivial_destructor;
            typedef __true_type    is_POD_type;
        };

        template<> struct __type_traits<unsigned long long> {
            typedef __true_type    has_trivial_default_constructor;
            typedef __true_type    has_trivial_copy_constructor;
            typedef __true_type    has_trivial_assignment_operator;
            typedef __true_type    has_trivial_destructor;
            typedef __true_type    is_POD_type;
        };

        #endif /* __STL_LONG_LONG */

        template<> struct __type_traits<float> {
            typedef __true_type    has_trivial_default_constructor;
            typedef __true_type    has_trivial_copy_constructor;
            typedef __true_type    has_trivial_assignment_operator;
            typedef __true_type    has_trivial_destructor;
            typedef __true_type    is_POD_type;
        };

        template<> struct __type_traits<double> {
            typedef __true_type    has_trivial_default_constructor;
            typedef __true_type    has_trivial_copy_constructor;
            typedef __true_type    has_trivial_assignment_operator;
            typedef __true_type    has_trivial_destructor;
            typedef __true_type    is_POD_type;
        };

        template<> struct __type_traits<long double> {
            typedef __true_type    has_trivial_default_constructor;
            typedef __true_type    has_trivial_copy_constructor;
            typedef __true_type    has_trivial_assignment_operator;
            typedef __true_type    has_trivial_destructor;
            typedef __true_type    is_POD_type;
        };

        #ifdef __STL_CLASS_PARTIAL_SPECIALIZATION

        template <class _Tp>
        struct __type_traits<_Tp*> {
            typedef __true_type    has_trivial_default_constructor;
            typedef __true_type    has_trivial_copy_constructor;
            typedef __true_type    has_trivial_assignment_operator;
            typedef __true_type    has_trivial_destructor;
            typedef __true_type    is_POD_type;
        };

        #else /* __STL_CLASS_PARTIAL_SPECIALIZATION */

        template<> struct __type_traits<char*> {
            typedef __true_type    has_trivial_default_constructor;
            typedef __true_type    has_trivial_copy_constructor;
            typedef __true_type    has_trivial_assignment_operator;
            typedef __true_type    has_trivial_destructor;
            typedef __true_type    is_POD_type;
        };

        template<> struct __type_traits<signed char*> {
            typedef __true_type    has_trivial_default_constructor;
            typedef __true_type    has_trivial_copy_constructor;
            typedef __true_type    has_trivial_assignment_operator;
            typedef __true_type    has_trivial_destructor;
            typedef __true_type    is_POD_type;
        };

        template<> struct __type_traits<unsigned char*> {
            typedef __true_type    has_trivial_default_constructor;
            typedef __true_type    has_trivial_copy_constructor;
            typedef __true_type    has_trivial_assignment_operator;
            typedef __true_type    has_trivial_destructor;
            typedef __true_type    is_POD_type;
        };

        template<> struct __type_traits<const char*> {
            typedef __true_type    has_trivial_default_constructor;
            typedef __true_type    has_trivial_copy_constructor;
            typedef __true_type    has_trivial_assignment_operator;
            typedef __true_type    has_trivial_destructor;
            typedef __true_type    is_POD_type;
        };

        template<> struct __type_traits<const signed char*> {
            typedef __true_type    has_trivial_default_constructor;
            typedef __true_type    has_trivial_copy_constructor;
            typedef __true_type    has_trivial_assignment_operator;
            typedef __true_type    has_trivial_destructor;
            typedef __true_type    is_POD_type;
        };

        template<> struct __type_traits<const unsigned char*> {
            typedef __true_type    has_trivial_default_constructor;
            typedef __true_type    has_trivial_copy_constructor;
            typedef __true_type    has_trivial_assignment_operator;
            typedef __true_type    has_trivial_destructor;
            typedef __true_type    is_POD_type;
        };

        #endif /* __STL_CLASS_PARTIAL_SPECIALIZATION */


        // The following could be written in terms of numeric_limits.
        // We're doing it separately to reduce the number of dependencies.

        template <class _Tp> struct _Is_integer {
            typedef __false_type _Integral;
        };

        #ifndef __STL_NO_BOOL

        template<> struct _Is_integer<bool> {
            typedef __true_type _Integral;
        };

        #endif /* __STL_NO_BOOL */

        template<> struct _Is_integer<char> {
            typedef __true_type _Integral;
        };

        template<> struct _Is_integer<signed char> {
            typedef __true_type _Integral;
        };

        template<> struct _Is_integer<unsigned char> {
            typedef __true_type _Integral;
        };

        #ifdef __STL_HAS_WCHAR_T

        template<> struct _Is_integer<wchar_t> {
            typedef __true_type _Integral;
        };

        #endif /* __STL_HAS_WCHAR_T */

        template<> struct _Is_integer<short> {
            typedef __true_type _Integral;
        };

        template<> struct _Is_integer<unsigned short> {
            typedef __true_type _Integral;
        };

        template<> struct _Is_integer<int> {
            typedef __true_type _Integral;
        };

        template<> struct _Is_integer<unsigned int> {
            typedef __true_type _Integral;
        };

        template<> struct _Is_integer<long> {
            typedef __true_type _Integral;
        };

        template<> struct _Is_integer<unsigned long> {
            typedef __true_type _Integral;
        };

        #ifdef __STL_LONG_LONG

        template<> struct _Is_integer<long long> {
            typedef __true_type _Integral;
        };

        template<> struct _Is_integer<unsigned long long> {
            typedef __true_type _Integral;
        };

        #endif /* __STL_LONG_LONG */

        #endif /* __TYPE_TRAITS_H */

        // Local Variables:
        // mode:C++
        // End:


---

        #ifndef MEM__H_
        #define MEM__H_

        #ifndef __TYPE_TRAITS_H
        #include "type_traits.h"
        #endif

        #include <cstdlib>
        #include <new>


        void*			Malloc(size_t size){
            return malloc(size);
        }

        void			Free(void* ptr){
            if(ptr)
                free(ptr);
        }

        namespace DO_NOT_USE_DIRECT_{
        //析构基本类型
        template<typename T>
        void			DeleteImp(__true_type,T* ptr){
            Free(ptr);
        }
        //析构自定义类型
        template<typename T>
        void			DeleteImp(__false_type,T* ptr){
            if(ptr){
                ptr->~T();
                Free(ptr);
            }
        }

        //构造基本类型
        template<typename T>
        T*				NewImp(__true_type){
            return static_cast<T*>(Malloc(sizeof(T)));
        }

        //构造自定义类型，无参数
        template<typename T>
        T*				NewImp(__false_type){
            void* ptr_ = Malloc(sizeof(T));
            if(!ptr_)
                return NULL;
            else
                return new(ptr_) T;
        }

        //构造自定义类型,一个参数
        template<typename T,typename A1>
        T*				NewImp(__false_type,A1 a1){
            void* ptr_ = Malloc(sizeof(T));
            if(!ptr_)
                return NULL;
            else
                return new(ptr_) T(a1);
        }


        }

        template<typename T>
        void			Delete(T* ptr){
            DO_NOT_USE_DIRECT_::DeleteImp(typename __type_traits<T>::has_trivial_destructor(),ptr);
        }

        template<typename T>
        T*				New(){
            return DO_NOT_USE_DIRECT_::NewImp<T>(typename __type_traits<T>::has_trivial_default_constructor());
        }

        template<typename T,typename A1>
        T*				New(A1 a1){
            return DO_NOT_USE_DIRECT_::NewImp<T>(typename __type_traits<T>::has_trivial_default_constructor(),a1);
        }

        #endif

---

        #include "mem.h"

        class Cls
        {
        public:
            Cls(int j){
                int i = 0;
                i++;
            }

            ~Cls()
            {
                int i = 0;
                i++;
            }
        };

        int main(){
            int* xxx = (int*)Malloc(34);
            Delete(xxx);
            
            Cls *cls = New<Cls>(99);
            Delete(cls);
            return 0;
        }

#重载 ::operator new\(\)
在使用自定义内存分配器时，如果考虑在前面插入一定的内存来进行审计，必须保证给客户的内存是sizeof(void*)对齐。

        static inline void*		Malloc(size_t size)
        {
            return size?malloc(size):NULL;
        }
        static inline void		Free(void* ptr){
            if(ptr)
                free(ptr);
        }

        //接口
        static inline void* operator new(size_t size){
            return Malloc(size);
        };
        static inline void operator delete(void* ptr){
            Free(ptr);
        }
        static inline void* operator new [](size_t size){
            return Malloc(size);
        };
        static inline void operator delete [](void*ptr){
            Free(ptr);
        }

#参考
1. <http://msdn.microsoft.com/en-us/library/a45x8asx\(v=vs.71\).aspx>
1. <http://www.cplusplus.com/reference/std/new/set_new_handler/>
1. <http://www.cppblog.com/Solstice/archive/2011/02/22/140410.aspx>

