---
layout: post
title: Google TCMalloc学习笔记2-页管理
category: cpp
---

写于 **2011年11月5日**

tcmalloc底层的内存分配直接使用操作系统的内存页分配接口来分配内存，这些也通过一个PageHeap来管理。整个内存流向如下图

	ThreadCache
	  ||
	CentralFreelist
	  ||
	PageHeap
	  ||
	操作系统

PageHeap直接从操作系统获得一个个内存页，并以页面的形式保存，中央空闲链（CentralFreeList）从PageHeap获得一个个页面，然后分成若干对象（一个对象就是上文所说的区间大小），线程缓存（ThreadCache）从CentralFreeList获得若干对象并交付应用程序。

在一个32位的地址空间中，中央阵列由一个2层的基数树来表示，其中根包含了32个条目，每个叶包含了 215个条目（一个32为地址空间包含了 220个 4K 页面，所以这里树的第一层则是用25整除220个页面）。这就导致了中央阵列的初始内存使用需要128KB空间（215*4字节），看上去还是可以接受的。

	1 2 3 4 ... ... 29 30 31
		      ||
		      ||
	      Page1
	      Page2
	      ... ...  2的15次方
	      PageN-1
	      PageN

如上图，整个基数树包含两层，第一层是一个32个元素的指针数组，每一个元素指向一个结构，这个结构包含一个215个条目的指针数组，这个数组元素（存储一个指针）指向一个内存页（到后面会发现，其实是指向一个Span）。每一个内存也是4K，那么一个就包含4k x 32 x 215 = 4G，恰好包含整个虚拟内存空间，tcmalloc这样做有一个好处。假设第一次分配一个页面，内存地址是X，接下来又分配一个内存也，内存地址是X+4K，现在如果有一个业务需要6K内存，正常情况下是做不到的，但是这里可以看到其实两个内存也是相连的，完全可以将其合并，并一起使用。为了做到这一点，就必须对所有内存也的地址进行排序，上面的结构就很方便（数组寻址），除了有点费内存，整个结构初始化之后需要4M内存。另一个问题是，如果打算在内存过多时返还给操作系统一部分，这里合并的时候还是必须记录每一次申请内存时的边界，否则释放内存时将出错，不过就我所知，tcmalloc并没有打算返还内存。


每一次申请内存，tcmalloc将通过一个span来管理这块内存，这些信息包括这个页在基数树的索引，以及页的数量。Span的定义如下（有删减）

	// Information kept for a span (a contiguous run of pages).
	struct Span {
		//此连续页面内存块，第一个页面的索引
		PageID        start;          // Starting page number
		//连续页面长度
		Length        length;         // Number of pages in span

		//span的位置，这些位置定义如下
		unsigned int  location : 2;   // Is the span on a freelist, and if so, which?

		//分别是“正在使用”，“未使用过的空闲块”，“使用过但是已经不再使用过的空闲块”
		// What freelist the span is on: IN_USE if on none, or normal or returned
		enum { IN_USE, ON_NORMAL_FREELIST, ON_RETURNED_FREELIST };
	};

每一次获得页面并准备插入到基数树之前，tcmalloc总是先检查与它相邻的前后两个页面是否也存在，并且是同样的属性（空闲/使用/…），若相同，则将其合并而组成一个更大的Span，

	|=|=|=|=|=|=|=|=|=|=|=|=|=|=|=|=|=|=|=|=
	| |     | |     |       |     |     |
	|a|     |b|     |    c  |     |  d  |
	
如上图，下面的a、b、c、d就是一个Span，Span a包含两个页面，Span c包含五个页面，对于多个页面的Span，中间页面的基数树索引并不一定指向实际的Span。比如Span c中间三个页面的索引就可能不存在。

根据源码看一下具体的实现（32位系统为例），基数树的实现如下，

	template<int BITS> class MapSelector {
	public:
		typedef TCMalloc_PageMap3<BITS - kPageShift> Type;
		typedef PackedCache<BITS - kPageShift, uint64_t> CacheType;
	};

	// A two-level map for 32-bit machines
	//kPageShift = 12  （4K）
	template<> class MapSelector<32> {
	public:
		typedef TCMalloc_PageMap2<32 - kPageShift> Type;
		typedef PackedCache<32 - kPageShift, uint16_t> CacheType;
	};

	
