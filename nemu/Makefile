NAME = nemu

ifneq ($(MAKECMDGOALS),clean) # ignore check for make clean
ISA ?= x86
ISAS = $(shell ls src/isa/)
$(info Building $(ISA)-$(NAME))

ifeq ($(filter $(ISAS), $(ISA)), ) # ISA must be valid
$(error Invalid ISA. Supported: $(ISAS))
endif
endif

INC_DIR += ./include ./src/isa/$(ISA)/include
BUILD_DIR ?= ./build

ifdef SHARE
SO = -so #-fPIC:Position Independent Code"的标志。这意味着生成的代码可以在任何地址运行，这是创建共享库所必需的
SO_CFLAGS = -fPIC #Shared Object Compilation Flags 用于编译共享对象的编译标志
-D_SHARE=1 #-D是定义宏的标志。这里，它定义了一个名为_SHARE的宏，并为其赋值1
SO_LDLAGS = -shared -fPIC #LDLAGS:这是"Shared Object Linker Flags"的缩写，意思是用于链接共享对象的链接标志
endif

OBJ_DIR ?= $(BUILD_DIR)/obj-$(ISA)$(SO)
BINARY ?= $(BUILD_DIR)/$(ISA)-$(NAME)$(SO)

include Makefile.git

.DEFAULT_GOAL = app

# Compilation flags
CC = gcc#CC是一个常见的Makefile变量，它表示C编译器
LD = gcc#LD是一个常见的Makefile变量，它表示链接器
INCLUDES  = $(addprefix -I, $(INC_DIR))#INCLUDES变量包含了一些-I选项，这些选项指定了编译器在查找头文件时应该搜索的目录
CFLAGS   += -O2 -MMD -Wall -Werror -ggdb3 $(INCLUDES) -D__ISA__=$(ISA) -fomit-frame-pointer
#CFLAGS += -O2 -MMD -Wall -Werror -ggdb3 $(INCLUDES) -D__ISA__=$(ISA) -fomit-frame-pointer：
#这行代码向CFLAGS变量添加了一些编译选项。CFLAGS是一个常见的Makefile变量，它表示C编译器的选项。这些选项包括优化级别-O2，生成依赖文件-MMD，开启所有
#警告-Wall，将警告视为错误-Werror，生成用于gdb调试的信息-ggdb3，添加包含目录$(INCLUDES)，定义宏__ISA__的值为$(ISA)，以及省略帧指针-fomit-frame-pointer。

QEMU_DIFF_PATH = $(NEMU_HOME)/tools/qemu-diff #QEMU_DIFF_PATH：这个变量定义了QEMU的差异工具的路径。差异:DIFF-different
QEMU_SO = $(QEMU_DIFF_PATH)/build/$(ISA)-qemu-so# /build在make之前没有

$(QEMU_SO):	#$(MAKE)：这是一个特殊的变量，它代表make命令本身。在递归地调用make命令时，通常使用这个变量而不是直接使用make，以确保使用与顶层make命令相同的版本和选项。							
	$(MAKE) -C $(QEMU_DIFF_PATH)  
#这个规则(上面这两行)的意思是，当需要构建$(QEMU_SO)目标时，make命令会切换到$(QEMU_DIFF_PATH)目录，并在那里执行另一个make命令。这通常意味着$(QEMU_DIFF_PATH)目录下有一个Makefile文件，它知道如何构建$(QEMU_SO)目标。

# Files to be compiled
SRCS = $(shell find src/ -name "*.c" | grep -v "isa")#$(shell ...)：这是一个Makefile函数，它允许你执行外部的shell命令并获取其输出。#find src/ -name "*.c"：这个shell命令在src/目录及其子目录中查找所有的.c文件。| grep -v "isa"：这个部分使用grep命令过滤掉包含“isa”字符串的文件。-v选项告诉grep排除匹配的行。
SRCS += $(shell find src/isa/$(ISA) -name "*.c")#这行代码向SRCS变量添加更多的文件。find src/isa/$(ISA) -name "*.c"：这个shell命令在src/isa/$(ISA)目录及其子目录中查找所有的.c文件。
OBJS = $(SRCS:src/%.c=$(OBJ_DIR)/%.o)#这行代码定义了OBJS变量，它表示由SRCS变量中的源文件生成的对象文件。$(SRCS:src/%.c=$(OBJ_DIR)/%.o)：这是一个字符串替换操作。它将SRCS变量中的每个字符串src/%.c替换为$(OBJ_DIR)/%.o。%是一个通配符，它匹配任何字符串。$(OBJ_DIR)是一个变量，它表示对象文件的目录。所以，OBJS变量包含了与SRCS变量中的每个源文件对应的对象文件的路径。

# Compilation patterns
$(OBJ_DIR)/%.o: src/%.c
	@echo + CC $<
	@mkdir -p $(dir $@)
	@$(CC) $(CFLAGS) $(SO_CFLAGS) -c -o $@ $<


# Depencies
-include $(OBJS:.o=.d)

# Some convenient rules

.PHONY: app run gdb clean run-env $(QEMU_SO)
app: $(BINARY)

override ARGS ?= -l $(BUILD_DIR)/nemu-log.txt
override ARGS += -d $(QEMU_SO)

# Command to execute NEMU
IMG :=
NEMU_EXEC := $(BINARY) $(ARGS) $(IMG)

$(BINARY): $(OBJS)
	$(call git_commit, "compile")
	@echo + LD $@
	@$(LD) -O2 -rdynamic $(SO_LDLAGS) -o $@ $^ -lSDL2 -lreadline -ldl

run-env: $(BINARY) $(QEMU_SO)

run: run-env
	$(call git_commit, "run")
	$(NEMU_EXEC)

gdb: run-env
	$(call git_commit, "gdb")
	gdb -s $(BINARY) --args $(NEMU_EXEC)

clean:
	-rm -rf $(BUILD_DIR)
	$(MAKE) -C tools/gen-expr clean
	$(MAKE) -C tools/qemu-diff clean
