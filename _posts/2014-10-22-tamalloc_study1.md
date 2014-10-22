---
layout: post
title: Google TCMalloc学习笔记1-区间设置
category: cpp
---

写于 **2011年11月4日**

TCMalloc是[google-perftools](http://code.google.com/p/gperftools/)套件的一部分, Google-perftools是Google开发的开放源代码C++性能调整工具库, 特别为多线程C++程序的开发提供支持，除了TCMalloc外, Google-perftools还包括了heap-checker, heap-profiler和cpu-profiler等工具, google-perftools在New BSD License下发布。more。

TCMalloc作为一种典型的slot模型的实现，自然要将内存分成若干区间。所以本文首先从这里入手一窥TCMalloc的内幕。

所谓区间就是一个内存区段，比如8～16字节（不包括8），15字节就位于这个区间中，一般情况下，内存分配器的区间从最小值网上，对每一个区间只需要一个值就可以了，比如8、16、32、48。第一个区间就是0～8，第四个区间就是32～48。当需要分配33字节内存，就从第四个区间分配。

内存区间的划分一般是设定一个初值，然后根据特定的算法来获得下一个区间，最后限定一个可分配的最大区间，或者最大的区间个数。当超过这个最大区间后就使用系统默认分配或者其他方式。最简单的可以是一个y=tx，如memcached的内存分配器，下一个区间就是上一个区间的t倍。比如2、4、8、16。当然作为一种优化，一般会考虑将区间用8字节对齐。另一种简单的方式y=t+x，例如侯捷老师的大作《stl源码剖析》中讲述的stl内存分配器。区间是8、16、24、32、48…。tcmalloc的区间设置在源码common.cc/.h。实现相对复杂一点。

tcmalloc区间初值是8，接下来的区间等于上一个区间加上一个可变的T，这个T的初值也是8。假设当前区间的最大值是X，那么当LgFloor(X)和LgFloor(X-1) (X-1表示上一个区间的最大值，初始值为-1) 不等，则更新T，这个算法如下：
      
	 //size 为当前区间最大值
	int AlignmentForSize(size_t size) {
		int alignment = kAlignment;
		if (size >= 2048) {
			// Cap alignment at 256 for large sizes.
			alignment = 256;
		} else if (size >= 128) {
			// Space wasted due to alignment is at most 1/8, i.e., 12.5%.
			alignment = (1 << LgFloor(size)) / 8;
		} else if (size >= 16) {
			// We need an alignment of at least 16 bytes to satisfy
			// requirements for some SSE types.
			alignment = 16;
		}
		return alignment;
	}

lg(x)的算法如下

	static inline int LgFloor(size_t n) {
		int log = 0;
		//这里依次检查
		/*
		 * n >> 16
		 * n >> 8
		 * n >> 4
		 * n >> 2
		 * n >> 1
		 * n >> 0
		 * 若前一次 n >> x 不等于0，则n = n >> x
		 */
		//实际上此算法就是获得32位数，二进制表示中为1的最高为
		//比如 LgFloor(10) = 2 LgFloor(10000) = 5
		for (int i = 4; i >= 0; --i) {
			int shift = (1 << i);
			size_t x = n >> shift;
			if (x != 0) {
				n = x;
				log += shift;
			}
		}
		return log;
	}
	

tcmalloc区间的最大值是32k，具体的初始化在SizeMap::Init函数中。tcmalloc通过SizeMap类来处理区间相关的逻辑，这个类是个单体。tcmalloc通过四个数组来存储区间相关的信息

	class SizeMap {
	private:
		//在线程本地存储从中央存储区获得内存时需要移动的对象数量
		//一个对象就是该区间的内存大小，比如第一个区间的大小是8，那么一个对象就是8字节。
		//线程本地存储每一次获取内存大小是64k，那么一次性就需要移动64*1024/8
		// Number of objects to move between a per-thread list and a central
		// list in one shot.  We want this to be not too small so we can
		// amortize the lock overhead for accessing the central list.  Making
		// it too big may temporarily cause unnecessary memory wastage in the
		// per-thread free list until the scavenger cleans up the list.
		int num_objects_to_move_[kNumClasses];

		//-------------------------------------------------------------------
		// Mapping from size to size_class and vice versa
		//-------------------------------------------------------------------

		// Sizes <= 1024 have an alignment >= 8.  So for such sizes we have an
		// array indexed by ceil(size/8).  Sizes > 1024 have an alignment >= 128.
		// So for these larger sizes we have an array indexed by ceil(size/128).
		//
		// We flatten both logical arrays into one physical array and use
		// arithmetic to compute an appropriate index.  The constants used by
		// ClassIndex() were selected to make the flattening work.
		//
		// Examples:
		//   Size       Expression                      Index
		//   -------------------------------------------------------
		//   0          (0 + 7) / 8                     0
		//   1          (1 + 7) / 8                     1
		//   ...
		//   1024       (1024 + 7) / 8                  128
		//   1025       (1025 + 127 + (120<<7)) / 128   129
		//   ...
		//   32768      (32768 + 127 + (120<<7)) / 128  376
		static const int kMaxSmallSize = 1024;
		static const size_t kClassArraySize = (((1 << kPageShift) * 8u + 127
				+ (120 << 7)) >> 7) + 1;
		//通过一个特定大小来索引区间，考虑到tcmalloc的算法相对复杂，每一次申请内存都动态的计算
		//区间不划算，所以这里建一个索引来加速。比如需要分配9字节，就知道应该从第二个区间分配，
		//实际分配16个字节
		unsigned char class_array_[kClassArraySize];

		//存储第kNumClasses个区间的大小（最大值）
		// Mapping from size class to max size storable in that class
		size_t class_to_size_[kNumClasses];

		//存储第kNumClasses个区间的page数量（一个page就是操作系统的页大小，默认是4KB）
		// Mapping from size class to number of pages to allocate at a time
		size_t class_to_pages_[kNumClasses];
	};

很多人可能会觉得class_to_pages_很奇怪，区间小于4K就一个页面，小于8K就两个页面…。这里举个例子，加入一个页面是100字节，加入区间是51字节，那一个页面只能分配一个对象，几乎浪费一半内存。那怎么解决呢？如果对这个区间一次性使用两个页面，那么两个页面可以分配3个对象，是不是浪费的内存减少到1/4。tcmalloc会保证内核一个区间的内存浪费率低于1/8。

	void SizeMap::Init() {

		// Compute the size classes we want to use
		int sc = 1; // Next size class to assign
		int alignment = kAlignment; //8字节
		int last_lg = -1;
		for (size_t size = kAlignment; size <= kMaxSize; size += alignment) {
			int lg = LgFloor(size);
			//如果LgFloor不一致
			if (lg > last_lg) {
				//重新计算区间累加的差值
				// Increase alignment every so often to reduce number of size classes.
				alignment = AlignmentForSize(size);
				last_lg = lg;
			}

			//如果浪费内存严重，则增加页面数，以保证更少的内存浪费
			// Allocate enough pages so leftover is less than 1/8 of total.
			// This bounds wasted space to at most 12.5%.
			size_t psize = kPageSize;
			while ((psize % size) > (psize >> 3)) {
				psize += kPageSize;
			}
			//最后获得实际需要分配的页面数
			const size_t my_pages = psize >> kPageShift;

			//如果上一区间和当前区间的对象数量一致，则没有区分两个两个区间的必要。
			//假如区间51～55和区间55～60都分配一个内存页（100字节，不考虑1/8的浪费检测）
			//那么两个区间都只能分配一个对象，当分配54字节和56字节没有区别，都需要100字节。
			if (sc > 1 && my_pages == class_to_pages_[sc - 1]) {
				// See if we can merge this into the previous class without
				// increasing the fragmentation of the previous class.
				const size_t my_objects = (my_pages << kPageShift) / size;
				const size_t prev_objects = (class_to_pages_[sc - 1] << kPageShift)
						/ class_to_size_[sc - 1];
				if (my_objects == prev_objects) {
					// Adjust last class to include this size
					class_to_size_[sc - 1] = size;
					continue;
				}
			}

			// Add new class
			class_to_pages_[sc] = my_pages;
			class_to_size_[sc] = size;
			sc++;
		}

		// Initialize the mapping arrays
		int next_size = 0;
		//对每一个区间的大小分别作索引
		for (int c = 1; c < kNumClasses; c++) {
			const int max_size_in_class = class_to_size_[c];
			//对此区间每隔8字节做一次索引，将区间索引值保存在class_array_
			for (int s = next_size; s <= max_size_in_class; s += kAlignment) {
				class_array_[ClassIndex(s)] = c;
			}
			next_size = max_size_in_class + kAlignment;
		}

		//计算64K有多少对象
		// Initialize the num_objects_to_move array.
		for (size_t cl = 1; cl < kNumClasses; ++cl) {
			num_objects_to_move_[cl] = NumMoveSize(ByteSizeForClass(cl));
		}
	}

##参考
1. <http://pan.baidu.com/s/1mgmiuqk>
