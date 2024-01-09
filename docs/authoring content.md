---
title: 创作内容
---

Quartz 中的所有内容都应当被放置在 `/content` 文件夹内，主页的内容应该在 `content/index.md`。如果你已经[[index#🪴 Get Started|安装好]]了 Quartz，`/content` 文件夹应该已经被初始化，处于此文件夹下的所有 Markdown 内容都将由 Quartz 处理。

推荐使用 [Obsidian](https://obsidian.md/) 作为编辑和维护 Quartz 的工具，它拥有一个漂亮的编辑器和图形界面，可以直观地预览、编辑，也能方便地将附件链接到您的文件。

如果一切都被设置好了，让我们在本地[[build|构建]]并预览您的 Quartz！

## 语法

由于 Quartz 使用 Markdown 文件作为编写内容的主要方式，因此它完全支持 Markdown 语法。默认情况下，Quartz 还附带一些语法扩展，例如 [Github Flavored Markdown](https://docs.github.com/en/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax)（脚注、删除线、表格、任务列表）和 [Obsidian Flavored Markdown](https://help.obsidian.md/Editing+and+formatting/Obsidian+Flavored+Markdown)（[[callouts|标注]]、[[wikilinks|Wiki 链接]]）。

此外，Quartz 还允许您在文件中指定被称为 **frontmatter** 的附加元数据。

```md title="content/note.md"
---
title: Example Title
draft: false
tags:
  - example-tag
---

The rest of your content lives here. You can use **Markdown** here :)
```

Quartz 原生支持的一些常见的 frontmatter 字段：

- `title`：页面标题。如果未提供，Quartz 将使用文件名作为标题。
- `aliases`：此文章的其他名称。这是一个字符串列表。
- `draft`：是否发布页面。这是在 Quartz 中将页面设置为[[private pages|私有]]的一种方法。
- `date`：表示注释发布日期的字符串。通常使用 `YYYY-MM-DD` 格式。

## 同步您的内容

当您对您 Quartz 的内容感到满意时，您可以通过执行 `npx quartz sync` 将更改同步保存到 GitHub。

> [!hint] 其他参数选项
> 要获得完整的帮助选项，您可以运行 `npx quartz sync --help`。
>
> 其中大多数都有合理的默认值，但如果您需要自定义设置，则可以覆盖它们：
>
> - `-d` 或 `--directory`: 指定内容文件夹，默认为 `content`
> - `-v` 或 `--verbose`: 输出额外的日志
> - `--commit` 或 `--no-commit`: 是否为您的更改进行 `git` 提交
> - `--push` 或 `--no-push`: 是否将您的更改推送到您的 Github 仓库
> - `--pull` 或 `--no-pull`: 在推送之前是否尝试从 GitHub 分支（即从其他设备）提取任何更新
