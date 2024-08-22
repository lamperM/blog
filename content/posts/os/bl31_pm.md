---
title: "ATF 解读：电源管理"
tags: ["Operating System"]
categories: ["Operating System"]
date: 2024-08-02T19:28:12+08:00
---

## PSCI

PSCI, Power State Coordination Interface，由 ARM 定义的电源管理接口规范。
目前 PSCI 最新规格为 v1.1，《POWER STATE COORDINATION INTERFACE (PSCI) System Software on ARM® Systems》。

### 为什么要有 PSCI 协议？

以前，ARM 中设备的电源管理由 OS 内核负责，随着多 OS、虚拟化技术的发展，电源管理需要有一个单独统一的组件去完成。ARMv8 引入 PSCI 协议，OS 内核里的所有电源管理行为通过 `smc/hvc` 下陷到 hypervisor/EL3 完成，通常在 EL3。ATF 里那就是 BL31 去做这件事。

PSCI 规定了 OS 内核发送 smc 指令的标准电源管理请求格式，BL31 会按照这个格式去实现具体的方案。所以，只要 OS 是遵循 PSCI 协议的，就能使得 OS 和 BL31 解绑，通用性更好。

## SCP

BL31 接收到 PSCI SMC 消息，然后呢？这一块又是解耦的设计，电源管理具体是要一个单独的硬件控制器去做，BL31 的任务就是将 PSCI SMC 请求转化为具体电源控制器的控制指令。

SCP, System Control Processor，是一个控制电源的硬件，里面运行的固件叫 SCP Firmware。BL31 需要想办法和 SCP 通信。

## SCMI

SCMI（System Control and Management Interface），是 BL31 和 SCP 的通信接口。当然，有了 SCMI 标准接口，SOC 中的其他 IP 也可以通过 SCMI 接口与 SCP 通信控制电源。

## 热插拔 Hotplug

```c
echo 0 > /sys/devices/system/cpu/cpu1/online //拔核操作
echo 1 > /sys/devices/system/cpu/cpu1/online //插核操作
```

CPU 热插拔是在不关闭系统电源的情况下，根据需求动态“关闭”任意 non-boot CPU，可避免因为 CPU 空转造成的能源浪费。

- AP：将要被拔掉的 cpu。
- BP：处理拔核流程的 cpu。

### CPU Down

#### Linux 侧行为

```c
// 如果config中启用了hotplug功能，则定义回调函数 cpu_subsys_online/cpu_subsys_offline
struct bus_type cpu_subsys = {
	.name = "cpu",
	.dev_name = "cpu",
	.match = cpu_subsys_match,
#ifdef CONFIG_HOTPLUG_CPU
	.online = cpu_subsys_online,
	.offline = cpu_subsys_offline,
#endif
};

// 如果config中启用了hotplug功能
const struct cpu_operations cpu_psci_ops = {
	.name		= "psci",
	.cpu_init	= cpu_psci_cpu_init,
	.cpu_prepare	= cpu_psci_cpu_prepare,
	.cpu_boot	= cpu_psci_cpu_boot,
#ifdef CONFIG_HOTPLUG_CPU
	.cpu_can_disable = cpu_psci_cpu_can_disable,
	.cpu_disable	= cpu_psci_cpu_disable,
	.cpu_die	= cpu_psci_cpu_die,
	.cpu_kill	= cpu_psci_cpu_kill,
#endif


cpu_subsys_offline()
    => cpu_down(cpu = dev->id, CPUHP_OFFLINE)
cpu_down(cpu, target)
    => cpuhp_kick_ap_work(cpu) // 使AP放弃手头工作，进入CPUHP_TEARDOWN_CPU状态
        =>  wake_up_process(ap->thread);
	    =>  wait_for_ap_thread();
    => cpuhp_down_callbacks() // 进一步 clean AP
        =>  takedown_cpu(cpu)
takedown_cpu(cpu)
    => stop_cpus()
        => queue_stop_cpus_work() // !! 将die自己的线程加入AP的运行队列，
                                  // 等待AP自己die，而不是BP完全帮他
    => __cpu_die() // 见下

// ====== 切换到 AP 的执行视角
take_cpu_down()
    => __cpu_disable()
        // 设置CPU状态为OFFLINE，一旦如此，IDLE线程里就会 call idle_dead()
        => set_cpu_online(cpu, false);
        => irq_migrate_all_off_this_cpu()// 中断迁移
    => // 任务迁移
idle_dead()
    // PSCI 接口，切换到BL31执行实际电源管理。
    => cpu_die()

// ====== 返回 BP 的执行视角
// 等待 BP 完成自己的die
=> __cpu_die(cpu)
    => cpu_wait_death(cpu)
    => op_cpu_kill(cpu)
        => cpu_kill(cpu) // cpu_psci_cpu_kill()

```

总结来看：

1. 用户输入`echo 0 > /sys/devices/system/cpu/cpu1/online`
2. 唤醒 AP per_cpu 线程，完成手头的工作
3. 设置 AP cpu 的状态为 OFFLINE，AP 的 idle 线程会自己调用 psci 接口 die
4. 做 AP 上的任务迁移和中断迁移
5. BP 会等到 AP 自杀完成

#### BL31 侧行为

### CPU UP

唤醒后执行哪里的代码？是 warm boot 还是 cold boot？

## 系统休眠唤醒

```c
echo mem > /sys/power/state
```

## 关机重启
