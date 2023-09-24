---
title: "Makefile 一些技巧"
date: 2022-12-03T19:08:22+08:00
tags: ["makefile"]
categories: ["Makefile"]
---

## 伪目标的依赖关系

Makefile 中的依赖关系指的是目标和依赖之间建立的关系，目标对应规则中的语句是否执行取决于依赖的状态。

最简单的依赖关系可以拿两个文件来举例:

```makefile
# gcc语句执行当前仅当 main.c 新于 main.elf
main.elf: main.c
    gcc main.c -o main.elf
```

make 在执行`main.elf`的规则时，会先判断依赖关系。拿上面的例子来说，
gcc 语句是否执行取决于`main.c` 和 `main.elf`的修改时间，**只有当
依赖新与目标时，规则语句才会执行**。

然而许多情况下，目标或者依赖并不是一个文件，而是**虚拟目标**。虚拟目标
并不是一个文件，即它没有修改时间这个属性，此时 make 就不能作比较，结果就是
**如果目标是伪目标，那么不管依赖如何都执行规则语句；如果依赖是伪目标，
那么目标的规则语句也永远被执行**。下面是两个例子：

```make
# 伪目标作为目标文件出现
# build finish总是输出， 而gcc语句仅当main.c比main.elf新时才执行
.PHONY : all
all: main.elf
	@echo 'build finish'
main.elf: main.c
	gcc $< -o $@
```

```make
# 伪目标作为依赖文件中出现
# 不管main.c是否比main.elf更新，因为pre-work是伪目标
# 所以gcc语句总是执行
.PHONY : pre-work
main.elf: main.c pre-work
	gcc $< -o $@

```

上面的代码的效果是：两条规则中的语句都会执行，即使你并没有对 main.c 做任何修改！

## 恐怖的空格

Makefile 中的变量结合很常见，例如`$(FIXDEP)=$(FIXDEP_PATH)/build/fixdep`.

特别是当我们这些语句是从某些地方粘贴过来，要特别注意变量中是否有空格，Makefile 非常重视这个。假如`$(FIXDEP_PATH)`中有一个空格，那么`$(FIXDEP)`就变成**两个宏**了（不知道叫宏合不合适）。而且 Make 的执行过程很难检查出来。

## 规则的执行顺序

如果不从命令行传入目标, Makefile 中定义的规则其实是以**从上而下**的顺序执行的, 但是我习惯把 `all` 这种默认规则放在最下面, 所以一般我们可以看到很多 Makefile 会在开头写一句规则`all:`, 作用就是告诉 make 默认(不显式指定)的目标是`all`.

> Busybox 根目录 Makefile 中的做法示例
>
> ```makefile
> # That's our default target when >none is given on the command line
> .PHONY: _all
> _all:
> ```

## 函数的魔法

### patsubst

```make
$(patsubst <pattern>,<replacement>,<text>)
```

功能：查找`<text>`中的单词（以空格，tab，回车，换行分割），看其是否符合`<pattern>`,
如果符合，将其使用`<replacement>`替换。可以使用通配符`%`。

以下两对是等效的, 明显还是直接使用变量的替换语法操作简单:

```make
$(patsubst <pattern>,<replacement>,$(var))
$(var:<pattern>=<replacement>;)

$(patsubst %<suffix>,%<replacement>,$(var))
$(var:<suffix>=<replacement>)
```

## 使用 shell 变量

Make 将 `$$var` 转义为`$var`, 供 shell 处理.

demo(源自 6.828 根目录`GNUmakefile`):

```makefile
handin-check:
    @if test -n "`git status -s`"; then \
        git status -s; \
        read -p "Untracked files will not be handed in.  Continue? [y/N] " r; \
        test "$$r" = y; \
    fi
```

> 以上 demo 还使用了 test 命令来终止 make 的执行, 如果用户没有输入`y`, make 将会终止执行

## 使用另一个 Makefile

常见的有三种方式， `make -C`, `make -f` 和 `include`

`make -C <dir>` 的作用等价与 `cd <dir>`+`make`, 常见于在一个工程
的主目录下，依次编译生成其他子目录的目标文件。有`cd`命令的效果，会切换
当前目录。

`make -f <file>` 更像临时使用某个 Makefile 来执行一些操作，在指定的
Makefile 中如果想使用之前的变量，需要`export`. 目前还没有发现有必要的
应用场景，大部分用`include`方式替代。

`include <file>` 一般用于引入一些通用规则，就像 C 语言的 include 头文件
一样，变量无需`export`.
