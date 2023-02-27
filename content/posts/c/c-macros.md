---
title: "C语言工具宏"
date: 2022-11-24T01:35:24+08:00
tags: [c]
---



### 计算数组元素的个数

```c
#define nelem(array)    sizeof(array)/sizeof(array[0])
```

