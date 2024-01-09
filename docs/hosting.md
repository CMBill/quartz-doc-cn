---
title: 托管
---

Quartz 能够高效地将您的 Markdown 文件和其他资源转换为一个 HTML、JS 和 CSS 文件的捆绑包（也就是一个网站！）。

但是，如果您想向全世界发布您的网站，则需要一种在线托管网站的方法。本指南将详细介绍如何使用常见的托管提供商进行部署，但任何允许您部署静态 HTML 的服务也应该可以工作。

> [!warning] 警告
> 本指南的其余部分假设您已经为 Quartz 创建了自己的 GitHub 存储库。如果还没有，您可以参阅[[setting up your GitHub repository|此页面]]此页面。

> [!hint] 提示
> Quartz 的某些功能（例如 [[RSS Feed]] 和站点地图生成）需要在[[configuration|配置]]中正确配置 `baseUrl` 才能正常工作。确保在部署之前进行设置！

## Cloudflare Pages

1. 登录 [Cloudflare 仪表板](https://dash.cloudflare.com/)并选择您的帐户。
2. 在帐户主页中，选择 **Workers & Pages** > **创建应用程序** > **Pages** > **链接到  Git**.
3. 选择您创建的新 GitHub 存储库，并在**设置构建和部署**部分中提供以下信息：

| 配置选项               | 值                 |
| ---------------------- | ------------------ |
| 生产分支               | `v4`               |
| 框架预设               | `无`               |
| 构建命令               | `npx quartz build` |
| 构建输出目录           | `public`           |

单击“保存并部署”，Cloudflare 应该会在大约一分钟内完成您站点的部署。而在之后您每次将 Quartz 更改同步到 GitHub 时，您的站点都会同步更新。

要添加自定义域，请参阅[Cloudflare 的文档](https://developers.cloudflare.com/pages/platform/custom-domains/)。

> [!warning] 警告
> Cloudflare Pages 仅允许浅层 `git` 克隆，因此如果您依赖 `git` 获取时间戳，建议您向 `frontmatter` 添加日期（请参阅[[authoring content#Syntax|语法]]）或使用其他托管服务。

## GitHub Pages

在您的本地 Quartz 中，创建一个新文件 `quartz/.github/workflows/deploy.yml`。

```yaml title="quartz/.github/workflows/deploy.yml"
name: Deploy Quartz site to GitHub Pages

on:
  push:
    branches:
      - v4

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-22.04
    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0 # Fetch all history for git info
      - uses: actions/setup-node@v3
        with:
          node-version: 18.14
      - name: Install Dependencies
        run: npm ci
      - name: Build Quartz
        run: npx quartz build
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v2
        with:
          path: public

  deploy:
    needs: build
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v2
```

然后：

1. 前往您 GitHub 仓库的 Settings 选项卡，在侧栏选择 Pages，在 Source 中选择 GitHub Actions。
2. 在本地仓库执行命令 `npx quartz sync` 来提交更改，这应当会将您的网站部署到 `<github-username>.github.io/<repository-name>`.

> [!hint] 提示
> 如果您收到有关 environment protection 相关规则而不能部署到 `github-pages`，请确保删除任何现有的 GitHub Pages 环境。
>
> 您可以通过在 GitHub 仓库的 Settings 选项卡中选择 Environments 选项，并单击垃圾桶的图标来删除，在下次同步 Quartz 时，GitHub Action 将正确地为您重新创建环境。

> [!info] 
> Quartz 将生成格式为 `file.html` 而不是 `file/index.html` 的文件，这意味着非文件夹路径的尾部斜杠将被删除。由于 GitHub Pages 不执行此重定向，因此这可能会导致使用尾部斜杠的现有网站链接中断。如果不破坏现有链接对您很重要（例如您要从 Quartz 3 迁移），请考虑使用 [[#Cloudflare Pages]]。

### 自定义域名

以下是将自定义域名添加到 GitHub Pages 的方法：

1. 前往您的 GitHub 仓库的 Settings 选项卡。
2. 在侧栏选择 Pages。
3. 在 Custom Domain 下，输入您的自定义域并单击 Save。
4. 下一步取决于您使用的是顶级域（`example.com`）还是子域（`subdomain.example.com`）。
   - 如果您使用的是顶级域，请在您的 DNS 提供商的控制台创建一个 `A` 记录，将您的顶级域指向以下 IP 地址：
     - `185.199.108.153`
     - `185.199.109.153`
     - `185.199.110.153`
     - `185.199.111.153`
   - 如果您使用子域，请导航到您的 DNS 提供商并创建一个 `CNAME` 记录，将您的子域指向您 Github Pages 网站的默认域。例如，如果您想要将子域 `quartz.example.com` 用于您的站点，请创建一个将 `quartz.example.com` 指向 `<github-username>.github.io` 的 `CNAME` 记录。

![[dns records.png]]_为 `jzhao.xyz` （顶级域）和 `quartz.jzhao.xyz` （子域）配置 DNS 记录的 Google Domains 的屏幕截图。_

有关为 GitHub Pages 设置您自己的自定义域的更多详细信息，请参阅[GitHub 文档](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site#configuring-a-subdomain) 。

> [!question] 为什么我的更改没有显示出来？
> 您的更改未显示的原因可能有很多，但最可能的原因是您忘记将更改推送到 GitHub。
>
> 确保将更改保存到 Git 并通过执行 `npx quartz sync` 将其同步到 GitHub。这也将确保提取您可能从其他设备进行的任何更新，以便同时同步到本地。

## Vercel

### Fix URLs

在部署到 Vercel 之前，项目目录的根目录下需要放置一个 `vercel.json` 文件，需要包含以下配置，以便 URL 不需要 `.html` 扩展名：

```json title="vercel.json"
{
  "cleanUrls": true
}
```

### 部署到 Vercel

1. 登录 [Vercel Dashboard](https://vercel.com/dashboard) 并单击 Add New... > Project
2. 导入包含 Quartz 项目的 Git 存储库
3. 为项目命名（仅限小写字符和连字符）
4. 检查这些配置选项是否已设置：

| Configuration option                      | Value              |
| ----------------------------------------- | ------------------ |
| Framework Preset                          | `Other`            |
| Root Directory                            | `./`               |
| Build and Output Settings > Build Command | `npx quartz build` |

5. 单击 Deploy，一旦上线，您将有 2 个 `*.vercel.app` 网址来查看该页面。

### 自定义域

> [!note]
> 如果在此域名上已托管某些内容，则如果不替换掉以前的内容，以下的步骤将不起作用。要解决此问题，您可以使用 Next.js 重写或参考下一部分[[#使用子域]]。

1. 如有必要，请先更新 `quartz.config.js` 中的 `baseUrl` 
2. 转到 Vercel 中的 [Domains - Dashboard](https://vercel.com/dashboard/domains) 页面
3. 将域连接到 Vercel
4. 单击 Add 将自定义域连接到 Vercel
5. 选择您的 Quartz 存储库并单击 Continue
6. 输入您想要连接到的域名
7. 按照说明更新您的 DNS 记录，直到您看到 Valid Configuration

### 使用子域

使用 `docs.example.com` 是子域的一个示例，它们是将多个部署连接到一个域名的一种简单方法。

1. 如有必要，请先更新 `quartz.config.js` 中的 `baseUrl`
2. 确保您的域已添加到 Vercel 中的 [Domains - Dashboard](https://vercel.com/dashboard/domains) 页面
3. 转到 [Vercel Dashboard](https://vercel.com/dashboard) 并选择您的 Quartz 项目
4. 转到 Settings 选项卡，然后单击侧栏中的 Domains
5. 在字段中输入您的子域，然后单击 Add

## Netlify

1. 登录到 [Netlify dashboard](https://app.netlify.com/) 并单击 Add new site
2. 选择 Git provider 和包含 Quartz 项目的仓库
3. 在 Build command 下输入 `npx quartz build`
4. 在 Publish directory 下输入 `public`
5. 单击 Deploy，一旦上线，您将有一个 `*.netlify.app` 的 URL 来查看该页面。
6. 要添加自定义域，请选中左侧边栏中的 Domain management，随后像使用 Vercel 一样添加
## GitLab Pages

在本地 Quartz 中，创建一个新文件 `.gitlab-ci.yaml`

```yaml title=".gitlab-ci.yaml"
stages:
  - build
  - deploy

variables:
  NODE_VERSION: "18.14"

build:
  stage: build
  rules:
    - if: '$CI_COMMIT_REF_NAME == "v4"'
  before_script:
    - apt-get update -q && apt-get install -y nodejs npm
    - npm install -g n
    - n $NODE_VERSION
    - hash -r
    - npm ci
  script:
    - npx quartz build
  artifacts:
    paths:
      - public
  cache:
    paths:
      - ~/.npm/
    key: "${CI_COMMIT_REF_SLUG}-node-${CI_COMMIT_REF_NAME}"
  tags:
    - docker

pages:
  stage: deploy
  rules:
    - if: '$CI_COMMIT_REF_NAME == "v4"'
  script:
    - echo "Deploying to GitLab Pages..."
  artifacts:
    paths:
      - public
```

提交 `.gitlab-ci.yaml` 后，GitLab 将构建网站并将其部署为 GitLab 页面。您可以在侧边栏中的 `Deploy > Pages` 下找到该网址。

默认情况下，该页面是私有的，仅在登录有权访问存储库的 GitLab 帐户时可见，但可以在 `Deploy` -> `Pages` 下的设置中打开。
