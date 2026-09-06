# GitHub 项目实战案例

## 案例 1：个人博客

### 目标
使用 GitHub Pages 搭建个人博客。

### 技术栈
- HTML/CSS/JavaScript
- GitHub Pages
- Jekyll（可选）

### 步骤

#### 1. 创建仓库
```bash
gh repo create my-blog --public --description "我的个人博客"
```

#### 2. 创建基本结构
```
my-blog/
├── index.html
├── css/
│   └── style.css
├── js/
│   └── main.js
├── posts/
│   └── 2024-01-01-hello.md
└── images/
```

#### 3. 编写首页
```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <title>我的博客</title>
    <link rel="stylesheet" href="css/style.css">
</head>
<body>
    <header>
        <h1>我的博客</h1>
        <nav>
            <a href="/">首页</a>
            <a href="/posts">文章</a>
            <a href="/about">关于</a>
        </nav>
    </header>
    <main>
        <article>
            <h2>欢迎来到我的博客</h2>
            <p>这是我的第一篇文章。</p>
        </article>
    </main>
</body>
</html>
```

#### 4. 启用 GitHub Pages
1. 进入仓库 Settings → Pages
2. 选择 main 分支
3. 点击 Save

#### 5. 访问网站
```
https://你的用户名.github.io/my-blog/
```

---

## 案例 2：开源工具库

### 目标
创建一个可复用的 JavaScript 工具库。

### 步骤

#### 1. 初始化项目
```bash
mkdir my-utils
cd my-utils
git init
npm init -y
```

#### 2. 创建目录结构
```
my-utils/
├── src/
│   ├── index.js
│   ├── string.js
│   └── array.js
├── test/
│   ├── string.test.js
│   └── array.test.js
├── package.json
├── README.md
├── LICENSE
└── .gitignore
```

#### 3. 编写工具函数
```javascript
// src/string.js
export const capitalize = (str) => {
  return str.charAt(0).toUpperCase() + str.slice(1);
};

export const camelCase = (str) => {
  return str.replace(/-([a-z])/g, (g) => g[1].toUpperCase());
};
```

#### 4. 编写测试
```javascript
// test/string.test.js
import { capitalize, camelCase } from '../src/string.js';

describe('String Utils', () => {
  test('capitalize', () => {
    expect(capitalize('hello')).toBe('Hello');
  });
  
  test('camelCase', () => {
    expect(camelCase('hello-world')).toBe('helloWorld');
  });
});
```

#### 5. 发布到 npm
```bash
npm login
npm publish
```

---

## 案例 3：团队协作项目

### 目标
创建一个适合团队协作的项目模板。

### 步骤

#### 1. 创建仓库并配置
```bash
gh repo create team-project --private --description "团队项目"
```

#### 2. 设置分支保护
1. Settings → Branches → Add rule
2. 配置：
   - Branch name pattern: `main`
   - ✅ Require pull request reviews
   - ✅ Require status checks

#### 3. 创建项目结构
```
team-project/
├── .github/
│   ├── ISSUE_TEMPLATE/
│   │   ├── bug_report.md
│   │   └── feature_request.md
│   ├── PULL_REQUEST_TEMPLATE.md
│   ├── CODEOWNERS
│   └── workflows/
│       └── ci.yml
├── src/
├── docs/
├── tests/
├── README.md
├── CONTRIBUTING.md
└── LICENSE
```

#### 4. 配置 CI/CD
```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - run: npm install
    - run: npm test
```

#### 5. 创建 Issue 模板
```markdown
<!-- .github/ISSUE_TEMPLATE/bug_report.md -->
---
name: Bug Report
about: 报告一个问题
labels: bug
---

## 描述
简要描述问题

## 复现步骤
1. 
2. 
3. 

## 期望行为
描述期望的行为

## 实际行为
描述实际的行为

## 环境
- OS: 
- Browser: 
- Version: 
```

---

## 案例 4：GitHub Actions 自动化

### 目标
创建一个自动发布 npm 包的工作流。

### 步骤

#### 1. 创建工作流文件
```yaml
# .github/workflows/release.yml
name: Release

on:
  push:
    tags:
      - 'v*'

jobs:
  release:
    runs-on: ubuntu-latest
    
    permissions:
      contents: read
      packages: write
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '20'
        registry-url: 'https://npm.pkg.github.com'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Build
      run: npm run build
    
    - name: Test
      run: npm test
    
    - name: Publish to GitHub Packages
      run: npm publish
      env:
        NODE_AUTH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    
    - name: Create Release
      uses: softprops/action-gh-release@v2
      with:
        generate_release_notes: true
```

#### 2. 使用工作流
```bash
# 创建标签
git tag -a v1.0.0 -m "Release v1.0.0"
git push origin v1.0.0
```

---

## 案例 5：全栈应用

### 目标
使用 GitHub 管理全栈应用开发。

### 技术栈
- 前端：React
- 后端：Node.js
- 数据库：MongoDB

### 步骤

#### 1. 创建仓库结构
```
fullstack-app/
├── client/          # 前端代码
│   ├── src/
│   └── package.json
├── server/          # 后端代码
│   ├── src/
│   └── package.json
├── .github/
│   └── workflows/
│       ├── frontend.yml
│       └── backend.yml
├── docker-compose.yml
├── README.md
└── .gitignore
```

#### 2. 配置前端 CI
```yaml
# .github/workflows/frontend.yml
name: Frontend CI

on:
  push:
    paths:
      - 'client/**'

jobs:
  test:
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: client
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '20'
    
    - run: npm ci
    - run: npm test
    - run: npm run build
```

#### 3. 配置后端 CI
```yaml
# .github/workflows/backend.yml
name: Backend CI

on:
  push:
    paths:
      - 'server/**'

jobs:
  test:
    runs-on: ubuntu-latest
    
    services:
      mongodb:
        image: mongo:5.0
        ports:
          - 27017:27017
    
    defaults:
      run:
        working-directory: server
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '20'
    
    - run: npm ci
    - run: npm test
    
    env:
      MONGODB_URI: mongodb://localhost:27017/test
```

---

## 总结

这些案例展示了 GitHub 在不同场景下的应用：

1. **个人博客**：GitHub Pages
2. **开源工具**：npm 发布
3. **团队协作**：分支保护、Issue 模板
4. **自动化**：GitHub Actions
5. **全栈应用**：monorepo 管理

根据你的需求选择合适的方案，逐步深入学习。
