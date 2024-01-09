---
title: 设置您的 GitHub 存储库
---

首先，确保您已在本地[[index#🪴 Get Started|克隆并设置了 Quartz]].

然后，在 GitHub.com 上创建一个新存储库。**不要**使用 `README` 、license或 `gitignore` 文件初始化新存储库。

![[github-init-repo-options.png]]

在 GitHub 存储库的 Quick Setup 页面顶部，单击剪贴板以复制远程存储库 URL。

![[github-quick-setup.png]]

在您选择的 Terminal 中，导航到 Quartz 文件夹的根目录。随后运行以下命令，将 `REMOTE-URL` 替换为您刚刚从上一步复制的 URL。

```bash
# add your repository
git remote add origin REMOTE-URL

# track the main quartz repository for updates
git remote add upstream https://github.com/jackyzha0/quartz.git
```

要验证您是否正确设置远程 URL，请运行以下命令。

```bash
git remote -v
```

然后，您可以同步您的内容到您的存储库。

```bash
npx quartz sync --no-pull
```

> [!hint]
> 如果执行 `npx quartz sync` 失败并出现 `fatal: --[no-]autostash option is only valid with --rebase` 的报错，您的 `git` 版本可能已过时，更新 `git` 应该可以解决此问题。
