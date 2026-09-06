# GitHub REST/GraphQL API 实战

## REST API 高级用法

### 分页处理

```javascript
async function getAllIssues(owner, repo) {
  const issues = [];
  let page = 1;
  
  while (true) {
    const response = await fetch(
      `https://api.github.com/repos/${owner}/${repo}/issues?page=${page}&per_page=100`,
      {
        headers: {
          Authorization: `token ${process.env.GITHUB_TOKEN}`,
          Accept: 'application/vnd.github.v3+json',
        },
      }
    );
    
    const data = await response.json();
    if (data.length === 0) break;
    
    issues.push(...data);
    page++;
  }
  
  return issues;
}
```

### 速率限制处理

```javascript
async function makeRequest(url) {
  const response = await fetch(url, {
    headers: {
      Authorization: `token ${process.env.GITHUB_TOKEN}`,
    },
  });
  
  // 检查速率限制
  const remaining = response.headers.get('x-ratelimit-remaining');
  const reset = response.headers.get('x-ratelimit-reset');
  
  if (remaining === '0') {
    const resetTime = new Date(reset * 1000);
    const waitTime = resetTime - new Date();
    console.log(`Rate limit exceeded. Waiting ${waitTime}ms...`);
    await new Promise(resolve => setTimeout(resolve, waitTime));
    return makeRequest(url);
  }
  
  return response.json();
}
```

### 批量操作

```javascript
// 批量添加标签
async function addLabels(owner, repo, issueNumber, labels) {
  return fetch(
    `https://api.github.com/repos/${owner}/${repo}/issues/${issueNumber}/labels`,
    {
      method: 'POST',
      headers: {
        Authorization: `token ${process.env.GITHUB_TOKEN}`,
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({ labels }),
    }
  );
}

// 批量关闭 issues
async function closeIssues(owner, repo, issueNumbers) {
  const promises = issueNumbers.map(number =>
    fetch(
      `https://api.github.com/repos/${owner}/${repo}/issues/${number}`,
      {
        method: 'PATCH',
        headers: {
          Authorization: `token ${process.env.GITHUB_TOKEN}`,
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({ state: 'closed' }),
      }
    )
  );
  
  return Promise.all(promises);
}
```

## GraphQL API

### 基础查询

```graphql
query {
  repository(owner: "your-org", name: "your-repo") {
    issues(first: 10, states: OPEN) {
      edges {
        node {
          title
          url
          createdAt
          author {
            login
          }
          labels(first: 5) {
            edges {
              node {
                name
              }
            }
          }
        }
      }
    }
  }
}
```

### 变更操作

```graphql
mutation {
  addComment(input: {
    subjectId: "I_xxx",
    body: "Thank you for the report!"
  }) {
    commentEdge {
      node {
        body
        createdAt
      }
    }
  }
}
```

### 使用 JavaScript

```javascript
async function graphqlQuery(query, variables = {}) {
  const response = await fetch('https://api.github.com/graphql', {
    method: 'POST',
    headers: {
      Authorization: `bearer ${process.env.GITHUB_TOKEN}`,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({ query, variables }),
  });
  
  return response.json();
}

// 使用示例
const query = `
  query ($owner: String!, $name: String!) {
    repository(owner: $owner, name: $name) {
      stargazerCount
      forkCount
      description
    }
  }
`;

const result = await graphqlQuery(query, {
  owner: 'your-org',
  name: 'your-repo',
});

console.log(result.data.repository);
```

## GitHub CLI API

```bash
# REST API
gh api repos/your-org/your-repo/issues --paginate

# GraphQL API
gh api graphql -f query='
  query {
    viewer {
      login
      repositories(first: 10) {
        nodes {
          name
        }
      }
    }
  }
'

# 创建 issue
gh api repos/your-org/your-repo/issues \
  --method POST \
  -f title="Bug report" \
  -f body="Description of the bug" \
  -f labels='["bug","urgent"]'
```

## Webhooks

### 创建 Webhook

```javascript
const crypto = require('crypto');

// 验证 webhook 签名
function verifyWebhook(payload, signature, secret) {
  const hmac = crypto.createHmac('sha256', secret);
  const digest = hmac.update(payload).digest('hex');
  return crypto.timingSafeEqual(
    Buffer.from(signature),
    Buffer.from(`sha256=${digest}`)
  );
}

// 处理 webhook
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
    case 'issues':
      handleIssues(payload);
      break;
  }
  
  res.status(200).send('OK');
});
```

### GitHub Action 接收 Webhook

```yaml
# .github/workflows/webhook.yml
name: Webhook Handler

on:
  repository_dispatch:
    types: [custom-event]

jobs:
  handle:
    runs-on: ubuntu-latest
    steps:
    - name: Handle webhook
      run: |
        echo "Event type: ${{ github.event.client_payload.event_type }}"
        echo "Data: ${{ github.event.client_payload.data }}"
```

## 最佳实践

1. **使用 GraphQL**：减少 API 调用次数
2. **处理速率限制**：实现重试和等待机制
3. **验证 Webhook**：确保请求来自 GitHub
4. **使用 GitHub CLI**：简化 API 调用
5. **缓存响应**：减少不必要的 API 调用

## 相关资源

- [REST API 文档](https://docs.github.com/en/rest)
- [GraphQL API 文档](https://docs.github.com/en/graphql)
- [Webhooks 文档](https://docs.github.com/en/webhooks)

---

**上一篇：[Feature Flags 功能开关](W20-feature-flags.md) | 下一篇：[开源项目商业化运营](W22-open-source-business.md)**
