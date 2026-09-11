# 第七章：安全与 DevOps 实践

## 7.1 GitHub 安全基础

### 账号安全

**两步验证（2FA）：**

1. 进入 **Settings** → **Password and authentication**
2. 点击 **Enable two-factor authentication**
3. 选择验证方式：
   - Authenticator App（推荐）
   - SMS/voice message
   - Security key
4. 保存恢复代码

**个人访问令牌（PAT）：**

1. 进入 **Settings** → **Developer settings** → **Personal access tokens**
2. 点击 **Generate new token**
3. 选择权限范围
4. 保存生成的令牌

### 仓库安全

**分支保护规则：**

1. 进入 **Settings** → **Branches**
2. 点击 **Add rule**
3. 配置保护规则：

```
┌─────────────────────────────────────────────┐
│  Branch protection rules                     │
│                                             │
│  Branch name pattern: [main           ]      │
│                                             │
│  ☑ Require a pull request before merging    │
│    ☑ Required number of approvals: [1  ▼]   │
│    ☑ Dismiss stale pull request approvals   │
│    ☑ Require review from Code Owners        │
│                                             │
│  ☑ Require status checks to pass            │
│    ☑ Require branches to be up to date      │
│                                             │
│  ☑ Do not allow bypassing the above settings│
│                                             │
│             [Create]                         │
└─────────────────────────────────────────────┘
```

**CODEOWNERS：**

创建 `.github/CODEOWNERS` 文件：

```yaml
# 默认所有者
* @your-org/core-team

# 前端代码
/src/components/ @your-org/frontend-team

# 后端代码
/src/api/ @your-org/backend-team

# 基础设施
/terraform/ @your-org/devops-team
```

## 7.2 Secret Scanning

### 什么是 Secret Scanning？

GitHub 自动扫描仓库中的敏感信息（如 API 密钥、密码等），并发出警告。

### 启用 Secret Scanning

1. 进入 **Settings** → **Code security and analysis**
2. 启用 **Secret scanning**
3. 启用 **Push protection**

```
┌─────────────────────────────────────────────┐
│  Code security and analysis                  │
│                                             │
│  Secret scanning                             │
│  ○ Disable  ● Enable  ← 启用                │
│                                             │
│  Push protection                             │
│  ○ Disable  ● Enable  ← 启用                │
│                                             │
└─────────────────────────────────────────────┘
```

### 自定义 Secret 模式

创建 `.github/secret-scanning.yml`：

```yaml
custom-patterns:
  - name: Internal API Key
    pattern: 'internal-api-key-[a-zA-Z0-9]{32}'
    description: Internal API keys
```

## 7.3 Dependabot

### 什么是 Dependabot？

Dependabot 自动检查项目依赖的安全漏洞，并提供更新建议。

### 启用 Dependabot

1. 进入 **Settings** → **Code security and analysis**
2. 启用 **Dependabot alerts**
3. 启用 **Dependabot security updates**

### 配置 Dependabot

创建 `.github/dependabot.yml`：

```yaml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
    open-pull-requests-limit: 10
    labels:
      - "dependencies"
      - "automated"
  
  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "weekly"
```

### 自动合并安全更新

```yaml
# .github/workflows/auto-merge.yml
name: Auto Merge Dependabot

on:
  pull_request:

permissions:
  contents: write
  pull-requests: write

jobs:
  auto-merge:
    if: github.actor == 'dependabot[bot]'
    runs-on: ubuntu-latest
    
    steps:
    - name: Fetch Dependabot metadata
      id: metadata
      uses: dependabot/fetch-metadata@v1
      with:
        github-token: "${{ secrets.GITHUB_TOKEN }}"
    
    - name: Auto-merge minor and patch updates
      if: steps.metadata.outputs.update-type != 'version-update:semver-major'
      run: gh pr merge --auto --squash "$PR_URL"
      env:
        PR_URL: ${{github.event.pull_request.html_url}}
        GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

## 7.4 Code Scanning

### 什么是 Code Scanning？

Code Scanning 使用 CodeQL 分析代码中的安全漏洞。

### 配置 Code Scanning

**第 1 步：** 进入 **Security** → **Code scanning**

**第 2 步：** 点击 **Set up**

**第 3 步：** 选择 CodeQL

```
┌─────────────────────────────────────────────┐
│  Set up code scanning                        │
│                                             │
│  Select a tool:                              │
│                                             │
│  ● CodeQL  ← 推荐                            │
│  ○ Third-party tools                         │
│                                             │
│  Select languages:                           │
│  ☑ JavaScript/TypeScript                     │
│  ☑ Python                                   │
│  ☑ Java/Kotlin                               │
│                                             │
│             [Set up CodeQL]                 │
└─────────────────────────────────────────────┘
```

### 创建工作流

创建 `.github/workflows/codeql.yml`：

```yaml
name: CodeQL

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 6 * * 1'

