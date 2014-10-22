---
layout: post
title: Google TCMalloc学习笔记4-线程本地存储
category: cpp
---

写于 **2011年11月18日**

前面提到了中央空闲链表，tcmalloc对每一个内存区间都设置一个中央空闲链表的类实例，线程本地存储和它类似，也是在内部对每一个内存区间做一个特殊的结构类实例，当不够的时候，就从中央空闲链表中取，太多了就还给它一些，线程结束后全部还给它。

线程本地存储通过下面的接口来访问，也就是通过它来获得线程绑定的一个ThreadCache类实例，然后在这个类中分配内存。

	inline ThreadCache* ThreadCache::GetCache() {
		ThreadCache* ptr = NULL;
		if (!tsd_inited_) {
			InitModule();
		} else {
			ptr = GetThreadHeap();
		}
		if (ptr == NULL)
			ptr = CreateCacheIfNecessary();
		return ptr;
	}

tcmalloc通过下面的静态变量来标识线程本地存储结构是否初始化，

	bool ThreadCache::tsd_inited_ = false;
	
也许很多人会想，上面的结构对此变量的访问并没有加锁，是否存在竞态呢？这个问题的确存在，但是tcmalloc在进入main函数之前就已经初始化，所以在之后的访问当中，是不会再有写入，所以共享读是没问题的。

	void ThreadCache::InitModule() {
		SpinLockHolder h(Static::pageheap_lock());
		if (!phinited) {
			Static::InitStaticVars();
			threadcache_allocator.Init();
			phinited = 1;
		}
	}

	static bool phinited = false;
	
当没有初始化，那么将调用InitModule函数来完成模块的初始化，是否初始化通过另一个静态变量phinited来控制，考虑到tcmalloc在进入main函数之前就完成了初始化，这种情况下也不会存在竞态的问题（实际上，当完成初始化之后，这个函数压根就不会跑进来）。Static::InitStaticVars 会完成前面几篇文章提到的相关模块的初始化，接下来要完成threadcache_allocator的初始化。

	PageHeapAllocator<ThreadCache> threadcache_allocator;
	
这个结构是一个全局变量，被所有线程共享，所以存在竞态，但是tcmalloc在使用它时有加锁，考虑到只有在创建线程的时候才会用到它，在这里加锁是没什么影响的。为什么要搞一个threadcache_allocator呢？tcmalloc本身是一个内存分配器，如果分配器内存的结构要分配内存，那会不会造成循环，造成所谓的鸡生蛋还是蛋生鸡的问题？所以引入一个threadcache_allocator来使用一种和tcmlloc无关的内存分配方式来分配内存。具体使用下面的函数：

	void* MetaDataAlloc(size_t bytes) {
		void* result = TCMalloc_SystemAlloc(bytes, NULL);
		if (result != NULL) {
			metadata_system_bytes_ += bytes;
		}
		return result;
	}
	
接下来就会通过CreateCacheIfNecessary函数获得线程绑定的ThreadCache。

	ThreadCache* ThreadCache::CreateCacheIfNecessary() {
		// Initialize per-thread data if necessary
		ThreadCache* heap = NULL;
		{
			//加锁
			SpinLockHolder h(Static::pageheap_lock());

			//获得pid，若尚未初始化，则赋值为0
			// Early on in glibc's life, we cannot even call pthread_self()
			pthread_t me;
			if (!tsd_inited_) {
				memset(&me, 0, sizeof(me));
			} else {
				me = pthread_self();
			}

			//找到一个匹配pid的ThreadCache
			// This may be a recursive malloc call from pthread_setspecific()
			// In that case, the heap for this thread has already been created
			// and added to the linked list.  So we search for that first.
			for (ThreadCache* h = thread_heaps_; h != NULL; h = h->next_) {
				if (h->tid_ == me) {
					heap = h;
					break;
				}
			}

			//如果找不到，则创建之
			//在这个函数里面，新申请的heap会挂载到thread_heaps_链表中去
			if (heap == NULL)
				heap = NewHeap(me);
		}

		// We call pthread_setspecific() outside the lock because it may
		// call malloc() recursively.  We check for the recursive call using
		// the "in_setspecific_" flag so that we can avoid calling
		// pthread_setspecific() if we are already inside pthread_setspecific().
		if (!heap->in_setspecific_ && tsd_inited_) {
			//如果已经初始化，就需要设置线程本地存储相关的数据
			heap->in_setspecific_ = true;
			//设置动态的线程本地存储
			perftools_pthread_setspecific(heap_key_, heap);
	#ifdef HAVE_TLS
			/*
			 #ifdef HAVE_TLS
			 __thread ThreadCache* ThreadCache::threadlocal_heap_
			 #endif
			 */
			//如果支持静态的线程本地存储，那么设置之，以加速访问
			// Also keep a copy in __thread for faster retrieval
			threadlocal_heap_ = heap;
	#endif
			heap->in_setspecific_ = false;
		}
		return heap;
	}
	
若线程已经初始化，则通过下面的函数来获得线程绑定的ThreadCache。

	inline ThreadCache* ThreadCache::GetThreadHeap() {
	#ifdef HAVE_TLS
		//如果支持TLS，则直接返回，以加速访问
		// __thread is faster, but only when the kernel supports it
		if (KernelSupportsTLS())
		return threadlocal_heap_;
	#endif
		//否则从动态分配的线程本地存储中获得
		return reinterpret_cast<ThreadCache *>(perftools_pthread_getspecific(
				heap_key_));
	}
	
