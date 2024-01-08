---
title: "构建你的 Quartz"
---

在您[[index#🪴 Get Started|初始化]] Quartz 之后，我们可以来看看它在本地构建出来的样子：

```bash
npx quartz build --serve
```

这将在您的计算机上启动一个本地 Web 服务器来运行 Quartz，打开浏览器并访问 `http://localhost:8080/` 进行查看。


> [!hint] 其他参数选项
> 要获得完整的帮助选项，您可以运行 `npx quartz build --help`.
>
> 其中大多数都有合理的默认值，但如果您需要自定义设置，则可以覆盖它们：
>
> - `-d` 或 `--directory`: 内容文件夹，一般即  `content`
> - `-v` 或 `--verbose`: 输出额外的日志信息
> - `-o` 或 `--output`: 输出文件夹，一般即 `public`
> - `--serve`: 在本地运行一个支持热重载的服务器来预览您的 Quartz
> - `--port`: 本地服务器运行的端口
> - `--concurrency`: 解析文档内容所使用的线程数
