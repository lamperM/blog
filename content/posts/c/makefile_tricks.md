---
title: "Makefile 一些技巧"
date: 2022-12-03T19:08:22+08:00
tags: ["makefile"]
---


## 规则的执行顺序
如果不从命令行传入目标, Makefile中定义的规则其实是以**从上而下**的顺序执行的, 但是我习惯把 `all` 这种默认规则放在最下面, 所以一般我们可以看到很多Makefile会在开头写一句规则`all:`, 作用就是告诉make默认(不显式指定)的目标是`all`.

>Busybox 根目录 Makefile 中的做法示例
> 
>```makefile
># That's our default target when >none is given on the command line
>.PHONY: _all
>_all:
>```


## 使用shell 变量

Make 将 `$$var` 转义为`$var`, 供shell处理. 

demo(源自6.828 根目录`GNUmakefile`):

```makefile
handin-check:
    @if test -n "`git status -s`"; then \
        git status -s; \
        read -p "Untracked files will not be handed in.  Continue? [y/N] " r; \
        test "$$r" = y; \
    fi
```

> 以上demo还使用了 test 命令来终止make的执行, 如果用户没有输入`y`, make将会终止执行
