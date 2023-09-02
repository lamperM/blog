---
title: "Hugo 主题创建(2): 添加侧边栏"
date: 2023-08-11T15:39:42+08:00
tags: [hugo]
---

> commit: https://github.com/wangloo/hugo-theme-puer/commit/32abfccc6bafd3763e07b751f0315a5403c6eaff

与顶栏相比，我更喜欢侧边栏，现在的屏幕纵向空间很宝贵。

本文创建了侧边栏模板的框架，预留了未来实现各种功能的布局，这个过程也是第一次接触`partials/`
下的文件的作用——页面的某个组成部分。而`_default/`下的模板则是描述不同类型的页面。


# 布局
基于hugo模板的分类思想，侧边栏属于页表的一个部分，所以侧边栏的模板需要放在`partials/`下，
同理的还有footer、toc、comment等。我们给侧边栏模板起一个名字`sidebar.html`。

因为想在站点所有的页面（section、single、list）都显示侧边栏，
所以在`baseof.html`中需要引入sidebar模板：
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
    <div id="app">
        {{- partial "sidebar.html" . -}}
        <main class="container">
            {{- block "main" . }}
            {{- end }}
        </main>
    </div>
</body>

</html>
```

`sidebar.html`的内容就比较简单了，目前的计划是添加**首页、TAG和一个搜索框**，
不着急，先展位，以后再实现这些功能，本次先实现框架。
```html
<div id="sideContainer" class="side-container">
    <div class="nav-link-list">
        {{/*  TODO: 回到首页  */}}
        <div class="a-block nav-link-item " href="">
            BACK
        </div>
        
        {{/*  TODO: articles...  */}}
        <div class="a-block nav-link-item " href="">
            POSTS
        </div>

        {{/*  TODO: tags  */}}
        <div class="a-block nav-link-item " href="">
            TAGS
        </div>
    </div>
</div>


```

# 样式

这是首次接触样式的修改，也就是用到css语言。顺便说下，我非常敬佩视频网站上那些对CSS玩的很溜
的人，我觉得CSS中的细节远比我的工作中的多，可能你糊弄一下能得到一个相同的效果，但是一个优秀的
前端工程师是要清楚每个属性的作用，他们之间是如何搭配的，**绝对不是凑出结果**。

主题中如何添加css文件呢？创建模板时会自动创建目录 `static/css/`，其中可以放置一些css文件，
比如我为sidebar.html创建的叫style.css, 在样式不多的情况下其他部分的样式也都可以放在这。

不在一个目录下，如何联系html和css呢？这就不得不提到`partials/`下的另一个重要模板`head.html`,
通过`baseof.html`中它的位置就能大概知道它的作用：`<head/>`标签的模板，基本上就是描述样式的css语法嘛。
这就是html和css连接的桥梁。

以下是`style.css`文件内容：
```css
.container {
    padding-left: 25%;
    width: 75%;
    min-height: 100vh;
    white-space: normal;

}

.side-container {
  position: fixed;
  top: 0;
  height: 100vh;
  width: 25%;
  text-align: left;
  padding: 20px 0 50px 0;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}

.nav-link-list {
    flex-grow: 1;
    .nav-link-item {
      margin-bottom: 10px;
      border-right: 2px solid transparent;
      /* padding: 8px 28px 8px 30px; */
      cursor: hand;
      transition: all 0.2s linear;
    }
  }
```
有了侧边栏后，我们需要启用内容的wrap line，且目前暂时用`padding-left+width=100%`的方式
来避免当文字过长出现滚动条时滑动会破坏布局。
本质还是由于侧边栏的实现是通过padding方式，sidebar和内容互相不能感知对方。