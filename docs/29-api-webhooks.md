# GitHub API 与 Webhooks

## GitHub API

### REST API

```bash
# 获取仓库信息
curl -H "Authorization: token YOUR_TOKEN" \
  https://api.github.com/repos/user/repo

# 使用 gh 命令
gh api repos/user/repo
```

### GraphQL API

```graphql
query {
  repository(owner: "user", name: "repo") {
    name
    description
    stargazerCount
  }
}
```

```bash
# 使用 gh 命令
gh api graphql -f query='
{
  repository(owner: "user", name: "repo") {
    name
    description
    stargazerCount
  }
}'
```

### 常用 API 端点

| 端点 | 说明 |
|------|------|
| `/repos/{owner}/{repo}` | 仓库信息 |
| `/repos/{owner}/{repo}/issues` | Issue 列表 |
| `/repos/{owner}/{repo}/pulls` | PR 列表 |
| `/repos/{owner}/{repo}/releases` | Release 列表 |
| `/user` | 当前用户信息 |

## Personal Access Tokens

### 创建令牌
1. **Settings** → **Developer settings** → **Personal access tokens**
2. 点击 **Generate new token**
3. 选择权限范围

### 权限范围

| 范围 | 说明 |
|------|------|
| `repo` | 仓库访问 |
| `workflow` | GitHub Actions |
| `admin:repo_hook` | Webhook 管理 |
| `read:org` | 组织信息 |

## Webhooks

### 什么是 Webhook？

Webhook 允许在仓库发生事件时通知外部服务。

### 配置步骤
1. **Settings** → **Webhooks** → **Add webhook**
2. 设置 Payload URL
3. 选择事件
4. 保存

### 可用事件

| 事件 | 说明 |
|------|------|
| `push` | 推送代码 |
| `pull_request` | PR 变更 |
| `issues` | Issue 变更 |
| `release` | Release 发布 |
| `workflow_run` | Action 完成 |

### Payload 示例

```json
{
  "action": "opened",
  "pull_request": {
    "number": 42,
    "title": "New feature",
    "user": {
      "login": "username"
    }
  }
}
```

## 使用场景

### 自动化部署
```yaml
# 监听 push 事件触发部署
on:
  push:
    branches: [main]
```

### 通知集成
```bash
# 发送到 Slack
curl -X POST -H 'Content-type: application/json' \
  --data '{"text":"New PR: #42"}' \
  https://hooks.slack.com/services/xxx
```

### 自定义机器人
```python
# Flask Webhook 处理
from flask import Flask, request

@app.route('/webhook', methods=['POST'])
def webhook():
    payload = request.json
    if payload['action'] == 'opened':
        # 处理新 Issue
        pass
    return 'OK', 200
```

## GitHub Apps

### 什么是 GitHub App？

GitHub App 是与 GitHub API 交互的更现代方式。

### 优势
- 更细粒度的权限
- 更好的身份验证
- 安装在组织/仓库级别

### 创建 GitHub App
1. **Settings** → **Developer settings** → **GitHub Apps**
2. 点击 **New GitHub App**
3. 配置权限和事件

## 最佳实践

1. **使用最小权限**：只授予必要的 API 权限
2. **保护令牌**：不要硬编码在代码中
3. **验证 Webhook**：检查 X-Hub-Signature
4. **处理速率限制**：注意 API 调用限制

## 下一步

[常用命令速查表 →](A-common-commands.md)
