---
layout: post
title: Google TCMalloc学习笔记3-中央空闲链表
category: cpp
---

写于 **2011年11月5日**

在上文讲到，tcmalloc通过PageHeap来管理最底层的内存页，往上则是中央空闲链表（CentralFreeList）来管理一个所有线程共享的内存对象。在《Google TCMalloc学习笔记1》讲到，tcmalloc将内存划分成了若干区间，一共有61个（32位系统）区间，tcmalloc对每一个区间都对应一个CentralFreeList。在同一个CentralFreeList内部，从PageHeap获得页都会划分成该区间大小的内存对象（object)。

每一个CentralFreeList都包含一个成员标识该类所对应的区间（size_class），通过这个size_class，CentralFreeList从SizeMap（见《Google TCMalloc学习笔记1》）获得该区间的内存对象大小、每一次从PageHeap获得多少个页面等信息。当ThreadCache从CentralFreeList申请内存时，首先会根据申请的大小定位到具体的CentralFreeList，接下来CentralFreeList会检查是否有非空的Span，若存在，则从该Span抽一个内存对象出去，否则向PageHeap要一个Span（这个Span可能包含一个或者多个内存页，具体来自SizeMap），并且将Span的内存分解成若干内存对象，最后再划分一个出去。

让我们再次看下Span的定义

	struct Span {
		//span首个页面的基数树ID
		PageID start; // Starting page number
		//页面数量
		Length length; // Number of pages in span
		//双向链表辅助指针
		Span* next; // Used when in link list
		Span* prev; // Used when in link list
		//内存对象
		void* objects; // Linked list of free objects
		//正在使用中的内存对象数量
		unsigned int refcount :16; // Number of non-free objects
		//内存区间ID
		unsigned int sizeclass :8; // Size-class for small objects (or 0)
		//位置（或者使用状态）
		unsigned int location :2; // Is the span on a freelist, and if so, which?
		unsigned int sample :1; // Sampled object?

		// What freelist the span is on: IN_USE if on none, or normal or returned
		enum {
			IN_USE, ON_NORMAL_FREELIST, ON_RETURNED_FREELIST
		};
	};

	
从PageHeap出来的Span只会初始化start、length、location，那么object、refcount、sizeclass等是干什么的呢？这些成员在CentralFreeList被使用，接下来通过一个函数来解析这个初始化过程。

	void CentralFreeList::Populate() {
		// Release central list lock while operating on pageheap
		lock_.Unlock();
		//查询sizemap，获得该区间每一次拉去多少个页面
		const size_t npages = Static::sizemap()->class_to_pages(size_class_);

		Span* span;
		{
			SpinLockHolder h(Static::pageheap_lock());
			//去pageheap申请页面
			span = Static::pageheap()->New(npages);
			//初始化span的sizeclass，并对Span的每一个页面在基数树中的索引完成初始化
			//在前面提到，基数树对每一个页面对应一个指针，在pageheap申请之后，只会
			//将Span首个和最后一个页面的指针初始化，
			//这里对Span的所有页面的指针都完成初始化，之所以这样，是因为在返还内存时
			//需要首先根据内存地址定位内存页，然后定位Span，最后才将内存回收到具体的Span中
			if (span)
				Static::pageheap()->RegisterSizeClass(span, size_class_);
		}
		if (span == NULL) {
			MESSAGE("tcmalloc: allocation failed", npages << kPageShift);
			lock_.Lock();
			return;
		}
		ASSERT(span->length == npages);
		// Cache sizeclass info eagerly.  Locking is not necessary.
		// (Instead of being eager, we could just replace any stale info
		// about this span, but that seems to be no better in practice.)
		//作cache
		for (int i = 0; i < npages; i++) {
			Static::pageheap()->CacheSizeClass(span->start + i, size_class_);
		}

		// Split the block into pieces and add to the free-list
		// TODO: coloring of objects to avoid cache conflicts?
		void** tail = &span->objects;
		//获得Span所管理内存的起始地址和终止地址
		char* ptr = reinterpret_cast<char*>(span->start << kPageShift);
		char* limit = ptr + (npages << kPageShift);
		//获得每一个内存对象的大小
		const size_t size = Static::sizemap()->ByteSizeForClass(size_class_);
		int num = 0;
		//对这些内存建一个链表
		while (ptr + size <= limit) {
			*tail = ptr;
			tail = reinterpret_cast<void**>(ptr);
			ptr += size;
			//num表示一个划分了多少个对象
			num++;
		}
		ASSERT(ptr <= limit);
		*tail = NULL;
		//全部内存对象都未使用
		span->refcount = 0; // No sub-object in use yet

		// Add span to list of non-empty spans
		lock_.Lock();
		//将span放入nonempty_链表
		tcmalloc::DLL_Prepend(&nonempty_, span);
		//CentralFreeList增加了num个内存对象
		counter_ += num;
	}
	//通过内存地址定位Span
	Span* MapObjectToSpan(void* object) {
		//通过内存地址定位页ID
		const PageID p = reinterpret_cast<uintptr_t>(object) >> kPageShift;
		//通过查询基数树，来获得特定的Span
		Span* span = Static::pageheap()->GetDescriptor(p);
		return span;
	}
	
