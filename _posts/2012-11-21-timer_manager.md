---
layout: post
title:  在单线程中管理多个定时器
category: cpp
---

休眠都是使用简单的Sleep函数，你可以将它整合到select，epoll里面去

    #ifdef _WIN32
    #include <windows.h>


    typedef DWORD Time_t;
    inline Time_t GetCurTime()
    {
        return ::GetTickCount();
    }


    #else // __linux__

    #include <time.h>
    #include <unistd.h>
    #include <cstdlib>
    #include <cstring>

    typedef long long Time_t;
    inline Time_t GetCurTime()
    {
        struct timespec tp_;
        if( -1 == clock_gettime(CLOCK_MONOTONIC,&tp_))
        {
            exit(1);
        }
        // 后者是纳秒
        return tp_.tv_sec*1000 + tp_.tv_nsec/(1000*1000);
    }
    inline int Sleep(int milliseconds)
    {
        return usleep(milliseconds*1000);
    }
    #endif

    #include <cassert>
    #include <cstdio>
    // linux 需要链接librt，  g++ xx.cpp -lrt

    class MinHeap;
    class Event
    {
        friend class MinHeap;
        //64位也足够了
        enum{HEAP_MAX_INDEX = 0xFFFFFFFF};
    public:
        Event():
            m_heap_index_(HEAP_MAX_INDEX),
            m_timeout_(-1),
            m_next_timeout_(-1),
            m_reuse_(false)
        {
        }

        //比较两个超时时间
        inline bool			Earlier(const Event& ev) const
        {
            return m_next_timeout_ < ev.m_next_timeout_;
        }

        //超时设置,不能中途取消超时
        inline void			SetTimeout(int timeout)
        {
            if(timeout > 0 && this->m_timeout_ == -1)
            {
                this->m_timeout_ = timeout;
                this->ResetNextTimeout();
                return;
            }
            assert(0);
        }
        inline int			Timeout() const{
            return this->m_timeout_;
        }
        inline void			ResetNextTimeout(){
            this->m_next_timeout_ =  ::GetCurTime() + m_timeout_;
        }
        inline Time_t	NextTimeout() const{
            return this->m_next_timeout_;
        }

        //////////////////////////////////////////////
        inline size_t		HeapIndex() const
        {
            return this->m_heap_index_;
        }
        inline void			UpdateHeapIndex(size_t index){
            m_heap_index_ = index;
        }

        void				Run()
        {
            printf("hello world %d\n",m_timeout_);
        }
        
        //////////////////////////////////////////////
        bool				IsReuse() const
        {
            return this->m_reuse_;
        }
        void				SetReuseFlag(bool reuse)
        {
            this->m_reuse_ = reuse;
        }
    private:
        //最小堆中的索引
        size_t				m_heap_index_;
        //超时相关（一个时间间隔），单位（ms)
        int					m_timeout_;
        //下一次超时的绝对时间
        Time_t				m_next_timeout_;
        // 是否重复
        bool				m_reuse_;
    };

    // 定时器的最小堆
    class MinHeap
    {
    public:
        MinHeap():
            m_data_(NULL),
            m_cnt_(0),
            m_total_(0)
        {
        }

        ~MinHeap()
        {
            if(m_data_)
            {
                // 务必删除动态分配的内存
                delete [] m_data_;
            }
        }

        void			Push(Event* value);
        Event*			Pop();
        Event*			Top();
        void			Erase(Event* value);
        inline bool		Empty() const
        {
            return !m_cnt_;
        }

        // 清空，但是内存并未释放
        inline void		Clear()
        {
            m_cnt_ = 0;
        }

    private:
        void			ShiftUp(size_t hole_index,Event* value);
        void			ShiftDown(size_t hole_index, Event* value);
        void			Reserve(size_t total);

        inline bool		Less(const Event* left,const Event* right)
        {
            return left->Earlier(*right);
        }
    private:
        // 事件数组，不管理内存
        Event**			m_data_;
        // 当前的Event数量
        size_t			m_cnt_;
        // 总的buf大小，若不够，需要重新分配一块更大的内存
        size_t			m_total_;
    };

    // 预留空间，若空间够用，将不做任何改变
    void			MinHeap::Reserve(size_t total)
    {
        if (m_total_ < total)
        {
            size_t new_total_ = m_total_?m_total_*2:8;
            Event** new_data_ = new Event*[new_total_*sizeof(int)];
            memset(new_data_,0,new_total_*sizeof(int));
            if(m_data_)
            {
                // 如果此前存在数据，那么需要拷贝有效部分
                if(m_cnt_)
                    memcpy(new_data_,m_data_,m_cnt_*sizeof(int));
                // 并释放内存
                delete [] m_data_;
            }
            m_data_ = new_data_;
            m_total_ = new_total_;
        }
    }

    // 希望往hole_index插入元素value
    //  根据hole_index的父节点大小，依次网上升
    //	前提：hole_index后面的节点都比它大
    void			MinHeap::ShiftUp(size_t hole_index,Event* value)
    {
        assert(value);
        // 获得父亲节点的索引
        // 若hole_index ==0，parent将是一个非法自，不过后面有判断
        size_t parent = (hole_index - 1) / 2;
        // 若hole_index不是根节点，并且比父亲节点小
        while (hole_index && Less(value,m_data_[parent]))
        {
            // 那么和父亲节点调换位置
            m_data_[hole_index] = m_data_[parent];
            // 需要更新父亲节点的索引
            m_data_[hole_index]->UpdateHeapIndex(hole_index);
            hole_index = parent;
            //继续查找
            parent = (hole_index - 1) / 2;
        }
        // 最后找到属于自己的位置
        m_data_[hole_index] = value;
        // 更新自己的索引
        m_data_[hole_index]->UpdateHeapIndex(hole_index);
    }

    // 插入一个元素
    void			MinHeap::Push(Event* value){
        assert(value);
        // 首先确保空间OK
        Reserve(m_cnt_ + 1);
        ShiftUp(m_cnt_ ++,value);
    }

    // 删除堆顶的元素
    Event*			MinHeap::Pop()
    {
        Event* ret_ = NULL;
        if (m_cnt_)
        {
            ret_ = *m_data_;
            // 第二个参数是堆中最末尾的节点
            ShiftDown(0, m_data_[-- m_cnt_]);
        }
        return ret_;
    }

    // hole_index元素被挖走，希望用value来填充，但是需要根据hole_index元素的子节点大小
    //   决定是否需要让直接点替换
    // 前提，hole_index前面的元素都比他小
    void			MinHeap::ShiftDown(size_t hole_index, Event* value)
    {
        // 获得子节点（右边）的索引
        size_t min_child = 2 * (hole_index + 1);
        // 存在之节点
        while (min_child <= m_cnt_)
        {
            // 若右节点不存在，或者左边的子节点比右边的子节点更小。那切换到左节点
            if(min_child == m_cnt_ || Less(m_data_[min_child - 1],m_data_[min_child]))
                -- min_child;

            // 如果最后一个节点比它的较小的子节点还小，那么直接用最后一个节点替换
            if(!Less(m_data_[min_child],value))
                break;

            // 将较小的子节点填入空缺，并更新索引
            m_data_[hole_index] = m_data_[min_child];
            m_data_[hole_index]->UpdateHeapIndex(hole_index);

            // 然后继续查找这个子节点的直接点
            hole_index = min_child;
            min_child = 2 * (hole_index + 1);
        }

        // 最后将最末一个节点放入进去，并更新索引
        m_data_[hole_index] = value;
        m_data_[hole_index]->UpdateHeapIndex(hole_index);
    }


    // 获得堆的顶部元素，一般是最小（或者最大）的元素
    Event* 			MinHeap::Top(){
        if(!m_cnt_)
            return NULL;
        else{
            return *m_data_;
        }
    }


    //  删除中间某个元素
    void			MinHeap::Erase(Event* value){
        assert(value);
        size_t cindex_ = value->HeapIndex();
        // 不能为空，也不能超过当前索引的范围
        if(Event::HEAP_MAX_INDEX != cindex_ && m_cnt_ > cindex_)
        {
            // 获得最后一个元素
            Event* last_ = m_data_[--m_cnt_];

            //若cindex_ == 0，则parent可能很大
            size_t parent = (cindex_ - 1)/2;

            // 如果不是顶元素，并且最后元素比空缺处还小，那么
            // 尝试直接将最后元素插入空缺，然后依次往上升
            if(cindex_ > 0 && Less(last_,m_data_[parent]))
                ShiftUp(cindex_, last_);
            else
                ShiftDown(cindex_, last_);

            // 清空索引
            value->UpdateHeapIndex(Event::HEAP_MAX_INDEX);
        }
    }

    int main(int argc, char* argv[])
    {

        MinHeap heap_;
        Event ev1_;
        ev1_.SetReuseFlag(true);
        ev1_.SetTimeout(1000);
        Event ev2_;
        ev2_.SetReuseFlag(true);
        ev2_.SetTimeout(2000);
        Event ev3_;
        ev3_.SetTimeout(3000);

        heap_.Push(&ev1_);
        heap_.Push(&ev2_);
        heap_.Push(&ev3_);

        while(true)
        {
            Time_t cur_time_ = ::GetCurTime();
            Event* ev_ = heap_.Top();
            if(!ev_)
                break;

            // 已经过了
            if(ev_->NextTimeout() <= cur_time_)
            {
                ev_->Run();
                heap_.Pop();

                if(ev_->IsReuse())
                {
                    ev_->ResetNextTimeout();
                    heap_.Push(ev_);
                }
                continue;
            }

            Time_t waittime_ = ev_->NextTimeout() - cur_time_;
            Sleep(waittime_);

        }
        return 0;
    }




