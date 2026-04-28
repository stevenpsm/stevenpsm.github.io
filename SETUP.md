# 赛博炒饭 — 站点构建与维护指南

本文档记录站点 `https://stevenpsm.github.io/` 的完整搭建过程、日常维护操作，
以及历次构建中遇到的关键问题与解决方案。适合本人重建或供其他模型参考复刻。

---

## 一、技术栈

| 层 | 选型 | 原因 |
|---|---|---|
| 静态站生成器 | Hugo 0.121.2 Extended | 单二进制、构建快、支持播客 RSS |
| 主题 | PaperMod **v8.0** | 简洁、双语、暗色/亮色自动、搜索归档内置 |
| 部署 | GitHub Actions → GitHub Pages | 零服务端，push 触发自动发布 |
| 评论 | giscus（GitHub Discussions） | 无广告、无追踪、数据属于自己 |
| 音频 | static/audio/（少量直存仓库） | 播客音频极少，无需 CDN |

> **安全保证**：GitHub Actions 构建产物 `public/` 里只有 `.html/.css/.js/.xml/.json`
> 静态文件，GitHub Pages 只做 CDN 分发，服务端零可执行代码，零数据库。

---

## 二、目录结构

```
stevenpsm.github.io/
├── .github/workflows/deploy.yml   # 自动部署 workflow
├── .gitignore                     # 排除 public/ resources/
├── .gitmodules                    # PaperMod submodule 声明
├── hugo.toml                      # 站点主配置
├── archetypes/
│   ├── posts.md                   # 博客文章模板
│   └── episodes.md                # 播客集模板
├── content/
│   ├── _index.md                  # 首页简介
│   ├── about.md                   # 关于页
│   ├── archives.md                # 归档页（PaperMod 专用 layout）
│   ├── search.md                  # 搜索页（PaperMod 专用 layout）
│   ├── posts/                     # 博客文章（.md）
│   └── episodes/                  # 播客集（.md）
├── layouts/
│   ├── shortcodes/audio.html      # 音频播放器 shortcode
│   ├── partials/comments.html     # giscus 评论组件
│   └── _default/rss.xml           # 自定义 Podcast RSS（含 <enclosure>）
├── static/
│   └── audio/                     # 音频文件存放处
└── themes/
    └── PaperMod/                  # git submodule，固定在 v8.0
```

---

## 三、首次搭建步骤

### 3.1 安装 Hugo Extended

**Ubuntu 20.04（glibc 2.31）重要说明：**
- Hugo 0.126+ 要求 glibc 2.32+，Ubuntu 20.04 不满足
- 必须使用 **Hugo 0.121.2 Extended**（最后兼容 glibc 2.31 的版本）
- GitHub Actions runner（ubuntu-latest = 24.04）glibc 2.35，可用任意版本

```bash
wget -O /tmp/hugo.tar.gz \
  https://github.com/gohugoio/hugo/releases/download/v0.121.2/hugo_extended_0.121.2_linux-amd64.tar.gz

mkdir -p ~/.local/bin
tar -xzf /tmp/hugo.tar.gz -C ~/.local/bin hugo

# 验证
~/.local/bin/hugo version
# 期望输出：hugo v0.121.2-...+extended linux/amd64
```

### 3.2 初始化 Hugo 项目

```bash
cd /path/to/stevenpsm.github.io   # 已有 .git 的目录
~/.local/bin/hugo new site . --force
rm index.html                      # 删除旧的 hello world
```

### 3.3 添加 PaperMod 主题（v8.0）

**重要：** 不能使用最新 PaperMod，最新版要求 Hugo 0.146+。v8.0 要求 ≥ 0.112.4。

```bash
git submodule add --depth=1 \
  https://github.com/adityatelange/hugo-PaperMod.git \
  themes/PaperMod

# 固定到 v8.0（兼容 Hugo 0.121.2）
git -C themes/PaperMod fetch --tags --depth=1 origin v8.0
git -C themes/PaperMod checkout v8.0
```

### 3.4 创建 hugo.toml

