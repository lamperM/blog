---
title: "Linux 中断管理: 软中断/tasklet/工作队列"
tags: ["Operating System"]
date: 2023-05-13T20:51:49+08:00
---

软中断、tasklet、工作队列都是中断上下部分离的具体实现方案。

## 软中断

我们可以将某些中断配置为软中断，相当于建立一张 INTID 到软中断的映射表，这样在
中断到来时就能判断是否为软中断。

这张“表”的建立是静态的，即编译时确定的。key 为 INTID，value 为描述一个软中断
的数据结构，在下面会介绍。

> **软中断的服务函数必须是可重入的**，即多个 CPU 可以同时执行同一个 softirq
> 的处理函数，涉及到的全局结构可以用 spinlock 钳制。

### 表示 softirq 的数据结构

`struct softirq_action`代表一个软中断，系统中所有支持的软中断组成一个数据
`softirq_vec[]`, 所有的软中断按照优先级来分配下标。

```c
struct softirq_action {
    // 指向softirq的处理函数
    void (*action)(struct softirq_action *);
};
```

### softirq 的中断流程

在中断的上部，如果识别到当前中断是一个 softirq， 那么系统会标记一个软中断发生，
即`raise_softirq()`函数。其做的事情包括:

1. 标记某个软中断发生，记录的结构是`irq_cpustate_t.__softirq_pending`
   (这个字段使`loca_softirq_pending()`访问)
2. 唤醒`ksoftirqd`内核线程，之后介绍

光标记不行，那么什么时候执行它们的服务函数呢？

几个可能的检查点:(1) 中断退出前 (2)`ksoftirq`被唤醒时

如果在检查点发现有标记挂起的 softirq(`local_softirq_pending()` != 0),
内核调用`do_softirq()`处理它们：

1. 如何`in_interrupt()`返回非 0， 直接返回。此时代表要么禁用了 softirq，要么当前是
   在中断嵌套的环境下，也可能正在执行`do_softirq()`时中断嵌套的，而`do_softirq()`
   函数是不能嵌套执行的。
2. 调用`__dosoft_irq()`, 对于`local_softirq_pending()`的每一位都调用其
   `softirq_vec[nr]->action()`

这里有个重要的问题，此时处于中断下部，即开中断的情况，所以在处理 softirq 时会有新的
softirq 到来，这里就有两种策略：

1. 不断的获取最新的`local_softirq_pending()`, 直到不再有新的 softirq 产生才返回
2. 忽略新来的 softirq，使其在下次检查点再被处理

这两种方案其实各有利弊，首先**第一种方案**，提高了 softirq 的响应速度，但如何 softirq
过多或者处理时间太长就会导致用户态线程已知得不到运行；而**第二种方案**则会增加
softirq 的响应延迟。

实际上，softirq 的处理函数`do_softirq()`是采用折中的方案，它会在内部循环检查 10 次
（是可配置的），检查有无新的 softirq 到来。对于那些在循环之后到来的 softirq，那么
唤醒 ksoftirqd 线程来处理剩下的，不延迟用户态的运行。

### ksoftirqd 内核线程

ksoftirqd 是一个内核线程，每个 CPU 都有，它的任务是不断检查是否存在挂起的 softirq， 并
执行其处理函数。

```c
for (;;) {
    set_current_state(TASK_INTERRUPTABLE);
    schedule();
    while (local_softirq_pending()) {
        preempt_disable();
        do_softirq();
        preempt_enable();
    }
}
```

ksoftirqd 的优先级较低，这样当`do_softirq()`循环 10 次还有新的 softirq 时，
唤醒 ksoftirqd 线程不会耽误用户态的执行，但当系统空闲时间，挂起的 softirq 又
会很快得到处理。

## tasklet

tasklet 是基于其中一个软中断(TASKLET_SOFTIRQ)构建，其关系有点像线程与用户态
线程之间那种嵌套关系。

tasklet 的分配可以是运行时确定的(例如使用 insmod)增加新的 tasklet。

> 内核对 tasklet 的服务函数进行了更加严格的控制：**不能在多个 CPU 上同时运行同一个类型的
> tasklet 函数(不同类型的 tasklet 可以)。**，这样就使得 tasklet 服务函数不必非得
> 实现为可重入的， 简化驱动开发者的工作。

### 表示 tasklet 的数据结构

描述一个 tasklet 的数据结构为`tasklet_struct`, 成员包括:

```c
struct tasklet_struct {
    // 指向下一个tasklet，所有tasklet链表串联
    struct tasklet_struct *next;
    unsigned long state;
    // tasklet 对应的处理函数
    void (*func)(unsigned long);
    // func 中可以使用的数据
    unsigned long data;
};
```

### tasklet 的中断流程

`TASKLET_SOFTIRQ`的 action 指向遍历所有`tasklet_struct`的方法，该方法中
执行每个 tasklet 的`func()`。

## 工作队列

工作队列创建了一个内核线程`kworker`, 原理与`ksoftirqd`差不多。

主要的区别是`ksoftirqd`运行在中断的上下文，因为其调用了`do_softirq()`,
而中断上下文中是禁用用户抢占的，也就是说不能发生调度(不影响嵌套中断)。

> 中断上下文中禁止抢占的原因是**开启了中断嵌套**，代价是必须禁止抢占。
> 如果同时允许中断嵌套和抢占，那么“嵌套的”中断返回时如果发生了调度，
> 返回别的高优先级的进程去了，此时初级的中断还未结束。如此时在新进程里
> 又发生了初级类型同样的中断，就很有可能发生数据不一定或者死锁。

工作队列是运行在进程的上下文中的，也就是一般情况下。此时当然可以发生
抢占，所以工作队列**适用于那种需要中断服务函数需要发生调度的情况**，
比如说调用了`sleep()`.

### 工作队列
