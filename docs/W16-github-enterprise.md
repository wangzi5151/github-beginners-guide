# GitHub Enterprise 企业版功能

## Enterprise 版本对比

| 功能 | Free | Team ($4/月) | Enterprise ($21/月) |
|------|------|--------------|---------------------|
| 仓库 | 无限 | 无限 | 无限 |
| 协作者 | 无限 | 无限 | 无限 |
| GitHub Pages | ✅ | ✅ | ✅ |
| GitHub Actions | 2000分钟/月 | 3000分钟/月 | 50000分钟/月 |
| 容量限制 | 1GB | 2GB | 50GB |
| SAML SSO | ❌ | ❌ | ✅ |
| 审计日志 | ❌ | ❌ | ✅ |
| IP 允许列表 | ❌ | ❌ | ✅ |
| 支持 | 社区 | 优先支持 | 24/7 支持 |

## SAML SSO 配置

### 配置 SAML

```
1. Settings → Authentication security → SAML single sign-on
2. 配置：
   - Sign on URL: https://your-idp.com/saml/sso
   - Issuer: your-entity-id
   - Public certificate: 上传证书
```

### 自动配置 SCIM

```bash
# 使用 GitHub CLI
gh api orgs/{org}/identity-provider \
  --method PUT \
  -f type='saml' \
  -f sso_url='https://your-idp.com/saml/sso' \
  -f issuer='your-entity-id' \
  -f certificate='-----BEGIN CERTIFICATE-----\n...\n-----END CERTIFICATE-----'
```

## 审计日志

### 访问审计日志

```bash
# 使用 CLI
gh api orgs/{org}/audit-log \
  --method GET \
  -f phrase='action:repo.create' \
  -f created='>=2024-01-01'

# 使用 API
curl -H "Authorization: Bearer $TOKEN" \
  "https://api.github.com/orgs/{org}/audit-log?phrase=action:repo.create"
```

### 审计日志事件

| 事件 | 说明 |
|------|------|
| `repo.create` | 创建仓库 |
| `repo.destroy` | 删除仓库 |
| `org.invite_member` | 邀请成员 |
| `org.remove_member` | 移除成员 |
| `team.create` | 创建团队 |
| `member.created` | 成员加入 |
| `member.removed` | 成员离开 |

### 导出审计日志

```yaml
# .github/workflows/audit-export.yml
name: Export Audit Log

on:
  schedule:
    - cron: '0 0 * * *'  # 每天执行

jobs:
  export:
    runs-on: ubuntu-latest
    steps:
    - name: Export audit log
      run: |
        gh api orgs/{org}/audit-log \
          --method GET \
          -f phrase='created:>=2024-01-01' \
          -q '.[] | @json' > audit-log.json
```

## IP 允许列表

### 配置 IP 允许列表

```
1. Settings → Authentication security → IP allow list
2. 添加 IP 地址或 CIDR 范围
3. 启用 "Enable IP allow list"
```

### API 配置

```bash
gh api orgs/{org}/actions/allowed-actions \
  --method PUT \
  -f enabled_all=true

gh api orgs/{org}/actions/allowed-actions \
  --method PUT \
  -f enabled_verified_only=true
```

## 合规和治理

### 配置分支保护

```yaml
# 组织级分支保护
gh api orgs/{org}/rulesets \
  --method POST \
  -f name='Production Branch Protection' \
  -f target='branch' \
  -f enforcement='active' \
  -f conditions='{"ref_name":{"include":["refs/heads/main"],"exclude":[]}}' \
  -f rules='[
    {"type":"pull_request","parameters":{"required_approving_review_count":2}},
    {"type":"required_status_checks","parameters":{"required_status_checks":[{"context":"ci/test"}]}},
    {"type":"non_fast_forward"}
  ]'
```

### 代码所有者

```yaml
# CODEOWNERS
* @your-org/core-team
/docs/ @your-org/docs-team
/src/security/ @your-org/security-team
/.github/ @your-org/devops-team
```

## 安全功能

### Secret Scanning

```yaml
# 启用 Push Protection
gh api repos/{org}/{repo}/secret-scanning/push-protection \
  --method PUT \
  -f status='enabled'
```

### 依赖图

```yaml
# 启用依赖图
gh api repos/{org}/{repo}/vulnerability-alerts \
  --method PUT \
  -f enabled='true'
```

## 管理 API

### 组织管理

```bash
# 获取组织信息
gh api orgs/{org}

# 列出成员
gh api orgs/{org}/members

# 列出团队
gh api orgs/{org}/teams

# 获取计费信息
gh api orgs/{org}/settings/billing
```

### 仓库管理

```bash
# 列出仓库
gh api orgs/{org}/repos --paginate

# 获取仓库安全功能
gh api repos/{org}/{repo}/vulnerability-alerts

# 获取仓库密钥
gh api repos/{org}/{repo}/actions/secrets
```

## 最佳实践

1. **启用 SSO**：统一身份认证
2. **配置审计日志**：跟踪所有操作
3. **设置 IP 允许列表**：限制访问来源
4. **使用 CODEOWNERS**：明确代码责任
5. **定期审查权限**：确保最小权限原则

## 相关资源

- [GitHub Enterprise 文档](https://docs.github.com/en/enterprise-cloud@latest)
- [SAML SSO 文档](https://docs.github.com/en/enterprise-cloud@latest/organizations/managing-saml-single-sign-on-for-your-organization)
- [审计日志文档](https://docs.github.com/en/enterprise-cloud@latest/admin/monitoring-activity-in-your-enterprise/reviewing-the-audit-log-for-your-enterprise)

---

**上一篇：[DevOps 实战](W15-devops.md) | 下一篇：[Docker + GitHub Actions 容器化](W17-docker-actions.md)**