```toml
baseURL = "https://stevenpsm.github.io/"
languageCode = "zh-CN"
title = "赛博炒饭"
theme = "PaperMod"
hasCJKLanguage = true        # 中文字数统计正确
enableRobotsTXT = true
buildDrafts = false
buildFuture = false
paginate = 10
summaryLength = 100

[params]
  author = "stevenpsm"
  description = "技术漫谈，偶尔播客。Tech ramblings, occasional podcast."
  defaultTheme = "auto"      # 跟随系统暗/亮色
  ShowReadingTime = true
  ShowPostNavLinks = true
  ShowBreadCrumbs = true
  ShowCodeCopyButtons = true
  ShowToc = true
  comments = true

  [params.homeInfoParams]
    Title = "赛博炒饭 🍳"
    Content = "技术漫谈，偶尔播客。 / Tech ramblings, occasional podcast."

  [[params.socialIcons]]
    name = "github"
    url = "https://github.com/stevenpsm"

  [[params.socialIcons]]
    name = "rss"
    url = "/index.xml"

[outputs]
  home = ["HTML", "RSS", "JSON"]   # JSON 供搜索索引

[menu]
  [[menu.main]]
    identifier = "archives"
    name = "归档"
    url = "/archives/"
    weight = 10
  [[menu.main]]
    identifier = "tags"
    name = "标签"
    url = "/tags/"
    weight = 20
  [[menu.main]]
    identifier = "search"
    name = "搜索"
    url = "/search/"
    weight = 30
  [[menu.main]]
    identifier = "about"
    name = "关于"
    url = "/about/"
    weight = 40

[markup]
  [markup.highlight]
    style = "dracula"
  [markup.goldmark.renderer]
    unsafe = true   # 允许 Markdown 中嵌入原始 HTML（音频播放器等）
```

### 3.5 创建必要的固定页面

```bash
# content/archives.md
---
title: "归档 / Archives"
layout: "archives"
url: "/archives/"
---

# content/search.md
---
title: "搜索 / Search"
layout: "search"
url: "/search/"
---
```

> PaperMod 的归档和搜索页**必须手动创建**，且 `layout` 字段不能省略。

### 3.6 创建 .gitignore

```
public/           # Hugo 构建产物，不提交
resources/        # Hugo 资源缓存
.hugo_build.lock
.DS_Store
```

### 3.7 创建 GitHub Actions Workflow

文件：`.github/workflows/deploy.yml`

```yaml
name: Deploy Hugo to GitHub Pages

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest
    env:
      HUGO_VERSION: 0.121.2
    steps:
      - name: Install Hugo Extended
        run: |
          wget -q -O /tmp/hugo.deb \
            https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb
          sudo dpkg -i /tmp/hugo.deb

      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive   # 必须，拉取 PaperMod submodule
          fetch-depth: 0

      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v5

      - name: Build
        run: hugo --minify --baseURL "${{ steps.pages.outputs.base_url }}/"

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

### 3.8 配置 GitHub Pages Source

**必须手动切换，否则 `actions/deploy-pages` 无法工作。**

方式一（CLI）：
```bash
gh api --method PUT repos/stevenpsm/stevenpsm.github.io/pages \
  -f build_type=workflow
```

方式二（网页）：
仓库 → Settings → Pages → Source → 选择 **"GitHub Actions"**

### 3.9 本地验证构建

```bash
~/.local/bin/hugo --minify

# 期望输出：
# Pages | 23+
# Total in ~50ms
# 无任何 ERROR
```

### 3.10 提交并推送

```bash
git add -A
git commit -m "feat: bootstrap Hugo site"
git push origin main
# GitHub Actions 自动触发，约 30 秒完成部署
```

---

## 四、giscus 评论激活

> 每个新仓库需要单独操作一次，ID 不可跨仓库复用。

1. 仓库 **Settings → Features → Discussions** 打勾
2. 安装 giscus GitHub App：https://github.com/apps/giscus
3. 访问 https://giscus.app，填入 `stevenpsm/stevenpsm.github.io`，获取配置
4. 将 `data-repo-id` 和 `data-category-id` 填入 `layouts/partials/comments.html`：

```html
<script src="https://giscus.app/client.js"
  data-repo="stevenpsm/stevenpsm.github.io"
  data-repo-id="R_kgDOSOuzDQ"
  data-category="Announcements"
  data-category-id="DIC_kwDOSOuzDc4C7330"
  data-mapping="pathname"
  data-theme="preferred_color_scheme"
  data-lang="zh-CN"
  crossorigin="anonymous"
  async>
