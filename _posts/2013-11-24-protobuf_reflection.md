---
layout: post
title: Google Protobuf 自动反射消息
category: cpp
---

最近翻了下同事**[《Linux多线程服务端编程:使用muduo C++网络库》](http://www.amazon.cn/Linux%E5%A4%9A%E7%BA%BF%E7%A8%8B%E6%9C%8D%E5%8A%A1%E7%AB%AF%E7%BC%96%E7%A8%8B-%E4%BD%BF%E7%94%A8muduo-C-%E7%BD%91%E7%BB%9C%E5%BA%93-%E9%99%88%E7%A1%95/dp/B00AYS2KL0/ref=pd_bxgy_b_img_y)**一书，发现Google Protobuf支持自动反射，这在一定程度上简化了消息（反）序列化的处理。

文中作者在自己博客也同步更新了文章，见这里<http://blog.csdn.net/solstice/article/details/6300108>

在网络程序中，通常需要对报文进行序列化和反序列化，以适用网络传输。不过这其中还有一个细节，你需要告诉对方，**你传输的是什么包**，对端才知道用什么结构来反序列化。

大多数情况下，我们用一个双方都统一的数字或者字符串来标识报文类型，收包时做一个映射。

现在的问题是，哪天新增了一个新的报文类型，这个映射就必须新增一项，发包的地方也一样。

不过Protobuf内建了这样的映射。

	bool init_send_buf_ex(
			const google::protobuf::Message& msg,
			char* buf,
			size_t* bufsize)
	{
		assert(buf && bufsize && *bufsize);
		google::protobuf::io::ArrayOutputStream aos(
				buf,
				*bufsize);
		google::protobuf::io::CodedOutputStream coded_output(&aos);

		// 写名字长度
		std::string msgname_ = msg.GetTypeName();
		if(msgname_.empty() || msgname_.length() >= ProtobufMessageMaxNamelen)
		{
			LogError("Invalid message name '%s' when init_send_buf_ex",msgname_.c_str());
			return false;
		}
		coded_output.WriteVarint32(uint32_t(msgname_.length()));
		// 写名字
		coded_output.WriteString(msgname_);
		
		// buffer是否够大
		if(msg.ByteSize() + coded_output.ByteCount() > *bufsize)
		{
			LogError("buf size '%ld' is too small,need '%ld'",
					*bufsize,
					msg.ByteSize() + coded_output.ByteCount());
			return false;
		}
		// 序列化
		msg.SerializeToCodedStream(&coded_output);
		*bufsize = coded_output.ByteCount();
		return true;
	}

---

	/**
	 * @brief 根据名字获取实际的Message对象
	 * @param typeName
	 * @return
	 */
	static google::protobuf::Message* Name2ProtobufMessage(
			const char* name)
	{
		assert(name);
		google::protobuf::Message* message = NULL;
		const google::protobuf::Descriptor* descriptor =
				google::protobuf::DescriptorPool::generated_pool()->FindMessageTypeByName(name);
		if (descriptor)
		{
			const google::protobuf::Message* prototype =
					google::protobuf::MessageFactory::generated_factory()->GetPrototype(descriptor);
			if (prototype)
			{
				message = prototype->New();
			}
		}
		return message;
	}

	void LanServerBase::read_imp(int fd)
	{
		char buf_[1024];
		struct sockaddr_in addr_;
		socklen_t addrlen_ = sizeof(addr_);

		int len_ = recvfrom_ex(fd,buf_,sizeof(buf_),0,
				(struct sockaddr*)&addr_,&addrlen_);
		if(len_ <= 0)
		{
			if(len_ < 0)
			{
				LogError("%s %d %s",strerror(errno),errno,"recvfrom_ex");
			}
			return;
		}
		assert(sizeof(addr_) <= addrlen_);
		google::protobuf::io::ArrayInputStream ais(buf_,len_);
		google::protobuf::io::CodedInputStream coded_input(&ais);
		
		// msg 类型是否合法
		bool result_ = false;

		// 读取名称长度
		uint32_t msgname_len_ = 0;
		result_ = coded_input.ReadVarint32(&msgname_len_);
		if(!result_ ||
			0 == msgname_len_ || 
			ProtobufMessageMaxNamelen <= msgname_len_)
		{
			LogError("ReadVarint32 msgname_len fail with size '%d'",msgname_len_);
			return;
		}

		// 读取名称
		std::string msgname_;
		result_ = coded_input.ReadString(&msgname_,msgname_len_);
		if(!result_)
		{
			LogError("ReadString msgname fail with size '%d'",msgname_len_);
			return;
		}
		
		google::protobuf::Message* msg_ = Name2ProtobufMessage(msgname_.c_str());
		if(!msg_)
		{
			LogError("Name2ProtobufMessage fail with name '%s'",msgname_.c_str());
			return;
		}
		
		if(!msg_->ParseFromCodedStream(&coded_input))
		{
			LogError("ParseFromCodedStream fail with name '%s'",msgname_.c_str());
			delete msg_;
			return;
		}
		delete msg_;
	}
	
##注意

在反序列化时，**ParseFromCodedStream**不要一次性给太多数据，这个报文需要多少数据，就给多少数据，否则会画蛇添足，导致反序列化失败
	
##参考
1. <http://blog.csdn.net/solstice/article/details/6300108>
