---
title: "Makefile 一些技巧"
date: 2022-12-03T19:08:22+08:00
tags: ["makefile"]
---

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