在上面函数中通过一个while循环对span的空闲内存做了一个内存对象的链表，这过程没有明确的数据结构，而是利用现有的内存来实现。如图

	Span.Object   Span.Start  Span.Length=2
	  ||                ||           ||
	  ||               ||           ||
	  ||             ||           ||
	  ||          ||           ||
	  ||       ||          ||
	  ||    ||         ||
	  ||  ||      ||
	  || ||   ||
	   ||||||
	    ||||
	     || 
	  |SizeClass|SizeClass|SizeClass|SizeClass|
	  |    Page1          |    Page1          | 
  
下面是获取一个内存对象的具体实现

	void* CentralFreeList::FetchFromSpansSafe() {
		//从非空span中获得内存对象
		void *t = FetchFromSpans();
		if (!t) {
			//如果获取不到，则先从PageMap获得一个Span
			Populate();
			//再次尝试
			t = FetchFromSpans();
		}
		return t;
	}

	void* CentralFreeList::FetchFromSpans() {
		//如果没有非空Span，则返回失败
		if (tcmalloc::DLL_IsEmpty (&nonempty_))
			return NULL;
		Span* span = nonempty_.next;

		ASSERT(span->objects != NULL);
		//被使用的内存对象多了一个
		span->refcount++;
		//将objects指向下一个内存对象，并返回当前第一个给上层应用
		void* result = span->objects;
		span->objects = *(reinterpret_cast<void**>(result));
		//如果没有内存对象了，则将Span移至空Span链表
		if (span->objects == NULL) {
			// Move to empty list
			tcmalloc::DLL_Remove(span);
			tcmalloc::DLL_Prepend(&empty_, span);
			Event(span, 'E', 0);
		}
		//总的内存对象少了一个
		counter_--;
		return result;
	}
	
释放一个内存对象

	void CentralFreeList::ReleaseListToSpans(void* start) {
		//依次释放每一个内存对象
		//记住如果有多个对象，如果被应用直接释放会有问题，因为应用可能把
		//内存中的链表结构给写坏了。
		while (start) {
			void *next = SLL_Next(start);
			ReleaseToSpans(start);
			start = next;
		}
	}

	void CentralFreeList::ReleaseToSpans(void* object) {
		Span* span = MapObjectToSpan(object);
		ASSERT(span != NULL);
		ASSERT(span->refcount > 0);

		//如果span之前已经空掉了，则将其移到非空链表
		// If span is empty, move it to non-empty list
		if (span->objects == NULL) {
			tcmalloc::DLL_Remove(span);
			tcmalloc::DLL_Prepend(&nonempty_, span);
			Event(span, 'N', 0);
		}

		//多了一个内存对象
		counter_++;
		span->refcount--;
		//若没有任何使用的内存对象，则直接释放这个span给pagemap
		if (span->refcount == 0) {
			Event(span, '#', 0);
			//减少技术
			counter_ -= ((span->length << kPageShift)
					/ Static::sizemap()->ByteSizeForClass(span->sizeclass));
			tcmalloc::DLL_Remove(span);

			// Release central list lock while operating on pageheap
			lock_.Unlock();
			{
				SpinLockHolder h(Static::pageheap_lock());
				Static::pageheap()->Delete(span);
			}
			lock_.Lock();
		} else {
			//将这个内存对象放在最前面
			*(reinterpret_cast<void**>(object)) = span->objects;
			span->objects = object;
		}
	}
	
实际使用中，使用的是以下两个接口，这个接口在前面接口的基础上增加了一个cache，这里不再说明

	void CentralFreeList::InsertRange(void *start, void *end, int N);
	int CentralFreeList::RemoveRange(void **start, void **end, int N);
	
##参考
1. <http://pan.baidu.com/s/1mgmiuqk>
