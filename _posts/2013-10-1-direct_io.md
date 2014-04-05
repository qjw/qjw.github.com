---
layout: post
title: linux 零拷贝技术
category: cpp
---

##使用mmap代替read/write

以下代码读取某文本中的内容

	#include <stdlib.h>
	#include <unistd.h>
	#include <signal.h>
	#include <stdio.h>
	#include <errno.h>
	#include <sys/types.h>
	#include <sys/stat.h>
	#include <fcntl.h>
	#include <sys/mman.h>


	int main(int argc, char** argv) 
	{
		int fd = open("/tmp/fuck",O_RDONLY);
		if(fd < 0)
		{
			printf("[%d]%d %s\n",__LINE__,errno,strerror(errno));
			return -1;
		}

		struct stat sb;
		if (fstat(fd, &sb) == -1)
		{
			printf("[%d]%d %s\n",__LINE__,errno,strerror(errno));
			return -1;
		}
		const char* buf = (const char*)mmap(0, sb.st_size, PROT_READ, MAP_SHARED, fd, 0);
		if (buf == MAP_FAILED)
		{
			printf("[%d]%d %s\n",__LINE__,errno,strerror(errno));
			return -1;
		}
		printf("%s\n",buf);
		close(fd);
		return 0;
	}
	  
##sendfile

以下代码拷贝一个文件

	#include <stdlib.h>
	#include <unistd.h>
	#include <signal.h>
	#include <stdio.h>
	#include <errno.h>
	#include <sys/types.h>
	#include <sys/stat.h>
	#include <fcntl.h>
	#include <sys/sendfile.h>


	int main(int argc, char** argv) 
	{
		int fd = open("/tmp/fuck",O_RDONLY | O_RDONLY);
		if(fd < 0)
		{
			printf("[%d]%d %s\n",__LINE__,errno,strerror(errno));
			return -1;
		}
		int fd2 = open("/tmp/fuck1",O_CREAT | O_TRUNC | O_WRONLY);
		if(fd2 < 0)
		{
			printf("[%d]%d %s\n",__LINE__,errno,strerror(errno));
			return -1;
		}

		struct stat sb;
		if (fstat(fd, &sb) == -1)
		{
			printf("[%d]%d %s\n",__LINE__,errno,strerror(errno));
			return -1;
		}
		if(sendfile(fd2,fd,NULL,sb.st_size) < 0)
		{
			printf("[%d]%d %s\n",__LINE__,errno,strerror(errno));
			return -1;
		}
		close(fd);
		close(fd2);
		return 0;
	}
	
##splice
	
	#define _GNU_SOURCE
	#include <stdlib.h>
	#include <unistd.h>
	#include <signal.h>
	#include <stdio.h>
	#include <errno.h>
	#include <sys/types.h>
	#include <sys/stat.h>
	#include <fcntl.h>
	#include <sys/sendfile.h>


	int main(int argc, char** argv) 
	{
		int fd = open("/tmp/fuck",O_RDONLY | O_RDONLY);
		if(fd < 0)
		{
			printf("[%d]%d %s\n",__LINE__,errno,strerror(errno));
			return -1;
		}
		int fd2 = open("/tmp/fuck1",O_CREAT | O_TRUNC | O_WRONLY);
		if(fd2 < 0)
		{
			printf("[%d]%d %s\n",__LINE__,errno,strerror(errno));
			return -1;
		}

		struct stat sb;
		if (fstat(fd, &sb) == -1)
		{
			printf("[%d]%d %s\n",__LINE__,errno,strerror(errno));
			return -1;
		}
		int pipefd[2];
		if(pipe( pipefd ) < 0)
		{
			printf("[%d]%d %s\n",__LINE__,errno,strerror(errno));
			return -1;
		}

		loff_t off_out=0;
		loff_t off_in=0;
		int max_read = 4096;
		while(sb.st_size > off_in)
		{
			int toread = (sb.st_size - off_in > max_read)?max_read:sb.st_size - off_in ;
			int readlen = splice(fd, &off_in, pipefd[1], NULL, toread, SPLICE_F_MORE |SPLICE_F_MOVE);
			splice(pipefd[0], NULL, fd2, &off_out, readlen, SPLICE_F_MORE |SPLICE_F_MOVE);
		}
		close(fd);
		close(fd2);
		return 0;
	}
	
##Direct IO

在open时指定O_DIRECT标志位.
	  
##参考
1. <http://www.ibm.com/developerworks/cn/linux/l-cn-zerocopy1/>
1. <https://www.ibm.com/developerworks/cn/linux/l-cn-zerocopy2/>
1. <http://www.ibm.com/developerworks/cn/linux/l-cn-directio/>