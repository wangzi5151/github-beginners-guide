# 第八章：GitHub 高级功能

## 8.1 GitHub Copilot

### 什么是 GitHub Copilot？

GitHub Copilot 是一个 AI 编程助手，可以帮助你编写代码、生成测试、编写文档等。

### 功能矩阵

| 功能 | Free | Pro | Business | Enterprise |
|------|------|-----|----------|------------|
| 代码补全 | ✅ 有限 | ✅ | ✅ | ✅ |
| Copilot Chat | ✅ 有限 | ✅ | ✅ | ✅ |
| Agent 模式 | ❌ | ✅ | ✅ | ✅ |
| Extensions | ❌ | ✅ | ✅ | ✅ |

### 使用方法

**安装扩展：**

1. 打开 VS Code
2. 点击扩展图标
3. 搜索 "GitHub Copilot"
4. 点击 **Install**

**基本使用：**

1. 编写注释或函数签名
2. Copilot 自动建议代码
3. 按 `Tab` 接受建议

**示例：**

```javascript
// 编写注释
// 计算两个数的和

// Copilot 会自动生成代码
function add(a, b) {
  return a + b;
}
```

### Copilot Chat

在 VS Code 中按 `Ctrl+I` 打开 Copilot Chat：

```
┌─────────────────────────────────────────────┐
│  Copilot Chat                                │
│                                             │
│  [输入问题或指令]                             │
│                                             │
│  例如：                                      │
│  - "解释这段代码的作用"                       │
│  - "帮我写一个单元测试"                       │
│  - "如何优化这段代码"                         │
│  - "修复这个 bug"                             │
└─────────────────────────────────────────────┘
```

### Agent 模式

Agent 模式可以执行多步骤任务：

```bash
# 让 Agent 创建完整的项目结构
"创建一个 Express API 项目，包含用户认证、数据库连接和 Docker 配置"

# 让 Agent 修复 CI/CD
"查看 GitHub Actions 失败日志并修复工作流"

# 让 Agent 重构代码
"将这个函数重构为更清晰的结构，添加错误处理"
```

### Copilot Extensions

Extensions 让 Copilot 可以调用外部服务：

```
┌─────────────────────────────────────────────┐
│  Copilot Extensions                          │
│                                             │
│  @github     - GitHub 平台操作               │
│  @terminal   - 终端命令辅助                  │
│  @workspace  - 项目工作区搜索                │
│  @docker     - Docker 配置辅助               │
│  @kubernetes - K8s YAML 生成                 │
└─────────────────────────────────────────────┘
```

## 8.2 GitHub Codespaces

### 什么是 Codespaces？

Codespaces 是 GitHub 提供的云端开发环境，让你可以在浏览器中直接编写代码。

### 启用 Codespaces

**第 1 步：** 打开仓库页面

**第 2 步：** 点击 **Code** 按钮

**第 3 步：** 选择 **Codespaces** 标签

**第 4 步：** 点击 **Create codespace on main**

```
┌─────────────────────────────────────────────┐
│  Code                                        │
│                                             │
│  Local    Codespaces                         │
│                                             │
│  ┌─────────────────────────────────────┐    │
│  │ Create codespace on main            │    │
│  │                                     │    │
│  │ 2-core • 8 GB RAM • 15 GB          │    │
│  │ Free for 120 hours/month            │    │
│  └─────────────────────────────────────┘    │
└─────────────────────────────────────────────┘
```

### 配置 Codespaces

创建 `.devcontainer/devcontainer.json`：

```json
{
  "name": "My Project",
  "image": "mcr.microsoft.com/devcontainers/javascript-node:20",
  "forwardPorts": [3000],
  "postCreateCommand": "npm install",
  "customizations": {
    "vscode": {
      "extensions": [
        "dbaeumer.vscode-eslint",
        "esbenp.prettier-vscode"
      ]
    }
  }
}
```

### 使用 Codespaces

