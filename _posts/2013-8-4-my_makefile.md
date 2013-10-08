---
layout: post
title: 自用makefile
category: cpp
---
	
	# 需求
	# 1. 支持多级目录,而无需在每个目录放置一个makefile
	# 2. 支持自动识别C/CPP,并可以混合存在
	# 3. 支持DEBUG模式
	# 4. 支持动态库,静态库,可执行程序编译
	# 5. 支持批量dos,unix换行格式相互转换
	###########################################################################

	# 当前目录
	PROJECT_DIR = $(shell pwd)
	###########################################################################
	# 生成的目标类型
	# so:动态库/bin:可执行程序/lib:静态库
	TARGET_TYPE = bin

	###########################################################################
	# 头文件包含
	INCLUDE_PATH = -I/usr/local/include -I$(PROJECT_DIR)/include 

	# C/CPP都适用的选项
	BUILD_FLAG = -Wall -Werror -Wextra 
	# C适用的选项
	BUILD_C_FLAG =
	# CPP适用的选项
	BUILD_CXX_FLAG =

	#debug/realse相关的编译选项
	BUILD_DEBUG_FLAG = -g3 -O0 -DDEBUG -D_DEBUG
	BUILD_RELEASE_FLAG = -O3 -DRELEASE -D_RELEASE

	# 预编译
	PRE_DEFINED = -DTEST 

	###########################################################################
	# 链接选项
	LD_LIB_STATIC = -lso
	LD_LIB = -lpthread
	LD_LIB_PATH = -L/usr/local/lib -L/root/test/so
	LD_FLAG = 

	# 目标程序名称
	#PROGRAM = your_programe_name
	PROGRAM =

	# The C/C++ program compiler.
	CC     = gcc
	CXX    = g++

	###########################################################################
	#枚举文件
	SOURCES = $(shell find . -mindepth 1 -name "*.c" )
	CPP_SOURCES = $(shell find . -mindepth 1 \( -name "*.cpp" -o -name "*.cc" \) )
	H_SOURCES = $(shell find . -mindepth 1 -name "*.h" )

	#中间文件
	OBJS    = $(addsuffix .o, $(basename $(SOURCES)))
	OBJS    += $(addsuffix .o, $(basename $(CPP_SOURCES)))

	#DEPS    = $(OBJS:.o=.d)

	# 自动包含当前目录
	INCLUDE_PATH += -I$(PROJECT_DIR)

	# 默认是release
	BUILD_TARGET_FLAG = $(BUILD_RELEASE_FLAG)

	# 合并编译选项
	BUILD_FLAG += $(BUILD_TARGET_FLAG)
	BUILD_FLAG += $(PRE_DEFINED)
	BUILD_C_FLAG += $(BUILD_FLAG)
	BUILD_CXX_FLAG += $(BUILD_FLAG)

	# 链接器
	ifeq ($(strip $(CPP_SOURCES)),)
		LINKER = $(CC)
	else
		LINKER = $(CXX)
	endif

	# 自动为静态库加上static
	ifneq ($(strip $(LD_LIB_STATIC)),)
		LD_LIB_STATIC_TMP := $(LD_LIB_STATIC)
		LD_LIB_STATIC = -static $(LD_LIB_STATIC_TMP)
	endif

	###########################################################################
	# 编译，链接命令
	C_COMPILE = $(CC) $(INCLUDE_PATH) $(BUILD_C_FLAG) -c
	CPP_COMPILE = $(CXX) $(INCLUDE_PATH) $(BUILD_CXX_FLAG) -c
	LINK_COMPILE = $(LINKER) $(LD_FLAG) $(LD_LIB_PATH) $(LD_LIB) $(OBJS) $(LD_LIB_STATIC) -o $@

	###########################################################################
	# 自动生成程序名称
	ifeq ($(strip $(PROGRAM)),)
		PROGRAM = $(shell pwd | sed "s/.*\///" )
	endif

	ifeq ($(strip $(TARGET_TYPE)),so)
		BUILD_FLAG += -fPIC
		LD_FLAG += -shared
		PROGRAM_TMP := $(PROGRAM)
		PROGRAM = lib$(PROGRAM_TMP).so
	endif
	ifeq ($(strip $(TARGET_TYPE)),lib)
		PROGRAM_TMP := $(PROGRAM)
		PROGRAM = lib$(PROGRAM_TMP).a
		LINK_COMPILE = ar crv $@ $(OBJS)
	endif

	###########################################################################
	.PHONY:all clean show debug unix dos
	###########################################################################
	all:$(PROGRAM)

	%.o:%.c
		$(C_COMPILE) $< -o $@
	%.o:%.cc
		$(CPP_COMPILE) $< -o $@
	%.o:%.cpp
		$(CPP_COMPILE) $< -o $@
	%.o:%.c++
		$(CPP_COMPILE) $< -o $@
	%.o:%.cp
		$(CPP_COMPILE) $< -o $@
	%.o:%.cxx
		$(CPP_COMPILE) $< -o $@

	$(PROGRAM):$(OBJS)
		$(LINK_COMPILE)

	debug: BUILD_TARGET_FLAG = $(BUILD_DEBUG_FLAG)
	debug: $(PROGRAM)

	release: BUILD_TARGET_FLAG = $(BUILD_RELEASE_FLAG)
	release: $(PROGRAM)

	clean:
		rm -rf $(PROGRAM) $(OBJS)

	show:
		@echo 'CC     :' $(CC)
		@echo 'CXX     :' $(CXX)
		@echo 'PROGRAM     :' $(PROGRAM)
		@echo 'SOURCES     :' $(SOURCES)
		@echo 'CPP_SOURCES     :' $(CPP_SOURCES)
		@echo 'H_SOURCES     :' $(H_SOURCES)

	unix:
		@dos2unix $(SOURCES) $(CPP_SOURCES) $(H_SOURCES)
	dos:
		@unix2dos $(SOURCES) $(CPP_SOURCES) $(H_SOURCES)

	indent:
		indent -cdw -nbad -bap -nbc -bl -bli0 -i4 -npcs -nut -npsl -bfda -nbfde -l80 $(SOURCES) $(CPP_SOURCES) $(H_SOURCES)
		
		
##参考
1. <http://www.gnu.org/software/indent/manual/html_section/indent_2.html>
2. <http://sourceforge.net/projects/gcmakefile/>
