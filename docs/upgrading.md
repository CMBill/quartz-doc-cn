---
title: 升级 Quartz
---

> [!note]
> 这是将 Quartz 4 版本升级到最新更新的指南，如果您需要从 Quartz 3 迁移到 Quartz 4，请参阅[[migrating from Quartz 3|迁移指南]]。

要获取最新的 Quartz 更新，只需运行

```bash
npx quartz update
```

由于 Quartz 在底层使用 [git](https://git-scm.com/) 进行版本控制，因此更新可以有效地从官方 Quartz GitHub 存储库中 pull 更新。如果您的本地更改可能与更新冲突，您可能需要自己手动解决这些问题（或者使用 `git pull origin upstream` 手动拉取）。

> [!hint]
> Quartz 会在更新之前尝试缓存您的内容，以防止合并冲突。如果在合并过程中发生冲突，您可以停止合并，然后运行 ​​ `npx quartz restore` 从缓存中恢复内容。

如果您安装了 [GitHub desktop app](https://desktop.github.com/)它将自动打开以帮助您解决冲突。否则，您将需要在 VSCode 等文本编辑器中解决此问题。有关手动解决冲突的更多帮助，请参阅 [GitHub guide on resolving merge conflicts](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/addressing-merge-conflicts/resolving-a-merge-conflict-using-the-command-line#competing-line-change-merge-conflicts).
