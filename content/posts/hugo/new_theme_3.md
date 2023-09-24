---
title: "Hugo 主题创建(3): 站内搜索"
date: 2023-08-11T16:39:42+08:00
tags: [hugo]
categories: ["Hugo"]
---

> commit: 

# 为什么选择fast search?
hugo本身是不支持站内搜索功能的, 如果你写的文章较多就只能按照tag去检索分类.
这样至少也需要三次点击操作, 如果每个页面的边栏或者顶栏有一个搜索框, 能够
搜索文章的内容或者标题、Tag这些，对我来说效率就能得到显著提升。

[fast search](https://gist.github.com/cmod/5410eae147e4318164258742dd053993)
是我检索到的目前比较简单、成熟的方案，它的亮点：
1. 最小外部依赖（无需jQuery）
2. 支持实现键盘唤出
3. 无需NPM, grunt等外部工具
4. 无需额外的编译步骤，你只需要像往常一样执行hugo
5. 可以方便地切换到任意可使用json索引的客户端搜索工具

# 集成
集成的步骤我是参照的[这篇文章](https://ttys3.dev/blog/hugo-fast-search)
, fast search官方也有说明类似的步骤，过程不难，大概可分为：
> 1. Add index.json file to `layouts/_default`
> 2. Add JSON as additional output format in `config.toml`
> 3. Add `search.js` and `fuse.js` (downloaded from fusejs.io) to `static/js`
>4. Add searchbox html 到你想布局的位置
>5. 对searchbox添加样式文件

具体的步骤看博文或者官方文档就行，这里不赘述。


## 改动
做了一些让自己舒服的改动：
1. 让搜索框常驻，只是搜索结果可以隐藏(ESC)
2. `/`聚焦搜索框，和vim相同
3. 简化样式，贴合我的主题
4. 搜索结果只显示title就够

这样以后不论在哪，想要切换到一篇文章只需要两次鼠标（或者两次键盘）就能精准定位并打开，不必使用鼠标的方式可能更有作用哈哈。

## TODO
1. 只能搜索标题，不能搜索内容、tag？
