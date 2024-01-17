---
title: "课程学习：cs61a"
tags: ["Course"]
categories: ["Course"]
date: 2023-07-17T19:28:12+08:00
---



## 学习日历 - 激励自己学习

因为最近在写论文，所以每两天能学习一次，并记录每次学习的事件

### 2024年1月4日22点09分

配置环境

lab00目录的组成：
* lab00.py: The template file you'll be adding your code to
* ok: A program used to test and submit assignments
* lab00.ok: A configuration file for ok

What Would Python Do? (WWPD)

`python3 ok -q python-basics -u --local` 结尾的`--local`避免输入伯克利邮箱。


解释一个函数的组成：
* The lines in the triple-quotes """ are called a docstring, which is a description of what the function is supposed to do. When writing code in 61A, you should always read the docstring!

* The lines that begin with >>> are called doctests. Recall that when using the Python interpreter, you write Python expressions next to >>> and the output is printed below that line. Doctests explain what the function does by showing actual Python code. It answers the question: "If we input this Python code, what should the expected output be?"

> 结束时间:23点22分，完成了Lab00和Lab01

### 2024年1月7日16点41分

> 结束时间:19点07分， 完成了Lab02和一半Hog

### 2024年1月9日19点33分

>结束时间:21点17分，完成了hog剩下的，和lab04的Q1

### 2024年1月11日11点24分

>结束时间:14点01分，完成了lab04

### 2024年1月12日22点00分

>结束时间:22点34分，完成了Cat的前几个问题

### 2024年1月13日17点31分

>结束时间:18点55分，完成了Cat。

### 2024年1月14日21点58分4

抽象数据类型（ADT）的含义是什么？
Tree的ADT在Python中就是根节点和分支的List。

tuple 是不可变的列表，意味着可以作为字典的key。

python中使用默认的可变函数参数是非常危险的，并不会在每次调用之后被释放，
相当于函数内部的静态变量。
```python
def f(s=[]):
  s.append(5)
  return len(s)
```

>结束时间:22点58分，完成了Lab05







## 参考链接
* [CS 61A Fall 2023 Lab](https://inst.eecs.berkeley.edu/~cs61a/fa23/)
* [CS61A 学习经验&感想 - 知乎](https://zhuanlan.zhihu.com/p/486323075)
* [FyisFe/UCB-CS61A-20Fall个人Solution](https://github.com/FyisFe/UCB-CS61A-20Fall?tab=readme-ov-file)