1. 浏览器中打开 Codespace
2. 像本地 VS Code 一样使用
3. 自动同步代码
4. 支持终端、调试、扩展

## 8.3 GitHub Models

### 什么是 GitHub Models？

GitHub Models 是 GitHub 提供的 AI 模型平台，让你可以直接在 GitHub 上使用各种 AI 模型。

### 支持的模型

| 模型 | 提供商 | 用途 |
|------|--------|------|
| GPT-4o | OpenAI | 通用对话、代码生成 |
| GPT-4o-mini | OpenAI | 轻量级任务 |
| Claude 3.5 Sonnet | Anthropic | 代码分析、对话 |
| Llama 3.1 | Meta | 开源通用模型 |
| Mistral | Mistral AI | 欧洲开源模型 |

### 使用方法

**通过 API 使用：**

```bash
# 设置 API Key
export GITHUB_TOKEN="your-github-token"

# 调用 GPT-4o
curl -X POST "https://models.inference.ai.azure.com/chat/completions" \
  -H "Authorization: Bearer $GITHUB_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-4o",
    "messages": [
      {"role": "user", "content": "Hello, who are you?"}
    ]
  }'
```

**使用 Python SDK：**

```python
from openai import OpenAI

client = OpenAI(
    base_url="https://models.inference.ai.azure.com",
    api_key=os.environ["GITHUB_TOKEN"],
)

response = client.chat.completions.create(
    model="gpt-4o",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "Write a Python function to sort a list"}
    ]
)

print(response.choices[0].message.content)
```

### 在 GitHub Actions 中使用

```yaml
name: AI Code Review

on:
  pull_request:

jobs:
  ai-review:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    
    - name: Get PR diff
      id: diff
      run: |
        DIFF=$(gh pr diff ${{ github.event.pull_request.number }})
        echo "diff<<EOF" >> $GITHUB_OUTPUT
        echo "$DIFF" >> $GITHUB_OUTPUT
        echo "EOF" >> $GITHUB_OUTPUT
    
    - name: AI Review
      run: |
        curl -X POST "https://models.inference.ai.azure.com/chat/completions" \
          -H "Authorization: Bearer ${{ secrets.GITHUB_TOKEN }}" \
          -H "Content-Type: application/json" \
          -d '{
            "model": "gpt-4o",
            "messages": [
              {"role": "system", "content": "Review this code change and provide feedback."},
              {"role": "user", "content": "${{ steps.diff.outputs.diff }}"}
            ]
          }'
```

## 8.4 GitHub Advanced Security

### Secret Scanning Push Protection

在推送时阻止包含敏感信息的提交：

1. 进入 **Settings** → **Code security and analysis**
2. 启用 **Push protection**

### Code Scanning

使用 CodeQL 分析代码中的安全漏洞：

1. 进入 **Security** → **Code scanning**
2. 点击 **Set up**
3. 选择 CodeQL
4. 配置扫描选项

### Dependency Review

在 PR 中审查依赖的安全漏洞：

```yaml
# .github/workflows/dependency-review.yml
name: Dependency Review

on:
  pull_request:

permissions:
  contents: read
  pull-requests: write

jobs:
  dependency-review:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    
    - name: Dependency Review
      uses: actions/dependency-review-action@v4
      with:
        fail-on-severity: high
```

## 8.5 GitHub API

### REST API

```bash
# 获取仓库信息
curl -H "Authorization: token $GITHUB_TOKEN" \
  "https://api.github.com/repos/owner/repo"

# 创建 Issue
curl -X POST \
  -H "Authorization: token $GITHUB_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"title": "Bug report", "body": "Description"}' \
  "https://api.github.com/repos/owner/repo/issues"
```

### GraphQL API

```graphql
query {
  repository(owner: "your-org", name: "your-repo") {
    issues(first: 10) {
      edges {
        node {
          title
          state
          author {
            login
          }
        }
      }
    }
  }
}
```

### 使用 GitHub CLI

