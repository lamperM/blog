---
title: "Hugo 主题创建(5): Tag 分类支持"
date: 2023-08-15T18:39:42+08:00
tags: [hugo]
---

通过 tag 可以实现对post进行分类，用到的支持是 HUGO Taxonomy Template（分类模板）

# 原理

实现tag的功能需要完成两类页面的设计： `/tags/` 和 `/tags/<one-tag>`

前者属于 Taxonomy Terms（分类术语）页面，用分类术语模板实现，
后者属于 Taxonomy List （分类列表）页面，用分类列表模板实现，他们都属于 Taxonomy 模板。

不难推测出，分类术语模板规定了如何展现某个分类方式，比如说用云图来展示tag分类方法。
而分类list模板的作用是展示选中某一类之后的页面，比如说在云图中选中了某个tag。

更加详细的描述可以看官方文档: https://gohugobrasil.netlify.app/templates/taxonomy-templates/

# 设计

正与文档中所说，分类terms模板可以有多个查找的优先级：
>1. /layouts/taxonomy/<SINGULAR>.terms.html
>1. /layouts/_default/terms.html
>1. /themes/<THEME>/layouts/taxonomy/<SINGULAR>.terms.html
>1. /themes/<THEME>/layouts/_default/terms.html

这样的好处是，比如说我有两种terms，tag和categories，我想在分类术语页面对这两种分类展示不用的页面，
就可以定义`tags.terms.html`和`categories.terms.html`, 而我目前就用`terms.html`，简单。

分类list模板也是，使用最通用的`list.html`, 和其他的list公用，并没有对分类list做单独的页面。

> 对应的 commit: https://github.com/wangloo/hugo-theme-puer/commit/63d8bb762b16a3d4657ba3523d6b6fb38cf5f9ca
>
> 上面的commit不小心提交了menu.html, 实际不属于taxonomy的目的，所以在这纠正: https://github.com/wangloo/hugo-theme-puer/commit/0af07f66807b540fd9d3be84e8d7faca7f962c4b
