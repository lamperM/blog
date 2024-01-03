---
title: "Hugo 引用图片"
date: 2024-01-03T13:39:42+08:00
tags: ["Hugo"]
categories: ["Hugo"]
---

写博文时避免不了插入一些图片，总结了几种方式。

## 图床

最早在其他平台写博客时，因为 Markdown 格式编辑，不方便内嵌图片（好像听说支持 Base64 编码，
没试过）， 此时想要维护单个 md 文件，最好的方式就是用网络图片，
本地引用必须同时维护图片和文件在同一个目录，而且一些博客平台上传图片太麻烦。

Markdown 插入图像的语法本就支持网络图片，这里防一张图片作为演示：

![百度Logo](https://www.baidu.com/img/PCtm_d9c8750bed0b3c7d089fa7d55720d6cf.png)

以前用的 Gitee（码云）搭建图床，后面码云官方禁止这种行为，考虑过换成其他收费的平台，
例如各家云公司的对象存储 OSS。但是仔细想想如果要迁移平台意味着所有的博客都要改动，
未免太麻烦了。

## Hugo

使用 Hugo 搭建静态博客页面之后，其实对于远程链接的引用方式的依赖性就消失了。
反正都是用一个 Hugo 工程管理所有笔记，那么也统一管理所有图片也没太大所谓。
以前觉得本地管理很麻烦，需要传图片之类的，但最近用 Latex 写论文发现也还好。
工程放在 Windows 上，传图片直接另存为改下目录就行了，用虚拟机上确实不太方面。
还好 Hugo 对 Windows 的支持还不错。综上，目前就在尝试使用本地的方法管理图片。

Hugo 引用图片有两种方式：

1. 建立一个 Page bundle，图片作为 Page source。通过`![](sunset.jpg)`
   即可访问，[Hugo 官方描述](https://gohugo.io/content-management/image-processing/#page-resource)。
   属于各自 blog 的图片放到各自的目录下，这样的好处是看起啦比较清晰。
   但是麻烦的地方就在于需要引用图片的博文都需要建立一个 Page bundle，
   而且我不喜欢 index.md 这种文件名，难以搜索。
    ```
    content/
    └── posts/
        └── post-1/           <-- page bundle
            ├── index.md
            └── sunset.jpg    <-- page resource
    ```
2. 所有博文的图片都放到的`/static`目录下，统一管理。
   `/static`可能不是必须存在，可以手动创建。此时的图片通过
   `![](/sunset)` 的方式来访问，多了一条斜杠。
   **Latex论文里的图片就是这样管理的**，以前担心混乱，
   实际上因为图片不多，命名规范也还好。

### Shortcode

Md支持的原生图片引用方式调整大小、对齐等操作比较麻烦，
Hugo在这方面引入了默认的Shortcode: 
[figure](https://gohugo.io/content-management/shortcodes/#figure)
帮助实现引用本地图片。

```
{{</* figure src="/test.jpg" width="100%" */>}}
```

另外，我自己根据网上的资料写了引用外部图片的shortcode: 
[insertFigure]({{< ref "new_theme_6.md" >}})。
