---
title: 欢迎使用 Quartz 4
---
**本站点提供了 Quartz 文档的非官方中文翻译，目前正在翻译中。由于译者水平有限，翻译质量可能不尽人意，如出现任何错误与遗漏，请以[官方英文原版](https://quartz.jzhao.xyz/)为准，如有任何修改建议或意见，欢迎在 [GitHub 仓库](https://github.com/CMBill/quartz-doc-cn)提交 PR 或 Issue。**

Quartz 是一款快速、具有丰富插件的静态站点生成器，可将 Markdown 内容转换为功能齐全的网站。成千上万的学生、开发人员和教师已经在使用 Quartz 将个人笔记、网站和[数字花园](https://jzhao.xyz/posts/networked-thought)发布到网络上。

## 🪴 开始使用

Quartz **至少需要 [Node](https://nodejs.org/) v18.14** 和 `npm` v9.3.1 才能正常运行。在继续之前，请确保您的计算机上配置好需要的软件。

随后，在您所使用的终端中，逐行输入以下命令：

```shell
git clone https://github.com/jackyzha0/quartz.git
cd quartz
npm i
npx quartz create
```

这将指导您使用自己的 `content` 文件夹初始化 Quartz。完成此操作后，请参阅以下页面来完成您的网站构建：

1. 在 Quartz 中[[authoring content|编写内容]]
2. [[configuration|配置]] Quartz
3. 改变 Quartz 的[[布局]]
4. [[build|构建并预览]] Quartz
5. [[hosting|在线托管]] Quartz

> [!info] 
> 由 Quartz 3 迁移而来？请参阅[[migrating from Quartz 3|迁移指南]]，了解 Quartz 3 和 Quartz 4 之间的差异以及如何迁移。

## 🔧 特性

- [[Obsidian compatibility|Obsidian 兼容性]]，[[full-text search|全文搜索]]，[[graph view|关系图谱]]，注释嵌入，[[wikilinks|Wiki 链接]]，[[backlinks|反向链接]]，[[Latex]]，[[syntax highlighting|语法高亮]]，[[popover previews|悬浮预览窗]]，[[Docker Support|支持 Docker]]，以及[更多开箱即用的功能](./features)
- 配置文件和内容的热重载
- 简单的 JSX 布局和[[creating components|页面组件]]
- [[SPA Routing|极快的页面加载速度]]和极小的捆绑包大小
- 通过[[making plugins|插件]]可以完全定制内容的处理、过滤和页面生成

对于详尽的特性，您可以访问[特性页面](./features)。您可以访问[[philosophy|哲学]]页面来了解这些特性背后的 _WHY_，同时可以在[[philosophy|架构]]页面阅读技术概述。

### 🚧 故障排除与更新

在使用 Quartz 时存在问题？尝试使用搜索功能来寻找您遇到的问题的解决方法。如果您没有使用最新版本，尝试将 Quartz [[upgrading|更新]] 到最新版本，再看看有没有解决问题。

如果您仍然还存有疑问或发现了 bug，可以在 Github 仓库[提交 issue](https://github.com/jackyzha0/quartz/issues)，或在我们的 [Discord 社区](https://discord.gg/cRFFHYye7t)寻求帮助。
