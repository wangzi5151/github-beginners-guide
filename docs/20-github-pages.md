# GitHub Pages 静态网站

## 什么是 GitHub Pages？

GitHub Pages 是免费的静态网站托管服务，可以直接从仓库部署网站。

**适用场景：**
- 个人博客
- 项目文档
- 作品集展示
- 学习笔记

## 创建网站（图文详解）

### 方法一：直接在 main 分支（最简单）

**第 1 步：创建 index.html 文件**

在仓库中创建一个 `index.html` 文件：

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>我的网站</title>
</head>
<body>
    <h1>欢迎来到我的网站！</h1>
    <p>这是我的第一个 GitHub Pages 网站。</p>
</body>
</html>
```

**第 2 步：提交并推送**

```bash
git add index.html
git commit -m "添加首页"
git push
```

**第 3 步：启用 GitHub Pages**

1. 打开仓库页面
2. 点击 **Settings** 标签
3. 左侧菜单找到 **Pages**
4. Source 选择 **main** 分支
5. 点击 **Save**

```
┌─────────────────────────────────────────────┐
│  GitHub Pages                                │
│                                             │
│  Build and deployment                        │
│                                             │
│  Source: [Deploy from a branch ▼]            │
│                                             │
│  Branch:                                     │
│  [main ▼]  [/ (root) ▼]                     │
│                                             │
│  ☑ Save  ← 点击保存                          │
│                                             │
│  Your site is live at                        │
│  https://username.github.io/repo-name/      │
└─────────────────────────────────────────────┘
```

**第 4 步：访问网站**

部署完成后（通常需要 1-2 分钟），访问：
```
https://你的用户名.github.io/仓库名/
```

### 方法二：使用 docs 目录

**第 1 步：创建 docs 目录**

```bash
mkdir docs
```

**第 2 步：创建 index.html**

```bash
echo "<h1>Hello World</h1>" > docs/index.html
```

**第 3 步：提交并推送**

```bash
git add docs/
git commit -m "添加文档站点"
git push
```

**第 4 步：配置 Pages**

1. 打开仓库 **Settings** → **Pages**
2. Source 选择 **main** 分支的 **/docs** 目录
3. 点击 **Save**

### 方法三：使用 GitHub Actions

**第 1 步：配置 Pages**

1. 打开仓库 **Settings** → **Pages**
2. Source 选择 **GitHub Actions**

```
┌─────────────────────────────────────────────┐
│  GitHub Pages                                │
│                                             │
│  Build and deployment                        │
│                                             │
│  Source: [GitHub Actions ▼]                  │
│                                             │
└─────────────────────────────────────────────┘
```

**第 2 步：创建工作流文件**

创建 `.github/workflows/pages.yml`：

```yaml
name: Deploy to Pages

on:
  push:
    branches: [main]

permissions:
  contents: read
  pages: write
  id-token: write

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - run: npm ci && npm run build
    - uses: actions/upload-pages-artifact@v3
      with:
        path: ./dist
        
  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
    - id: deployment
      uses: actions/deploy-pages@v4
```

## 自定义域名

### 购买域名

推荐域名服务商：
- 阿里云
- 腾讯云
- 万网
- GoDaddy

### 设置步骤

**第 1 步：添加 CNAME 文件**

在仓库根目录创建 `CNAME` 文件：

```bash
echo "yourdomain.com" > CNAME
git add CNAME
git commit -m "添加自定义域名"
git push
```

**第 2 步：配置 DNS**

在域名服务商控制台添加：

| 记录类型 | 主机记录 | 记录值 |
|----------|----------|--------|
| CNAME | @ | username.github.io |
| CNAME | www | username.github.io |

**第 3 步：启用 HTTPS**

1. 打开 **Settings** → **Pages**
2. 勾选 **Enforce HTTPS**

```
┌─────────────────────────────────────────────┐
│  GitHub Pages                                │
│                                             │
│  Custom domain: [yourdomain.com]            │
│                                             │
│  ☑ Enforce HTTPS  ← 勾选启用                │
│                                             │
└─────────────────────────────────────────────┘
```

## 使用 Jekyll（内置工具）

Jekyll 是 GitHub Pages 内置的静态站点生成器，可以使用 Markdown 写作。

### 基本配置

创建 `_config.yml` 文件：

```yaml
title: 我的博客
description: 一个简单的博客
theme: minima
```

### 创建文章

在 `_posts` 目录创建 Markdown 文件：

```markdown
---
layout: post
title: "Hello World"
date: 2024-01-01
categories: blog
---

这是我的第一篇文章。

## 标题

正文内容...
```

### 主题选择

```yaml
# _config.yml
remote_theme: pages-themes/cayman@v0.2.0
plugins:
  - jekyll-remote-theme
```

## 查看部署状态

### 查看 Pages 部署历史

1. 打开仓库 **Actions** 标签
2. 点击 **pages build and deployment** 工作流
3. 查看部署状态

```
┌─────────────────────────────────────────────┐
│  Actions                                     │
│                                             │
│  workflows / pages build and deployment      │
│                                             │
│  ✓ Deploy to GitHub Pages                    │
│    ✓ build                                   │
│    ✓ deploy                                  │
│                                             │
│  部署时间: 2 分钟前                          │
│  状态: 成功 ✓                                │
└─────────────────────────────────────────────┘
```

### 查看 Pages 设置

```
┌─────────────────────────────────────────────┐
│  GitHub Pages                                │
│                                             │
│  Your site is published at                   │
│  https://username.github.io/repo-name/      │
│                                             │
│  Last deployment: 2 minutes ago              │
│  Deployment status: ✓ Success               │
│                                             │
│  [Visit site]  ← 点击访问网站               │
└─────────────────────────────────────────────┘
```

## 限制

| 限制 | 说明 |
|------|------|
| 仓库大小 | 1GB |
| 发布站点大小 | 1GB |
| 带宽 | 100GB/月 |
| 构建次数 | 每小时最多 10 次 |

## 常用框架部署

### React (Create React App)

```yaml
- name: Build
  run: npm run build
  
- name: Deploy
  uses: peaceiris/actions-gh-pages@v3
  with:
    github_token: ${{ secrets.GITHUB_TOKEN }}
    publish_dir: ./build
```

### Vue

```yaml
- name: Build
  run: npm run build
  
- name: Deploy
  uses: peaceiris/actions-gh-pages@v3
  with:
    github_token: ${{ secrets.GITHUB_TOKEN }}
    publish_dir: ./dist
```

---

## 实践练习

### 练习：创建你的第一个 GitHub Pages 网站

**任务 1：创建仓库**
1. 在 GitHub 创建新仓库 `my-first-website`
2. 克隆到本地

**任务 2：创建网站文件**
1. 创建 `index.html` 文件
2. 输入简单的 HTML 代码
3. 提交并推送

**任务 3：启用 GitHub Pages**
1. 打开仓库 **Settings** → **Pages**
2. Source 选择 **main** 分支
3. 点击 **Save**

**任务 4：访问网站**
1. 等待 1-2 分钟
2. 访问 `https://你的用户名.github.io/my-first-website/`
3. 看到网站内容表示成功

**验证方法：**
- 网站可以正常访问
- 显示你创建的内容

## 下一步

[GitHub Projects 项目管理 →](21-github-projects.md)
