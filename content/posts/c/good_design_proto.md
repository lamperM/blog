---
title: "Good Design: 抽象消息参数"
date: 2023-11-26T16:21:27+08:00
tags: ["C"]
categories: ["C"]
---


在设计一个消息传递类似的子系统时，消息经常需要各种参数，
通常消息的个数和类型是根据**消息自身的类型**决定的。

```c
void
handle_open(..., int flags, int mode);
void
handle_read(..., size_t len, int offset);
// ...
```

有的消息/命令参数比较多，不想写这么长的参数那就把这些参数封装到struct里
```c
struct arg_open {
  int flag;
  int mode;
};
struct arg_read {
  size_t len;
  int offset;
};

// 这里用结构体还是结构体指针都可以，不是重点!
void
handle_open(..., struct arg_open *arg);
void
handle_read(..., struct arg_read *arg);
```

这种方法有什么缺点呢?
1. 不具有通用性；无法用函数指针来实现进一步抽象，即跳表。
2. ...（暂时没想到）

所以说，**一个更好的抽象方式来了**，将所有的参数利用union放到一个结构体中。
```c
struct proto_open {
  int flag;
  int mode;
};
struct proto_read {
  size_t len;
  int offset;
};

// GOOD DESIGN
struct proto {
  // ... 可能有公共的参数, 例如ID
  union {
    struct proto_open open;
    struct proto_read read;
    // ...
  };
};

// 因为每个类型的消息/命令都有自己的处理函数
// 所以各个函数知道自己应该从那个union成员里取
void
handle_open(..., struct proto *proto)
{
  int flag, mode;
  flag = proto->open.flags;
  mode = proto->open.mode;
}
void
handle_read(..., struct proto *proto) {...}
```

这样做的最大好处**就是可以用跳表来设计了**，减少了代码量，增加了易读性！！