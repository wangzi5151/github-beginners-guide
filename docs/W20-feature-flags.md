# Feature Flags 功能开关

## 什么是 Feature Flags？

Feature Flags 是一种在运行时控制功能开启/关闭的技术，允许你渐进式地发布新功能。

## 常见工具

| 工具 | 特点 | 免费额度 |
|------|------|----------|
| LaunchDarkly | 功能强大 | 无 |
| Split.io | 实验驱动 | 有 |
| Flagsmith | 开源自托管 | 无限 |
| Unleash | 开源 | 无限 |
| GitHub Feature Flags | GitHub 原生 | 有 |

## 使用 Flagsmith

### 配置

```yaml
# .github/workflows/feature-flags.yml
name: Feature Flags

on:
  push:
    branches: [main]

jobs:
  update-flags:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    
    - name: Update feature flags
      run: |
        curl -X POST https://api.flagsmith.com/api/v1/flags/ \
          -H "Authorization: Bearer ${{ secrets.FLAGSMITH_TOKEN }}" \
          -H "Content-Type: application/json" \
          -d '{
            "feature": {
              "name": "new_dashboard",
              "description": "New dashboard UI"
            },
            "default_value": "false"
          }'
```

### 代码集成

```javascript
// Node.js
const flagsmith = require('flagsmith-node');

const flagsmithClient = new flagsmith.Flagsmith({
  environmentKey: 'your-env-key',
  identity: 'user-123',
});

// 检查功能开关
const showNewDashboard = await flagsmithClient.getValue('new_dashboard', false);

if (showNewDashboard) {
  // 显示新仪表板
  renderNewDashboard();
} else {
  // 显示旧仪表板
  renderOldDashboard();
}
```

## 使用 LaunchDarkly

### 配置

```javascript
// 初始化
const ldClient = require('launchdarkly-node-server-sdk');

const client = ldClient.init('your-sdk-key');

await client.waitForInitialization();

// 检查功能开关
const showNewFeature = await client.variation('new-feature-flag', user, false);

if (showNewFeature) {
  // 显示新功能
}
```

## 使用 GitHub Feature Flags

```yaml
# .github/workflows/feature-flag.yml
name: Feature Flag

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    
    - name: Deploy with feature flag
      run: |
        # 根据 feature flag 决定部署策略
        if [[ "${{ github.event.head_commit.message }}" == *"[canary]"* ]]; then
          echo "Deploying canary..."
          # 金丝雀部署
        else
          echo "Deploying full..."
          # 完整部署
        fi
```

## 渐进式发布

### 金丝雀发布

```yaml
# .github/workflows/canary.yml
name: Canary Deployment

on:
  push:
    branches: [main]

jobs:
  canary:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    
    - name: Deploy canary
      run: |
        # 部署到 10% 的用户
        kubectl set image deployment/my-app \
          my-app=ghcr.io/your-org/your-app:${{ github.sha }}
        
        # 等待并监控
        sleep 300
        
        # 检查错误率
        ERROR_RATE=$(curl -s https://prometheus.example.com/api/v1/query?query=rate(http_requests_total{status=~"5.."}[5m]))
        
        if (( $(echo "$ERROR_RATE > 0.01" | bc -l) )); then
          echo "Error rate too high, rolling back..."
          kubectl rollout undo deployment/my-app
        else
          echo "Canary looks good, proceeding with full deployment..."
          # 完整部署
        fi
```

### A/B 测试

```javascript
// React 组件中使用
import { useFeatureFlag } from 'flagsmith-react';

function Dashboard() {
  const showNewUI = useFeatureFlag('new-dashboard-ui');
  
  if (showNewUI) {
    return <NewDashboardUI />;
  }
  
  return <OldDashboardUI />;
}
```

## 最佳实践

1. **渐进式发布**：先小范围测试，再逐步扩大
2. **监控指标**：发布时监控错误率和性能
3. **快速回滚**：有问题时能快速关闭功能
4. **清理旧代码**：功能稳定后移除 flag 相关代码
5. **文档化**：记录每个 flag 的用途和过期时间

## 相关资源

- [Feature Flags 文档](https://docs.launchdarkly.com)
- [Flagsmith 文档](https://docs.flagsmith.com)
- [Unleash 文档](https://docs.getunleash.io)

---

**上一篇：[数据库 CI/CD 工作流](W19-database-cicd.md) | 下一篇：[GitHub REST/GraphQL API 实战](W21-api-advanced.md)**
