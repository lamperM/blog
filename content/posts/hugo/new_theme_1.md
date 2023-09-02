---
title: "Hugo 主题创建(1): 内置样式"
date: 2023-08-11T07:39:42+08:00
tags: [hugo]
---

> 本次对应的commit，应该属于站点的仓库，因为仅修改 `config.toml`

# 代码高亮
hugo 内置一套highlight引擎, 参见[官网的描述](https://gohugo.io/content-management/syntax-highlighting/)
, 所以我们只需要对站点的配置文件(注意不是模板的配置文件)进行修改, 就能最简单的实现代码高亮.

如果你需要对其进行自定义, 且将其固化到你的主题中, 那么就可能需要使用highlight.js来完成,
遵循"提前优化是万恶之源"的理论, 暂时使用hugo提供的高亮支持就能符合我们的目标.

这是我的配置文件`config.toml`中关于代码高亮的启用:
```toml
[markup]
  [markup.highlight]
    anchorLineNos = false     # 行号格式化为<span>
    codeFences = true         # 代码围栏, 不启用高亮无效
    guessSyntax = true        # 自动推断高亮语言
    hl_Lines = ''             # 突出显示某些特定的行
    hl_inline = false         # 高亮 inline code, ver>=0.101.0
    lineAnchors = ''          #　与 anchorLineNos 配合
    lineNoStart = 1           # 行号开始
    lineNos = true            # 是否显示行号
    lineNumbersInTable = true # 生成html中分开行号和代码
    noClasses = true          
    noHl = false
    style = 'vs'
    tabWidth = 4
```


## 参考
- hugo代码高亮引擎描述引导页: https://gohugo.io/content-management/syntax-highlighting/
- 参数的详细描述: https://gohugo.io/functions/highlight/