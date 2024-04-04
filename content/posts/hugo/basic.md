---
title: "Hugo 基础概念"
date: 2022-05-21T17:39:42+08:00
tags: [hugo]
categories: ["Hugo"]
---


本章将解答Hugo是什么, 以及Hugo是如何工作的. 只有了解Hugo的工作机制之后, 才能发挥想象力进行DIY. 

本章内容大多来自[官方手册](https://gohugo.io/documentation/)或者搜索引擎提供的结果.

## Hugo 项目结构

一个hugo 项目通常包含以下内容:

```
.
├── archetypes
├── config.toml
├── content
├── data
├── layouts
├── public
├── static
└── themes
```

这里面有些是必须的, 有些是可选的.

**archetypes**

定义新创建post时, header的格式. 

**asserts**

> Note: assets directory is not created by default.

**config**

Hugo uses the `config.toml`, `config.yaml`, or `config.json` (if found in the site root) as the default site config file.

The user can choose to override that default with one or more site config files using the command-line `--config` switch.

```
hugo --config debugconfig.toml
hugo --config a.toml,b.toml,c.toml
```

> `Config` directory is not created by default.

**content**

显然, 存储所有的post.

**data**

This directory is used to store configuration files that can be used by Hugo when generating your website. 

像是你 website 的一个mini 数据库, 你可以放置 toml, yaml, json格式的文件.

**layouts**

Stores templates in the form of `.html` files that specify how views of your content will be rendered into a static website. Templates include [list pages](https://gohugo.io/templates/list/), your [homepage](https://gohugo.io/templates/homepage/), [taxonomy templates](https://gohugo.io/templates/taxonomy-templates/), [partials](https://gohugo.io/templates/partials/), [single page templates](https://gohugo.io/templates/single-page-templates/), and more.

**public**

保存build生成的站点. 当运行`hugo [flag]`时生成. 

拷贝该目录下的内容, 可以部署到web 服务器上了.

**static**

Stores all the static content: images, CSS, JavaScript, etc. 当Hugo构建您的站点时，静态目录中的所有资源都会按原样复制。

即当构建website时, `static/`下的所有文件都会复制到 `public/`下. 

The static files are served on the site root path (eg. if you have the file `static/image.png` you can access it using `http://{server-url}/image.png`, to include it in a document you can use `![Example image](/image.png) )`.

**resources**

一些缓存文件

> `resources` directory is not created by default.

## Hugo Cli Command

hugo 支持的所有命令可以通过 `hugo help` 命令来查看. 每一条命令的具体用法, 可以执行 `hugo [command] help` 来查看

```
Usage:
  hugo [flags]
  hugo [command]
```

**hugo completion**

用来配置补全 hugo command 和 flag 的. 该命令会输出一段脚本, 将该脚本复制到你的 shell 的配置文件中就可以使用 hugo tab 补全了.

**hugo config**

打印hugo的配置文件, 即根目录下的 `config.toml`.

**hugo env**

打印 hugo 的版本和环境信息

**hugo list**

打印所有post的info, 包含标题, 发布日志, 链接等.

**hugo new**

非常重要的命令, 可以用来新建一个 website, 主题, 或者一篇post(常用). 带有许多 flag可以使用.

**hugo server**

执行`hugo server`之后, 首先构建了你的网站(但是默认并不在本地创建文件, 而是放在内存), 然后启动hugo 自带的 web服务器让我们能看见网站的效果.

{{< notice warning >}}
不知道从哪个版本开始，反正在hugo v0.124.1下，默认是直接生成文件，如果还是希望放在内存，需要手动添加参数： `--renderToMemory`。
（PS：搞不懂为什么要做这个改动！？
{{< /notice >}}

同时, 默认情况下, server 会同步你的本地更改, 然后实时的reload你的页面. 这样你就能同时看到修改的效果.

hugo server 的常用flag:

```
-D              包含标记为草稿的post. 默认不构建草稿.
--theme strings 使用[strings]主题进行构建
```

**hugo [flags]**

`hugo` 自身就是一个命令, 用于build website, 放到 `public/`目录下. 

常用 Flag([All supported flags](https://gohugo.io/getting-started/usage/#test-installation:~:text=The%20output%20you%20see%20in%20your%20console%20should%20be%20similar%20to%20the%20following%3A)): 

```
--gc                         在build后会清除一些cache文件. 与 resource/有关
--minify                     minify any supported output format (HTML, XML etc.)
```



> `hugo` 命令不会删除之前的文件. 而是仅新增改动. 所以每次build时需要你手动删除 `public/` 目录.





## Hugo 内容管理

hugo build 后的website页面的布局和你源文件的布局相同, 所有源文件都放置在 `content/` 目录下.

```
└── content
    ├── _index.md   // <- https://example.com
    |
    ├── about
    |   └── index.md  // <- https://example.com/about/
    ├── posts
    |   ├── _index.md     // https://example.com/posts/
    |   ├── firstpost.md   // <- https://example.com/posts/firstpost/
    |   ├── happy
    |   |   └── ness.md  // <- https://example.com/posts/happy/ness/
    |   └── secondpost.md  // <- https://example.com/posts/secondpost/
    └── quote
        ├── first.md       // <- https://example.com/quote/first/
        └── second.md      // <- https://example.com/quote/second/
```

> hugo 将content/ 下的那级目录(例如 content/posts)特殊看待, 称为 *section*.

### 页面资源(Page Resources)

页面资源指每个页面**私有的**图片, 文档等静态资源. 与`static/` 中全局的资源不同.

页面资源放在`content/`下的任意位置, 但不是所有页面都能访问. page bundles 中的`index.md` or `_index.md` 能够访问该 bundles 下的资源.

```
content
└── post
    ├── first-post
    │   ├── images
    │   │   ├── a.jpg
    │   │   ├── b.jpg
    │   │   └── c.jpg
    │   ├── index.md (root of page bundle, 能够访问first-post/下的所有资源)
    │   ├── notice.md  不能访问任何资源, 但其自身作为一个资源可被index.md访问
    │   ├── office.mp3
    │   ├── pocket.mp4
    │   ├── rating.pdf
    │   └── safety.txt
    └── second-post
        └── index.md (root of page bundle, 但不能访问first-post/下的资源)
```

### 内容分类(Taxonomy)

Taxonomy: How to group the content together. Two default taxonomies are *tags* and *categories*.

### 代码高亮(Syntax Highlight)

[代码高亮的配置](https://gohugo.io/getting-started/configuration-markup/#configure-markup:~:text=anchorize%20template%20func.-,Highlight,-This%20is%20the)(in `config.toml`): 


Hugo 使用[chroma](https://github.com/alecthomas/chroma)来执行代码高亮。
- Example of all style: https://xyproto.github.io/splash/docs/index.html

### 页面分类

从布局上来看, 页面可以分为两类: List page 和 single page.

显而易见, list page比较特殊, 它负责列出当前目录下的所有post. 所以一个目录地址必然是一个list page. 

在下面的例子中, `https://example.com` , `https://example.com/posts/happy/` 都可以叫做 list page. 

* `https://example.com/posts/happy/` 是list page, 目录下的`_index.md` 不是必须的, hugo 会默认仅显示所有post的title. [详见](https://gohugo.io/templates/lists/#list-pages-without-_indexmd)

*  `https://example.com/about/` 不是list page, 因为其目录下有`index.md`, 强制表明这是一个 single page. [详见](https://gohugo.io/content-management/page-bundles/#:~:text=CONTENT%20MANAGEMENT-,Page%20Bundles,-Content%20organization%20using)

```
└── content
    ├── _index.md   // <- https://example.com
    |
    ├── about
    |   └── index.md  // <- https://example.com/about/
    └── posts
        ├── _index.md     // https://example.com/posts/
        ├── firstpost.md   // <- https://example.com/posts/firstpost/
        ├── happy
        |   └── ness.md  // <- https://example.com/posts/happy/ness/
        └── secondpost.md  // <- https://example.com/posts/secondpost/
```

> Homepage 和 section page 都属于特殊的 list page. 
>
> * homepage 特指 `content/_index.md`
> * section page 特指 `content/[section]/_index.md`



### shortcodes

shortcode 可以理解为 hugo 为了封装了一些代码块, 通过 shortcode 来调用.



## 模板(Template)

模板是hugo的一个高级用法, 用来定义你网站的style. 模板不等同与主题(themes), 可以理解为主题是一套模板的集合. 我们可以在使用模板的同时添加DIY的 style. 😎 Hugo 会有优先级的判断.

不同的页面类型需要定义不同的模板. List page 的模板称为 List template, single page 的模板称为 single template. 同理还有 homepage template, section template.

存储模板的目录为`layout/`, 上面介绍hugo的目录结构时已经说过. 如果你使用了一个 theme, 那么`themes/[your-theme]/layout/`就是该theme的模板.

### homepage 模板

### Base 模板

对应`layouts/_default/baseof.html`

base 模板是整个website的核心. 所有的模板包括 list template, single template, homepage template... 都是独立的, base template 将其他的模板联系到一起.

### 分页模板

对于条数过多的场景（比如所有的post list，某个tag的post list），可以构建一个分页器，用【上一页】【下一页】来使单个页面简洁一点，避免滑不到头的情况 :X

不过对于目前，我还没有那么多条目看不过来，所以暂且没细看。

[Pagination模板 官方说明](https://gohugo.io/templates/pagination/)

### partial 模板

包含网站的许多元素, 增加模块化. 我可以为网站的 header 或者 footer 写一个模板(html), 这些HTML可以嵌入其他的模板.



### 模板优先级

既然同一种页面的模板可以定义在多个位置, 如果他们同时存在时, 优先级规则必然存在. 常见的情况比如我们使用了某个模板, 然而, 我们对模板中的一些布局不满意, 直接修改模板中的文件显然不是一个好方法, 那么该怎么做呢?

**一般来说**, 如果你只想重写theme中的某个模板, 例如section template. 那么你只需要新建 `layout/_default/section.html` 即可, hugo 构建你的网站时, 如果检测到本地和theme的`layout/_default`下都有 `section.html`, 它会使用我们自己定义的那个. 

> 完整的, 多级的优先级规则: [Hugo's Lookup Order | Hugo (gohugo.io)](https://gohugo.io/templates/lookup-order/)



## 变量(Variables)

❗ Hugo 变量仅设计给模板使用, 即在`layouts/`下的html文件.

### Page Variables

与post相关的变量, 定义在post 的 front matter中. 

```
// Define Page variables in front matter of post
----------------
title: "使用 HuGo 搭建个人网站"
description: 学习正确的 Hugo 食用方式, DIY 属于自己的 website~
Myvar: "my value"
----------------

// Use Page Variables
{{.Description}}   // Get the description of the post
{{.Params.Myvar}}  // Get the value of Myvar, that is, "my value"
```



### Site Variables

站点层面的变量大部分是网站配置相关. 



## 函数(Functions)

函数是hugo为你封装的一些方法你可以直接调用.

❗ Hugo 函数仅设计给模板使用, 即在`layouts/`下的html文件. Same as variables.

## Hugo pipes


## Hugo相关参考

- https://lewky233.top/categories/hugo%E7%B3%BB%E5%88%97/ 一系列的hugo配置和美化记录
- https://github.com/heartnn/hugo-theme-test/blob/master/README.md hugo基础知识

## 支持 Emoji

[Adding emoji tutorial](https://stackoverflow.com/questions/41047920/adding-emoji-to-a-hugo-page-variable)

[Emoji chart](https://www.webfx.com/tools/emoji-cheat-sheet/)


