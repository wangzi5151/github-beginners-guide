# 第五章：GitHub 核心功能详解

## 5.1 仓库管理

### 创建仓库

#### 方法一：网页创建（推荐新手）

**第 1 步：** 登录 GitHub，点击右上角的 **+** 号，选择 **New repository**

```
┌─────────────────────────────────────────────┐
│  +  [🔔]  [头像]                             │
│  │                                          │
│  └─→ New repository  ← 点击这个              │
│      Import repository                      │
│      New organization                        │
└─────────────────────────────────────────────┘
```

**第 2 步：** 填写仓库信息

```
┌─────────────────────────────────────────────┐
│  Create a new repository                     │
│                                             │
│  Owner: [你的用户名 ▼]                       │
│                                             │
│  Repository name: [my-project          ]     │
│  Description:     [这是一个示例项目    ]      │
│                                             │
│  ○ Public  ← 任何人都可以看到                 │
│  ○ Private ← 只有你和协作者可以看到           │
│                                             │
│  ☑ Add a README file  ← 强烈推荐勾选        │
│                                             │
│  Add .gitignore: [None ▼]                   │
│  Choose a license: [None ▼]                 │
│                                             │
│        [Create repository]                  │
└─────────────────────────────────────────────┘
```

**第 3 步：** 点击 **Create repository** 完成创建

#### 方法二：命令行创建

```bash
# 使用 GitHub CLI
gh repo create my-repo --public --description "我的项目"

# 创建私有仓库
gh repo create my-repo --private
```

### 仓库页面详解

```
┌─────────────────────────────────────────────┐
│  your-username / my-project                  │
│                                             │
│  [Code]  [Issues]  [Pull requests]          │
│  [Actions]  [Projects]  [Wiki]              │
│  [Security]  [Insights]  [Settings]          │
├─────────────────────────────────────────────┤
│                                             │
│  📁 README.md                               │
│  📁 src/                                    │
│  📁 .gitignore                              │
│                                             │
│  This is a sample project...                │
│                                             │
└─────────────────────────────────────────────┘
```

**标签说明：**

| 标签 | 功能 |
|------|------|
| **Code** | 查看代码文件 |
| **Issues** | 问题追踪 |
| **Pull requests** | 代码合并请求 |
| **Actions** | 自动化工作流 |
| **Projects** | 项目看板 |
| **Wiki** | 项目文档 |
| **Security** | 安全设置 |
| **Insights** | 数据分析 |
| **Settings** | 仓库设置 |

### 仓库设置

**进入设置：**
1. 打开仓库页面
2. 点击 **Settings** 标签

**基本设置：**

```
┌─────────────────────────────────────────────┐
│  Settings                                    │
│                                             │
│  General  ← 基本设置                         │
│  Access                                       │
│  Branches                                    │
│  Tags                                        │
│  Webhooks                                    │
│  ...                                         │
├─────────────────────────────────────────────┤
│                                             │
│  Repository name: [my-project          ]     │
│  Description:     [这是一个示例项目    ]      │
│  Website:         [https://example.com]      │
│  Topics:          [react] [javascript]       │
│                                             │
│           [Save changes]                    │
└─────────────────────────────────────────────┘
```

### 添加协作者

**操作步骤：**
1. 进入 **Settings** → **Collaborators**
2. 点击 **Add people**
3. 输入对方的 GitHub 用户名
4. 点击 **Add [username] to this repository**

```
┌─────────────────────────────────────────────┐
│  Collaborators                                │
│                                             │
│  Search by username, full name, or email:    │
│  ┌─────────────────────────────────────┐    │
│  │ username                            │    │
│  └─────────────────────────────────────┘    │
│                                             │
│           [Add to repository]               │
└─────────────────────────────────────────────┘
```

### 删除仓库

**⚠️ 警告：删除操作不可逆！**

**操作步骤：**
1. 进入 **Settings**
2. 滚动到页面最底部
3. 找到 **Danger Zone** 区域
4. 点击 **Delete this repository**
5. 输入仓库名称确认删除

```
┌─────────────────────────────────────────────┐
│  Danger Zone                                 │
│                                             │
│  Delete this repository                      │
│  Once you delete a repository, there is no  │
│  going back.                                │
│                                             │
│  [Delete this repository]                   │
└─────────────────────────────────────────────┘
```

## 5.2 Issue 管理

### 创建 Issue

**第 1 步：** 进入仓库的 **Issues** 页面

**第 2 步：** 点击 **New issue**

```
┌─────────────────────────────────────────────┐
│  Issues                                      │
│                                             │
│  [New issue]  ← 点击这个按钮                 │
│                                             │
│  Filters: [Open ▼] [Labels ▼] [Assignee ▼] │
└─────────────────────────────────────────────┘
```

**第 3 步：** 填写 Issue 信息

