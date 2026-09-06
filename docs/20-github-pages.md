# GitHub Pages 静态网站

## 什么是 GitHub Pages？

GitHub Pages 是免费的静态网站托管服务，可以直接从仓库部署网站。

## 创建网站

### 方法一：直接在 main 分支

```bash
# 创建 index.html
echo "<h1>Hello World</h1>" > index.html

# 提交并推送
git add index.html
git commit -m "添加首页"
git push
```

### 方法二：使用 docs 目录

1. 仓库 **Settings** → **Pages**
2. Source 选择 **main** 分支的 **/docs** 目录

```bash
mkdir docs
echo "<h1>Hello World</h1>" > docs/index.html
git add docs/
git commit -m "添加文档站点"
git push
```

### 方法三：使用 GitHub Actions

1. 仓库 **Settings** → **Pages**
2. Source 选择 **GitHub Actions**

```yaml
# .github/workflows/pages.yml
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

### 设置步骤
1. 购买域名
2. 添加 CNAME 文件：
   ```bash
   echo "yourdomain.com" > CNAME
   git add CNAME
   git commit -m "添加自定义域名"
   git push
   ```
3. 在域名服务商设置 DNS：
   - 类型：CNAME
   - 主机记录：@ 或 www
   - 值：`username.github.io`

### 启用 HTTPS
**Settings** → **Pages** → ✅ **Enforce HTTPS**

## 使用 Jekyll

Jekyll 是 GitHub Pages 内置的静态站点生成器。

### 基本配置

```yaml
# _config.yml
title: My Site
description: A website powered by Jekyll
theme: minima
```

### 创建文章

```markdown
---
layout: post
title: "Hello World"
date: 2024-01-01
categories: blog
---

这是我的第一篇文章。
```

### 主题

```yaml
# _config.yml
remote_theme: pages-themes/cayman@v0.2.0
plugins:
  - jekyll-remote-theme
```

## 限制

- 仓库大小限制 1GB
- 发布站点大小限制 1GB
- 带宽限制 100GB/月
- 每小时最多 10 次构建

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

## 下一步

[GitHub Projects 项目管理 →](21-github-projects.md)
