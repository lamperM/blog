---
title: "aptitude修复apt安装依赖"
tags: [tools]
categories: ["DevTools"]
date: 2024-04-08T15:28:12+08:00
---

Ubuntu 下用 apt 安装包出现依赖问题：
{{< figure src="/apt_err.png" width="90%" >}}

尝试加`-f`安装也仍然报相同错误。

在网上查到 aptitude 专用鱼解决 apt 依赖问题，遂尝试。

1. 下载 aptitude

   ```bash
   sudo apt install aptitude
   ```

2. 用 aptitude 重新安装，aptitude 会给出几种解决方案，第一种是不安装，第二种是将依赖的软件降级消除依赖。我们显然选择第二种方案。

{{< figure src="/apt_ok.png" width="90%" >}}

>参考：[解决Ubuntu下因依赖包而无法安装问题 - soarli博客](https://blog.soarli.top/archives/22.html)
