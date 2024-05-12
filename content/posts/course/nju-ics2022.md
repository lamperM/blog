---
title: "课程笔记：NJU ICS2022"
tags: ["Course"]
categories: ["Course"]
date: 2024-04-16T19:28:12+08:00
---

ISA: RISCV32


## 编译一个客户程序

### 内存布局

貌似所有的格式，所有的平台都使用同一个链接脚本。
- 栈空间在ELF中预留，好处是统一，缺点是增加了ELF的大小。
- bss跟在数据段后面，导致数据段的filesz和memsz可能不同。NanOS是加载APP的时候，load segment时直接清零，而不是需要单独遍历一下bss，更加方便。

```c
ENTRY(_start)
PHDRS { text PT_LOAD; data PT_LOAD; }

SECTIONS {
  /* _pmem_start and _entry_offset are defined in LDFLAGS */
  . = _pmem_start + _entry_offset;
  .text : {
    *(entry)
    *(.text*)
  } : text
  etext = .;
  _etext = .;
  .rodata : {
    *(.rodata*)
  }
  .data : {
    *(.data)
  } : data
  edata = .;
  _data = .;
  .bss : {  /* bss段跟在.data的后面，会被放入同一个segment，所以加载时需要清零 */
	_bss_start = .;
    *(.bss*)
    *(.sbss*)
    *(.scommon)
  }
  /* 栈空间在ELF中定义 */
  _stack_top = ALIGN(0x1000);
  . = _stack_top + 0x8000;
  _stack_pointer = .;
  end = .;
  _end = .;
  _heap_start = ALIGN(0x1000);
}

```



## Difftest 的设计理念

揭开Difftest的魔法。

进行DiffTest需要提供一个和DUT(Design Under Test, 测试对象) 功能相同但实现方式不同的REF(Reference, 参考实现), 然后让它们接受相同的有定义的输入, 观测它们的行为是否相同.


拿NEMU作为DUT，Spike作为REF来说，Difftest的原理就是：
1. 微调REF，使得两者初始化环境相同，同时输入的客户机程序也相同。
2. DUT执行一条指令
3. 发送信号让REF执行一条指令。
4. 拷贝REF的运行环境（寄存器）到DUT中
5. 在DUT中检查是否一致。
6. DUT和REF继续执行指令，直至客户程序结束


```c
// (1) 如何保证两者初始环境相同？
init_difftest() // NEMU
  => dlopen()  // 打开REF的动态链接库（Spike）
  => dlsym()   // 过动态链接对动态库中的上述API符号进行符号解析和重定位
               // 返回REF中函数的地址，可以在DUT里强制调用
  => ref_difftest_init() // REF初始化，等待DUT的命令
                         // Spike怎么做的不清楚，但是QEMU是用Gdb监听，DUT给Gdb发命令
  => ref_difftest_memcpy() // 将DUT的guest memory拷贝到REF中
  => ref_difftest_regcpy() // 将DUT的寄存器状态拷贝到REF中
// (2) DUT执行一条指令
exec_once() 
// (3) 并让REF也执行指令，比较结果
difftest_step() 
  => ref_difftest_exec(1) // REF执行一条指令，QEMU来说就是让Gdb发送一条si命令
  => ref_difftest_regcpy(DIFFTEST_TO_DUT); // 将REF的寄存器拷贝过来，准备比较
  => checkregs(); // 比较DUT和REF的寄存器值
```

NEMU的简化会导致某些指令的行为与REF有所差异, 因而无法进行对比。
Difftest支持跳过对比某些指令，大概的过程如下：
```C
// (1) 情况1: NEMU执行一条指令，但是QEMU如果也执行会发生差异，
//     所以让QEMU跳过执行这条指令。
difftest_skip_ref() // 在执行NEMU指令的任意阶段调用
  => is_skip_ref = true // 标记跳过下次REF指令执行阶段
  => skip_dut_nr_inst = 0 // 与情况2有关，下面再介绍

difftest_step() {
  if (is_skip_ref) {
    ref_difftest_regcpy(DIFFTEST_TO_REF); // 同步DUT到REF，相当于跳过
    is_skip_ref = false;
    return;   // 跳过REF，直接返回到下一条指令
  }
}

// (2) 情况2: NEMU跳过执行，让REF执行
difftest_skip_dut() // 在执行NEMU指令的任意阶段调用
  => skip_dut_nr_inst += 1
  => ref_difftest_exec(1) // REF先执行一次，更新了REF上下文

difftest_step() {
  if (skip_dut_nr_inst > 0) {
    ref_difftest_regcpy(DIFFTEST_TO_DUT); // 不管NEMU此次的结果，同步REF到DUT
    if (ref_r.pc == npc) {
      skip_dut_nr_inst = 0;
      checkregs(&ref_r, npc);
      return;
    }
    skip_dut_nr_inst --;
    return;
  }
}

```