```bash
# 列出仓库
gh repo list

# 创建 Issue
gh issue create --title "Bug" --body "Description"

# 创建 PR
gh pr create --title "Feature" --body "Description"

# 查看 PR
gh pr view 123
```

## 8.6 GitHub Webhooks

### 什么是 Webhooks？

Webhooks 允许你在 GitHub 上发生事件时，自动向外部服务发送通知。

### 创建 Webhook

1. 进入仓库 **Settings** → **Webhooks**
2. 点击 **Add webhook**
3. 配置：

```
┌─────────────────────────────────────────────┐
│  Add webhook                                 │
│                                             │
│  Payload URL: [https://your-server.com/webhook]│
│  Content type: [application/json ▼]          │
│  Secret: [your-secret-token     ]            │
│                                             │
│  Events:                                     │
│  ● Just the push event                      │
│  ○ Send me everything                       │
│  ○ Let me select individual events          │
│                                             │
│       [Add webhook]                          │
└─────────────────────────────────────────────┘
```

### 处理 Webhook

```javascript
const crypto = require('crypto');

// 验证 Webhook 签名
function verifyWebhook(payload, signature, secret) {
  const hmac = crypto.createHmac('sha256', secret);
  const digest = hmac.update(payload).digest('hex');
  return crypto.timingSafeEqual(
    Buffer.from(signature),
    Buffer.from(`sha256=${digest}`)
  );
}

// 处理 Webhook
app.post('/webhook', (req, res) => {
  const signature = req.headers['x-hub-signature-256'];
  
  if (!verifyWebhook(JSON.stringify(req.body), signature, process.env.WEBHOOK_SECRET)) {
    return res.status(401).send('Invalid signature');
  }
  
  const event = req.headers['x-github-event'];
  const payload = req.body;
  
  switch (event) {
    case 'push':
      handlePush(payload);
      break;
    case 'pull_request':
      handlePullRequest(payload);
      break;
  }
  
  res.status(200).send('OK');
});
```

## 8.7 GitHub Projects

### 什么是 Projects？

GitHub Projects 是一个项目管理工具，提供看板、表格、路线图等视图。

### 创建项目

```bash
# 使用 CLI 创建项目
gh project create --title "My Project" --owner your-org
```

### 项目视图

| 视图 | 说明 |
|------|------|
| **Board** | 看板视图，类似 Trello |
| **Table** | 表格视图，类似 Excel |
| **Roadmap** | 路线图视图，时间线 |
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

## 8.8 GitHub Packages

### 什么是 Packages？

GitHub Packages 是一个包管理服务，可以发布和管理包。

### 支持的包管理器

| 包管理器 | 语言 |
|----------|------|
| npm | JavaScript |
| NuGet | .NET |
| RubyGems | Ruby |
| Maven | Java |
| Docker | 容器 |

### 发布 npm 包

**第 1 步：** 配置 `.npmrc`

```
registry=https://npm.pkg.github.com
```

**第 2 步：** 发布

```bash
npm publish
```

### 使用 GitHub Actions 发布

```yaml
name: Publish Package

on:
  push:
    tags: ['v*']

jobs:
  publish:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    
    steps:
    - uses: actions/checkout@v4
    
    - uses: actions/setup-node@v4
      with:
        node-version: '20'
        registry-url: 'https://npm.pkg.github.com'
    
    - run: npm ci
    - run: npm publish
      env:
        NODE_AUTH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

## 8.9 本章小结

本章详细介绍了 GitHub 的高级功能，包括：

- GitHub Copilot：AI 编程助手
- GitHub Codespaces：云端开发环境
- GitHub Models：AI 模型平台
- GitHub Advanced Security：安全功能
- GitHub API：自动化接口
- GitHub Webhooks：事件通知
- GitHub Projects：项目管理
- GitHub Packages：包管理

**关键要点：**
- 这些高级功能可以大大提高开发效率
- 根据项目需求选择合适的功能
- 持续学习新功能

**下一步：**
[中国开发者专区 →](Q-china-acceleration.md)
