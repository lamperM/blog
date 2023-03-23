---
title: "GCC '-M' and Related Parameters"
date: 2022-04-26T19:08:22+08:00
tags: [c, makefile]
---

As we all know, there are huge number of parameters for GCC. With them, we can make many things possible. Now we talk about -M and related ones.
After reading this article, you will know the meaning of there magic parameters. And I will put some little demos follows. Finally, we will see what can they do in really project. Let's go ahead.

## 实例规则

以下的分析都是基于这样一个生成目标文件的规则, 应该来说具有一定的通用性。

```makefile
build/obj/main.o: src/main.c
    $(CC) $(CFLAGS) $(INCLUDES) -c $< -o $@
```

`main.c`中的内容：

```c
/* File: main.c */
#include <stdio.h> // system header file
#include "header.h"   // user defined header file
int main() {
    return 0;
}
```

## -M

Output the dependencies of the input source file. Incluing the names of itself and all included files.

**`-M`(and 下面的`-MM`)和`-o` 不能同时使用**，因为都隐含`-E`。
假设我们只想输出依赖文件，我们可以将示例中的规则如此改造：

```makefile
build/obj/main.o: src/main.c
    $(CC) $(CFLAGS) $(INCLUDES) -c $< -M
```

We will get messy output like following. Notice that the first two words is object filename and a colon.

```shell
main.o: src/main.c /usr/include/stdc-predef.h /usr/include/stdio.h \
 /usr/include/x86_64-linux-gnu/bits/libc-header-start.h \
 /usr/include/features.h /usr/include/x86_64-linux-gnu/sys/cdefs.h \
 /usr/include/x86_64-linux-gnu/bits/wordsize.h \
 /usr/include/x86_64-linux-gnu/bits/long-double.h \
 /usr/include/x86_64-linux-gnu/gnu/stubs.h \
 /usr/include/x86_64-linux-gnu/gnu/stubs-64.h \
 /usr/lib/gcc/x86_64-linux-gnu/9/include/stddef.h \
 /usr/lib/gcc/x86_64-linux-gnu/9/include/stdarg.h \
 /usr/include/x86_64-linux-gnu/bits/types.h \
 /usr/include/x86_64-linux-gnu/bits/timesize.h \
 /usr/include/x86_64-linux-gnu/bits/typesizes.h \
 /usr/include/x86_64-linux-gnu/bits/time64.h \
 /usr/include/x86_64-linux-gnu/bits/types/__fpos_t.h \
 /usr/include/x86_64-linux-gnu/bits/types/__mbstate_t.h \
 /usr/include/x86_64-linux-gnu/bits/types/__fpos64_t.h \
 /usr/include/x86_64-linux-gnu/bits/types/__FILE.h \
 /usr/include/x86_64-linux-gnu/bits/types/FILE.h \
 /usr/include/x86_64-linux-gnu/bits/types/struct_FILE.h \
 /usr/include/x86_64-linux-gnu/bits/stdio_lim.h \
 /usr/include/x86_64-linux-gnu/bits/sys_errlist.h
 src/header.h

```

## -MM

Like `-M` but do NOT output system header files.

```shell
main.o: src/main.c src/header.h
```

## -MF \<file>

Use with `-M` or `-MM`. Specify output dependencies to file instead of STDOUT.

注意，只要使用追加上`-MF`，就可以和`-o`选项并存了，可以写在一条语句中

## -MD

`-MD` is same as `-M -MF <file>`. But the filename is basd on the object file but replacing `.o` with `.d`.

如果将示例中的代码换成：

```makefile
build/obj/main.o: src/main.c FORCE
    $(CC) $(CFLAGS) $(INCLUDES) -c $< -o $@ -MD
```

```shell
$ ll build/obj/

total 16
drwxrwxr-x 2 soben soben 4096 3月  23 20:45 ./
drwxrwxr-x 3 soben soben 4096 3月  23 20:35 ../
-rw-rw-r-- 1 soben soben 1144 3月  23 20:45 main.d
-rw-rw-r-- 1 soben soben 1368 3月  23 20:45 main.o

```

> Note: `-MD` and `-MMD` 因为有`-MT`，也不隐含 `-E`.

## -MMD

`-MMD` is same as `-MM -MF <file>`. Also named on object file but replacing `.o` with `.d`.

## -MT \<target>

`MT` 是一个单独的选项，不与上面的冲突。作用是**改变生成依赖规则的目标格式**。在此之前，默认的格式是`文件名.o`，去除任何前缀目录。

而使用`-MT`之后可以自定义规则中目标的格式， 由`<target>`指定。

例如，对于前面的选项，依赖规则目前总是`main.o`，很多使用，我们需要的是其编译规则中目标的形式，包含路径，并不仅仅是文件名本身。这时我们就需要使用`-MT`，可以将示例中的规则做如下修改:

```makefile
build/obj/main.o: src/main.c FORCE
    $(CC) $(CFLAGS) $(INCLUDES) -c $< -o $@ -MMD -MT $@
```

依赖文件的内容就变为:

```makefile
build/obj/main.o: src/main.c src/header.h
```

> 实际上，从我的开发经验来看，大项目中编译规则的目标并不直接是目标文件，总有一个路径前缀，例如：`$(objdir)/%.o: $(srcdir)/%c`, 这时如果 include 的依赖文件的目标只是一个文件名，其实没什么意义。 所以 `-MT` 应该是在开发大型项目中很常见的。

## -MQ \<target>

与`MT`类似，而且我没有验证成功[官网](https://gcc.gnu.org/onlinedocs/gcc/Preprocessor-Options.html)说出的和 MT 的区别.
所以，这是一个 **TODO**。

## Application

Here is an important question you may ask me: _Why do we struggle to get the dependencies formats? What can they do?_  
If you are familiar with `make` and `Makefile`, aha, that's it!
With the help of M-related parameters, you can easily handle the problem of **tracing header files**.  
Give you a little demo about my point.

```makefile
-include *.d
%.o:%.c
        $(CC) $(CFLAGS)  $(INCLUDES) $< -c  -MMD -o $@
```

Actually, we do two things in order:

1. When complieing source files, we generate dependency files `xxx.d` at the same time.
2. After geting `xxx.d`, we include them in makefile. As its format is exactly the dependency format required by makefile.

## Summary

Hope this article can give you a clear understanding of M-related parameters in GCC. We can sometimes find them in large projects' makefile. It's very useful to automatic build dependency for header files. So try to use them in your current or next project.

## Reference

1. [GNU GCC options](https://gcc.gnu.org/onlinedocs/gcc/Preprocessor-Options.html#Preprocessor-Options)
2. [GCC -M, -MM, -MMD, -MF, -MT](https://programmer.group/gcc-m-mm-mmd-mf-mt.html)
