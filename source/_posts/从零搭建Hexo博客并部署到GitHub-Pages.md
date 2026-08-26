---
title: 从零搭建Hexo博客并部署到GitHub Pages
date: 2026-08-25 16:00:00
tags:
  - Hexo
  - GitHub Pages
  - 博客搭建
  - 教程
categories:
  - 技术折腾
---

# 从零搭建Hexo博客并部署到GitHub Pages

记录一下从零开始搭建 Hexo 博客的完整过程，包括踩坑和解决方案。

<!-- more -->

## 一、环境准备

首先需要安装 Node.js 和包管理器（npm/yarn/pnpm/bun）。

```bash
# 安装 Hexo CLI
npm install -g hexo-cli

# 或者使用 bun
bun install -g hexo-cli
```

## 二、初始化 Hexo 项目

```bash
# 初始化博客
hexo init hexo-blog
cd hexo-blog

# 安装依赖
npm install
# 或
bun install
```

项目结构如下：

```
hexo-blog/
├── _config.yml      # 主配置文件
├── package.json     # 依赖配置
├── scaffolds/       # 文章模板
├── source/          # 博客源码
│   └── _posts/      # 文章目录
└── themes/          # 主题目录
```

## 三、安装 Async 主题

我选择了 [hexo-theme-async](https://github.com/MaLuns/hexo-theme-async)，一款简洁优雅的主题。

### 方法一：npm 安装（推荐）

```bash
bun add hexo-theme-async
```

然后创建主题软链接：

```bash
ln -s ../node_modules/hexo-theme-async themes/hexo-theme-async
```

### 方法二：Git Clone

```bash
cd themes
git clone https://github.com/MaLuns/hexo-theme-async.git hexo-theme-async
```

在 `_config.yml` 中设置主题：

```yaml
theme: hexo-theme-async
```

## 四、配置 GitHub 仓库

1. 在 GitHub 创建仓库 `hexo`（或任意名称）

2. 初始化 Git 并关联远程仓库：

```bash
git init
git remote add origin https://github.com/hesirui12/hexo.git
```

## 五、配置 GitHub Actions 自动部署

在项目根目录创建 `.github/workflows/deploy.yml`：

```yaml
name: Deploy Hexo

on:
  push:
    branches: [ main ]

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
    steps:
      - uses: actions/checkout@v4

      - name: Setup Bun
        uses: oven-sh/setup-bun@v1
        with:
          bun-version: latest

      - name: Install dependencies
        run: bun install

      - name: Build
        run: bunx hexo generate

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

## 六、⚠️ 踩坑：资源全部 404

推送代码后，发现网站样式全丢了，所有 CSS、JS、图片都 404！

### 问题原因

GitHub Pages 的访问地址是：
```
https://hesirui12.github.io/hexo/
```

但 `_config.yml` 中的 `url` 默认是：
```yaml
url: http://example.com
```

这导致生成的资源路径是 `/css/...`，而不是 `/hexo/css/...`。

浏览器请求 `https://hesirui12.github.io/css/...`，但资源实际在 `https://hesirui12.github.io/hexo/css/...`，所以 404！

### 解决方案

修改 `_config.yml` 中的 `url` 为你的 GitHub Pages 地址：

```yaml
# 修改前
url: http://example.com

# 修改后
url: https://hesirui12.github.io/hexo
```

重新生成并推送：

```bash
hexo generate
git add .
git commit -m "fix: update url to correct GitHub Pages path"
git push origin main
```

验证生成的 HTML，资源路径现在正确了：

```html
<!-- 之前 -->
<link rel="stylesheet" href="/css/index.css">

<!-- 之后 -->
<link rel="stylesheet" href="/hexo/css/index.css">
```

## 七、本地预览

```bash
hexo server
```

访问 http://localhost:4000/hexo/ 预览。

## 八、常用命令

```bash
# 新建文章
hexo new "文章标题"

# 本地预览
hexo server

# 生成静态文件
hexo generate

# 清除缓存和生成的文件
hexo clean

# 一键部署（本地）
hexo deploy
```

## 总结

踩坑记录：
1. **资源 404** → 修改 `url` 为正确的 GitHub Pages 地址
2. **主题不生效** → 检查软链接和 `_config.yml` 中的 theme 配置
3. **部署失败** → 检查 GitHub Actions 日志和 Pages 设置

搭建完的博客地址：https://hesirui12.github.io/hexo/

---

*本文记录了从零搭建 Hexo 博客并部署到 GitHub Pages 的完整过程，包括遇到的问题和解决方案。希望对同样在折腾博客的你有帮助！*
