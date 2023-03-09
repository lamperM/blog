---
title: "C语言自身的难点"
date: 2023-03-09T17:18:57+08:00
tags: [c]
---

# 函数指针
## 指针的数组 or 指向数组的指针?

``` 
>> int (*p)[10]   p是指针, 指向长度为10的数组. 加括号是为了强调p是一个指针, 区别包含10个指针的array.
>> int *(p[10])   p是数组, 它的元素类型是int *, 加括号是为了强调p是数组.
>> int *p[10]     等效于int *(p[10])
```



## 基础架构

```
// 函数指针
>> int (*f)(int)  说明f是一个指向函数的指针, 加括号为了区别返回值为int*的函数
>> f = function;  函数指针的赋值
>> (*f)(x)        函数指针指向函数的调用, 可简化为f(x). 但是容易将f误认为是函数.
    
// 函数指针的数组
>> int (*(f[10])) (int)  f是数组,元素为10个函数指针. 内层括号说明f是数组,外层括号说明元素类型是函数指针
>> int (*f[10]) (int)    与上面等效. 但外层括号不能省略
>> f[0] = function()     赋值
>> (*f[0])()             指向函数的调用, 可简化为f[0]()
    
// 返回函数指针的函数
>> void (*signal(int sig, ...))(int);  signal是一个函数, 参数有sig.... 它的返回值是一个函数指针, 指向任意返回值为void, 参数为int的函数.
```


## `typedef`帮助理解函数指针

`signal()`是一个系统调用, 用于告诉系统, 当某种特定"软件中断"发生时调用*特定的程序*. 它的真正名称应当是: *Call that routine when the interrupt comes in*.

看`signal()`的原型, 非常复杂. 根据上面基础架构的铺垫, 可以看出`signal()`的返回值是函数指针, 同时它的参数也是一个函数指针. 且这两个函数指针所指向函数的返回值和参数相同.

```c
void (*signal(int sig, void(*func)(int)))(int);
```

可以借用`typedef`表示通用部分.

```c
typedef void (*sighandler_t)(int);
```

而后`signal`的声明就是人能看懂的了:

```c
sighandler_t signal(int signum, sighandler_t handler);
```


