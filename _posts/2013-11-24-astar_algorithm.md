---
layout: post
title: A*算法
category: cpp
---

最近玩魔塔游戏，觉得挺有意思，不过操控没有自动寻路，很浪费时间，所以网上搜了下相关的寻路算法。找到一个A*,挺有意思，便用最熟悉的C++重新造了个轮子。

算法完全使用标准的组件，可以兼容Windows和Linux。
1. 在OpenList寻找最短路径时，为了省篇幅，使用**std::multimap**（全局排序）。为了更优，可以使用**最小堆**
1. 加载地图部分，不算太完善
1. 以下代码可以直接编译成可执行程序,先用vi（*或者其他文本编辑器*）编辑地图。**.：道路、+：障碍、s：源地址、d：目的地址、-：生成的路径**

	..............
	.....++++++++.
	....s+.....++.
	.....+...d.+..
	.....+.....+.+
	.....+++++.+..
	.....+.....++.
	.....+.++++++.
	.....+........

寻路后如下
	
	....----------
	....-++++++++-
	....s+.....++-
	.....+...d.+--
	.....+...--+-+
	.....+++++-+--
	.....+-----++-
	.....+-++++++-
	.....+--------

---

	#include <cstdio>
	#include <cstdlib>
	#include <cstring>
	#include <cassert>
	#include <cstdint>
	#include <vector>
	#include <map>
	#include <vector>

	enum{InvalidPosition = 0xFFFFFFFF};

	class Entry
	{
	public:
		enum Type_t
		{
			TypeWall, ///墙
			TypeRoad,///路
			TypeNPC,///NPC
			TypeEnemy,/// 怪物
		};
		enum Position_t
		{
			PositionNormal = 1,
			PositionOpenList,
			PositionCloseList,
		};
		
	private:
		struct Extra
		{
		public:
			Extra():
				m_position_(PositionNormal),
				m_parent_(InvalidPosition),
				m_gval_(InvalidPosition),
				m_hval_(InvalidPosition),
				m_path_flag_(false)
			{}

		public:
			Position_t m_position_;
			uint32_t m_parent_;

			uint32_t m_gval_;
			uint32_t m_hval_;

			bool m_path_flag_;
		};
	public:
		Entry(Type_t type,uint32_t index):
			m_type_(type),
			m_index_(index),
			m_extra_(NULL)
		{
		}
		~Entry()
		{
			if(m_extra_)
				delete m_extra_;
		}
		inline Type_t Type() const
		{
			return this->m_type_;
		}
		inline uint32_t Index() const
		{
			return this->m_index_;
		}
		inline Position_t Position() const
		{
			return m_extra_?m_extra_->m_position_:PositionNormal;
		}
		inline void SetPosition(Position_t position)
		{
			assert(m_extra_);
			m_extra_->m_position_ = position;
		}
		inline uint32_t FVal() const
		{
			assert(m_extra_);
			return m_extra_->m_gval_ + m_extra_->m_hval_;
		}

		inline void InitFinder(const Entry& end,uint32_t width)
		{
			assert(!m_extra_);
			uint32_t x1_ = this->m_index_%width;
			uint32_t y1_ = this->m_index_/width;
			
			uint32_t x2_ = end.m_index_%width;
			uint32_t y2_ = end.m_index_/width;
			
			m_extra_ = new Extra;
			// 这里不能用std::abs (unsigned)
			m_extra_->m_hval_ = ((x1_ >= x2_)?(x1_ - x2_):(x2_ - x1_)) +
				((y1_ >= y2_)?(y1_ - y2_):(y2_ - y1_));
		}

		inline void SetParent(const Entry& parent,uint32_t width)
		{
			assert(m_extra_ && parent.m_extra_);
				
			uint32_t x1_ = this->m_index_%width;
			uint32_t y1_ = this->m_index_/width;
			
			uint32_t x2_ = parent.m_index_%width;
			uint32_t y2_ = parent.m_index_/width;

			// 这里不能用std::abs (unsigned)
			uint32_t gval_ = ((x1_ >= x2_)?(x1_ - x2_):(x2_ - x1_)) +
				((y1_ >= y2_)?(y1_ - y2_):(y2_ - y1_));

			// 第一次设置parent
			if(m_extra_->m_parent_ == InvalidPosition)
			{
				m_extra_->m_parent_ = parent.Index();
				m_extra_->m_gval_ = gval_;
				if(parent.Index() !=  this->Index())
				{
					m_extra_->m_gval_ += parent.m_extra_->m_gval_;
				}
			}
			else
			{
				// 是否需要重新调整
				if(m_extra_->m_gval_ > parent.m_extra_->m_gval_ + gval_)
				{
					m_extra_->m_parent_ = parent.Index();
					m_extra_->m_gval_  = parent.m_extra_->m_gval_ + gval_;
				}
			}
		}
		uint32_t Parent() const
		{
			return m_extra_?m_extra_->m_parent_:InvalidPosition;
		}
		inline void FiniFinder()
		{
			assert(m_extra_);
			delete m_extra_;
			m_extra_ = NULL;
		}

		inline void SetPathFlag()
		{
			assert(m_extra_);
			m_extra_->m_path_flag_ = true;
		}
		inline bool IsPath() const
		{
			return m_extra_?m_extra_->m_path_flag_:false;
		}
	private:
		const Type_t m_type_;
		const uint32_t m_index_;
		Extra* m_extra_;
	};

	class Map
	{
		enum Direction_t
		{
			Direction_Left,
			Direction_Right,
			Direction_Top,
			Direction_Bottom,
		};
		typedef std::vector<Entry> MapData;
		typedef MapData::iterator MapDataIter;

		typedef std::multimap<uint32_t,uint32_t> OpenList;

		typedef std::vector<uint32_t> CloseList;
	public:
		Map():
			m_init_(false),
			m_width_(0),
			m_height_(0),
			m_start_(InvalidPosition),
			m_end_(InvalidPosition)
		{
		}

		/// 从字符文本中初始化
		bool InitFromFile(const char* path)
		{
			assert(path && !m_init_);
			FILE* fp_ = fopen(path,"r");
			if(!fp_)
			{
				printf("open file %s fail\n",path);
				return false;
			}
			// 也就是最大只能有100*100(或者等效）的地图
			uint8_t buf_[10240];
			size_t readed_ = fread(buf_,1,sizeof(buf_),fp_);
			if(ferror(fp_))
			{
				printf("fread file %s fail\n",path);
				fclose(fp_);
				return false;
			}

			// 分析内容
			assert(readed_ <= sizeof(buf_));
			uint8_t* tmp_buf_ = new uint8_t[readed_];

			size_t width_ = 0,height_ = 0;
			size_t cur_width_ = 0;
			size_t valid_cnt_ = 0;
			for(size_t i = 0;i < readed_;i++)
			{
				if(buf_[i] == '.' || buf_[i] == '+' ||
					buf_[i] == 's' || buf_[i] == 'd')
				{
					// 判断开始结束
					if(buf_[i] == 's')
					{
						// 这里直接做断言（判断重复开始/结束位置），不做判断（下同）
						assert(InvalidPosition == this->m_start_);
						this->m_start_ = valid_cnt_;
						buf_[i] = '.';
					}
					else if(buf_[i] == 'd')
					{
						assert(InvalidPosition == this->m_end_);
						this->m_end_ = valid_cnt_;
						buf_[i] = '.';
					}

					cur_width_ ++;
					*(tmp_buf_ + valid_cnt_) = buf_[i];
					valid_cnt_ ++;
				}
				else if(buf_[i] == '\n')
				{
					// 第一行决定宽度
					if(!width_)
					{
						assert(cur_width_);
						width_ = cur_width_;
					}
					// 后续宽度必须一致
					else if(width_ != cur_width_)
					{
						delete [] tmp_buf_;
						fclose(fp_);
						return false;
					}
					cur_width_ = 0;
					height_ ++;
				}
			}

			// 是否有开始，结束点（由于符号的设计，开始结束点不可能在墙上，也不可能重复）
			if(this->m_start_ == InvalidPosition ||
				this->m_end_ == InvalidPosition)
			{
				printf("there is NO start(%u)/end(%u) position\n",this->m_start_,this->m_end_);
				delete [] tmp_buf_;
				fclose(fp_);
				return false;
			}

			// 尺寸
			m_width_ = width_;
			m_height_ = height_;

			// 数据
			//m_map_data_.resize(valid_cnt_);
			//memcpy(&(m_map_data_[0]),tmp_buf_,valid_cnt_);
			Entry::Type_t type_;
			for(size_t i = 0;i < valid_cnt_;i++)
			{
				if(tmp_buf_[i] == '.')
					type_ = Entry::TypeRoad;
				else 
					type_ = Entry::TypeWall;
				m_map_data_.push_back(Entry(type_,i));
			}

			delete [] tmp_buf_;
			fclose(fp_);
			m_init_ = true;
			return true;
		}

		/// 打印
		void Print()
		{
			assert(this->m_init_);
			size_t cnt_ = 0;
			for(MapDataIter iter_ = m_map_data_.begin();
				iter_ != m_map_data_.end();
				++ iter_)
			{
				if(cnt_ == this->m_start_)
				{
					printf("s");
				}
				else if(cnt_ == this->m_end_)
				{
					printf("d");
				}
				else if(iter_->IsPath())
				{
					printf("-");
				}
				else if(iter_->Type() == Entry::TypeRoad)
				{
					printf(".");
				}
				else 
				{
					printf("+");
				}
				cnt_ ++;
				if(cnt_%m_width_ == 0)
					printf("\n");
			}
		}

		/// 寻路
		inline bool FindWay()
		{
			return FindWayImp(m_start_,m_end_);
		}

		void EndFindWay()
		{
			for(OpenList::iterator iter_ = m_openlist_.begin();
				iter_ != m_openlist_.end();
				++ iter_)
			{
				m_map_data_[iter_->second].FiniFinder();
			}
			// 清空信息
			m_openlist_.clear();
			
			for(CloseList::iterator iter_ = m_closelist_.begin();
				iter_ != m_closelist_.end();
				++ iter_)
			{
				m_map_data_[*iter_].FiniFinder();
			}
			m_closelist_.clear();
		}
	private:
		void Add2Openlist(Entry& entry)
		{
			entry.SetPosition(Entry::PositionOpenList);
			m_openlist_.insert(OpenList::value_type(entry.FVal(),entry.Index()));
		}
		void Add2Closelist(Entry& entry)
		{
			entry.SetPosition(Entry::PositionCloseList);
			m_closelist_.push_back(entry.Index());
		}

		uint32_t GetNeighbor(Direction_t direction,uint32_t cur)
		{
			if(direction == Direction_Left)
			{
				if(cur%m_width_ > 0)
					return cur - 1;
			}
			else if(direction == Direction_Right)
			{
				if(cur%m_width_ + 1 < m_width_)
					return cur + 1;
			}
			else if(direction == Direction_Top)
			{
				if(cur/m_width_ > 0)
					return cur - m_width_;
			}
			else if(direction == Direction_Bottom)
			{
				if(cur/m_width_ + 1 < m_height_)
					return cur + m_width_;
			}
			return InvalidPosition;
		}
		
		bool ProcessNeighborEx(Direction_t direction,uint32_t cur,uint32_t end)
		{
			uint32_t index_ = GetNeighbor(direction,cur);
			if(index_ == InvalidPosition)
				return false;

			// 排除障碍物
			Entry& entry_ = m_map_data_[index_];
			if(entry_.Type() != Entry::TypeRoad)
				return false;

			// 排除在closelist中
			if(entry_.Position() == Entry::PositionCloseList)
				return false;

			// 如果在openlist中
			if(entry_.Position() == Entry::PositionOpenList)
			{
				// 重新计算G值
				entry_.SetParent(m_map_data_[cur],m_width_);
				printf("update neighbor %u (%uX%u) %u\n",
					index_,index_%m_width_,index_/m_width_,entry_.FVal());
			}
			else
			{
				// 全新的
				entry_.InitFinder(m_map_data_[end],m_width_);
				entry_.SetParent(m_map_data_[cur],m_width_);
				this->Add2Openlist(entry_);
				printf("add neighbor %u (%uX%u) %u\n",
					index_,index_%m_width_,index_/m_width_,entry_.FVal());
			}
			return index_ == end;
		}

		bool FindWayImp(uint32_t start,uint32_t end)
		{
			assert(this->m_init_);
			assert(start != end);
			assert(start < m_map_data_.size() && 
				end <= m_map_data_.size());

			// 先放入Open列表
			{
				Entry& start_ = m_map_data_[start];
				start_.InitFinder(m_map_data_[end],m_width_);
				start_.SetParent(start_,m_width_);
				this->Add2Openlist(start_);
			}

			uint32_t cur_index_;
			do{
				if(m_openlist_.empty())
				{
					printf("can NOT find path from %u to %u\n",start,end);
					return false;
				}

				/// 取最小的f值，并转移到closelist
				cur_index_ = InvalidPosition;
				{
					OpenList::iterator iter_ = m_openlist_.begin();
					cur_index_ = iter_->second;
					
					printf("process %u (%uX%u) %u\n",
						cur_index_,cur_index_%m_width_,cur_index_/m_width_,iter_->first);

					m_openlist_.erase(iter_);
					this->Add2Closelist(m_map_data_[cur_index_]);
				}

				if(ProcessNeighborEx(Direction_Left,cur_index_,end) ||
					ProcessNeighborEx(Direction_Right,cur_index_,end) ||
					ProcessNeighborEx(Direction_Top,cur_index_,end) ||
					ProcessNeighborEx(Direction_Bottom,cur_index_,end))
					break;
			}while(true);

			// 回溯路径
			while(cur_index_ != start)
			{
				m_map_data_[cur_index_].SetPathFlag();
				cur_index_ = m_map_data_[cur_index_].Parent();
			};
			return true;
		}
	private:
		bool m_init_;
		uint32_t m_width_;
		uint32_t m_height_;
		MapData m_map_data_;

		uint32_t m_start_;
		uint32_t m_end_;

		OpenList m_openlist_;
		CloseList m_closelist_;
	};

	int main()
	{
		Map map_;
		if(!map_.InitFromFile("map.txt"))
			return -1;
		map_.Print();
		map_.FindWay();
		map_.Print();
		map_.EndFindWay();
		map_.Print();
		map_.FindWay();
		map_.Print();
		return 0;
	}
	
	
##参考
1. <http://www.policyalmanac.org/games/aStarTutorial.htm>
1. <http://www.cppblog.com/mythit/archive/2013/11/07/80492.html>