```
┌─────────────────────────────────────────────┐
│  New issue                                   │
│                                             │
│  Title: [Bug: 首页加载失败              ]     │
│                                             │
│  Description:                                │
│  ┌─────────────────────────────────────┐    │
│  │ ## 问题描述                         │    │
│  │ 首页在某些情况下无法正常加载           │    │
│  │                                     │    │
│  │ ## 复现步骤                         │    │
│  │ 1. 打开首页                         │    │
│  │ 2. 点击登录按钮                      │    │
│  │ 3. 页面显示空白                      │    │
│  │                                     │    │
│  │ ## 期望行为                         │    │
│  │ 应该显示登录表单                     │    │
│  │                                     │    │
│  │ ## 环境信息                         │    │
│  │ - OS: Windows 11                    │    │
│  │ - Browser: Chrome 120               │    │
│  └─────────────────────────────────────┘    │
│                                             │
│  ☑ Assignees: [选择负责人]                  │
│  ☑ Labels: [bug] [help wanted]              │
│  ☑ Milestone: [v1.0]                       │
│                                             │
│        [Submit new issue]                   │
└─────────────────────────────────────────────┘
```

**第 4 步：** 点击 **Submit new issue**

### 使用 Issue 模板

很多仓库提供 Issue 模板，可以更快地创建标准 Issue：

```
┌─────────────────────────────────────────────┐
│  Choose a template                           │
│                                             │
│  ┌─────────────┐  ┌─────────────┐          │
│  │ 🐛 Bug     │  │ ✨ Feature  │          │
│  │ Report      │  │ Request     │          │
│  │             │  │             │          │
│  │ [Use        │  │ [Use        │          │
│  │  template]  │  │  template]  │          │
│  └─────────────┘  └─────────────┘          │
└─────────────────────────────────────────────┘
```

### 管理 Issue 标签

**创建自定义标签：**

1. 在 Issues 页面点击 **Labels**
2. 点击 **New label**
3. 填写信息：
   - **Label name**：标签名称
   - **Description**：描述
   - **Color**：颜色
4. 点击 **Add label**

```
┌─────────────────────────────────────┐
│  New label                           │
│                                     │
│  Label name: [priority: high   ]    │
│  Description: [高优先级问题      ]    │
│  Color: [🔴]                        │
│                                     │
│     [Add label]                     │
└─────────────────────────────────────┘
```

### 关闭 Issue

**方法一：在 Issue 页面关闭**

1. 打开 Issue 页面
2. 点击底部的 **Close issue**

**方法二：使用关键词自动关闭**

```bash
# 在提交信息中使用关键词
git commit -m "fix: 修复登录问题，closes #42"
git commit -m "fix: 修复多个问题，fixes #42, fixes #43"
```

**关键词：**
- `closes #42`
- `fixes #42`
- `resolves #42`

## 5.3 Pull Request

### 创建 PR

**第 1 步：** 推送功能分支到远程

```bash
git push -u origin feature-new-button
```

**第 2 步：** 打开创建 PR 页面

```
┌─────────────────────────────────────────────┐
│  feature-new-button had recent pushes       │
│                                             │
│  [Compare & pull request]  ← 点击创建 PR     │
└─────────────────────────────────────────────┘
```

**第 3 步：** 选择分支

```
┌─────────────────────────────────────────────┐
│  Compare changes                             │
│                                             │
│  base: [main ▼]  ← 目标分支                  │
│     ...                                     │
│  compare: [feature-new-button ▼]  ← 你的分支 │
│                                             │
│  [Create pull request]                       │
└─────────────────────────────────────────────┘
```

**第 4 步：** 填写 PR 信息

```
┌─────────────────────────────────────────────┐
│  Open a pull request                         │
│                                             │
│  Title: [feat: 添加新按钮功能           ]     │
│                                             │
│  Description:                                │
│  ┌─────────────────────────────────────┐    │
│  │ ## 变更说明                         │    │
│  │ 添加了一个新按钮                     │    │
│  │                                     │    │
│  │ ## 变更类型                         │    │
│  │ - [x] 新功能 (feat)                 │    │
│  │ - [ ] Bug 修复 (fix)                │    │
│  │                                     │    │
│  │ ## 测试                             │    │
│  │ - [x] 已添加测试                    │    │
│  │ - [x] 已通过所有测试                │    │
│  │                                     │    │
│  │ ## 关联 Issue                       │    │
│  │ Closes #42                          │    │
│  └─────────────────────────────────────┘    │
│                                             │
│  ☑ Reviewers: [添加审查者]                  │
│  ☑ Labels: [enhancement]                    │
│                                             │
│        [Create pull request]                │
└─────────────────────────────────────────────┘
```

**第 5 步：** 点击 **Create pull request**

### 审查 PR

**查看代码差异：**

1. 点击 **Files changed** 标签
2. 查看代码变更

```
┌─────────────────────────────────────────────┐
│  Files changed  (3)                          │
│                                             │
│  ─ src/button.ts (+5 -2)                    │
│                                             │
│     1  │ const button = () => {             │
│  -   2  │   return <button>Click</button>;  │
│  +   2  │   return <button className="new"> ││
│  +   3  │     Click                         │
│  +   4  │   </button>;                      │
│     5  │ };                                 │
│                                             │
│  点击行号旁边的 + 可以添加评论                  │
└─────────────────────────────────────────────┘
```

