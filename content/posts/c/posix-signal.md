---
title: "A Demo of Posix Signal"
tags: ["C"]
categories: ["C"]
date: 2024-04-05T20:51:49+08:00
---

## 数据结构和 API 介绍

```c
// 信号配置中最关键的数据结构
struct sigaction {
    void     (*sa_handler)(int); // 信号触发钩子函数（简单版）
    void     (*sa_sigaction)(int, siginfo_t *, void *); // 信号触发钩子函数（复杂版）
    sigset_t   sa_mask;
    int        sa_flags;  // 一些标识位，下面会介绍
    void     (*sa_restorer)(void);
};
// 几种常用的sa_flags:
// SA_SIGINFO ==> 触发信号后会调用更强大的 sa_sigaction()
// SA_ONSTACK ==> 使用单独的栈运行handler，避免函数太大撑爆原来程序栈
// SA_RESTART ==> 使信号触发可以打断阻塞状态（如read()等待），
//                此时errno将被置为 EINTR

// 为某个信号自定义触发handler，以及某些配置项
int sigaction(int signum, const struct sigaction *act,
                     struct sigaction *oldact);
// 在当前进程中触发一个信号
int raise(int sig);
// 向pid进程发送一个信号（本案例中用不到）
int kill(pid_t pid, int sig);
```

## 信号从触发到被捕获的过程

{{< figure src="/posix_signal.jpg" width="70%" >}}

## 简单版 Demo

只有一个进程

- 为 `SIGUSR1` 设置 handler
- while(1)循环接受用户输入字符
- 当输入 q 时为当前进程触发信号
- 进入设定的信号 handler，设置进程退出标志
- 进程退出，终止

```c
#include <stdio.h>
#include <string.h>
#include <signal.h>
#include <assert.h>

static int need_exit;

static void
sig_handler(int sig)
{
  assert (sig == SIGUSR1);

  printf("SIGUSR1 triggered! Set need_exit, program will exit\n");
  need_exit = 1;
}

static void
install_signal_handler()
{
  struct sigaction s;
  int ret;

  memset(&s, 0, sizeof(s));
  s.sa_handler = sig_handler; // 重设信号的处理函数
  // struct sigaction 的其他参数暂不关心

  ret = sigaction(SIGUSR1, &s, NULL);
  assert(ret == 0);
}

int
main(void)
{
  char c;
  pid_t pid;

  install_signal_handler();

   while ((c = getchar())) {
     // printf("char: %c:\n", c);
     if (c == 'q') {
       raise(SIGUSR1);
     }
     if (need_exit)
       break;

  }
  return 0;
}
```

## 高级版 Demo

- 设置 SA_SIGINFO 标志，即启用参数更多的 handler
- 实现简易的批处理系统？
  - info.si_addr 来判断发生的地址，

重新注册了三个信号的 handler，分别有不同的作用

- SIGUSR1 模拟用户态程序发送 IO 请求
- SIGUSR2 模拟用户态程序调用 yield()主动让出 CPU
- SIGVTALRM 模拟 timer 中断
- SIGSEGV 不是显式触发，程序执行出错自动触发被捕捉。
  - 出错可能有多种原因，利用 hangler 中的 `siginfo_t` 参数区分不同情况
  - `siginfo_t` 的结构和成员说明在下方
  - 主要用到 si_addr, si_code 来区分 syscall、iret、page_fault 行为

```c

static void
sig_handler(int sig, siginfo_t *info, void *ucontext) {
  thiscpu->ev = (Event) {0};
  thiscpu->ev.event = EVENT_ERROR;
  switch (sig) {
    case SIGUSR1: thiscpu->ev.event = EVENT_IRQ_IODEV; break;
    case SIGUSR2: thiscpu->ev.event = EVENT_YIELD; break;
    case SIGVTALRM: thiscpu->ev.event = EVENT_IRQ_TIMER; break;
    case SIGSEGV:
      if (info->si_code == SEGV_ACCERR) {
        switch ((uintptr_t)info->si_addr) {
          case 0x100000: thiscpu->ev.event = EVENT_SYSCALL; break;
          case 0x100008: iret(ucontext); return;
        }
      }
      if (__am_in_userspace(info->si_addr)) {
        assert(thiscpu->ev.event == EVENT_ERROR);
        thiscpu->ev.event = EVENT_PAGEFAULT;
        switch (info->si_code) {
          case SEGV_MAPERR: thiscpu->ev.cause = MMAP_READ; break;
          // we do not support mapped user pages with MMAP_NONE
          case SEGV_ACCERR: thiscpu->ev.cause = MMAP_WRITE; break;
          default: assert(0);
        }
        thiscpu->ev.ref = (uintptr_t)info->si_addr;
      }
      break;
  }

  if (thiscpu->ev.event == EVENT_ERROR) {
    thiscpu->ev.ref = (uintptr_t)info->si_addr;
    thiscpu->ev.cause = (uintptr_t)info->si_code;
    thiscpu->ev.msg = strsignal(sig);
  }
  setup_stack(thiscpu->ev.event, ucontext);
}

// signal handlers are inherited across fork()
static void
install_signal_handler() {
  struct sigaction s;
  memset(&s, 0, sizeof(s));
  s.sa_sigaction = sig_handler;
  s.sa_flags = SA_SIGINFO | SA_RESTART | SA_ONSTACK;
  __am_get_intr_sigmask(&s.sa_mask);

  int ret = sigaction(SIGVTALRM, &s, NULL);
  assert(ret == 0);
  ret = sigaction(SIGUSR1, &s, NULL);
  assert(ret == 0);
  ret = sigaction(SIGUSR2, &s, NULL);
  assert(ret == 0);
  ret = sigaction(SIGSEGV, &s, NULL);
  assert(ret == 0);
}
```

{{< notice info siginfo_t >}}

从 `siginfo_t` 中截取了某些关键成员，对他们的含义进行说明：

```c
// [si_code] indicating why this signal was sent.
//           需要 man sigaction 才能找到对应与某个信号的所有si_code。
// [si_addr] the address of the fault.
siginfo_t {
    int      si_signo;     /* Signal number */
    int      si_errno;     /* An errno value */
    int      si_code;      /* Signal code */
    int      si_trapno;    /* Trap number that caused
                              hardware-generated signal
                              (unused on most architectures) */
    pid_t    si_pid;       /* Sending process ID */
    uid_t    si_uid;       /* Real user ID of sending process */
    int      si_status;    /* Exit value or signal */

    union sigval si_value; /* Signal value */

    void    *si_addr;      /* Memory location which caused fault */

    int      si_fd;        /* File descriptor */
}
```

{{< /notice >}}

## 参考

- 南京大学 AM 中 native 实现 CTE 的方案
