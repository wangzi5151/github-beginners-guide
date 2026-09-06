# GitHub App 开发指南

## 什么是 GitHub App？

GitHub App 是与 GitHub API 集成的应用程序，可以访问 GitHub 的各种功能。

## GitHub App vs OAuth App

| 特性 | GitHub App | OAuth App |
|------|-----------|-----------|
| 权限管理 | 精细化 | 笼统 |
| 安装方式 | 组织/用户安装 | 逐用户授权 |
| API 访问 | 以 App 身份 | 以用户身份 |
| 推荐 | ✅ | ❌ |

## 创建 GitHub App

### 1. 基本信息

```
1. 访问 https://github.com/settings/apps/new
2. 填写：
   - GitHub App name: your-app-name
   - Homepage URL: https://your-domain.com
   - Webhook URL: https://your-domain.com/webhook
   - Webhook secret: 生成并保存
```

### 2. 权限配置

```yaml
Repository permissions:
  - Contents: Read & Write
  - Issues: Read & Write
  - Pull requests: Read & Write
  - Actions: Read only

Organization permissions:
  - Members: Read only

Subscribe to events:
  - push
  - pull_request
  - issues
  - issue_comment
```

### 3. 安装 App

```bash
# 获取安装访问令牌
curl -X POST \
  -H "Authorization: Bearer $(jwt)" \
  -H "Accept: application/vnd.github.v3+json" \
  "https://api.github.com/app/installations/{installation_id}/access_tokens"
```

## 开发 GitHub App

### 使用 Octokit

```javascript
const { App } = require("@octokit/app");
const { Octokit } = require("@octokit/rest");

// 初始化 App
const app = new App({
  appId: process.env.APP_ID,
  privateKey: process.env.PRIVATE_KEY,
  webhooks: {
    secret: process.env.WEBHOOK_SECRET,
  },
});

// 处理 webhook
app.webhooks.on("push", async ({ payload }) => {
  console.log(`Push to ${payload.repository.name}`);
  console.log(`Branch: ${payload.ref}`);
  console.log(`Commits: ${payload.commits.length}`);
});

// 处理 pull_request
app.webhooks.on("pull_request.opened", async ({ payload }) => {
  const octokit = await app.getInstallationOctokit(
    payload.installation.id
  );
  
  // 添加评论
  await octokit.rest.issues.createComment({
    owner: payload.repository.owner.login,
    repo: payload.repository.name,
    issue_number: payload.pull_request.number,
    body: "感谢你的 PR！我们已经收到并会尽快审查。",
  });
});

// 启动服务器
const port = process.env.PORT || 3000;
app.webhooks.listen(port, () => {
  console.log(`Server listening on port ${port}`);
});
```

### 使用 Probot

```javascript
// index.js
const { Probot, ProbotOctokit } = require("probot");

module.exports = (app) => {
  app.on("issues.opened", async (context) => {
    const issueComment = context.issue({
      body: "感谢你提交 Issue！我们会尽快处理。",
    });
    await context.octokit.issues.createComment(issueComment);
  });

  app.on("pull_request.opened", async (context) => {
    // 自动添加标签
    await context.octokit.issues.addLabels(
      context.issue({
        labels: ["needs-review"],
      })
    );
  });
};
```

## 使用场景

### 1. 自动化代码审查

```javascript
app.webhooks.on("pull_request.opened", async ({ payload }) => {
  const octokit = await app.getInstallationOctokit(
    payload.installation.id
  );
  
  // 获取 PR 文件
  const files = await octokit.rest.pulls.listFiles({
    owner: payload.repository.owner.login,
    repo: payload.repository.name,
    pull_number: payload.pull_request.number,
  });
  
  // 检查文件大小
  const largeFiles = files.data.filter((file) => file.changes > 100);
  
  if (largeFiles.length > 0) {
    await octokit.rest.issues.createComment({
      owner: payload.repository.owner.login,
      repo: payload.repository.name,
      issue_number: payload.pull_request.number,
      body: `⚠️ 以下文件更改过多：\n${largeFiles
        .map((f) => `- ${f.filename}: ${f.changes} 处更改`)
        .join("\n")}`,
    });
  }
});
```

### 2. 自动标签管理

```javascript
app.webhooks.on("pull_request.opened", async ({ payload }) => {
  const octokit = await app.getInstallationOctokit(
    payload.installation.id
  );
  
  const labels = [];
  
  // 根据文件路径添加标签
  const files = await octokit.rest.pulls.listFiles({
    owner: payload.repository.owner.login,
    repo: payload.repository.name,
    pull_number: payload.pull_request.number,
  });
  
  const paths = files.data.map((f) => f.filename);
  
  if (paths.some((p) => p.startsWith("src/"))) {
    labels.push("source-code");
  }
  
  if (paths.some((p) => p.startsWith("docs/"))) {
    labels.push("documentation");
  }
  
  if (paths.some((p) => p.includes("test"))) {
    labels.push("tests");
  }
  
  if (labels.length > 0) {
    await octokit.rest.issues.addLabels({
      owner: payload.repository.owner.login,
      repo: payload.repository.name,
      issue_number: payload.pull_request.number,
      labels,
    });
  }
});
```

### 3. 部署通知

```javascript
app.webhooks.on("deployment", async ({ payload }) => {
  const octokit = await app.getInstallationOctokit(
    payload.installation.id
  );
  
  // 更新部署状态
  await octokit.rest.repos.createDeploymentStatus({
    owner: payload.repository.owner.login,
    repo: payload.repository.name,
    deployment_id: payload.deployment.id,
    state: "in_progress",
    description: "开始部署...",
  });
  
  // 执行部署逻辑
  try {
    await deploy(payload.deployment.ref);
    
    await octokit.rest.repos.createDeploymentStatus({
      owner: payload.repository.owner.login,
      repo: payload.repository.name,
      deployment_id: payload.deployment.id,
      state: "success",
      description: "部署成功！",
    });
  } catch (error) {
    await octokit.rest.repos.createDeploymentStatus({
      owner: payload.repository.owner.login,
      repo: payload.repository.name,
      deployment_id: payload.deployment.id,
      state: "failure",
      description: `部署失败: ${error.message}`,
    });
  }
});
```

## 部署

### 使用 Vercel

```json
{
  "version": 2,
  "builds": [
    {
      "src": "index.js",
      "use": "@vercel/node"
    }
  ],
  "routes": [
    {
      "src": "/webhook",
      "dest": "index.js"
    }
  ]
}
```

### 使用 Docker

```dockerfile
FROM node:20-slim

WORKDIR /app

COPY package*.json ./
RUN npm install --production

COPY . .

EXPOSE 3000

CMD ["node", "index.js"]
```

## 最佳实践

1. **最小权限**：只请求必要的权限
2. **验证签名**：验证 webhook 签名
3. **处理速率限制**：实现重试机制
4. **错误处理**：优雅处理错误
5. **日志记录**：记录重要操作

## 相关资源

- [GitHub App 文档](https://docs.github.com/en/apps/creating-github-apps)
- [Octokit 文档](https://octokit.github.io/)
- [Probot 文档](https://probot.github.io/)

---

**上一篇：[供应链安全](W13-supply-chain-security.md) | 下一篇：[DevOps 实战](W15-devops.md)**
