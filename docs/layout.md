---
title: 布局
---

某些发射器可以输出 [HTML](https://developer.mozilla.org/en-US/docs/Web/HTML) 文件，为了方便自定义，这些发射器允许您完全重新排列页面布局。默认页面布局可以在 `quartz.layout.ts` 中找到。

每个页面由多个不同的包含 `QuartzComponents` 的部分组成，以下代码片段列出了您可以添加组件的部分：

```typescript title="quartz/cfg.ts"
export interface FullPageLayout {
  head: QuartzComponent // single component
  header: QuartzComponent[] // laid out horizontally
  beforeBody: QuartzComponent[] // laid out vertically
  pageBody: QuartzComponent // single component
  left: QuartzComponent[] // vertical on desktop, horizontal on mobile
  right: QuartzComponent[] // vertical on desktop, horizontal on mobile
  footer: QuartzComponent // single component
}
```

对应于页面的以下部分：

![[quartz layout.png|800]]

> [!note]
> 还有两个附加布局字段未在上图中显示
>
> 1. `head` 是在 HTML 文件中呈现 `<head>` [tag](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/head) 的组件。它不会以视觉形式出现在页面上，仅负责有关文档的元数据，例如选项卡标题、脚本和样式。
> 2. `header` 是一组水平布局的组件，出现在 `beforeBody` 部分之前。这使您能够复制旧的 Quartz 3 标题栏（包含标题、搜索栏和黑暗模式切换）。默认情况下，Quartz 4 不会在 `header` 中放置任何组件。

Quartz **组件**与插件一样，可以采用附加属性作为配置选项。如果您熟悉 React ，您可以将它们视为高阶组件。

参阅 [a list of all the components](component.md) 以了解所有可用组件的所有组件列表及其配置选项，如果您有兴趣进一步定制 Quartz 的行为，您还可以查阅  [[creating components|创建组件]]指南。

### 样式

最常见的样式更，例如配色方案和字体，只需通过[[configuration#通用配置]]选项。如果您想进行复杂的样式更改，可以通过编写自己的样式来实现。 Quartz 4 与 Quartz 3 一样，使用 [Sass](https://sass-lang.com/guide/) 进行样式设置。

您可以在 `quartz/styles/base.scss` 中查看基本样式表，并在 `quartz/styles/custom.scss` 中编写自己的样式表。

> [!note]
> 有些组件也可能提供自己的样式！例如 `quartz/components/Darkmode.tsx` 从 `quartz/components/styles/darkmode.scss` 导入样式。如果您想为特定组件自定义样式，请仔细检查组件定义以了解其样式是如何定义的。
