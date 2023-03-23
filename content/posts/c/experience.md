---
title: "C语言程序设计的一些经验"
date: 2023-02-27T19:20:20+08:00
tags: [c]
---







## 外部库的使用方式
最近我在开发项目是, 需要使用到libelf库, 我在Github上找到了其源代码.

我之前使用一个lib都是以链接的形式使用动态库/静态库, 但是既然它提供了源码, 那么我可以直接将源码拷贝到我的项目中吗? 答案肯定是可以, 那么这两种方案该如何抉择呢?

在查阅了一些资料后, 我总结了以下几个判断依据:

1. 库的大小/对编译时间的敏感度; 如果使用源代码, 每次编译项目时需要额外对库文件进行编译(起码是第一次), 而库文件的定义是不常修改的, 如果库文件比较大, 则会延长整个项目的编译时间.
2. 是否需要版本控制; 要使用的库如果需要区分版本, 或者分配给其他的团队成员使用, 那么用库的形式似乎更为方便
3. 发挥git submodule的优势;





> Ref: [c++ - Should I add the source of libraries instead of linking to them? - Software Engineering Stack Exchange](https://softwareengineering.stackexchange.com/questions/313907/should-i-add-the-source-of-libraries-instead-of-linking-to-them)





## `const` 修饰符的妙用

有些时候, 我们设计的结构体中会有*name*字段, 类型是`char *`. 在使用时为它分配空间, 不使用时需要回收.

其实还有另一种情况, 就是*name*要指向预先定义好的"static name list", 适用于name的取值是确定的范围. 例如, libdwarf库中的描述section name的`dss_name` 成员.

这时, 为了防止使用者调用`free()`来释放它, 我们可以将其声明为`const char *`, 此时如果调用`free(.dss_name)`, 编译器会给出警告:

```sh
const.c:16:10: warning: passing argument 1 of ‘free’ discards ‘const’ qualifier from pointer target type [-Wdiscarded-qualifiers]         
   16 |     free(dss_name);             
      |          ^~~~~~~~  
```



