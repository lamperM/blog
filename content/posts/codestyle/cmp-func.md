---
title: "C/python: cmp函数应该怎么写"
tags: [c, python]
date: 2023-08-20T17:59:22+08:00
---

C 中的`qsort`, python 中的`sorted()`很多时间需要自己构造比较的规则，也就是告诉排序函数怎么衡量两个值的大小关系？

# TL;DR

升序的写法(C-qsort):

```c
int cmp(const void *a, const void *b)
{
  return *(int *)a - *(int *)b;
}

int main(void)
{
  int nums[] = {2, 1, 3, 5, 4};

  qsort(nums, 5, sizeof(int), cmp);
  return 0;
}
```

升序的写法(python-sorted()):

```python
from functools import cmp_to_key
nums = [2, 3, 1, 4, 5]

nums = sorted(nums, key=cmp_to_key(lambda x,y: x-y))
print(nums)
```

> python3 丢弃了`sorted()`中的`cmp`选项， 全部用 key 选项进行指定，
> 所以需要`cmp_to_key`进行转换

升序的写法(C++-sort()):

```c++
// sort()原型
void sort (RandomAccessIterator first, RandomAccessIterator last, Compare comp);

bool cmp(int a, int b){
    return a < b;
}

int main(){
    int a[10]={8 ,3 ,10 ,9 ,5};
    sort(a,a+10,cmp);
    return 0;
}

```

# 详解

正如上面所说，cmp 函数的作用是给排序函数一个比较的依据。

- c 和 python 的 cmp 函数属于同一种，返回的是 int 类型的值，代表的含义是：**a 相对于 b 的位置**。 返回正数代表 a 在 b 之后，负数则反之。所以用`a-b`算式就能表达升序排序

- 而 C++返回的则是布尔值，代表的含义是：**a 排在 b 的前面吗?**，所以用`a<b`， 也能表达升序排序

这样看起来，貌似 c++的写法更好理解一些。

> 有时我们会看到 python 中这样表示升序:
>
> ```python
> lambda x,y: -(x < y)
> ```
>
> 这其实是和`x-y`的效果是相同的, 只是有的时候一些类型不能够相减，比如说字符串，
> 但是可以使用<>比较，所以执行这样一个转换的小 trick。
