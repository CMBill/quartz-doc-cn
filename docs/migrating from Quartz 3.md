---
title: 从 Quartz 3 迁移
---

由于您本地已经有 Quartz，因此无需再次 Folk 或 Clone。只需 checkout v4 分支，安装依赖项，然后导入旧的保管库即可。

```bash
git fetch
git checkout v4
git pull upstream v4
npm i
npx quartz create
```

如果出现类似 `fatal: 'upstream' does not appear to be a git repository` 的报错，请确保添加 `upstream` 作为远程源：

```shell
git remote add upstream https://github.com/jackyzha0/quartz.git
```

运行 `npx quartz create` 时，系统将提示您如何初始化内容文件夹。在这里，您可以选择导入或链接以前的内容文件夹，Quartz 应该按照您的预期工作。

> [!note]
> 如果您要使用的现有内容文件夹位于不同分支上的*同一路径*，请在*不同路径*的某个位置再次克隆存储库以便使用它。

## 主要变化

1. **移除 Hugo 与 `hugo-obsidian`**：Hugo 在 Quartz 的早期版本中运行良好，但它也使不在 Golang 和 Hugo 社区的人难以完全理解 Quartz 的作用并按照自己的想法来定制。Quartz 4 现在使用基于节点的静态站点生成过程，这应该会产生更有用的错误消息，并带来更流畅的整体用户体验。
2. **完全的热重载**：`hugo-obsidian` 与 Hugo 粗糙的集成方式使得预览模式下不会触发 `hugo-obsidian` 来更新内容索引，这会导致预览模式下输出异常的情况很多奇怪的情况，Quartz 4 现在使用一个紧密的解析、过滤和输出流程，在每次更改时均会自动运行，因此热重载输出的页面始终准确。
3. **用 JSX 替换 Go templates 语法**：Quartz 3 使用 [Go templates](https://pkg.go.dev/text/template) 来创建页面布局。然而，该语法并不适合进行任何类型的复杂渲染（例如[文本处理](https://github.com/jackyzha0/quartz/blob/hugo/layouts/partials/textprocessing.html)），并且很难对 Quartz 3 进行任何有意义的布局更改。Quartz 4 使用称为 JSX 的 JavaScript 语法扩展，它允许您编写布局代码看起来像 JavaScript 中的 HTML，更容易理解和维护。
4. **新的可扩展[[configuration|配置]] 和 [[configuration#插件]] 系统**：如果没有关于 Hugo 的知识，Quartz 3 很难配置，甚至很难制作扩展。 Quartz 4 的配置和插件系统旨在供用户扩展，同时使更新到 Quartz 新版本变得容易。

## 要更新的内容

- 您将需要更新部署脚本。有关更多详细信息，请参阅[[hosting|托管]]指南。
- 确保 GitHub 上的默认分支从 `hugo` 更新为 `v4`。
- [[folder and tag listings|文件夹和标签列表]]也发生了变化。
  - 文件夹描述应位于 `content/<folder-name>/index.md` 下，其中 `<folder-name>` 是文件夹的名称。
  - 标签描述应位于 `content/tags/<tag-name>.md` 下，其中 `<tag-name>` 是标签的名称。
- Quartz 3 和 Quartz 4 之间的某些 HTML 布局可能不同。如果您依赖于特定的 HTML 层次结构或类名称，您可能需要更新自定义 CSS 以应对这些更改。
- 如果您自定义了 Quartz 3 的布局，则可能需要将这些更改从 Go 模板转换回 JSX，因为 Quartz 4 不再使用 Hugo。对于组件，请查看[[creating components|创建组件指南]]以获取更多详细信息。