如果是32位平台，会使用下面模板特化的版本。

	template <int BITS> //BITS = 20
	class TCMalloc_PageMap2 {
	private:
		// Put 32 entries in the root and (2^BITS)/32 entries in each leaf.
		static const int ROOT_BITS = 5;
		static const int ROOT_LENGTH = 1 << ROOT_BITS; //32

		static const int LEAF_BITS = BITS - ROOT_BITS; //15
		static const int LEAF_LENGTH = 1 << LEAF_BITS; //1024*32

		// Leaf node
		//每一个根数组元素指向的结构，这个结构包含一个数组
		struct Leaf {
			void* values[LEAF_LENGTH];
		};

		//32各根指针数组
		Leaf* root_[ROOT_LENGTH]; // Pointers to 32 child nodes
		void* (*allocator_)(size_t); // Memory allocator

	public:
		typedef uintptr_t Number;

		explicit TCMalloc_PageMap2(void* (*allocator)(size_t)) {
			allocator_ = allocator;
			memset(root_, 0, sizeof(root_));
		}

		void* get(Number k) const {
			//获得根节点的索引
			const Number i1 = k >> LEAF_BITS;
			//获得特定页结构 数组的索引
			const Number i2 = k & (LEAF_LENGTH - 1);
			if ((k >> BITS) > 0 || root_[i1] == NULL) {
				return NULL;
			}
			//可见效率还是很高的
			return root_[i1]->values[i2];
		}

		void set(Number k, void* v) {
			ASSERT(k >> BITS == 0);
			const Number i1 = k >> LEAF_BITS;
			const Number i2 = k & (LEAF_LENGTH - 1);
			root_[i1]->values[i2] = v;
		}

		//...
	}

使用如下结构来管理页面，

	// Pick the appropriate map and cache types based on pointer size
	typedef MapSelector<kAddressBits>::Type PageMap;
	typedef MapSelector<kAddressBits>::CacheType PageMapCache;
	//基数树
	PageMap pagemap_;
	//一个缓存类，缓存一些数据。没仔细看，里面用hash实现快速查询
	mutable PageMapCache pagemap_cache_;

	// We segregate spans of a given size into two circular linked
	// lists: one for normal spans, and one for spans whose memory
	// has been returned to the system.
	//一个Span链表头

	struct SpanList {
		Span normal;
		Span returned;
	};

	// List of free spans of length >= kMaxPages
	//超过256个页面的连续块，使用large span来存储
	SpanList large_;

	//小于256个页面的连续块，对每一种页面数大小，使用一个链表，这些链表通过free_数组索引
	// Array mapping from span length to a doubly linked list of free spans
	SpanList free_[kMaxPages];

	//统计信息
	// Statistics on system, free, and unmapped bytes
	Stats stats_;

	//插入一个空闲页面 
	void PageHeap::PrependToFreeList(Span* span) {
		ASSERT(span->location != Span::IN_USE);
		//如果大于256，则插入large_，否则插入数组free_对应的链表
		SpanList* list =
				(span->length < kMaxPages) ? &free_[span->length] : &large_;
		if (span->location == Span::ON_NORMAL_FREELIST) {
			stats_.free_bytes += (span->length << kPageShift);
			//置入链表中
			DLL_Prepend(&list->normal, span);
		} else {
			stats_.unmapped_bytes += (span->length << kPageShift);
			//置入链表中
			DLL_Prepend(&list->returned, span);
		}
	}

	//删除一个空闲页面
	void PageHeap::RemoveFromFreeList(Span* span) {
		ASSERT(span->location != Span::IN_USE);
		if (span->location == Span::ON_NORMAL_FREELIST) {
			stats_.free_bytes -= (span->length << kPageShift);
		} else {
			stats_.unmapped_bytes -= (span->length << kPageShift);
		}
		//从链表中删除，因为是一个双向链表，所以这里并不需要知道它属于哪一个链表
		DLL_Remove(span);
	}

