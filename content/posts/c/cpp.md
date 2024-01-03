---
title: "C++ 特性的底层原理"
tags: ["C"]
categories: ["C"]
date: 2023-12-22T18:51:49+08:00
---

## const 修饰成员函数

C++允许将成员函数添加const修饰符，代表此成员函数不会对成员变量进行修改，
否则会发生编译错误。在下面的示例中，set函数用const修饰就会出错，
而get函数用const修饰就能清楚的告诉别人这个函数不会修改类的成员。

对于一个声明为const的类实例，C++规定它只能调用const修饰的成员函数，
也就是说明这个类的成员是不允许被修改的。
```c
class A {
  int num;
public:
  void set_num(int x) { num = x; }
  int get_num(void) const { return num; }
};

const A a;
a.set_num(10); // Compile error
a.get_num();   // Success
```




说起a为什么不允许调用set函数，我这里尝试从C的角度去进行解释，
毕竟C和C++本是同根生。C++定义的成员函数会有一个隐形的参数叫this指针，
this总是指向这个类的示例，所以实际上我理解调用成员函数的时候会将成员地址作为参数也传递给成员函数，毕竟这样才能用this指针嘛。然后就说为什么不能调用set函数呢？
我猜测对于一般的成员函数，规定接受的隐形参数this的类型是 `A*`，
而const修饰的成员函数接受的this类型为 `const A*`，
这样做就能限制用const修饰的类实例在将其自身地址传递给普通成员函数的时候出错，
即`const A *` 不能传递给参数类型为 `A*` 的函数哦，导致编译错误。
```c
// 猜测C实现C++类
struct A {
  int num;
  void (*set_num)(struct A *a, int x){...}
  void (*get_num)(const struct A *a) {...}
};

const struct A a;
a.set_num(&a, 10); // Compile error
a.get_num(&a);     // Ok
```


查看编译后的结果能够证实上述的猜想，确实需要将类变量的地址传递给成员函数作为隐式参数。https://godbolt.org/z/dT9rnsvKz

{{< figure src="/posts/c/cpp_const.jpg" title="" width=100% >}}

