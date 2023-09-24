---
title: "Hugo 主题创建(7): footer"
date: 2023-09-02T18:39:42+08:00
tags: [hugo]
categories: ["Hugo"]
---

footer属于 partial模板之一, 创建一个新文件`footer.html`, 然后在baseof模板中, 
指定footer内容显示的位置.
```html
<body>
    <div id="app">
        {{- partial "sidebar.html" . -}}
        <main class="container">
            {{- block "main" . }}
            {{- end }}
            {{- partial "footer.html" . -}}
        </main>
        {{- partial "script.html" . -}}
    </div>

</body>
```

下面将按照功能划分, 添加各种内容到footer模板中.

# 文件创建和lastmode时间

>commit: https://github.com/wangloo/hugo-theme-puer/commit/d263d9af65808ff03b2307abfb4db397ae1bcc2a


文件创建时间是获取的footer中的变量, lastmod其实也可以通过这种方式获取,
但是这样每次修改都要手动更新太复杂, **我们可以借助git追踪的文件的修改时间来作为lastmod**,
默认不是这样的, 需要在config.toml中指定.
```toml
[frontmatter]
  lastmod = ['lastmod', ':git', ':fileModTime', 'date', 'publishDate']
```

然后就是在footer.html中引用这两个变量即可:
```html
<HR width="100%" id="EOF">
    
{{- if not .Lastmod.IsZero -}}
    <p style="color:#777;">创建于: {{ .Date.Format "2006-01-02T15:04:05"}},   Lastmod: {{ .Lastmod.Format "2006-01-02T15:04:05"}}</p>
{{- end -}}
```