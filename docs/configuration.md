---
title: 配置
---

即使您不了解任何代码，Quartz 也具有极高的可配置性。您需要的大部分配置只需编辑 `quartz.config.ts` 文件，或者在 `quartz.layout.ts` 中更改[[layout|布局]]。

> [!tip]
> 如果您使用支持 TypeScript 语言的文本编辑器（如 VSCode）编辑 Quartz 配置，当您在配置中出现错误时，它会向您发出警告，帮助您避免配置错误！

Quartz 的配置可以分为两个主要部分：

```ts title="quartz.config.ts"
const config: QuartzConfig = {
  configuration: { ... },
  plugins: { ... },
}
```

## 通用配置

这部分配置涉及任何可能影响整个站点的内容，以下是您可以配置的所有内容：

- `pageTitle`: 网站的标题。为您的站点生成 [[RSS Feed]]时也会使用此信息。
- `enableSPA`: 是否在您的网站上启用 [[SPA Routing]]。
- `enablePopovers`: 是否在您的网站上启用 [[popover previews|悬浮预览窗]]。
- `analytics`: 设置网站使用的分析器，可以为以下值：
  - `null`: 不使用分析器;
  - `{ provider: 'plausible' }`: 使用 [Plausible](https://plausible.io/)，这是 Google Analytics 的隐私友好替代方案；或者
  - `{ provider: 'google', tagId: <your-google-tag> }`: 使用 Google Analytics
- `baseUrl`: 用于需要绝对 URL 才能知道站点的规范“主页”所在位置的情况，例如站点地图和 RSS 提要。这通常是您网站的已部署 URL，比如此网站的 `quartz.jzhao.xyz`，请勿包含协议（即 `https://` ）或任何前导以及后面的斜杠。
  - 如果您在没有自定义域名的 GitHub 页面上[[hosting|托管]]，这还应该包括子路径。例如，如果我的存储库是 `jackyzha0/quartz` ，GitHub 页面将部署到 `https://jackyzha0.github.io/quartz` ，而 baseUrl 将是 `jackyzha0.github.io/quartz`。
  - 请注意，Quartz 4 将尽可能避免使用此功能，而是尽可能使用相对 URL，以确保您的网站无论最终实际部署在何处都能正常运行。
- `ignorePatterns`: Quartz 在 content 文件夹中查找文件时应忽略且不搜索的文件，这是一个通配符列表，您可以在 [[private pages]] 获得更多信息。
- `defaultDateType`: 是否使用创建、修改或发布作为在页面和页面列表上显示的默认日期。
- `theme`: 配置站点的外观。
  - `typography`: 配置字体，[Google Fonts](https://fonts.google.com/) 上的字体均可用。
    - `header`: 标题字体
    - `code`: 内联和块引用的字体
    - `body`: 所有内容的字体
  - `colors`: 控制网站的主题。
    - `light`: 页面背景
    - `lightgray`: 边框
    - `gray`: 图形链接，较粗的边框
    - `darkgray`: 正文
    - `dark`: 标题文本和图标
    - `secondary`: 链接颜色，[[graph view|关系图谱]]中当前节点的颜色
    - `tertiary`: [[graph view|关系图谱]]中鼠标指向时以及已访问过的节点颜色
    - `highlight`: 内部链接背景、突出显示的文本、高亮的[[syntax highlighting|代码]]

## 插件

您可以将 Quartz 插件视为对内容的一系列转换。

![[quartz transform pipeline.png]]

```ts
plugins: {
  transformers: [...],
  filters: [...],
  emitters: [...],
}
```

- [[making plugins#Transformers|Transformers]] **map** over content (例如解析 frontmatter、生成描述)
- [[making plugins#Filters|Filters]] **过滤**内容 (例如过滤掉草稿)
- [[making plugins#Emitters|Emitters]] **归约** 内容 (例如创建 RSS 提要或列出具有特定标签的所有文件的页面)

通过在 `tranformers`、`filters` 和 `emitters` 字段中添加、删除和重新排序插件，您可以自定义 Quartz 的行为。

> [!note]
> 每个节点都被每一个 Transformer _按顺序_ 修改，有些 Transformer 对位置敏感，因此您可能需要特别注意它的位置，是否需要位于某个特定插件之前或之后。

此外，插件也可能有自己的配置选项可以设定，例如 [[Latex]] 插件允许您指定 `renderEngine` 的字段，以在 Katex 和 MathJax 之间选择。

```ts
transformers: [
  Plugin.FrontMatter(), // 使用默认值
  Plugin.Latex({ renderEngine: "katex" }), // 指定某个选项
]
```
如果您想制作自己的插件，请阅读[[making plugins|插件制作指南]]插件制作指南以获取更多信息
