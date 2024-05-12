# KMonitor 用户手册

## 修改记录

## 简介

KMonitor

## 使用方法

### 调试命令
#### 断点

```bash
# 添加断点到地址处，地址必须以0x开头的十六进制形式
bp 0xffff000012345678
# 添加断点到符号
bp pagetable_map
# 列出所有断点
bp 
# 禁用断点 1
bd 1
# 启用断点 1
be 1
# 删除断点 1
bc 1
```

#### 查看/修改内存
```bash
# 查看内存
md # TODO 格式
# 修改内存，单位是8字节
# *(0xffff000012345678) = 0x1234
mm 0xffff000012345678 0x1234
# 修改内存，单位是1字节
# *(0xffff000012345678) = 0xab
mmb 0xffff00001234567 0xab

```

#### 查看/修改寄存器
```bash
# 输出所有寄存器(hex)
rd 
# 输出某个寄存器(hex)
rd sp_el0
# 修改寄存器
rm x18 0x12345678
```

#### 栈回溯
```bash
# 打印栈回溯
bt
```



## TODO
用户如何查看记录数据？
- 用KMonitor自带的终端输出
- 类似于Linux抽象为文件，在用户侧进行读取

日志系统可以加入KMonitor，用统一的接口进行存储。

想要监控用户态程序？
- 底层用同一套接口

## 参考
- https://blog.csdn.net/oyangyufu/article/details/6245931 kdb命令