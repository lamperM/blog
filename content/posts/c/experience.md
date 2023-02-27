---
title: "C语言程序设计的一些经验"
date: 2023-02-27T19:20:20+08:00
draft: true
---











## `const` 修饰符的妙用

有些时候, 我们设计的结构体中会有*name*字段, 类型是`char *`. 在使用时为它分配空间, 不使用时需要回收.

其实还有另一种情况, 就是*name*要指向预先定义好的"static name list", 适用于name的取值是确定的范围. 例如, libdwarf库中的描述section name的`dss_name` 成员.

这时, 为了防止使用者调用`free()`来释放它, 我们可以将其声明为`const char *`, 此时如果调用`free(.dss_name)`, 编译器会给出警告:

```sh
const.c:16:10: warning: passing argument 1 of ‘free’ discards ‘const’ qualifier from pointer target type [-Wdiscarded-qualifiers]         
   16 |     free(dss_name);             
      |          ^~~~~~~~  
```



