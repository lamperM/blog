---
title: "Linux 内核开发经验技巧"
tags: ["Operating System", "Linux"]
categories: ["Operating System"]
date: 2024-05-17T14:51:49+08:00
---


查看内核的符号

```bash
cat /proc/kallsyms | grep tracepoint
```


查看内核启动log

```bash
dmesg | more
```

运行时查看内核的设备输

在 `/sys/fireware` 下是关于Linux内核运行时的设备信息，其中fdt是dtb格式。而devicetree/ 按照层级目录的形式展现dts。

查看内核的.config

```bash
# zcat 快速预览压缩文件内容
zcat /proc/config.gz
```