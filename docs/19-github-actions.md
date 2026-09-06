# GitHub Actions 自动化

## 什么是 GitHub Actions？

GitHub Actions 是 GitHub 的 CI/CD 平台，可以自动化构建、测试和部署流程。

## 基本概念

| 概念 | 说明 |
|------|------|
| **Workflow** | 自动化工作流，由 YAML 定义 |
| **Event** | 触发工作流的事件 |
| **Job** | 一组步骤 |
| **Step** | 单个任务 |
| **Action** | 可复用的工作单元 |
| **Runner** | 执行工作流的服务器 |

## 创建 Workflow

### 文件结构

```
.github/
└── workflows/
    └── ci.yml
```

### 基本模板

```yaml
name: CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '20'
        
    - name: Install dependencies
      run: npm install
      
    - name: Run tests
      run: npm test
```

## 常见工作流

### Node.js 项目

```yaml
name: Node.js CI

on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    
    strategy:
      matrix:
        node-version: [18, 20, 22]
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Use Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v4
      with:
        node-version: ${{ matrix.node-version }}
        
    - run: npm ci
    - run: npm run build
    - run: npm test
```

### 部署到 GitHub Pages

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    permissions:
      contents: read
      pages: write
      id-token: write
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Setup Pages
      uses: actions/configure-pages@v4
      
    - name: Build
      run: npm run build
      
    - name: Upload artifact
      uses: actions/upload-pages-artifact@v3
      with:
        path: './dist'
        
    - name: Deploy to GitHub Pages
      uses: actions/deploy-pages@v4
```

## Secrets 管理

在仓库 **Settings** → **Secrets and variables** → **Actions** 中添加：

```yaml
- name: Deploy
  env:
    API_KEY: ${{ secrets.API_KEY }}
  run: ./deploy.sh
```

## 缓存优化

```yaml
- name: Cache dependencies
  uses: actions/cache@v4
  with:
    path: ~/.npm
    key: ${{ runner.os }}-npm-${{ hashFiles('**/package-lock.json') }}
```

## Workflow 语法

### 触发事件

```yaml
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 0 * * *'  # 每天执行
  workflow_dispatch:       # 手动触发
```

### 条件执行

```yaml
- name: Deploy
  if: github.ref == 'refs/heads/main'
  run: ./deploy.sh
```

## Actions 市场

访问 [github.com/marketplace?type=actions](https://github.com/marketplace?type=actions) 发现常用 Action。

## 下一步

[GitHub Pages 静态网站 →](20-github-pages.md)
