

## Reference
1. [Measuring Function Duration with Ftrace
](https://citeseerx.ist.psu.edu/document?repid=rep1&type=pdf&doi=f48e8682c14355df15567be2c95307485edb1587#page=47)
2. [Finding Origins of Latencies Using Ftrace](https://static.lwn.net/lwn/lwn/images/conf/rtlws11/papers/proc/p02.pdf)
3. [Slides: Ftrace](https://events.static.linuxfound.org/slides/2010/linuxcon_japan/linuxcon_jp2010_rostedt.pdf)
4. [Slides: Ftrace Kernel Hooks:More than just tracing](https://blog.linuxplumbersconf.org/2014/ocw/system/presentations/1773/original/ftrace-kernel-hooks-2014.pdf)

## 介绍
ftrace功能 ：帮助了解Linux内核的运行时行为，可以查看系统调用情况，以及某个函数的调用流程。
2.6内核之后引入内核的。以便进行故障调试或性能分析。

Ftrace 跟踪工具由性能分析器（profiler）和跟踪器（tracer）两部分组成，

性能分析器：用来提供追踪数据的解析和图形化作战时（需要 CONFIG_FUNCTION_PROFILER=y）
* 函数性能分析
* 直方图

跟踪器：负责不同追踪事件的实现，数据的来源
* 函数跟踪（function）
* 点跟踪（tracepoint）
* kprobe
* uprobe
* 函数调用关系（function_graph）
* hwlat等

### Debugfs提供用户层控制接口

ftrace的目录：/sys/kernel/debug/tracing/ ，常用文件介绍：



* dynamic tracing，动态trace进行过滤的接口，是需要在编译时支持该功能，需要打开对应的宏开关：
* available_events
* available_filter_functions: 可追对函数的完整列表
* available_tracers，当前内核中可用的插件追踪器。
* buffer_size_kb，以KB为单位指定各个CPU追踪缓冲区的大小。系统追踪缓冲区的总大小就是这个值乘以CPU的数量。设置buffer_size_kb时，必须设置current_tracer为nop追踪器。
* buffer_total_size_kb
* current_tracer，通过该接口指定当前ftrace要使用的tracer，也就是要追踪的函数/时间。 
* dyn_ftrace_total_info:
* enabled_functions:
* max_graph_depth:
* printk_formats:
* saved_cmdlines:
* saved_cmdlines_size:
* set_event:
* set_event_pid:
* set_ftrace_filter，指定要追踪的函数名称，函数名称仅可以包含一个通配符。
* set_ftrace_notrace，指定不要追踪的函数名称。
* set_ftrace_pid，指定作为追踪对象的进程的PID号。
* set_graph_function:
* set_graph_notrace:
* trace，以文本格式输出内核中追踪缓冲区的内容，是查看trace日志的接口。
* trace_clock:
* trace_marker:
* trace_marker_raw:
* trace_options:
* trace_pipe，与trace相同，但是运行时像管道一样，可以在每次事件发生时读出追踪信息，但是读出的内容不能再次读出
* tracing_cpumask，以十六进制的位掩码指定要作为追踪对象的处理器，例如，指定0xb时仅在处理器0、1、3上进行追踪。
* tracing_on，启用/禁用向追踪缓冲区写入功能。1为启用，0为禁用。
* tracing_thresh:
* uprobe_events:
* uprobe_profile:
* 

支持的tracer包括:
* nop，不执行任何操作。不使用插件追踪器时指定。
* function，函数调用追踪器，可以看出哪个函数何时调用。
* function_graph，函数调用图表追踪器，可以看出哪个函数被哪个函数调用，何时返回。
* mmiotrace，MMIO( Memory MappedI/O)追踪器，用于Nouveau驱动程序等逆向工程。
* blk，block I/O追踪器。
* wakeup，进程调度延迟追踪器。
* wakeup_rt，与wakeup相同，但以实时进程为对象。
* irqsoff，当中断被禁止时，系统无法响应外部事件，造成系统响应延迟，irqsoff跟踪并记录内核中哪些函数禁止了中断，对于其中禁止中断时间最长的，irqsoff将在log文件的第一行标示出来，从而可以迅速定位造成系统响应延迟的原因。
* preemptoff，追踪并记录禁止内核抢占的函数，并清晰显示出禁止内核抢占时间最长的函数。
* preemptirqsoff，追踪并记录禁止内核抢占和中断时间最长的函数
* sched_switch，进行上下文切换的追踪，可以得知从哪个进程切换到了哪个进程。


ftrace有两种主要跟踪机制可以往缓冲区中写数据，一种是函数，一种是事件。前者比较酷，很多教程都会先讲前者。但对我来说，后者才比较可靠实用，所以我先讲后者。

事件是固定插入到内核中的跟踪点，我们看Linux代码的时候，经常看到这种trace_开头的函数调用：

```c
  if (likely(prev != next)) {
          rq->nr_switches++;
          rq->curr = next;
          ++*switch_count;

          trace_sched_switch(preempt, prev, next);
          rq = context_switch(rq, prev, next, cookie); /* unlocks the rq */
  } else {
          lockdep_unpin_lock(&rq->lock, cookie);
          raw_spin_unlock_irq(&rq->lock);
  }

```


## 实验

使用ftrace：分为三步

* 设置tracer类型
* 设置tracer参数
* 使能tracer







### 内核函数跟踪

```shell
cd /sys/kernel/debug/tracing

# Set tracer 
echo function > current_tracer
# Set function filter
echo vma_link > set_ftrace_filter
# Enable selected tracer
echo 1 > tracing_on

# See trace result
cat trace

```

应用的场景不多，只限于想看某几类函数的调用事件。
但是有些场景我们更可能希望获取调用该内核函数的流程（即该函数是在何处被调用），
这需要通过设置 options/func_stack_trace 选项实现。

```sh
#先关闭跟踪
echo 0 > tracing_on
# Set tracer 
echo function > current_tracer
# Set function filter
echo vma_link > set_ftrace_filter
#开启跟踪函数的调用栈
echo 1 > options/func_stack_trace
# Enable selected tracer
echo 1 > tracing_on

# See trace result
cat trace
```

如果想要分析内核函数调用的子流程（即本函数调用了哪些子函数，处理的流程如何），
这时需要用到 function_graph 跟踪器，从字面意思就可看出这是函数调用关系跟踪。

```sh
echo function_graph > current_tracer
echo *vfs* > set_ftrace_filter
echo 1 > tracing_on
cat trace
```

### 事件

可基于 ftrace 跟踪内核静态跟踪点，可跟踪的完整列表可通过 available_events 查看。



[1小时掌握ftrace内核跟踪技术 - 知乎](https://zhuanlan.zhihu.com/p/659390893)

[高效调试与分析：利用ftrace进行Linux内核追踪 - 知乎](https://zhuanlan.zhihu.com/p/661794875)


## Perf性能分析

### Wsl2编译安装Perf
apt工具总是提示找不到，所以就手动编译安装。

```shell
git clone https://github.com/microsoft/WSL2-Linux-Kernel --depth 1
cd WSL2-Linux-Kernel/tools/perf
make -j8
sudo cp perf /usr/local/bin
```