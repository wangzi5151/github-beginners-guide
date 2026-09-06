# GitHub 企业治理

## 治理框架

### 治理层次

```
组织级治理
├── 策略管理
├── 权限管理
├── 安全管理
└── 合规管理

仓库级治理
├── 分支策略
├── 代码审查
├── 访问控制
└── 自动化

团队级治理
├── 协作规范
├── 沟通机制
└── 知识管理
```

## 策略管理

### 组织策略

```yaml
# 组织策略文档
组织策略：
  代码管理：
    - 使用 Conventional Commits 规范
    - PR 必须通过代码审查
    - 合并前必须通过 CI 检查
  
  安全管理：
    - 所有仓库必须启用 Dependabot
    - 必须启用 Secret Scanning
    - 生产分支必须启用分支保护
  
  发布管理：
    - 使用语义化版本
    - 发布必须通过审批
    - 自动生成 Release Notes
```

### 仓库模板

```yaml
# 仓库初始化模板
模板内容：
  README.md: 基础模板
  CONTRIBUTING.md: 贡献指南
  CODE_OF_CONDUCT.md: 行为准则
  .github/
    ISSUE_TEMPLATE/: Issue 模板
    PULL_REQUEST_TEMPLATE.md: PR 模板
    CODEOWNERS: 代码所有者
    dependabot.yml: Dependabot 配置
    workflows/: 工作流模板
```

## 权限管理

### 权限矩阵

| 角色 | 仓库 | 分支 | Issue | PR | Actions |
|------|------|------|-------|-----|---------|
| Owner | 完全控制 | 完全控制 | 完全控制 | 完全控制 | 完全控制 |
| Admin | 完全控制 | 完全控制 | 完全控制 | 完全控制 | 完全控制 |
| Write | 读写 | 读写 | 读写 | 读写 | 读写 |
| Triage | 读取 | 读取 | 管理 | 管理 | 读取 |
| Read | 读取 | 读取 | 读取 | 读取 | 读取 |

### CODEOWNERS

```yaml
# CODEOWNERS
# 默认所有者
* @your-org/core-team

# 前端代码
/src/components/ @your-org/frontend-team
/src/pages/ @your-org/frontend-team

# 后端代码
/src/api/ @your-org/backend-team
/src/services/ @your-org/backend-team

# 基础设施
/terraform/ @your-org/devops-team
/.github/ @your-org/devops-team

# 文档
/docs/ @your-org/docs-team

# 安全相关
/src/security/ @your-org/security-team
```

### 分支保护

```json
// 组织级分支保护规则
{
  "rules": [
    {
      "pattern": "main",
      "protections": {
        "required_pull_request_reviews": {
          "required_approving_review_count": 2,
          "dismiss_stale_reviews": true,
          "require_code_owner_reviews": true
        },
        "required_status_checks": {
          "strict": true,
          "contexts": ["ci/test", "ci/lint"]
        },
        "enforce_admins": true,
        "restrictions": null
      }
    }
  ]
}
```

## 安全管理

### 安全策略

```markdown
# SECURITY.md

## 安全政策

### 支持的版本

| 版本 | 支持状态 |
|------|----------|
| 5.0.x | ✅ 完全支持 |
| 4.0.x | ✅ 安全更新 |
| < 4.0 | ❌ 不再支持 |

### 报告漏洞

请通过以下方式报告安全漏洞：

1. **不要**公开报告
2. 发送邮件到 security@your-domain.com
3. 使用 GitHub Security Advisory

### 响应时间

- 严重漏洞：24 小时内响应
- 高危漏洞：72 小时内响应
- 中危漏洞：1 周内响应
- 低危漏洞：2 周内响应
```

### Secret Scanning

```yaml
# 启用 Secret Scanning Push Protection
gh api repos/{org}/{repo}/secret-scanning/push-protection \
  --method PUT \
  -f status='enabled'
```

### 依赖审查

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
        deny-licenses: GPL-3.0, AGPL-3.0
```

## 合规管理

### 审计日志

```bash
# 查看审计日志
gh api orgs/{org}/audit-log \
  --method GET \
  -f phrase='action:repo.create' \
  -f created='>=2024-01-01'

# 导出审计日志
gh api orgs/{org}/audit-log \
  --method GET \
  -f phrase='created:>=2024-01-01' \
  -q '.[] | @json' > audit-log.json
```

### 合规检查

```yaml
# .github/workflows/compliance.yml
name: Compliance Check

on:
  schedule:
    - cron: '0 0 * * 1'  # 每周一

jobs:
  check:
    runs-on: ubuntu-latest
    steps:
    - name: Check branch protection
      run: |
        gh api repos/{org}/{repo}/branches/main/protection
    
    - name: Check security features
      run: |
        gh api repos/{org}/{repo}/vulnerability-alerts
        gh api repos/{org}/{repo}/secret-scanning/alerts
    
    - name: Check CODEOWNERS
      run: |
        if [ ! -f .github/CODEOWNERS ]; then
          echo "Missing CODEOWNERS file"
          exit 1
        fi
```

## 知识管理

### 文档结构

```yaml
文档管理：
  README.md: 项目介绍
  CONTRIBUTING.md: 贡献指南
  CODE_OF_CONDUCT.md: 行为准则
  SECURITY.md: 安全政策
  CHANGELOG.md: 变更日志
  docs/: 详细文档
    architecture/: 架构文档
    api/: API 文档
    guides/: 指南
    troubleshooting/: 故障排除
```

### 知识库

```markdown
# 知识库结构

## 开发指南
- 编码规范
- 架构设计
- 最佳实践

## 运维指南
- 部署流程
- 监控告警
- 故障排除

## 安全指南
- 安全策略
- 漏洞处理
- 合规要求
```

## 指标和度量

### 关键指标

| 指标 | 目标 | 说明 |
|------|------|------|
| PR 审查时间 | < 24 小时 | PR 提交到合并的时间 |
| CI 通过率 | > 95% | CI 检查通过的比例 |
| 安全漏洞修复时间 | < 7 天 | 发现到修复的时间 |
| 文档覆盖率 | > 80% | 有文档的 API 比例 |
| 测试覆盖率 | > 70% | 代码测试覆盖率 |

### 度量工具

```yaml
# .github/workflows/metrics.yml
name: Metrics Collection

on:
  schedule:
    - cron: '0 0 * * *'  # 每天执行

jobs:
  collect:
    runs-on: ubuntu-latest
    steps:
    - name: Collect metrics
      run: |
        # 收集 PR 指标
        gh api repos/{org}/{repo}/pulls?state=closed \
          --jq '.[] | {created_at: .created_at, merged_at: .merged_at}'
        
        # 收集 Issue 指标
        gh api repos/{org}/{repo}/issues?state=closed \
          --jq '.[] | {created_at: .created_at, closed_at: .closed_at}'
```

## 最佳实践

1. **建立清晰的策略**：制定明确的治理策略
2. **自动化治理**：使用工具自动化治理流程
3. **定期审查**：定期审查和更新策略
4. **培训团队**：确保团队理解和遵守策略
5. **持续改进**：根据反馈持续改进治理

## 相关资源

- [GitHub 组织管理](https://docs.github.com/en/organizations)
- [GitHub 安全最佳实践](https://docs.github.com/en/code-security)
- [GitHub 审计日志](https://docs.github.com/en/organizations/keeping-your-organization-secure/managing-security-settings-for-your-organization/reviewing-the-audit-log-for-your-organization)

---

**上一篇：[GitHub Enterprise 企业版](W16-github-enterprise.md) | 下一篇：[GitHub 团队协作规范](W26-team-collaboration.md)**