threadlocal_heap_ 作为一个静态分配的TLS，无需任何初始化即可使用，而heap_key_是动态分配，所以需要显式的初始化，这个初始化在下面的函数完成

	void ThreadCache::InitTSD() {
		ASSERT(!tsd_inited_);
		//创建一个动态tls 槽，
		perftools_pthread_key_create(&heap_key_, DestroyThreadCache);
		tsd_inited_ = true;

		// We may have used a fake pthread_t for the main thread.  Fix it.
		pthread_t zero;
		memset(&zero, 0, sizeof(zero));
		SpinLockHolder h(Static::pageheap_lock());
		for (ThreadCache* h = thread_heaps_; h != NULL; h = h->next_) {
			//如果存在一个已经初始化，但是尚未分配（pid = 0）的ThreadCache，则据为己有。
			//这里只是做个标记，到CreateCacheIfNecessary函数中就会自动识别
			if (h->tid_ == zero) {
				h->tid_ = pthread_self();
			}
		}
	}
	
对于Windows平台perftools_pthread_key_create将跑到下面的函数

	pthread_key_t PthreadKeyCreate(void (*destr_fn)(void*)) {
		// Semantics are: we create a new key, and then promise to call
		// destr_fn with TlsGetValue(key) when the thread is destroyed
		// (as long as TlsGetValue(key) is not NULL).
		pthread_key_t key = TlsAlloc();
		if (destr_fn) {   // register it
			// If this assert fails, we'll need to support an array of destr_fn_infos
			assert(destr_fn_info.destr_fn == NULL);
			destr_fn_info.destr_fn = destr_fn;
			destr_fn_info.key_for_destr_fn_arg = key;
		}
		return key;
	}
	
关于Windows TLS，见<http://msdn.microsoft.com/en-us/library/6yh4a9k1(v=vs.80).aspx>

关于Linux TLS 见<http://linux.die.net/man/3/pthread_key_create>

在InitTSD函数中创建一个动态tls槽之后,在下一次调用CreateCacheIfNecessary函数时,将线程相关的ThreadCache放入这个tls槽中,之后就可以通过GetThreadHeap函数直接获得这个线程相关的ThreadCache实例了。

前面提到，当线程终止时，会自动释放所有的内存到中央空闲链表中，那这个怎么实现呢？对于linux，这很方便，因为pthread_key_create函数本身就支持这种行为。

	An optional destructor function may be associated with each key value. At thread exit, 
	if a key value has a non-NULL destructor pointer, 
	and the thread has a non-NULL value associated with that key, 
	the value of the key is set to NULL, and then the function pointed to 
	is called with the previously associated value as its sole argument. 
	The order of destructor calls is unspecified if more than one destructor 
	exists for a thread when it exits.
	
	int pthread_key_create(pthread_key_t *key, void (*destructor)(void*)); 

对于Windows，在PthreadKeyCreate函数中，会将注册的回调函数放到一个特殊的结构当中

	// When destr_fn eventually runs, it's supposed to take as its
	// argument the tls-value associated with key that pthread_key_create
	// creates.  (Yeah, it sounds confusing but it's really not.)  We
	// store the destr_fn/key pair in this data structure.  Because we
	// store this in a single var, this implies we can only have one
	// destr_fn in a program!  That's enough in practice.  If asserts
	// trigger because we end up needing more, we'll have to turn this
	// into an array.
	struct DestrFnClosure {
		void (*destr_fn)(void*);
		pthread_key_t key_for_destr_fn_arg;
	};

	static DestrFnClosure destr_fn_info;   // initted to all NULL/0.
	
当线程终止时，会调用on_tls_callback函数，最终到on_process_term函数，最后前面注册的回调会被调用，并且携带动态tls槽中的数据作为参数。

	BOOL WINAPI DllMain(HINSTANCE h, DWORD dwReason, PVOID pv) {
		if (dwReason == DLL_THREAD_DETACH)
			on_tls_callback(h, dwReason, pv);
		else if (dwReason == DLL_PROCESS_DETACH)
			on_process_term();
		return TRUE;
	}

	static void NTAPI on_tls_callback(HINSTANCE h, DWORD dwReason, PVOID pv) {
		if (dwReason == DLL_THREAD_DETACH) {   // thread is being destroyed!
			on_process_term();
		}
	}
	static int on_process_term(void) {
		if (destr_fn_info.destr_fn) {
			void *ptr = TlsGetValue(destr_fn_info.key_for_destr_fn_arg);
			// This shouldn't be necessary, but in Release mode, Windows
			// sometimes trashes the pointer in the TLS slot, so we need to
			// remove the pointer from the TLS slot before the thread dies.
			TlsSetValue(destr_fn_info.key_for_destr_fn_arg, NULL);
			if (ptr)  // pthread semantics say not to call if ptr is NULL
				(*destr_fn_info.destr_fn)(ptr);
		}
		return 0;
	}
	
在这个回调中，线程本地存储的内存最终被归还于中央空闲链表区

	void ThreadCache::DestroyThreadCache(void* ptr) {
		// Note that "ptr" cannot be NULL since pthread promises not
		// to invoke the destructor on NULL values, but for safety,
		// we check anyway.
		if (ptr == NULL)
			return;
	#ifdef HAVE_TLS
		// Prevent fast path of GetThreadHeap() from returning heap.
		threadlocal_heap_ = NULL;
	#endif
		DeleteCache(reinterpret_cast<ThreadCache*>(ptr));
	}
	
前面所有的初始化操作都是通过TCMallocGuard类来驱动，这个类包含一个静态的实例，所以在main函数之前，会调用它的构造函数，上面的初始化操作就是通过它的构造来调用的。

ThreadCache对内存的管理和CentralFreeList差不多，这个就不再说了。此外，ThreadCache的内存归还策略也可以看一下。

##参考
1. <http://pan.baidu.com/s/1mgmiuqk>
