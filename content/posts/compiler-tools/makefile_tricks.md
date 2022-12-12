---
title: "Makefile 一些技巧"
date: 2022-12-03T19:08:22+08:00
tags: ["makefile", "make"]
---

..



### 使用shell 变量

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