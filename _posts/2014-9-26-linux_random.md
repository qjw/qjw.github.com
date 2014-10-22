---
layout: post
title: Linux下获取随机数
category: bash
---

	#define _GNU_SOURCE                                                                
	#include <unistd.h>                                                                
	#include <fcntl.h>                                                                 
	#include <stdio.h>                                                                 
	#include <stdlib.h>                                                                
	#include <time.h>                                                                  
	#include <stdint.h>                                                                
		                                                                           
	int main( int argc, char** args )                                                  
	{                                                                                  
	    srand(time(NULL));                                                             
	    printf("srand/rand '%d'\n",rand());                                            
		                                                                           
	    {                                                                              
		uint64_t random_;                                                          
		int fd_ = open("/dev/random",O_CLOEXEC | O_RDONLY);                        
		read(fd_,&random_,sizeof(random_));                                        
		printf("/dev/random '%lu'\n",random_);                                     
	    }                     
		                  
		                  
	    {                     
		uint64_t random_; 
		int fd_ = open("/dev/urandom",O_CLOEXEC | O_RDONLY);                       
		read(fd_,&random_,sizeof(random_));                                        
		printf("/dev/urandom '%lu'\n",random_);                                 
	    }        
	    return 0;
	} # gcc main.cpp -fno-exceptions -fno-rtti

----

	/tmp <king@king-HP-Compaq-6530b-FP587PA-AB2> 10:29:20 $  od -An -N8 -t u8 /dev/urandom
	 17520312826055098811
	/tmp <king@king-HP-Compaq-6530b-FP587PA-AB2> 10:29:20 $  od -An -N8 -t u8 /dev/random
	  6282395179181610988
	/tmp <king@king-HP-Compaq-6530b-FP587PA-AB2> 10:29:23 $ echo $RANDOM
	15935



