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
> Quartz 将生成格式为 `file.html` 而不是 `file/index.html` 的文件，这意味着非文件夹路径的尾部斜杠将被删除。由于 GitHub Pages 不执行此重定向，因此这可能会导致使用尾部斜杠的现有网站链接中断。如果不破坏现有链接对您很重要（例如您要从 Quartz 3 迁移），请考虑使用 [[#Cloudflare Pages]。

### 自定义域名

Here's how to add a custom domain to your GitHub pages deployment.

1. Head to the "Settings" tab of your forked repository.
2. In the "Code and automation" section of the sidebar, click "Pages".
3. Under "Custom Domain", type your custom domain and click "Save".
4. This next step depends on whether you are using an apex domain (`example.com`) or a subdomain (`subdomain.example.com`).
   - If you are using an apex domain, navigate to your DNS provider and create an `A` record that points your apex domain to GitHub's name servers which have the following IP addresses:
     - `185.199.108.153`
     - `185.199.109.153`
     - `185.199.110.153`
     - `185.199.111.153`
   - If you are using a subdomain, navigate to your DNS provider and create a `CNAME` record that points your subdomain to the default domain for your site. For example, if you want to use the subdomain `quartz.example.com` for your user site, create a `CNAME` record that points `quartz.example.com` to `<github-username>.github.io`.

![[dns records.png]]_The above shows a screenshot of Google Domains configured for both `jzhao.xyz` (an apex domain) and `quartz.jzhao.xyz` (a subdomain)._

See the [GitHub documentation](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site#configuring-a-subdomain) for more detail about how to setup your own custom domain with GitHub Pages.

> [!question] Why aren't my changes showing up?
> There could be many different reasons why your changes aren't showing up but the most likely reason is that you forgot to push your changes to GitHub.
>
> Make sure you save your changes to Git and sync it to GitHub by doing `npx quartz sync`. This will also make sure to pull any updates you may have made from other devices so you have them locally.

## Vercel

### Fix URLs

Before deploying to Vercel, a `vercel.json` file is required at the root of the project directory. It needs to contain the following configuration so that URLs don't require the `.html` extension:

```json title="vercel.json"
{
  "cleanUrls": true
}
```

### Deploy to Vercel

1. Log in to the [Vercel Dashboard](https://vercel.com/dashboard) and click "Add New..." > Project
2. Import the Git repository containing your Quartz project.
3. Give the project a name (lowercase characters and hyphens only)
4. Check that these configuration options are set:

| Configuration option                      | Value              |
| ----------------------------------------- | ------------------ |
| Framework Preset                          | `Other`            |
| Root Directory                            | `./`               |
| Build and Output Settings > Build Command | `npx quartz build` |

5. Press Deploy. Once it's live, you'll have 2 `*.vercel.app` URLs to view the page.

### Custom Domain

> [!note]
> If there is something already hosted on the domain, these steps will not work without replacing the previous content. As a workaround, you could use Next.js rewrites or use the next section to create a subdomain.

1. Update the `baseUrl` in `quartz.config.js` if necessary.
2. Go to the [Domains - Dashboard](https://vercel.com/dashboard/domains) page in Vercel.
3. Connect the domain to Vercel
4. Press "Add" to connect a custom domain to Vercel.
5. Select your Quartz repository and press Continue.
6. Enter the domain you want to connect it to.
7. Follow the instructions to update your DNS records until you see "Valid Configuration"

### Use a Subdomain

Using `docs.example.com` is an example of a subdomain. They're a simple way of connecting multiple deployments to one domain.

1. Update the `baseUrl` in `quartz.config.js` if necessary.
2. Ensure your domain has been added to the [Domains - Dashboard](https://vercel.com/dashboard/domains) page in Vercel.
3. Go to the [Vercel Dashboard](https://vercel.com/dashboard) and select your Quartz project.
4. Go to the Settings tab and then click Domains in the sidebar
5. Enter your subdomain into the field and press Add

## Netlify

1. Log in to the [Netlify dashboard](https://app.netlify.com/) and click "Add new site".
2. Select your Git provider and repository containing your Quartz project.
3. Under "Build command", enter `npx quartz build`.
4. Under "Publish directory", enter `public`.
5. Press Deploy. Once it's live, you'll have a `*.netlify.app` URL to view the page.
6. To add a custom domain, check "Domain management" in the left sidebar, just like with Vercel.

## GitLab Pages

In your local Quartz, create a new file `.gitlab-ci.yaml`.

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

When `.gitlab-ci.yaml` is commited, GitLab will build and deploy the website as a GitLab Page. You can find the url under `Deploy > Pages` in the sidebar.

By default, the page is private and only visible when logged in to a GitLab account with access to the repository but can be opened in the settings under `Deploy` -> `Pages`.
