---
title: "ARMv8 基础概念"
tags: ["armv8"]
date: 2023-05-09T21:19:01+08:00
---


{{< figure src="/n2_core.jpg" caption="" attr="" width="90%" >}}

指令执行的过程：取指、译码、执行和写回。
根据这个将Core分为两部分：
1. 前端：指令从内存预取到Cpu，解码，发射。在Arm中，
   前端的流程是顺序的。一个cycle里最多可以解码4条指令给后端。
2. 后端：后端的作用是执行指令。后端一般包含几个执行单元，
   整数、浮点数、Load/Store、Branch相关的执行单元。
   后端中，如果指令之间没有依赖，支持乱序执行。
   
Core中的Cache分布：Icache在前端，Dcache属于后端。

Core中的Tlb分布：ITlb在前端，DTlb在后端

指令执行完后，到达Retire，指令退役。


## 程序性能的分析方法

性能的问题可能出在前端，称为前端Stall，在后端时则称为后端Stall。

性能指标：
- IPC：IPC=INST_RETIRED / CPU_CYCLES，IPC并不能单独判断
  是否性能比较好，比如说在某个处理器上，前端最多一个Cycle发射4
  条指令，那么IPC是不是越接近4越好呢？其实不是，**还要结合CPU
  此时正在做什么事情**，如果是死循环，那么就不代表什么。
- Pipeline Stalls
  - Stall Front-end rate=STALL_FRONTEND/CPU_CYCLES
  - Stall Back-end rate=STALL_BACKEND/CPU_CYCLES
- Frontend Bound
  - ITLB events
  - I-Cache events
- Backend Bound
  - DTLB events
  - Memory System related events
  - D-Cache events
- Retiring
  - Instruct Mix
- Bad Speculation
  - Branch Effectiveness events

## 程序性能的分析工具

### Linux Perf

```sh
# Counting
perf stat -e <event list>
# Event based sampling
perf record -e <event list>
# SPE sampling
perf record -e árm_spe_0/ts_enable=1'
```



## 与 ARMv7 相比的改动

- 指令集： 新增 A64 指令集， 但也兼容原来的 A32 指令集
- 权限等级： AArch64 下新增 EL0-EL3 异常等级，对应 V7 的特权等级
- 通用寄存器：31 个通用寄存器，V7 15 个
- 虚拟地址长度：64 位的地址长度，理论支持 256TB 的寻址范围

## Arm 处理器的架构与微架构

架构可以理解为由**指令集、内存模型等**组成的一个**行为规范**，
或叫做 specification。相当于一种标准，会定义 Cpu 工作行为的预期，
并不会限制具体是如何实现。

微架构就是**整个流水线的设计，包括前端和后端**具体的设计与实现，
由芯片厂商自行开发。


## Armv8.2 SPE

SPE的全称为Statistical Profiling Extension, 统计分析扩展，
是Armv8.2引入的一个特性。

PMU能统计事件发生的次数，但是无法直到是哪一条指令导致的。
采样因为要通过中断的方式，所以准确性不高，太高的准确率会造成系统很大的负担。
因此开发者一般只能确定是哪一个函数，但是确定哪一行就比较困难。

SPE就是通过硬件的方式解决这个问题，直接在流水线上对指令进行采样。
用硬件对性能的损耗就会很低。

有个计数器，没取指一次，计数器-1，减到0之后的指令就是要采样的指令。

采集的指令非常全面：时间戳、PC、指令的分类、运行的时间等。

采集之后，可以做一次过滤，只保存关心的事件（执行时间大于几个Cycle？指令类型lr/str？）

保存的buffer满了之后，触发中断，软件读取。


### 借助perf使用SPE

- FEAT_SPE   Armv8.2  Support from 4.14 and 5.3
- FEAT_SPEv1p1 Armv8.5 Only support from 5.11

```sh
perf list | grep arm_spe_0

perf record -e arm_spe_0/branch_filter=1,ts_enable=1,pct_enable=1,pa_enable=1,\
load_filter=1,jitter=1,store_filter=1,min_latency=0/ -- ./user.app

perf report -D -i perf.data
```





- [Arm架构下性能分析与优化介绍-云视频-阿里云开发者社区](https://developer.aliyun.com/live/252287?spm=a2c6h.28322828.J_3909373260.17.88865488AzlJRR)
