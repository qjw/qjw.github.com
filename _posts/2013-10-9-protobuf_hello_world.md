---
layout: post
title: Protobuf Hello World
category: cpp
---

##安装

	apt-get install libprotobuf-dev libprotobuf-c0 libprotobuf-c0-dev 
	apt-get install libprotobuf6 protobuf-compiler protobuf-c-compiler
	
或者下载源码,然后**./configure && make && make install**

##编写pro文件

	message Person {
		required string name=1;
		required int32 id=2;
		optional string email=3;

		enum PhoneType {
			MOBILE=0;
			HOME=1;
			WORK=2;
		}

		message PhoneNumber {
			required string number=1;
			optional PhoneType type=2 [default=HOME];
		}

		repeated PhoneNumber phone=4;
	}

##编译proto
运行以下命令编译成cpp文件

	protoc --cpp_out=cpp/ fuck.proto
	
在cpp目录下生成两个文件"**fuck.pb.cc  fuck.pb.h**"

##编写测试代码

	#include "fuck.pb.h"                                                                                                                       
	#include <fstream>
	#include <iostream>

	using namespace std;

	int main(int argc,const char** argv)
	{
		if(argc == 1)
		{   
			Person person;
			person.set_name("John Doe");
			person.set_id(1234);
			person.set_email("jdoe@example.com");
			fstream output("myfile",ios::out | ios::binary);
			person.SerializeToOstream(&output);
		}   
		else
		{   
			fstream input("myfile",ios::in | ios::binary);
			Person person;
			person.ParseFromIstream(&input);
			cout << "Name: " << person.name() << endl;
			cout << "E-mail: " << person.email() << endl;
		}   
	}

##编译源码

	g++ main.cpp fuck.pb.cc -g -lprotobuf -lpthread	
	
运行./a.out将类序列化到文件,运行./a.out fuck 从文件中反序列化.
	
##参考
1. <http://www.cppblog.com/woaidongmao/archive/2009/06/23/88391.aspx>
1. <http://www.cnblogs.com/brainy/archive/2012/05/13/2498660.html>