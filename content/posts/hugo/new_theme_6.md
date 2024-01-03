---
title: "Hugo 主题创建(6): shortcode"
date: 2023-08-20T18:39:42+08:00
tags: [hugo]
categories: ["Hugo"]
---

shortcode 可以当成是一些对 html 代码块封装的函数，在写 markdown 的时候就会方便一些，
举个例子来说，我有时需要往 post 中插入图片，并调整它的大小，这时候每次都手动写一些
html 简直是太麻烦了，使用 shortcode 就像是调用函数一样，告诉它函数名和必要的参数，
它会在生成网页时自动转换为对应的 html 语法。

shortcode 分为两种：Hugo 默认和自定义的。Hugo 默认支持的 shortcode 有这些
https://gohugo.io/content-management/shortcodes/
，这里面同时包含了告诉我们如果使用 shortcode 的基本语法。
- figure 插入图片
- ref/relref 引用本地文档

当然hugo支持创建自定义 shortcode，详细的使用方法可以看这里，
https://gohugo.io/templates/shortcode-templates/
，我会大概说一下。

1. 定义一个新的shortcode，即在`layouts/shortcodes/`下创建一个新的`xxx.html`文件，文件名就是你的函数名
2. 这个shortcode会做什么事，就是在这个html中进行实现

# 插入链接图片 
remoteFigure，参考的是[diary主题的实现](https://github.com/AmazingRise/hugo-theme-diary/wiki/Inserting-Figures)支持调整图片大小、填充样式、对齐、添加图片描述等。

>puer 主题的 [Github commit](https://github.com/wangloo/hugo-theme-puer/commit/0865c662f834fb273c1dfa11f6f30af570c40b3b)



## Reference

- [中文介绍Shortcode](https://matnoble.github.io/tech/hugo/shortcodes-practice-tutorial-for-hugo/#google_vignette)