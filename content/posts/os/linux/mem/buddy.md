---
title: "Linux Buddy 内存分配器"
tags: ["linux", "Operating System"]
categories: ["Operating System"]
date: 2023-09-18T17:51:49+08:00
---

# 伙伴系统的优势

作为一个页分配器，伙伴系统主要解决外部碎片过多的问题，
保证系统中尽可能有大的连续空间可以使用。

这也正是伙伴系统要设计成相邻内存块合并的原因。