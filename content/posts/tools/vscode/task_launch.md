---
title: "Vscode task.json & launch.json"
tags: [tools]
categories: ["DevTools"]
date: 2023-12-17T15:25:12+08:00
---

## Task.json

Vscode中，可以为编译、打包等过程创建自动化任务，避免每次手动敲一些命令。
在我看来，Vscode Task就像是一个强大的、与Vscode联动的Shell脚本。

### 创建一个Task
创建一个Task很简答，`Terminal-Configure Tasks`，
然后根据引导就可以创建一个默认的task，对他进行配置的文件是workspace/.vscode/task.json。

展示一下我刚刚创建的一个编译并执行单元测试的任务，关键的参数是`label`也就是任务的名字，
`type`除了shell不知道还有啥，`command`就是该任务会执行的shell命令。
更多的参数下面会介绍。
```json
{
  // See https://go.microsoft.com/fwlink/?LinkId=733558
  // for the documentation about the tasks.json format
  "version": "2.0.0",
  "tasks": [
    {
      "label": "build-ut",
      "type": "shell",
      "command": "bash tools/vscode_build_ut.sh",
      "presentation": {
        "echo": true,
        "reveal": "always",
        "focus": true,
        "panel": "shared",
        "showReuseMessage": true,
        "clear": false
      },
      "problemMatcher": [],
    }
  ]
}
```

### 支持的参数

支持的参数很多，我主要介绍几个，Vscode的[官方文档](https://code.visualstudio.com/docs/editor/tasks)说的非常通俗易懂，修改参数时最好参考一下。

- label: 此任务的名字
- type：类型
  - shell：作为shell命令执行
  - process：创建一个新进程执行
- command：任务实际执行的命令
- group：任务的分组
- presentation：定义如何处理Task的输出
  - reveal：终端是否显示
  - echo：任务输出是否到终端中
  - focus：任务执行时是否聚焦到终端
  - showReuseMessage：是否显示最后的提示信息
  - clear：任务运行前是否清理终端输出
- options：定义当前目录和一些环境变量
- runOptions：定义任务何时运行以及如何运行
- problemMatcher: 自定义错误匹配机制，这个应该很强大，我这里单纯是为了运行时不需要再选一次所以用了一个默认值。具体怎么用可以参考: https://code.visualstudio.com/docs/editor/tasks#_defining-a-problem-matcher




## Launch.json

早就看Vscode左边的"Run and Debug"栏不爽了，呆在那也没啥用。
其实在实习的时候看过他用Vscode调试Qemu guest，
抽时间把这个给研究了一下，感觉比直接执行gdb方便一些。

Launch.json就是定义一些运行与Debug的动作，结合Vscode的界面，
比Gdb Tui还是好看了一些。这里列出我当前项目所使用的配置项。

>注意：如果同时在项目根目录有`.gitinit`文件，可能会导致错误，
>详细: [QEMU: Terminated via GDBstub error](https://stackoverflow.com/questions/5550555/qemu-terminated-via-gdbstub-error)

```json
{
  // Use IntelliSense to learn about possible attributes.
  // Hover to view descriptions of existing attributes.
  // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
  "version": "0.2.0",
  "configurations": [
    {
      "name": "qemu-kernel",
      "type": "cppdbg",
      "request": "launch",
      "miDebuggerPath": "/usr/bin/gdb-multiarch",
      "miDebuggerServerAddress": "localhost:1234",
      "program": "${workspaceFolder}/your/exec/file",
      "cwd": "${workspaceFolder}",
      "MIMode": "gdb",
      "stopAtConnect": true,
    }

  ]
}
```
