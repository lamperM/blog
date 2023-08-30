---
title: "Linux追踪系统学习(1): tracepoint"
tags: ["linux", "Operating System", "trace"]
date: 2023-04-23T23:51:49+08:00
---

tracepoint 是 Linux trace system 中 data source 之一，
其 trace 的对象是 kernel，属于一种静态的插桩方法。

- 添加和删除需要手动修改内核源码
- 可以向上提供接口，可以通过 frontend 来开启或者关闭，也可以自定义数据处理方式
- 在 disable 时， 仅有一次 if 判断的损耗，所以效率还算高。但缺点是不够灵活。

## tracepoint 的组成

看其源码`struct tracepoint`就能知道它的组成结构：

```c
struct tracepoint {
    const char *name;
#define TP_STATE_DISABLE 0
#define TP_STATE_ENABLE  1
    int state;

    // 并非用于注册hook的函数，而是注册hook时的hook
    int (*reghook)(void);
    void (*unreghook)(void);

    // 在tracepoint触发时将调用的hook
    struct tracepoint_hook *hooks;
};
```

- name: 是该 tracepoint 的名称
- state: 用于控制其开关状态
- hooks: 是一系列的函数指针，当 tracepoint hit 时，这些函数会被依次调用
- reghook/unreghook: 在注册/注销 hook 时将被调用，可以用来输出一些提示信息

为了提供对 tracepoint 操作的接口，定义一个 tracepoint 时，会同时定义一系列功能函数,
包括：

1. 放在内核代码之中的插桩函数；其被调用说明 tracepoint hit， 如果 enable 状态，
   则依次执行其 hook
2. 用于注册 hook 的接口；为该 tracepoint 添加新的 hook
3. 注销某个 hook 的接口；

## tracepoint 工作原理

类似于一般的日志记录函数， 在合适的位置放置`trace_##event()`作为“插桩”，运行到
此处代表该做一些事了，至于做什么事不是 tracepoint 该管的。它只能负责提供给你一些接口
，让你能把“做事”的函数与 tracepoint 联系起来，到时候触发时调用它们。

这就是上面说的，tracepoint 仅负责 data source 这一部分。

## 小结

下一节将介绍如何 Linux trace system 中的**数据记录**组件，毕竟每次触发都输出到控制台还是
太乱了，而且不是 trace 这个系统的工作内容（log 系统应该是干这个的）。
