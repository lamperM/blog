---
title: "前端学习: position"
date: 2023-08-11T21:02:04+08:00
tags: [css]
categories: ["HTML/CSS"]
---

position 属性决定了一个元素在页面中的排放方式, 通过与 top、bottom、left、right
结合可以决定任一元素在页面中应该在什么位置上。

position 的取值可以是: static/absolute/relative/fixed/sticky ，下面我将依次对他们的使用方法和场景进行介绍。

## static

static 是元素默认的 position，它使得元素按照顺序排列（什么样的顺序取决于`display`)。

它不能与 top、bottom 等属性结合，就是最简单的依次排布。

## relative

relative 与 static 相比，支持了 tom、bottom 这些属性，使得元素在依次排布的同时
能调整**相对于**上一个元素的位置变化。

据我所知，relative 并不常见。

## absolute

absolute 也就是我们常称的"绝对定位"，
产生的效果相对于父元素做了一些偏移，而不是上面所说的**上一个元素**，只有父元素的位置改变，它才按照偏移数值进行改变。
偏移数值的指定通过 top、bottom 来实现。

absolute 可以与 fixed 进行对比，两者相差很小。

> absolute 无视 static。上面说 absolute 是基于父元素进行调整，仅当父元素是
> static 时例外，absolute 会跳过这一层，找它的爷爷元素。

## fixed

fixed的含义是使元素的排列始终固定在页面的某个位置，换句话也可以说它总是基于body做relative的
排列。当然，偏移是通过top、bottom给出的。

一些页面的小广告用的排列就是fixed。

## sticky

sticky像默认的static，但它也有top、bottom等属性值，这些值有特殊含义：
当元素随着页面滚动变化，而使元素的页面绝对位置（相对于body）达到top、bottom值时，
便固定在那不会再移动，**使元素永远不会被移动出页面**。

看起来就好像是用页面的外框对sticky元素画了一个笼子，它永远跑不出页面之外。

sticky目前被广泛应用与导航栏，只要设置`top=0`
> sticky是新增的属性，某些浏览器支持的可能不是很好