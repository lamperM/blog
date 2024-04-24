---
title: "Linux Kconfig 概述"
categories: ["Operating System"]
date: 2023-03-28T16:15:03+08:00
---

## Kconfig 介绍

Kconfig 是一个通用的配置系统，最初由 Linux 开发，提供了一种简单的可扩展的配置语言，
实现工程的模块化和可定制需求。

在简单的工程中，或许我们使用简单的 C 语言宏就能解决问题，不必用到 Kconfig。
但对于大型项目，并不只是一个选项的开启和关闭，并不只是影响到源代码，还有基于 Makefile 的编译工作。并且，当可配置的选项增加时，开发者手动管理这些配置选项就显得不那么智能了。

Kconfig 的出现使得上述工作变得轻松：

- 在 build 前，我们可以使用图形化界面来选择此次构建启用了哪些编译选项, (menuconfig)
- 对于不同的厂商驱动，可以有不同的配置文件，厂商可以为我们提供一个默认的 config 文件，我们可以方便的应用这些默认选项进行编译, 或者基于这些默认选项进行改动，保存为自己的配置项文件(defconfig)
- 选项之间可以建立依赖关系，例如，只有 A 启用时，BCD 选项才有意义。对此 Kconfig 提供了一套简单上手的编辑语言

Kconfig 的使用必须配合 Makefile 进行，应该说，Kconfig 与 Makefile 结合是简化了 Kconfig 的使用步骤。

## Kconfig 的使用方法

我们常用的功能大概包括：

1. 使用一些默认配置文件来构建项目
2. 自定义配置构建项目
3. 将某套配置选项保存下来，方便下次使用

在分别介绍这些功能的使用方法之前，先得说明这些 Kconfig 中各种配置文件的用途了，还是比较容易混淆的。

- `Kconfig`: Kconfig 不仅在根目录，还可能存在于各级子目录下。里面的内容是整个 Kconfig 系统的所有配置项，以及每个配置项的默认值、描述等。如果想要添加、删除配置项，需要改动这个文件
- `xxxdefconfig`: xxx 可以替换为任意字符，这些都是默认的配置项，通常由开发人员提供给使用者
- `.config`: 这个文件存在于根目录下，我们可以叫他**当前配置**。可以把他当为服务与一次构建的“临时文件”，每次构建都是基于当前配置进行的
- `autoconfig.h`: 由 Kconfig 系统根据当前构建使用的配置自动生成的头文件，C 源代码可以通过它来知道当前的配置情况

其实它们之间的关系不是那么复杂，到底是谁根据谁生成的谁，下面将要说明。

### 使用默认配置(xxxdefconfig)

一般我们拿到一个 SDK，厂商会提供一些默认的配置项供我们使用。我们的使用方法一般是执行`make xxxdefconfig`，make 会调用 Kconfig 程序来读取这个 defcofig 文件，然后生成**当前配置**（即`.config`）

`xxxdefconfig`中的配置项是需要和`Kconfig`文件配合的，其中的属性和值都是 Kconfig 中支持的配置项。并且，xxxdefconfig 不会记录 Kconfig 每个项目的值，只是记录那些**非默认值**的变化，这样大大减少了文件大小。

> 以上行为可以通过执行`make xxxdefconfig`后，查看`.config`和`xxxdefconfig`的文件差异来验证

### 改变当前配置

总是使用默认配置可不行，那么怎么修改当前配置呢? 一种显而易见的方法是修改`defconfig`或者`.config`，取决于你想你这次更改永远有效，还是只是这次编译有效，**其实，最好不要改动厂商给的默认配置，你可以复制一份，然后进行修改**。

这里要介绍与 Kconfig 配合使用的工具——menuconfig，它为我们提供交互式的配置菜单，比面对 Kconfig 的语言来修改配置更加方便。可以把 menuconfig 理解为 Kconfig 的一个前端。

menuconfig 的使用方式是执行`make menuconfig`, 它会加载**当前配置**的内容（或者是 Kconfig 中定义的默认值），生成一个图形化的菜单，修改后还是保存为`.config`。每次构建都是使用最新的当前配置，就达到了**临时**修改配置的效果了。这不比直接修改`.config`方便多了？

### 保存当前配置

有时，我们发现当前这个配置很好，想要将其保存为新的`defconfig`文件，这样不用每次都修改当前配置了。Kconfig 当然也支持这个功能，叫做`savedefconfig`。

使用的方法是:`make savedefconfig`， 会将当前配置(.config)中的内容提取为一个 defconfig 的文件，保存的位置取决于 Kconfig 的配置（一般是`conf.c`的源码中），我们对他改个名字就成为自己的配置了。下次执行`make xxxdefconfig` 即可套用这一套配置

### 其他的功能...

Kconfig 提供的功能很多，包括：一键启用所有的配置选项、最小的构建选项等等，但是不常用就不介绍了。

## Kconfig 系统的工作原理

上面说道，Kconfig 的使用是需要配合修改 Makefile 的，在 Makefile 会增加几个目标: `%defconfig`, `savedefconfig`, `menuconfig`。 它们向上给用户提供选项，向下调用 Kconfig 系统的功能接口。

Kconfig 系统主要任务是由`conf`程序完成，它是一个 host 程序，负责解析 Kconfig，defconfig 等文件的内容，完成功能的实现。还有一个重要的程序是`mconf`，它负责实现 menuconfig 的功能。