## 构建系统

### 编译Guest程序并追加参数

```bash
make ARCH=$ISA-nemu run mainargs=I-love-PA
```

### 编译Guest程序流程分析

编译dummy可执行文件的命令如下，很好奇是如何实现的。
```bash
cd am-kernels/am-kernels/tests/cpu-tests
make ARCH=riscv32-nemu ALL=dummy
```

既然在当前目录执行make，自然先从 `am-kernels/am-kernels/tests/cpu-tests/Makefile`开始看起。
- `ALL`默认是所有test的集合，参数指定会覆盖它。
- `ALL`的生成规则属于 **静态规则** ，通配符%代表ALL的全称即依赖Makefile.dummy。[更多关于静态规则](https://seisman.github.io/how-to-write-makefile/rules.html#id8)
- Makefile.dummy的生成规则也就在下面，这个文件默认是不存在的，需要临时生成
- 生成的规则是 `echo -e "NAME = dummy\nSRCS = tests/dummy.c\ninclude $${AM_HOME}/Makefile" > $@`。包含了AM_HOME下的Makefile
- AM是各个平台版本可执行文件的“制造机”，理念是将平台信息传入，生成指定格式的镜像。运行时库也包含在内。
- 最后就是执行 `make -f` 去调用刚才生成的临时Makefile.dummy，传入指定参数
```makefile
ALL = $(basename $(notdir $(shell find tests/. -name "*.c")))

all: $(addprefix Makefile., $(ALL))
	@echo "test list [$(words $(ALL)) item(s)]:" $(ALL)

$(ALL): %: Makefile.%

Makefile.%: tests/%.c latest
	@/bin/echo -e "NAME = $*\nSRCS = $<\ninclude $${AM_HOME}/Makefile" > $@
	@if make -s -f $@ ARCH=$(ARCH) $(MAKECMDGOALS); then \
		printf "[%14s] $(COLOR_GREEN)PASS$(COLOR_NONE)\n" $* >> $(RESULT); \
	else \
		printf "[%14s] $(COLOR_RED)***FAIL***$(COLOR_NONE)\n" $* >> $(RESULT); \
	fi
	-@rm -f Makefile.$*
```

目前为止大概的思路还算比较清晰：
1. 生成 `Makefile.dummy` 
2. 执行它，传递命令行参数（`$(MAKECMDGOALS)`）

`Makefile.dummy` 是一个临时文件，编译命令执行后会被删除。注销删除命令，直接打开看它的内容。

```makefile
NAME = dummy
SRCS = tests/dummy.c
include /home/soben/ics2023/abstract-machine/Makefile
```

需要结合AM_HOME下的Makefile去理解了。
这么来看AM的构建系统像是一个“加工厂”，**在软件设计术语叫“模板“**。
- 外部传入加工零件的要求（ARCH、NAME、SRCS...）
- 输出对应需求的产品（可执行文件）。
- 既可以要求它将我们传入的源文件编译成native架构下可执行文件，或者是NEMU、QEMU这类模拟器上。

### AM的构建思路


{{< notice tip 帮助理解AM的Makefile >}}

AM_HOME/Makefile 有命令转化makefile为html文件，帮助理解。
```makefile
### *Get a more readable version of this Makefile* by `make html` (requires python-markdown)
html:
	cat Makefile | sed 's/^\([^#]\)/    \1/g' | markdown_py > Makefile.html
.PHONY: html
```
{{< /notice >}}

上面说过，AM的构建系统是一个模版，将源文件编译，使其能运行在指定的平台上。
甚至，运行时库也是集成到AM中的，最大程度上减少重复劳动。

下面是Makefile的源代码，我们选取关键的部分进行拆解。
1. 提前说明，传入的参数有ARCH、NAME、SRCS，剩余参数在$(MAKECMDGOALS)
2. 环境检查（clean、html命令跳过）
3. 从参数ARCH中分离出ISA和PLATFORM
4. 设置编译器和编译选项
5. 引入和ISA、Platform相关的一些规则，**include `scripts/$ARCH.mk`**，下面会详细分析
6. 一些通用的编译目标规则，比如.c/.S如何生成.o

```makefile
## 1. Basic Setup and Checks

### Default to create a bare-metal kernel image
ifeq ($(MAKECMDGOALS),)
  MAKECMDGOALS  = image
  .DEFAULT_GOAL = image
endif

### Override checks when `make clean/clean-all/html`
ifeq ($(findstring $(MAKECMDGOALS),clean|clean-all|html),)

# 其他的检查暂时省略...

### Extract instruction set architecture (`ISA`) and platform from `$ARCH`. Example: `ARCH=x86_64-qemu -> ISA=x86_64; PLATFORM=qemu`
ARCH_SPLIT = $(subst -, ,$(ARCH))
ISA        = $(word 1,$(ARCH_SPLIT))
PLATFORM   = $(word 2,$(ARCH_SPLIT))

### Check if there is something to build
ifeq ($(flavor SRCS), undefined)
  $(error Nothing to build)
endif

### Checks end here
endif

## 2. General Compilation Targets

### Compilation targets (a binary image or archive)
IMAGE_REL = build/$(NAME)-$(ARCH)
IMAGE     = $(abspath $(IMAGE_REL))
ARCHIVE   = $(WORK_DIR)/build/$(NAME)-$(ARCH).a

### Collect the files to be linked: object files (`.o`) and libraries (`.a`)
OBJS      = $(addprefix $(DST_DIR)/, $(addsuffix .o, $(basename $(SRCS))))
LIBS     := $(sort $(LIBS) am klib) # lazy evaluation ("=") causes infinite recursions


## 3. General Compilation Flags
# 省略：编译器和编译选项

## 4. Arch-Specific Configurations

### Paste in arch-specific configurations (e.g., from `scripts/x86_64-qemu.mk`)
-include $(AM_HOME)/scripts/$(ARCH).mk

### Fall back to native gcc/binutils if there is no cross compiler
ifeq ($(wildcard $(shell which $(CC))),)
  $(info #  $(CC) not found; fall back to default gcc and binutils)
  CROSS_COMPILE :=
endif

## 5. Compilation Rules

### Rule (compile): a single `.c` -> `.o` (gcc)
$(DST_DIR)/%.o: %.c
	@mkdir -p $(dir $@) && echo + CC $<
	@$(CC) -std=gnu11 $(CFLAGS) -c -o $@ $(realpath $<)

# 省略：一些其他的通用规则

### Rule (recursive make): build a dependent library (am, klib, ...)
$(LIBS): %:
	@$(MAKE) -s -C $(AM_HOME)/$* archive

### Rule (link): objects (`*.o`) and libraries (`*.a`) -> `IMAGE.elf`, the final ELF binary to be packed into image (ld)
$(IMAGE).elf: $(OBJS) $(LIBS)
	@echo + LD "->" $(IMAGE_REL).elf
	@$(LD) $(LDFLAGS) -o $(IMAGE).elf --start-group $(LINKAGE) --end-group

### Rule (archive): objects (`*.o`) -> `ARCHIVE.a` (ar)
$(ARCHIVE): $(OBJS)
	@echo + AR "->" $(shell realpath $@ --relative-to .)
	@$(AR) rcs $(ARCHIVE) $(OBJS)

### Rule (`#include` dependencies): paste in `.d` files generated by gcc on `-MMD`
-include $(addprefix $(DST_DIR)/, $(addsuffix .d, $(basename $(SRCS))))

## 6. Miscellaneous

### Build order control
image: image-dep
archive: $(ARCHIVE)
image-dep: $(OBJS) $(LIBS)
	@echo \# Creating image [$(ARCH)]
.PHONY: image image-dep archive run $(LIBS)

### Clean a single project (remove `build/`)
clean:
	rm -rf Makefile.html $(WORK_DIR)/build/
.PHONY: clean

### Clean all sub-projects within depth 2 (and ignore errors)
CLEAN_ALL = $(dir $(shell find . -mindepth 2 -name Makefile))
clean-all: $(CLEAN_ALL) clean
$(CLEAN_ALL):
	-@$(MAKE) -s -C $@ clean
.PHONY: clean-all $(CLEAN_ALL)
```

$ARCH.mk 文件规定一些架构、运行平台相关的选项：
- 架构相关的：比如说交叉编译器是啥，和特殊的CFLAGS(`-march`)
- 平台相关的：因为命令行参数可以添加如`run`，编译完后直接运行，此时需要提前将平台准备好，编译好NEMU。

```makefile
include $(AM_HOME)/scripts/isa/riscv.mk
include $(AM_HOME)/scripts/platform/nemu.mk
CFLAGS  += -DISA_H=\"riscv/riscv.h\"
COMMON_CFLAGS += -march=rv32im_zicsr -mabi=ilp32   # overwrite
LDFLAGS       += -melf32lriscv                     # overwrite

AM_SRCS += riscv/nemu/start.S \
           riscv/nemu/cte.c \
           riscv/nemu/trap.S \
           riscv/nemu/vme.c

```
