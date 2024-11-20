---
title: "ATF 解读：简介"
tags: ["Operating System", "Arm Trusted Firmware"]
categories: ["Operating System"]
date: 2024-08-31T19:28:12+08:00
---

Arm Trusted Firmware
首先，必须要说明的是，ATF 属于一个代码框架，基于他的代码框架实现的服务能够保证是安全的。

ATF 的主要特性包括：

1. 安全设备的初始化框架。ATF 的 boot 流程会走到 EL3/SEL1，提供了一个框架添加安全设备的初始化代码。
2. 完整的 boot 启动框架。其实并不只是安全相关，从 bootROM 到 Linux，其中会经过 BL1、BL2、BL31、BL32、BL33，这都属于 ATF 的一部分，ATF 提供了完整的启动框架，包括了每个安全状态下的每个 EL。
3. 为什么要提供一套完整框架的原因是，确保每个启动阶段都是安全的。每个镜像在执行前都会有验签过程，确保镜像的完整性。
4. BL31 中有 REE 和 TEE 切换的实现。
5. 支持整合多种 TrustOS
6. 有 recovery mode 的实现，进行镜像的替换升级
7. 电源管理代码框架
8. BL31 有一些运行时服务，不像其他阶段过去了就被释放

基于 ARMv7 的不同 mode，和 ARMv8 更近一步的等级和安全状态划分，ATF 将各个区域进行分割，使得各个功能模块更加独立。要不然，在此之前每个 OS 都要包括自己的电源管理、安全状态切换的代码。特别是 EL3 有自己的中断向量和内存映射之后，这种隔离性又大大增强。

## Trust OS

BL32 是 TrustOS，最初，TrustOS 提供基本的设备安全服务，例如可信引导或加密服务。如今，TrustOS 已发展到支持定制应用程序，这些应用程序可用于多种安全场景，例如安全支付或摄像头隐私保护等。

### 没有 SEL2 时的安全服务请求路径

{{< figure src="/atf_no_sel2.jpg" width="70%" attr="" attrlink="" >}}

图中绿色的都是和 Trusty 相关的

- 为什么有 SEL1 和 SEL0？因为安全功能服务的增多，bearmetal 的代码难以维护且容易出错，所以有了运行在 SEL1 的 trustOS，其中有安全外设的驱动。
- 市面上有多种 trustOS，每个开发安全应用 SEL0 的厂价都需要依赖某个特定 TrustOS 提供的 TA library，这对需要发布大量应用程序的 OEM 来说是一个问题，因为不同的应用程序与不同的可信操作系统绑定。SEL1 要运行多个 OS 很困难。
- 如上图所示，EL3 中需要有对 SMC 请求的处理，既

解决上述挑战需要一种能够促进组件化和硬件隔离的架构。该解决方案需要从 TrustZone 提供的安全隔离（仅提供两个世界之间的隔离）扩展到可以提供多个相互不信任的镜像之间隔离的架构。

• 不相互信任的的 trust 服务，驱动需要地址隔离。

• 为了使各个供应商相互协作更好，边界处需要提供标准的 API。This enables removing trusted OS vendor specific code from EL3 and non-secure EL2. 原本情况下，REE 想请求什么服务，服务提供商都得在 NSEL2/NSEL1 加特殊的 SMC 请求号。

### SEL2 改变了什么

SEL2 的引入隔离了不同的安全服务供应商，正如上面所说，他们的服务可能基于不同的 TrustOS 开发。

- Secure EL2 and SMMU provide the hardware isolation needed to meet the challenges described above. 通过 Stage2 实现内存访问隔离，SMMU 实现外设访问隔离。
- This architecture also establishes standard APIs that enable collaboration between vendors of Secure world software.

{{< figure src="/atf_sel2.jpg" width="70%" attr="" attrlink="" >}}

- SEL1 里的每个 TrustOS，甚至有些是 baremetal 的，都称之为一个 Secure partitions。 他们都是独立的环境，相互隔离，OEM 可以将不同厂商的同时放进去，不用移植。
- Secure Partition Manager (SPM) 在 SEL2，负责管理和调度这些 Secure partitions，像一个 hypervisor。
- Secure Partition Client Interface(SPCI)和 Secure Partition Run Time(SPRT) 实现统一的 REE、TEE 服务之间通信交互的逻辑，或者是 TEE 各个 SP 之间。

{{< figure src="/atf_sel2_spci.jpg" width="40%" attr="" attrlink="" >}}

### Trust 参考

[OSFC 2018 - Secure partitions in Arm Trusted Firmware-A | Sandrine Bailleux - YouTube](https://www.youtube.com/watch?v=o0czo37sCng&t=45s)

ARM 官方文档：Isolation using virtualization in the Secure world
