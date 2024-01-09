---
title: 架构
---

Quartz 是一个静态站点生成器，它是如何工作的？

这个问题的最佳答案是跟踪用户（您！）在命令行中运行 `npx quartz build` 时发生的情况：

## 在服务器上

1. 运行 `npx quartz build` 后，npm 将查看 `package.json` 以查找 `quartz` 的 `bin` 条目，该条目指向 `./quartz/bootstrap-cli.mjs`。
2. 此文件顶部存在 [shebang](<https://en.wikipedia.org/wiki/Shebang_(Unix)>) 行告诉 npm 使用 Node.js 来执行。
3. `bootstrap-cli.mjs` 负责一些事情：
   1. 使用 [yargs](http://yargs.js.org/) 解析命令行参数。
   2. 使用 [esbuild](https://esbuild.github.io/) 将 Quartz 的其余部分（使用Typescript）转换和打包为常规的 JavaScript。这里的 `esbuild` 配置有点特殊，因为它还使用 [esbuild-sass-plugin v2](https://www.npmjs.com/package/esbuild-sass-plugin) 处理 `.scss` 文件。此外，我们使用一个自定义的 `esbuild` 插件，将组件声明的“内联”客户端脚本（任何 `.inline.ts` 文件）打包在一起。该插件会运行另一个 `esbuild` 实例，用于为浏览器而不是 `node` 进行打包。
   3. 如果指定了 `--serve` 参数，则会运行本地预览服务器。这会启动两个服务器：
      1. 端口 `3001` 上的 WebSocket 服务器用于处理热重载信号。这会跟踪所有入站连接，并在检测到服务器端更改（内容或配置）时发送“重建”消息。
      2. 用户定义端口（默认为 `8080`）上的 HTTP 文件服务器用于为实际网站文件提供服务。
   4. 如果指定了 `--serve` 参数，还会启动一个文件观察器来检测源代码更改（例如 `.ts` 、 `.tsx` 、 `.scss` 以及包文件）。如果发生改变，我们将会使用 `esbuild` 提供的[重建 API](https://esbuild.github.io/api/#rebuild) 来重新构建模块（上方的第二步），这可以大幅降低构建消耗的时间。
   5. 在转译主 Quartz 构建模块 ( `quartz/build.ts` ) 后，我们将其写入缓存文件 `.quartz-cache/transpiled-build.mjs` 中，然后使用 `await import(cacheFile)` 动态导入它。然而，我们需要非常聪明地处理 Node 的[导入缓存](https://github.com/nodejs/modules/issues/307)，因此我们添加一个随机查询字符串来使 Node 使其认为它是一个新模块。然而，这确实会导致内存泄漏，因此我们只是希望用户不要在单个会话中热重载其配置太多次:)（每次重新加载时会泄漏约 350kB 内存）。导入模块后，我们调用它并传入之前解析的命令行参数以及回调函数来通知客户端刷新。
4. 在 `build.ts` 中，我们首先手动安装源映射支持，以解决我们之前介绍的查询字符串缓存破坏问题。然后，我们开始处理内容：
   1. 清理输出目录。
   2. 递归地遍历 `content` 文件夹中的所有文件，尊重 `.gitignore` 。
   3. 解析 Markdown 文件。
      1. Quartz会检测可用的线程数量，并在需要解析的内容超过128个片段时选择生成工作线程（粗略的启发式算法）。如果需要生成工作线程，Quartz将再次调用esbuild来转译工作线程脚本 `quartz/worker.ts` 。接下来，会创建一个工作窃取的[工作线程池](https://www.npmjs.com/package/workerpool)，并将每个批次包含128个文件的任务分配给工作线程。
      2. 每个工作线程（如果没有并发，则只是主线程）将根据[[configuration|配置]]中定义的插件创建一个[统一](https://github.com/unifiedjs/unified)的解析器。
      3. 解析的过程分为三步：
         1. 将文件读入 [vfile](https://github.com/vfile/vfile)。
         2. 根据插件对内容应用文本转换。
         3. 将文件路径进行转化为 URL 友好的格式，并将其存储在文件的数据中。有关 Quartz 中路径逻辑如何工作的更多详细信息，请参阅[[paths|路径]]页面（提示：它很复杂）。
         4. 使用 [remark-parse](https://www.npmjs.com/package/remark-parse) 进行 Markdown 解析（文本到[mdast](https://github.com/syntax-tree/mdast)）。
         5. 根据插件对内容应用 Markdown 到 Markdown 的转换。
         6. 使用 [remark-rehype](https://github.com/remarkjs/remark-rehype) 将 Markdown 转换为 HTML（[mdast](https://github.com/syntax-tree/mdast) to [hast](https://github.com/syntax-tree/hast)）。
         7. 根据插件对内容应用 HTML 到 HTML 的转换。
   4. 使用插件过滤掉不需要的内容。
   5. 使用插件输出文件。
      1. 收集每个发射器插件声明的所有静态资源（例如外部 CSS、JS 模块等）。
      2. 输出 HTML 文件的发射器需要额外的工作，因为因为它们需要将解析步骤中生成的 [hast](https://github.com/syntax-tree/hast) 转换为JSX，这一步使用了 [hast-util-to-jsx-runtime](https://github.com/syntax-tree/hast-util-to-jsx-runtime) 和 [Preact](https://preactjs.com/) 运行时。 最后，使用 [preact-render-to-string](https://github.com/preactjs/preact-render-to-string) 将 JSX 静态地渲染为 HTML（即不关心useState、useEffect或任何其他React/Preact的交互部分）。 在这里，我们还会进行一些有趣的操作，比如依据 `quartz.layout.ts` 中的[[layout|布局]]来组装页面、组装实际发送到客户端的所有内联脚本以及所有经过转译的样式。该逻辑的大部分可以在 `quartz/components/renderPage.tsx` 中找到，其他值得注意的有趣事项还包括：
         1. 使用 [Lightning CSS](https://github.com/parcel-bundler/lightningcss) 对 CSS 进行缩小和转换，以添加供应商前缀并进行语法降级。
         2. 将 Scripts 分为 `beforeDOMLoaded` 和 `afterDOMLoaded` 两部分，并分别插入 `<head>` 和 `<body>` 中。
      3. 最后，每个发射器插件负责输出并将其自己输出的文件写入磁盘。
   6. If the `--serve` flag was detected, we also set up another file watcher to detect content changes (only `.md` files). We keep a content map that tracks the parsed AST and plugin data for each slug and update this on file changes. Newly added or modified paths are rebuilt and added to the content map. Then, all the filters and emitters are run over the resulting content map. This file watcher is debounced with a threshold of 250ms. On success, we send a client refresh signal using the passed in callback function.如果检测到 `--serve` 参数，我们还会设置另一个文件监视器来检测内容更改（仅 `.md` 文件）。我们会保留一个内容映射用于跟踪每个 slug 的已解析 AST 和插件数据，并在文件更改时更新此映射。新添加或修改的路径将被重建并添加到内容映射中，随后所有过滤器和发射器都在新生成的内容映射上运行。该文件监视器以 250ms 的阈值进行去抖动。成功后，我们使用传入的回调函数发送客户端刷新信号。

## 在客户端

1. 在浏览器打开 Quartz 页面并加载 HTML。在 `<head>` 中还链接到页面样式（输出到 `public/index.css` ）和页面关键 JS（输出到 `public/prescript.js` ）
2. 随后，一旦页面主体加载完成，浏览器会加载非关键的JS（输出到`public/postscript.js`）。
3. 页面加载完成后，页面将分派一个自定义合成浏览器事件 `"nav"`。这将使由组件声明的客户端脚本能够在需要访问页面 DOM 的任何内容时进行“设置”。
   1. 如果在[[configuration|配置]]中启用了 [[SPA Routing|enableSPA option]] 选项，则还会在任何客户端导航上触发此 `"nav"` 事件，以允许组件取消注册和重新注册任何事件处理程序和状态。
   2. 如果不是，我们会将 `"nav"` 事件连接起来，使其在页面加载后仅触发一次，以实现 SPA 和非 SPA 上下文中状态设置方式的一致性。

插件系统的架构和设计在这里故意留下相当模糊的内容，因为在[[making plugins|制作插件]]的指南中对此进行了更深入的描述。
