---
title: "Python：三方库"
tags: ["Python"]
categories: ["Python"]
date: 2024-07-30T19:28:12+08:00
---

## jinja2
要了解jinja2，那么需要先理解模板的概念。
python中自带一个简单的模板，就是string提供的。
```python
>>> import string
>>> a = string.Template('$who is $role')
>>> a.substitute(who='daxin',role='Linux')
'daxin is Linux'
>>> a.substitute(who='daxin',role='cat')
'daxin is cat'
>>>
```

Python自带的模板功能极其有限，如果我们想要在模板中使用控制语句，和表达式，以及继承等功能的话，就无法实现了。

目前主流的模板系统，最常用的就是jinja2和mako

jinja2是Flask作者开发的一个模板系统，之所以被广泛使用是因为它具有以下优点：
1. 相对于Template，jinja2更加灵活，它提供了控制结构，表达式和继承等。
2. 相对于Mako，jinja2仅有控制结构，不允许在模板中编写太多的业务逻辑。
3. 相对于Django模板，jinja2性能更好。
4. Jinja2模板的可读性很棒。