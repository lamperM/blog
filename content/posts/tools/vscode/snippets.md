---
title: "Vscode Snippets"
tags: [tools]
categories: ["DevTools"]
date: 2024-01-06T12:25:12+08:00
---

Snippets 的含义是代码片段，帮助我们快速补全一段代码。
今天发现这个功能还挺强大的，尤其是写 Markdown 时，关键字写起来麻烦，
加上 Vscode 补全还乱七八糟（代码块的自动匹配不能关闭）。
先解我燃眉之急，先介绍 Markdown 的 Snippest。

全局搜索中找到 snippets 的配置：
{{< figure src="/vscode_snippets.jpg" width="60%" >}}

全局配置中选择 Insert 还能看到当前所有支持的 Snippets。

以下是我自己添加了一段与 Hugo notice 主题相关的，变量的运用比较关键。
$1 表示插入后光标所在的位置，$2，＄ 3...依次是按 Tab 键之后的位置，
$0 则表示最终将停在哪，不会继续循环。

```json
{
  // Place your snippets for markdown here. Each snippet is defined under a snippet name and has a prefix, body and
  // description. The prefix is what is used to trigger the snippet and the body will be expanded and inserted. Possible variables are:
  // $1, $2 for tab stops, $0 for the final cursor position, and ${1:label}, ${2:another} for placeholders. Placeholders with the
  // same ids are connected.
  // Example:
  // "Print to console": {
  // 	"prefix": "log",
  // 	"body": [
  // 		"console.log('$1');",
  // 		"$2"
  // 	],
  // 	"description": "Log output to console"
  // }
  "Hugo notice": {
    "prefix": "notice",
    "description": "Add hugo notice snip",
    "body": ["{{</* notice $1  */>}}", "$0", "{{</* /notice */>}}"]
  }
}
```

注意，需要在 editor 配置中插入对 Markdown 语言启用建议的配置。
部分情况下如果上下文中有和 snippet 重复的关键字，默认 snippet 的优先级较低，
所以添加了升高优先级的配置。

```json
  "[markdown]": {
    "editor.quickSuggestions": {   // 开启markdown中的智能提示建议
      "other": "on",
      "comments": "off",
      "strings": "off"
    },
    "editor.snippetSuggestions": "top", // 提高snippet在所有提示中的优先级
  },
```
