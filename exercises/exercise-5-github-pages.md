# 练习 5：使用 GitHub Pages 部署网站

## 目标

学习使用 GitHub Pages 部署静态网站。

## 步骤

### 1. 创建练习仓库

```bash
mkdir github-pages-practice
cd github-pages-practice
git init
```

### 2. 创建网站文件

创建 `index.html`：

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>My GitHub Pages Site</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            max-width: 800px;
            margin: 0 auto;
            padding: 20px;
            background-color: #f5f5f5;
        }
        .container {
            background: white;
            padding: 40px;
            border-radius: 10px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
        }
        h1 { color: #333; }
        p { color: #666; line-height: 1.6; }
        .footer {
            margin-top: 40px;
            padding-top: 20px;
            border-top: 1px solid #eee;
            color: #999;
            font-size: 14px;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Hello GitHub Pages!</h1>
        <p>这是我的第一个 GitHub Pages 网站。</p>
        <p>部署时间：2024年</p>
        <div class="footer">
            <p>Built with ❤️ using GitHub Pages</p>
        </div>
    </div>
</body>
</html>
```

### 3. 提交并推送

```bash
git add index.html
git commit -m "Initial commit"

git remote add origin git@github.com:你的用户名/github-pages-practice.git
git push -u origin main
```

### 4. 启用 GitHub Pages

1. 进入仓库 **Settings**
2. 左侧菜单点击 **Pages**
3. Source 选择 **main** 分支
4. 点击 **Save**

### 5. 访问网站

等待几分钟后，访问：
```
https://你的用户名.github.io/github-pages-practice/
```

## 进阶：使用自定义域名

### 1. 添加 CNAME 文件

```bash
echo "yourdomain.com" > CNAME
git add CNAME
git commit -m "添加自定义域名"
git push
```

### 2. 配置 DNS

在你的域名服务商添加：
- 类型：CNAME
- 主机记录：@ 或 www
- 值：`你的用户名.github.io`

### 3. 启用 HTTPS

**Settings** → **Pages** → ✅ **Enforce HTTPS**

## 进阶：使用 Jekyll

### 1. 创建 `_config.yml`

```yaml
title: My Site
description: A website powered by Jekyll
theme: minima
```

### 2. 创建文章

创建 `_posts/2024-01-01-hello-world.md`：

```markdown
---
layout: post
title: "Hello World"
date: 2024-01-01
categories: blog
---

这是我的第一篇博客文章。
```

### 3. 提交并推送

```bash
git add .
git commit -m "添加 Jekyll 配置"
git push
```

## 进阶：使用 GitHub Actions 部署

### 1. 创建工作流

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
    
    - name: Setup Pages
      uses: actions/configure-pages@v4
      
    - name: Upload artifact
      uses: actions/upload-pages-artifact@v3
      with:
        path: '.'
        
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

### 2. 启用 GitHub Pages

**Settings** → **Pages** → Source 选择 **GitHub Actions**

## 知识点

- GitHub Pages 基本使用
- 自定义域名配置
- Jekyll 静态站点生成器
- GitHub Actions 自动部署

## 下一步

[练习 6：参与开源项目 →](exercise-6-open-source.md)
