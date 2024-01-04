---
title: "杂乱的开发日记"
tags: ["Thinking"]
categories: ["Thinking"]
date: 2023-12-17T17:19:44+08:00
---

零零碎碎的开发笔记，如果思考比较多应该写成单独的博文。


## 6.828

### 还能这么拷贝代码
将一段汇编代码从一个地址拷贝到另一个地址，你会怎么做？

我能想到的是利用链接脚本，将该段代码限定在某个段里，然后利用变量来定位代码所在的地址，执行拷贝。

init.c的boot_aps()提供了一种新的思路：在代码的前后定义一个全局变量，就能在外面访问到代码的地址和范围了。

### 转换函数的一种写法
我们经常会用到两种指代之间的转换，比如用id找到结构体指针（JOS中的`envid2env`），你会怎么设置函数的参数和返回值？

我以前只能想到：
```c
struct Env *envid2env(envid_t envid)
```
正常情况下返回转换完成的结构体指针，否则返回NULL。

然而，JOS提供了一种别样的实现方式：
```c
int
envid2env(envid_t envid, struct Env **env_store, bool checkperm)
{
	struct Env *e;

	// If envid is zero, return the current environment.
	if (envid == 0) {
		*env_store = curenv;
		return 0;
	}

	e = &envs[ENVX(envid)];
	if (e->env_status == ENV_FREE || e->env_id != envid) {
		*env_store = 0;
		return -E_BAD_ENV;
	}

	if (checkperm && e != curenv && e->env_parent_id != curenv->env_id) {
		*env_store = 0;
		return -E_BAD_ENV;
	}

	*env_store = e;
	return 0;
}
```

外面传入一个指针的地址（无需申请空间），是传出参数。这样的好处是**空出一个返回地址，可以用来表示多种出错的类型**。

## spring OS
### 2023-07-01

如果kernel也用低地址， 其实是不行的，因为这样在切换到用户
进程的时候，需要切页表对吧。 但是切换之后用户进程的ttbr是
没有内核的页面映射的。 它们又都用了用一个页表。

所以说看来是还是需要用两个页表实现起来比较方便些，
让内核用ttbr1，即映射在高地址。 做法是：
1. 首先boot阶段的代码是要在低地址的，因为此时没有开mmu（用uboot+PIC就没有这个顾虑）。
2. 在boot代码中，mmu开启前需要建立映射，除了映射内核的代码段、数据段之外。
还有一个很重要的是恒等映射ttbr0，需要将boot代码建立恒等映射。
1. 等到mmu启动后，访问的还是boot的代码，这时恒等映射生效。
2. 但是当跳转到内核的代码时， 因为VMA是高地址， 所以用ttbr1，之前就映射好了。可以直接访问的。

### 2023-07-02

task_init():

task->affinity = -1

如果不是内核任务, 默认情况下task->state = TASK_STATE_WAIT_EVENT, 在create hook中进行的
内核任务的话, task->state = TASK_STATE_SUSPEND, 在task_init()中进行的


除了idle外, 其他task的task->cpu = -1, 在task_init()中进行的



wakeup_common():
task->task = TASK_STATE_WAKING
task->pend_stat = pend_state


task_ready():
1. 更新task->cpu, 如果初始值是-1, 更新为NR_CPUS-1, 这是TBD, 否则是affinity
2. 然后根据当前cpu是否为task->cpu, 做出判断:
	1. 如果是, 那么直接调用add_task_to_ready_list() 将任务加到readylist中
	2. 否则,将任务加到new_list, 而不是readylist, 然后发送核间中断通知task->cpu


sched_tick_handler():
1. 当前的task->ti.flags | __TIF_TICK_EXHAUST
2. 当前的task->ti.flags | __TIF_NEED_RESCHED

irqwork_handler():
1. 遍历当前pcpu->next_list:
	1. 将task->state = TASK_STATE_READY
	2. task加入pcpu的readylist