jobs:
  analyze:
    runs-on: ubuntu-latest
    permissions:
      security-events: write
    
    strategy:
      fail-fast: false
      matrix:
        language: ['javascript', 'python']
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Initialize CodeQL
      uses: github/codeql-action/init@v3
      with:
        languages: ${{ matrix.language }}
    
    - name: Autobuild
      uses: github/codeql-action/autobuild@v3
    
    - name: Perform CodeQL Analysis
      uses: github/codeql-action/analyze@v3
```

## 7.5 CI/CD 基础

### 什么是 CI/CD？

- **CI（Continuous Integration）**：持续集成，频繁地将代码集成到主干
- **CD（Continuous Deployment）**：持续部署，自动将代码部署到生产环境

### GitHub Actions 基础

**工作流文件结构：**

```
.github/
└── workflows/
    └── ci.yml
```

**基本模板：**

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

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
    
    - name: Build
      run: npm run build
```

### Secrets 管理

**添加 Secrets：**

1. 进入 **Settings** → **Secrets and variables** → **Actions**
2. 点击 **New repository secret**
3. 输入名称和值

```
┌─────────────────────────────────────────────┐
│  Secrets and variables / Actions             │
│                                             │
│  Repository secrets                          │
│                                             │
│  Name: [API_KEY            ]                 │
│  Value: [••••••••••••       ]                │
│                                             │
│           [Add secret]                      │
└─────────────────────────────────────────────┘
```

**使用 Secrets：**

```yaml
- name: Deploy
  env:
    API_KEY: ${{ secrets.API_KEY }}
  run: ./deploy.sh
```

## 7.6 部署实践

### 部署到 GitHub Pages

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [main]

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

### 部署到 Vercel

```yaml
name: Deploy to Vercel

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Deploy to Vercel
      uses: amondnet/vercel-action@v25
      with:
        vercel-token: ${{ secrets.VERCEL_TOKEN }}
        vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
        vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
```

### 部署到 Docker

```yaml
name: Docker Build and Push

on:
  push:
    tags: ['v*']

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Login to Docker Hub
      uses: docker/login-action@v3
      with:
        username: ${{ secrets.DOCKERHUB_USERNAME }}
        password: ${{ secrets.DOCKERHUB_TOKEN }}
    
    - name: Build and push
      uses: docker/build-push-action@v5
      with:
        context: .
        push: true
        tags: ${{ secrets.DOCKERHUB_USERNAME }}/my-app:latest
```

## 7.7 监控和告警

### 监控工作流

```yaml
name: Monitor

on:
  schedule:
    - cron: '0 * * * *'  # 每小时

jobs:
  health-check:
    runs-on: ubuntu-latest
    
    steps:
    - name: Health check
      run: |
        STATUS=$(curl -s -o /dev/null -w "%{http_code}" https://api.example.com/health)
        if [ "$STATUS" != "200" ]; then
          echo "Service is down!"
          exit 1
        fi
```

### 告警通知

```yaml
- name: Notify on failure
  if: failure()
  uses: slackapi/slack-github-action@v1
  with:
    payload: |
      {
        "text": "Build failed: ${{ github.repository }}"
      }
  env:
    SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
```

## 7.8 本章小结

本章详细介绍了 GitHub 的安全和 DevOps 实践，包括：

- 账号和仓库安全
- Secret Scanning
- Dependabot
- Code Scanning
- CI/CD 基础
- 部署实践
- 监控和告警

**关键要点：**
- 安全是开发的重要环节
- CI/CD 可以提高开发效率
- 自动化是 DevOps 的核心

**下一步：**
[实战练习 →](exercises/exercise-1-create-repo.md)
