---
title: "reveal.js Tutorial"
date: 2022-05-08T19:34:44+08:00
---

# Change code theme
Default use `monokai.css`. see [官方文档](https://revealjs.com/code/)

修改需要下载新的`css`放到`plugin/highlight/`目录下.

其他可用的`css`在[highlight.js仓库](https://github.com/highlightjs/highlight.js/tree/main/src/styles)中下载.


# Align
## Slide Align
取消center对齐方式: 
```js
Reveal.initialize({
    ...
    center: false,
    ... })
```


所有slide左对齐: https://github.com/hakimel/reveal.js/issues/1897

用markdown写的方式下使某一幻灯片左对齐: https://github.com/hakimel/reveal.js/issues/890#issuecomment-129735291


