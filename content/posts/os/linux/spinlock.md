---
title: "Linux spinlock 实现分析（ARM）"
tags: ["Operating System", "Linux", "armv8"]
categories: ["Operating System"]
date: 2024-08-18T10:51:49+08:00
---

## 背景

本文介绍 Linux 内核【票据自旋锁】的实现逻辑，票据自旋锁是为了解决原始自旋锁的公平问题。具体来说，因为有 cache 的存在，刚释放锁的 CPU 会比其他 CPU 更快到达获取锁的路径（可能不用访存），进而更有可能获得锁。

票据自旋锁大概的逻辑就是：给每个锁拍号，即便你先来的，锁也只分配给下一个被叫到号的 CPU。

## 基础算法

```c
typedef struct {
	union {
		u32 slock;
		struct __raw_tickets {
			u16 next;  // 每个CPU的local变量中，指代自己的号
			u16 owner; // 当前叫到的号，每次锁更新需要更新该值
		} tickets;
	};
} arch_spinlock_t;
```

汇编中对 lock->slock 进行原子加，实际上是操作的 lock->tickets->next 成员（低地址）。

这个值+1 成功并不代表获得了锁，只表明你取了一个号。每个 CPU 自己的号保存到 lockval 变量中，叫号机下一个号的数字存储在原子变量 lock->slock 中，这个变量必须是原子操作的，保证每个 CPU 叫到的号都非重复。

下面的 while 循环就是叫号的过程，owner 就是当前叫到的号，每个 CPU 和自己本地存储的号对比，如果相等说明到了自己，可以拥有锁，继续往下执行。

```c
static inline void arch_spin_lock(arch_spinlock_t *lock)
{
	unsigned long tmp;
	u32 newval;
	arch_spinlock_t lockval;

	prefetchw(&lock->slock);
	__asm__ __volatile__(
"1:	ldrex	%0, [%3]\n"
"	add	%1, %0, %4\n"
"	strex	%2, %1, [%3]\n"
"	teq	%2, #0\n"
"	bne	1b"
	: "=&r" (lockval), "=&r" (newval), "=&r" (tmp)
	: "r" (&lock->slock), "I" (1 << TICKET_SHIFT)
	: "cc");

	while (lockval.tickets.next != lockval.tickets.owner) {
		wfe();
		lockval.tickets.owner = READ_ONCE(lock->tickets.owner);
	}

	smp_mb();
}

static inline void arch_spin_unlock(arch_spinlock_t *lock)
{
	smp_mb();
	lock->tickets.owner++;
	dsb_sev();
}
```

## smp_mb()

这是一个证明 smp_mb()意义的例子，多个 CPU 可能同时执行这段代码，
屏障保证当一个 CPU 拿到锁后，其他 CPU 能够及时得到锁的最新状态，因为和预取的值是不同的。

```c
// 多CPU同时执行这段代码
{
    spin_lock();
    opt_share_vars();
    spin_unlock();
}

```

## dsb_sev()

当一个锁被释放，需要通知所有等待叫号的 CPU，每个 CPU 再看是不是轮到自己了。

思考：为什么不能定向唤醒到下一位等待者，不影响其他 wfe 的 CPU。

```c
static inline void dsb_sev(void)
{

	dsb(ishst);
	__asm__(SEV);
}

```

## spinlock 的发展

其他 CPU 循环死等 ==> 其他 CPU 可以 wfe 休眠，释放时唤醒 ==> 更加公平的票据自旋锁
