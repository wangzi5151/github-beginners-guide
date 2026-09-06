# GitHub API 使用指南

## 什么是 GitHub API？

GitHub API 允许你通过编程方式与 GitHub 交互，可以用来自动化操作、获取数据、集成到其他系统中。

## API 类型

| 类型 | 说明 | 端点 |
|------|------|------|
| REST API | 传统 RESTful API | `https://api.github.com` |
| GraphQL API | 灵活的查询语言 | `https://api.github.com/graphql` |

## 认证方式

### 1. 个人访问令牌 (PAT)

```bash
# 使用 curl
curl -H "Authorization: token YOUR_TOKEN" \
  https://api.github.com/user

# 使用 gh 命令
gh api user
```

### 2. OAuth

```javascript
// OAuth 认证流程
// 1. 重定向到 GitHub 授权页面
// 2. 用户授权
// 3. 获取 access token
// 4. 使用 token 调用 API
```

### 3. GitHub App

```javascript
// 使用 GitHub App 认证
// 1. 生成 JWT
// 2. 获取 installation token
// 3. 使用 token 调用 API
```

## REST API 示例

### 获取用户信息

```bash
# 使用 curl
curl https://api.github.com/users/octocat

# 使用 gh
gh api users/octocat
```

### 获取仓库信息

```bash
# 获取仓库
curl https://api.github.com/repos/octocat/Hello-World

# 使用 gh
gh api repos/octocat/Hello-World
```

### 创建 Issue

```bash
curl -X POST \
  -H "Authorization: token YOUR_TOKEN" \
  -H "Accept: application/vnd.github.v3+json" \
  https://api.github.com/repos/OWNER/REPO/issues \
  -d '{"title":"Bug report","body":"Description of the bug"}'
```

### 使用 gh 命令

```bash
# 创建 Issue
gh api repos/OWNER/REPO/issues \
  -f title='Bug report' \
  -f body='Description of the bug'

# 创建 PR
gh api repos/OWNER/REPO/pulls \
  -f title='New feature' \
  -f head='feature-branch' \
  -f base='main'
```

## GraphQL API 示例

### 查询仓库信息

```graphql
query {
  repository(owner: "octocat", name: "Hello-World") {
    name
    description
    stargazerCount
    issues(first: 5) {
      edges {
        node {
          title
          state
        }
      }
    }
  }
}
```

### 使用 gh 命令

```bash
gh api graphql -f query='
{
  repository(owner: "octocat", name: "Hello-World") {
    name
    description
    stargazerCount
  }
}'
```

## 常用 API 端点

### 用户相关

| 端点 | 说明 |
|------|------|
| `GET /user` | 获取当前用户 |
| `GET /users/{username}` | 获取用户信息 |
| `GET /user/repos` | 获取用户仓库 |

### 仓库相关

| 端点 | 说明 |
|------|------|
| `GET /repos/{owner}/{repo}` | 获取仓库信息 |
| `POST /repos/{owner}/{repo}/issues` | 创建 Issue |
| `GET /repos/{owner}/{repo}/pulls` | 获取 PR 列表 |
| `POST /repos/{owner}/{repo}/pulls` | 创建 PR |

### 组织相关

| 端点 | 说明 |
|------|------|
| `GET /orgs/{org}` | 获取组织信息 |
| `GET /orgs/{org}/repos` | 获取组织仓库 |
| `GET /orgs/{org}/members` | 获取组织成员 |

## 速率限制

### 认证请求

- **每小时 5,000 次请求**

### 未认证请求

- **每小时 60 次请求**

### 检查速率限制

```bash
# 使用 curl
curl -I https://api.github.com/users/octocat | grep -i 'x-ratelimit'

# 使用 gh
gh api rate_limit
```

## 错误处理

### 常见错误码

| 错误码 | 说明 |
|--------|------|
| 401 | 未授权 |
| 403 | 禁止访问 |
| 404 | 未找到 |
| 422 | 请求无效 |
| 403 | 速率限制 |

### 错误响应示例

```json
{
  "message": "Not Found",
  "documentation_url": "https://docs.github.com/rest"
}
```

## 使用场景

### 1. 自动化工作流

```yaml
# GitHub Actions 中使用 API
- name: Create Issue
  run: |
    gh api repos/${{ github.repository }}/issues \
      -f title='Build failed' \
      -f body='Build ${{ github.run_id }} failed'
```

### 2. 数据分析

```python
import requests

# 获取仓库统计
response = requests.get(
    'https://api.github.com/repos/octocat/Hello-World',
    headers={'Authorization': 'token YOUR_TOKEN'}
)

data = response.json()
print(f"Stars: {data['stargazers_count']}")
```

### 3. 集成到其他系统

```javascript
// 集成到 Slack
const { WebClient } = require('@slack/web-api');
const github = require('@octokit/rest');

// 当有新 PR 时通知 Slack
```

## 最佳实践

1. **使用认证请求**：获得更高的速率限制
2. **缓存响应**：减少 API 调用
3. **处理速率限制**：实现退避重试
4. **使用分页**：获取大量数据时使用分页
5. **验证输入**：确保请求数据有效
6. **错误处理**：妥善处理 API 错误

## 相关资源

- [GitHub REST API 文档](https://docs.github.com/en/rest)
- [GitHub GraphQL API 文档](https://docs.github.com/en/graphql)
- [GitHub API 探索工具](https://docs.github.com/en/rest/overview/api-versions)
