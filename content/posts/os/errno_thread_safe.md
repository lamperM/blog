---
title: "浅谈 errno 的线程安全问题"
date: 2022-12-21T19:08:22+08:00
tags: [Operating System]
---





我始终以为，C库中常用的 `errno` 仅是一个全局变量，使用了全局变量就无法保证线程安全了，因为全局变量在所有线程中都是共享的。

要实现线程安全的`errno` 就必须将其设置为**线程私有的变量**，下面就来看看GCC是如何巧妙的实现的。

## 正文

现在的`errno`定义并非一个全局变量, 而是一个**宏定义**, 以下是在`usr/include/errno`中的声明:

```c
extern int *__errno_location (void);
# define errno (*__errno_location ())
```

 这种方式下其实现原理大概是: `__errno_location` 函数返回一个`int`指针, 而这个函数的实现中, 返回的就恰好是**实际的**`errno` 变量(与宏同名)的地址, 所以对其**解引用**就相当于对其值进行操作. 所以, 这种定义规则下, 左值和右值表达式均成立.

```c
errno = 10;    // *__errno_location () = 10
int x = errno; // x = *__errno_location ();
```

`__errno_location` 的实现就至关重要, 因为如果其返回的变量地址不包含任何技巧的话, 就和原先直接定义全局变量的方式没差了, **说到底能否实现线程安全, 还得看实际保存errno的变量是否为线程独有的**. 目前还没有发掘到其精髓, 只是套壳而已.

以下给出`/csu/errno-loc.c`中`__errno_location` 的实现, 与我们预期一致, 返回变量的地址. 而同名变量`errno`则定义在`/csu/errno.c`中, **决定了能够实现errno的线程安全**.

```c
int *
__errno_location (void)
{
  return &errno;
}
```

```c
__thread int errno;
```

"`__thread`" 是GCC提供的扩展前缀, 表示该变量将被库处理为线程私有的, 注意这一步是C库完成的, 对程序员透明. 相关的理论叫 *Thread-local Storage*, AArch64 架构实现的原理是利用`TPIDR_EL0` 寄存器, 其他架构可以参考[此PDF](https://akkadia.org/drepper/tls.pdf)

> :question: 以上源文件中有注释为 *non-threaded*版本的实现, 是代表什么含义呢?

​                                            

虽然我暂时没有查阅到errno的其他线程安全的实现原理, 但起码GCC下该方式这是可行的. 依靠的是"`__thread`"的支持, **与换成宏定义的方式无关**, 不排除可能为了考虑兼容其他实现方式的可能性.

## 参考

[c - How is thread-safe errno initialized if #define substitutes errno symbol? - Stack Overflow](https://stackoverflow.com/questions/18025995/how-is-thread-safe-errno-initialized-if-define-substitutes-errno-symbol)