分配N个页面

	//请求分配N个页面
	Span* PageHeap::New(Length n) {
		ASSERT (Check());ASSERT(n > 0);
		//看当前还有没有足够的页面
		Span* result = SearchFreeAndLargeLists(n);
		if (result != NULL)
		//有了，

		return result;

		//否则像操作系统要一些
		// Grow the heap and try again.
		if (!GrowHeap(n)) {
			ASSERT(Check());
			return NULL;
		}
		//再次分配
		return SearchFreeAndLargeLists(n);
	}

	//查找当前是否有足够大的空闲页面
	Span* PageHeap::SearchFreeAndLargeLists(Length n) {
		ASSERT (Check());ASSERT(n > 0);
		//依次从匹配的页面链表开始查找，例如申请两个页面，先从free_[1]查找
		//若没有，则往上查找，比如free_[3]找到了，则获得对应的span，然后分成两个
		//span，给出一个页面的span返回，然后将三个页面的span交给free_[2]
		// Find first size >= n that has a non-empty list
		for (Length s = n; s < kMaxPages; s++) {
			Span* ll = &free_[s].normal;
			// If we're lucky, ll is non-empty, meaning it has a suitable span.
			if (!DLL_IsEmpty(ll)) {
				ASSERT(ll->next->location == Span::ON_NORMAL_FREELIST);
				return Carve(ll->next, n);
			}
			// Alternatively, maybe there's a usable returned span.
			ll = &free_[s].returned;
			if (!DLL_IsEmpty(ll)) {
				ASSERT(ll->next->location == Span::ON_RETURNED_FREELIST);
				return Carve(ll->next, n);
			}
		}
		//找不到，则从大页面链表查找
		// No luck in free lists, our last chance is in a larger class.
		return AllocLarge(n);// May be NULL
	}

	Span* PageHeap::Carve(Span* span, Length n) {
		ASSERT(n > 0);
		ASSERT(span->location != Span::IN_USE);
		//保存原来的状态，
		const int old_location = span->location;
		//从空闲列表中删除
		RemoveFromFreeList(span);
		//将他标记为使用状态
		span->location = Span::IN_USE;
		Event(span, 'A', n);

		const int extra = span->length - n;
		ASSERT(extra >= 0);
		//如果这个Span比实际的大,则将多余的部分重新分配一个span
		if (extra > 0) {
			//新建一个span
			Span* leftover = NewSpan(span->start + n, extra);
			//恢复原来的状态
			leftover->location = old_location;
			Event(leftover, 'S', extra);
			//初始化span
			RecordSpan(leftover);
			//插入空闲列表
			PrependToFreeList(leftover); // Skip coalescing - no candidates possible
			span->length = n;
			//插入基数树
			pagemap_.set(span->start + n - 1, span);
		}
		ASSERT (Check());
		return span	;
	}

	Span* PageHeap::AllocLarge(Length n) {
		// find the best span (closest to n in size).
		// The following loops implements address-ordered best-fit.
		Span *best = NULL;

		//依次查询normal和returned链表
		//因为超过256个页面的span都放在这里，所以这里总是找一个合适并且最小的span
		// Search through normal list
		for (Span* span = large_.normal.next; span != &large_.normal;
				span = span->next) {
			if (span->length >= n) {
				if ((best == NULL)
				//如果页数量小于当前最合适的
						|| (span->length < best->length)
						//或者span的初始地址较低
						|| ((span->length == best->length)
								&& (span->start < best->start))) {
					best = span;
					ASSERT(best->location == Span::ON_NORMAL_FREELIST);
				}
			}
		}

		// Search through released list in case it has a better fit
		for (Span* span = large_.returned.next; span != &large_.returned; span =
				span->next) {
			if (span->length >= n) {
				if ((best == NULL) || (span->length < best->length)
						|| ((span->length == best->length)
								&& (span->start < best->start))) {
					best = span;
					ASSERT(best->location == Span::ON_RETURNED_FREELIST);
				}
			}
		}

		return best == NULL ? NULL : Carve(best, n);
	}

	bool PageHeap::GrowHeap(Length n) {
		ASSERT(kMaxPages >= kMinSystemAlloc);
		if (n > kMaxValidPages)
			return false;
		//tcmalloc限制了最小分配的页面数，默认是256（1M），所以即使只需要一个页面，也
		//直接给申请256个页面
		Length ask =
				(n > kMinSystemAlloc) ? n : static_cast<Length>(kMinSystemAlloc);
		size_t actual_size;
		void* ptr = TCMalloc_SystemAlloc(ask << kPageShift, &actual_size,
				kPageSize);
		if (ptr == NULL) {
			//如果失败了，则尝试申请上层需要的页面数
			if (n < ask) {
				// Try growing just "n" pages
				ask = n;
				ptr = TCMalloc_SystemAlloc(ask << kPageShift, &actual_size,
						kPageSize);
			}
			if (ptr == NULL)
				return false;
		}
		ask = actual_size >> kPageShift;
		RecordGrowth(ask << kPageShift);

		uint64_t old_system_bytes = stats_.system_bytes;
		stats_.system_bytes += (ask << kPageShift);
		const PageID p = reinterpret_cast<uintptr_t>(ptr) >> kPageShift;
		ASSERT(p > 0);

		// If we have already a lot of pages allocated, just pre allocate a bunch of
		// memory for the page map. This prevents fragmentation by pagemap metadata
		// when a program keeps allocating and freeing large blocks.
		//如果申请内存之后，跨过了kPageMapBigAllocationThreshold（128M）这个阀值，
		//就让基数树预分配更多的结构来存储span
		if (old_system_bytes < kPageMapBigAllocationThreshold
				&& stats_.system_bytes >= kPageMapBigAllocationThreshold) {
			pagemap_.PreallocateMoreMemory();
		}

		// Make sure pagemap_ has entries for all of the new pages.
		// Plus ensure one before and one after so coalescing code
		// does not need bounds-checking.
		//确保基数树，申请的页面数，以及前后两个页面的数据结构存在
		if (pagemap_.Ensure(p - 1, ask + 2)) {
			// Pretend the new area is allocated and then Delete() it to cause
			// any necessary coalescing to occur.
			//创建一个新的Span来管理这些内存
			Span* span = NewSpan(p, ask);
			//插入到基数树
			RecordSpan(span);
			//进一步检查这个span
			Delete(span);
			ASSERT (Check());
			return true		;
		} else {
			// We could not allocate memory within "pagemap_"
			// TODO: Once we can return memory to the system, return the new span
			return false;
		}
	}

	void PageHeap::Delete(Span* span) {
		const Length n = span->length;
		span->sizeclass = 0;
		span->sample = 0;
		span->location = Span::ON_NORMAL_FREELIST;
		Event(span, 'D', span->length);
		//检查前后的span是否存在，并且具有相同的属性
		MergeIntoFreeList(span); // Coalesces if possible
		//是否需要释放内存，但是这函数似乎从来都没有释放内存，
		IncrementalScavenge(n);
		ASSERT (Check());}

	void PageHeap::MergeIntoFreeList(Span* span) {
		ASSERT(span->location != Span::IN_USE);

		// Coalesce -- we guarantee that "p" != 0, so no bounds checking
		// necessary.  We do not bother resetting the stale pagemap
		// entries for the pieces we are merging together because we only
		// care about the pagemap entries for the boundaries.
		//
		// Note that only similar spans are merged together.  For example,
		// we do not coalesce "returned" spans with "normal" spans.
		const PageID p = span->start;
		const Length n = span->length;
		Span* prev = GetDescriptor(p - 1);
		//如果前一个span不为空，并且属性一致，则合并之
		if (prev != NULL && prev->location == span->location) {
			// Merge preceding span into this span
			ASSERT(prev->start + prev->length == p);
			//保存前一个span的长度并删除对应的span
			const Length len = prev->length;
			RemoveFromFreeList(prev);
			DeleteSpan(prev);

			//将当前的span长度家常，并将初始页面往前移动
			span->start -= len;
			span->length += len;
			//插入基数树
			pagemap_.set(span->start, span);
			Event(span, 'L', len);
		}
		Span* next = GetDescriptor(p + n);
		//如果后一个span不为空，并且属性一致，则合并之
		if (next != NULL && next->location == span->location) {
			// Merge next span into this span
			ASSERT(next->start == p + n);
			//保存旧的长度，并删除span
			const Length len = next->length;
			RemoveFromFreeList(next);
			DeleteSpan(next);
			//将span长度增加
			span->length += len;
			//写入基数树
			pagemap_.set(span->start + span->length - 1, span);
			Event(span, 'R', len);
		}
		//放入空闲链表
		PrependToFreeList(span);
	}


##参考
1. <http://shiningray.cn/tcmalloc-thread-caching-malloc.html>
1. <http://pan.baidu.com/s/1mgmiuqk>
