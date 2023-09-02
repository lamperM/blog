---
title: "前端学习: display"
date: 2023-08-11T22:02:04+08:00
tags: [css]
---

display 是规定元素排列方式的属性，总的来说，元素的排列方式可分两种：block 和 inline。

- block 的含义是，该元素默认情况下的 width 表现为充满整个父元素，height 表现为根据内容决定。
- inline 的含义是，该元素的 width 和 height 都是必须根据内容决定，不能使用显示的`width`和`height`来改变。

即便 block 可以去设置 width, 比如为 50%, 但是它永远必须独占一行，下面的元素也不会排到它的空白处，
这就是 block 称之为 block 的原因。

细分来说，其实 display 这个属性共有五种取值： block, inline, inline-block, flex, grid。
我们将依次介绍。

# block

默认 display 方式为 block 的标签有: p, h1-h6, div, li 等

# inline

默认 display 方式为 inline 的标签有: span, a, strong

# inline-block

inline-block 是结合了 block 和 inline 的优势：既不必独占一行，又可以调整 width 和 height。

一些 button 经常使用的 display 就是 inline-block。

# flex

# grid

# 不同 display 文字居中的方法

文字居中是开发中常见的需求，然而根据 display 的不同，决定了实现居中的方式也不同。

居中大概分为水平居中和垂直居中

## 水平居中

对于 inline、inline-block 来说，直接设置`text-align=center`即可实现。

对于 block，设置`text-align=center`的效果是将 block 内部的文字相对与 block 居中，并不是
相对于 block 的父元素。

但是很多情况下，我们是想让整个 block 同时再相对于父元素居中（因为block的width不一定是100%）
此时需要对 block 元素额外添加`margin-left=auto`和`margin-right=auto`属性。

## 垂直居中

用 flex 比较方便实现。
