# 练习 18：GitHub 企业配置实战

## 学习目标

- 配置组织级策略
- 设置分支保护
- 配置安全功能

## 步骤

### 步骤 1：创建组织

```bash
# 创建组织
gh api orgs \
  --method POST \
  -f login="my-org" \
  -f profile_name="My Organization" \
  -f billing_email="admin@example.com"
```

### 步骤 2：配置 CODEOWNERS

```yaml
# .github/CODEOWNERS
* @your-username
/docs/ @your-username
/src/ @your-username
```

### 步骤 3：配置分支保护

```bash
# 使用 API 配置分支保护
gh api repos/{org}/{repo}/branches/main/protection \
  --method PUT \
  -f required_status_checks='{"strict":true,"contexts":["ci/test"]}' \
  -f enforce_admins=true \
  -f required_pull_request_reviews='{"required_approving_review_count":1}'
```

### 步骤 4：配置 Dependabot

```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
```

### 步骤 5：配置安全警报

```bash
# 启用 Secret Scanning
gh api repos/{org}/{repo}/secret-scanning \
  --method PUT \
  -f status='enabled'

# 启用 Dependabot alerts
gh api repos/{org}/{repo}/vulnerability-alerts \
  --method PUT \
  -f enabled='true'
```

## 实战任务

1. 创建 GitHub 组织
2. 配置 CODEOWNERS
3. 配置分支保护规则
4. 配置 Dependabot
5. 启用安全功能

## 验证清单

- [ ] 组织已创建
- [ ] CODEOWNERS 已配置
- [ ] 分支保护已启用
- [ ] Dependabot 已配置
- [ ] 安全功能已启用

## 下一步

继续 [练习 19：GitHub Actions 复用工作流](exercise-19-reusable-workflows.md)