**添加行内评论：**

1. 将鼠标悬停在代码行号旁边
2. 点击出现的 **+** 按钮
3. 在弹出框中输入评论
4. 点击 **Start review**

```
┌─────────────────────────────────────┐
│  💬 Leave a comment                  │
│                                     │
│  这里建议使用更具体的类名              │
│                                     │
│  ○ Comment  ← 仅评论                │
│  ○ Approve  ← 批准                  │
│  ○ Request changes ← 要求修改        │
│                                     │
│     [Start review]                  │
└─────────────────────────────────────┘
```

### 合并 PR

**合并条件：**
- ✅ 所有审查者已批准
- ✅ CI 检查通过
- ✅ 没有冲突

**合并操作：**

1. 在 PR 页面底部找到合并按钮
2. 选择合并方式
3. 点击 **Merge pull request**
4. 点击 **Confirm merge**

```
┌─────────────────────────────────────────────┐
│  Merge pull request                          │
│                                             │
│  ┌─────────────────────────────────────┐    │
│  │  ○ Create a merge commit            │    │
│  │    保留完整提交历史                   │    │
│  │                                     │    │
│  │  ○ Squash and merge                 │    │
│  │    压缩为一个提交                    │    │
│  │                                     │    │
│  │  ○ Rebase and merge                 │    │
│  │    线性历史                          │    │
│  └─────────────────────────────────────┘    │
│                                             │
│        [Merge pull request]                 │
└─────────────────────────────────────────────┘
```

## 5.4 GitHub Actions

### 什么是 GitHub Actions？

GitHub Actions 是 GitHub 的 CI/CD 平台，可以自动化构建、测试和部署流程。

### 创建工作流

**第 1 步：** 在仓库中创建 `.github/workflows` 目录

**第 2 步：** 创建 YAML 文件（如 `ci.yml`）

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

**第 3 步：** 提交并推送

```bash
git add .github/workflows/ci.yml
git commit -m "ci: 添加 CI 工作流"
git push
```

### 查看工作流

1. 进入仓库的 **Actions** 页面
2. 查看工作流运行状态

```
┌─────────────────────────────────────────────┐
│  Actions                                     │
│                                             │
│  workflows / CI                              │
│                                             │
│  ✓ Build and Test                            │
│    ✓ build                                   │
│    ✓ test                                    │
│                                             │
│  部署时间: 2 分钟前                          │
│  状态: 成功 ✓                                │
└─────────────────────────────────────────────┘
```

### 常用工作流模板

**Node.js 项目：**

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

**部署到 GitHub Pages：**

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

## 5.5 GitHub Pages

### 启用 GitHub Pages

**第 1 步：** 进入仓库 **Settings** → **Pages**

**第 2 步：** 选择 Source

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
│           [Save]                            │
└─────────────────────────────────────────────┘
```

**第 3 步：** 等待部署完成

### 访问网站

部署完成后，访问：
```
https://你的用户名.github.io/仓库名/
```

### 自定义域名

**第 1 步：** 购买域名

**第 2 步：** 添加 CNAME 文件

```bash
echo "yourdomain.com" > CNAME
git add CNAME
git commit -m "添加自定义域名"
git push
```

**第 3 步：** 配置 DNS

在域名服务商控制台添加：

| 记录类型 | 主机记录 | 记录值 |
|----------|----------|--------|
| CNAME | @ | username.github.io |
| CNAME | www | username.github.io |

**第 4 步：** 启用 HTTPS

1. 打开 **Settings** → **Pages**
2. 勾选 **Enforce HTTPS**

## 5.6 GitHub Projects

### 创建项目

```bash
# 使用 CLI 创建项目
gh project create --title "项目名称" --owner your-org
```

### 项目视图

| 视图 | 说明 |
|------|------|
| **Board** | 看板视图 |
| **Table** | 表格视图 |
| **Roadmap** | 路线图视图 |
| **Calendar** | 日历视图 |

### 自动化

```yaml
# .github/workflows/project-automation.yml
name: Project Automation

on:
  issues:
    types: [opened, closed]
  pull_request:
    types: [opened, closed, ready_for_review]

jobs:
  auto-add:
    runs-on: ubuntu-latest
    steps:
    - name: Add to project
      uses: actions/add-to-project@v0.5.0
      with:
        project-url: https://github.com/orgs/your-org/projects/1
        github-token: ${{ secrets.GITHUB_TOKEN }}
```

## 5.7 本章小结

本章详细介绍了 GitHub 的核心功能，包括：

- 仓库管理：创建、设置、协作者
- Issue 管理：创建、标签、关闭
- Pull Request：创建、审查、合并
- GitHub Actions：工作流配置
- GitHub Pages：网站托管
- GitHub Projects：项目管理

**关键要点：**
- 熟练掌握这些功能是使用 GitHub 的基础
- 多实践，在实际项目中应用这些功能

**下一步：**
[协作与进阶 →](22-fork-contribute.md)
