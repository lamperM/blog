---
title: "Hugo 主题创建(0): 脚手架"
date: 2023-08-10T17:39:42+08:00
tags: [hugo]
---
> commit: https://github.com/wangloo/hugo-theme-puer/commit/c014d1fae09eea1fcc44e03c69b6dd4d185f91fd


# 背景交代
到现在为止我使用hugo也一年多了, 记了几十条的博客，对于使用频率如此高的工具来说，
有一个顺眼的外观、方便的功能布局简直是梦寐以求。

然而，试过了这么多的现有主题，始终没有一个让我觉得满意，可能我的要求过于苛刻：
1. 搜索；我经常需要翻阅之前的博客/笔记，期望可以检索Tag，且不需要二次点击（
   上方直接是一个搜索框而不是一个按钮）。
2. TOC；要求可是展开显示三级的目录，且布局好看些。
3. 外观；简洁，不花里胡哨，代码高亮看起来舒服。
4. xxx

所以，既然Hugo是一个开源的、社区环境较好的工具，那么为什么不尝试打造一款属于自己主题呢。

我是一名嵌入式开发工程师，对于前端的知识生疏，希望在良好的社区环境下能帮助我早日完成满足我个人需求的主题。

# 计划

1. 搭建框架
2. 制作模板，熟悉模板的概念，各个模板负责的区域
3. 在上面的了解过程中逐渐加入对布局的调整，这一块可能需要学习css的知识
4. 观摩学习前人的代码，结合百家之长，磨合出适合自己的布局和功能

# 开始动手：搭建脚手架

创建的过程可以参考这个[博客](https://blog.gimo.me/posts/creating-a-hugo-theme/)
, 我主要想按照我的理解对整个框架进行详细的介绍。

# 目录结构
```
.
├── layouts
│   ├── 404.html
│   ├── _default   <--- 此次重点研究
│   │   ├── baseof.html 
│   │   ├── section.html
│   │   ├── single.html
│   │   └── list.html
│   ├── index.html  <--- 此次重点研究
│   └── partials 
│       ├── footer.html
│       ├── header.html
│       ├── head.html
│       └── script.html
├── LICENSE
├── static      <--- 目前是空的, 后续再研究
│   ├── css
│   └── js
└── theme.toml  <--- 干什么用的不清楚,后续研究
```

此次重点探讨`index.html`和`_default`目录下的文件.

index.html 是整个网站的首页, 打开你的站点最先看到的就是它的内容. 
```html
{{ define "title" }}
    首页
{{ end }}

{{ define "main" }}
    <p> 这里是个人站的首页, 仅展示用, 请DIY自己的首页覆盖它 :] </p>
{{ end }}

```
你可能看不懂这段代码, 这不是正常的html, 但马上你就能明白了.

`_dafult`下的文件被hugo称之为**模板**, 顾名思义他们的存在是为了减少代码的重复度.
不同名称的模板代表了hugo将页面划分为不同的类, 这个在官方文档中应该有介绍, 
后续可以插入个链接(TODO).
- single.html: 普通页面的模板, 我们写的博客中的内容页面就算是single page
- list.html: 列表页面的模板, 好几个博客分布在同一个子目录下, 访问这个目录的
  地址就是 list page
- section.html: `content/`是hugo工程放置内容的目录, 其下需要建立子目录,
  比如`posts/`代表这是一些博客

baseof.html需要着重介绍, 它相当于**模板的模板**, 目的是为了消除模板代码中的重复,
其中使用hugo提供的`block`语法, 建立基础模板, 目前我们baseof.html的内容如下:
```html
<!DOCTYPE html>
<html>

<head>
    {{- partial "head.html" . -}}
    <title>
        {{ block "title" . }}
        {{ .Site.Title }}
        {{ end }}
    </title>
</head>

<body>
    {{- partial "header.html" . -}}
    <main class="container">
        {{- block "main" . }} 
        {{- end }}
    </main>
    {{- partial "footer.html" . -}} 
    {{- partial "script.html" . -}}
</body>

</html>
```

这还稍微像一点html, 能看出是一个html的框架, 其中用`block`声明了`title`和`main`变量.


其他的模板文件中(包括`index.html`), 只要实现这两个变量, 就相当于把整段的代码都拷过来,
并且将`{{ block "xx" }}` 和 `{{ end }}` 之间的内容替换为你define的内容.


在`baseof.html`中对`{{ block "xx" }}` 和 `{{ end }}` 之间提前填充就实现了, 
对网页的title设置了一个默认值, 如果重定义了, 那么覆盖原先的默认值.
看`index.html`就能明白, 它将网站的title替换成了"首页", 内容也改变了, 其他的都是相当于和
`baseof.html`相同. 



> 这里不介绍hugo的更多详细语法使用, 官方文档中都有

> 这里先不介绍优先级的概念, 假设这些模板文件在全局中只有一个


# 参考

- https://gohugo.io/templates/base/