</script>
```

5. `git commit` + `git push` → 自动重新部署

---

## 五、日常操作

### 5.1 写新博客文章

```bash
~/.local/bin/hugo new content posts/我的文章标题.md
```

文章 front matter 模板：

```yaml
---
title: "文章标题"
date: 2026-05-01T10:00:00+08:00
draft: false
tags: ["Chinese", "技术"]    # 双语用 Chinese/English tag 区分
categories: ["工程"]
description: "文章摘要（用于 SEO 和列表页展示）"
---

<!-- 中文内容 -->

---

<!-- English content -->
```

### 5.2 发布播客集

```bash
~/.local/bin/hugo new content episodes/ep01-标题.md
```

播客集额外 front matter 字段：

```yaml
audio_url: "/audio/ep01.mp3"   # 音频路径（相对 static/）
duration: "32:15"              # 时长，写入 RSS <itunes:duration>
episode: 1
```

在正文中嵌入播放器：

```
{{</* audio src="/audio/ep01.mp3" */>}}
```

将 MP3 文件放入 `static/audio/ep01.mp3`，提交即可。

### 5.3 本地预览

```bash
~/.local/bin/hugo server --buildDrafts
# 访问 http://localhost:1313
# 支持热重载，修改文件自动刷新
```

### 5.4 发布流程

```bash
git add -A
git commit -m "post: 文章标题"
git push origin main
# 约 30 秒后 https://stevenpsm.github.io/ 自动更新
```

查看部署状态：
```bash
gh run list --repo stevenpsm/stevenpsm.github.io --limit 3
```

---

## 六、已知问题与解决方案

### 问题 1：Hugo 版本与 glibc 不兼容

**现象：** Ubuntu 20.04 运行 Hugo 0.126+ 报 `GLIBC_2.32 not found`

**原因：** Ubuntu 20.04 自带 glibc 2.31，Hugo 0.126 起要求 2.32+

**解决：** 本地使用 Hugo **0.121.2**，CI（ubuntu-latest = 24.04，glibc 2.35）不受影响

### 问题 2：PaperMod 最新版不兼容 Hugo 0.121.2

**现象：** `ERROR => hugo v0.146.0 or greater is required`

**原因：** PaperMod master 分支将 `min_version` 升至 `0.146.0`

**解决：** 固定 submodule 到 **v8.0**（要求 Hugo ≥ 0.112.4）

```bash
git -C themes/PaperMod fetch --tags --depth=1 origin v8.0
git -C themes/PaperMod checkout v8.0
```

### 问题 3：GitHub Pages 不触发部署

**现象：** push 成功但页面不更新，`deploy-pages` 报权限错误

**原因：** Pages source 默认是 `legacy`（branch 模式），需切换为 `workflow` 模式

**解决：**
```bash
gh api --method PUT repos/OWNER/REPO/pages -f build_type=workflow
```

### 问题 4：归档/搜索页 404

**现象：** `/archives/` 和 `/search/` 返回 404

**原因：** PaperMod 这两个页面需手动创建 `.md` 文件并指定专属 `layout`

**解决：** 创建 `content/archives.md`（`layout: archives`）和 `content/search.md`（`layout: search`）

---

## 七、Podcast RSS 说明

RSS 地址：`https://stevenpsm.github.io/index.xml`

自定义模板位于 `layouts/_default/rss.xml`，在标准 Hugo RSS 基础上增加：
- `xmlns:itunes` 命名空间（Apple Podcasts 兼容）
- `<enclosure>` 标签（当文章 front matter 含 `audio_url` 时自动输出）
- `<itunes:duration>` 时长标签
- `<content:encoded>` 完整正文（非截断摘要）

可直接提交到 Apple Podcasts Connect 或 Spotify for Podcasters。

---

## 八、版本记录

| 日期 | 事件 |
|---|---|
| 2026-04-28 | 初始化 Hugo 站点，首次部署 |
| 2026-04-28 | 激活 giscus 评论（Discussions + App 安装完成） |
| 2026-04-28 | 写入本文档 SETUP.md |
