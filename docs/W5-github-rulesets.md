# GitHub Rulesets 介绍

## 什么是 Rulesets？

GitHub Rulesets 是分支保护规则的下一代替代方案。它提供了更灵活的方式来控制分支和标签的推送、合并和删除操作。

## Rulesets vs 旧版保护规则

| 特性 | Rulesets | 旧版保护规则 |
|------|----------|--------------|
| 应用范围 | 仓库、组织、企业 | 仅仓库 |
| 条件 | 多种条件组合 | 仅分支名匹配 |
| 嵌套规则 | 支持 | 不支持 |
| 优先级 | 支持 | 不支持 |
| 绕过权限 | 更灵活 | 较简单 |

## 创建 Ruleset

### 通过 Web UI 创建

1. 进入仓库 **Settings** → **Rules** → **Rulesets**
2. 点击 **New ruleset** → **New branch ruleset**
3. 配置规则

### 通过 API 创建

```bash
gh api repos/{owner}/{repo}/rulesets \
  --method POST \
  -f name='Main Branch Protection' \
  -f target='branch' \
  -f enforcement='active' \
  -f conditions='{"ref_name":{"include":["refs/heads/main"],"exclude":[]}}' \
  -f rules='[{"type":"pull_request","parameters":{"required_approving_review_count":1,"dismiss_stale_reviews_on_push":true,"require_last_push_approval":false}}]'
```

## 常用规则类型

### 分支保护规则

```json
{
  "type": "pull_request",
  "parameters": {
    "required_approving_review_count": 2,
    "dismiss_stale_reviews_on_push": true,
    "require_last_push_approval": false,
    "require_review_thread_resolution": true
  }
}
```

### 要求状态检查通过

```json
{
  "type": "required_status_checks",
  "parameters": {
    "required_status_checks": [
      {"context": "ci/build"},
      {"context": "ci/test"}
    ],
    "strict_required_status_checks_policy": true
  }
}
```

### 禁止强制推送

```json
{
  "type": "non_fast_forward"
}
```

### 要求签名提交

```json
{
  "type": "required_linear_history"
}
```

### 限制文件大小

```json
{
  "type": "file_parameters",
  "parameters": {
    "max_file_size": 10240
  }
}
```

## 组织级 Ruleset

### 创建组织 Ruleset

1. 进入组织 **Settings** → **Repository** → **Rulesets**
2. 点击 **New ruleset** → **New branch ruleset**
3. 选择应用范围

### 继承组织 Ruleset

仓库可以继承组织级 Ruleset：

1. 进入仓库 **Settings** → **Rules** → **Rulesets**
2. 查看组织级 Ruleset 是否已应用
3. 仓库级 Ruleset 会覆盖组织级规则

## 高级用法

### 多条件组合

```yaml
name: Release Branch
target: branch
enforcement: active
conditions:
  ref_name:
    include:
      - "refs/heads/release/*"
      - "refs/heads/main"
    exclude: []
rules:
  - type: pull_request
    parameters:
      required_approving_review_count: 2
  - type: required_status_checks
    parameters:
      required_status_checks:
        - context: "ci/build"
        - context: "ci/test"
  - type: non_fast_forward
```

### 创建 Ruleset 集合

```bash
# 创建 Ruleset 集合
gh api orgs/{org}/rulesets \
  --method POST \
  -f name='Default Branch Rules' \
  -f target='branch' \
  -f enforcement='active' \
  -f conditions='{"ref_name":{"include":["refs/heads/main"]}}' \
  -f rules='[{"type":"pull_request","parameters":{"required_approving_review_count":1}}]'
```

## 管理 Rulesets

### 查看所有 Ruleset

```bash
# 仓库级
gh api repos/{owner}/{repo}/rulesets

# 组织级
gh api orgs/{org}/rulesets
```

### 禁用 Ruleset

```bash
gh api repos/{owner}/{repo}/rulesets/{ruleset_id} \
  --method PATCH \
  -f enforcement='disabled'
```

### 删除 Ruleset

```bash
gh api repos/{owner}/{repo}/rulesets/{ruleset_id} \
  --method DELETE
```

## 最佳实践

1. **从简单开始**：先创建基本保护规则，逐步添加复杂规则
2. **使用组织级规则**：统一团队的分支管理策略
3. **文档化规则**：在 README 中说明分支策略
4. **定期审查**：检查规则是否仍然适用
5. **使用条件组合**：针对不同分支应用不同规则

## 常见问题

### Q: Ruleset 和 Ruleset 集合的区别？
A: Ruleset 是单个规则集合，Ruleset 集合是多个 Ruleset 的组合，可以按优先级应用。

### Q: 如何绕过 Ruleset？
A: 需要配置 bypass 选项，通常只有仓库管理员或指定用户可以绕过。

### Q: Ruleset 会影响现有分支吗？
A: 是的，Ruleset 会立即应用到符合条件的现有分支。

## 相关资源

- [GitHub Rulesets 官方文档](https://docs.github.com/en/repositories/configuring-your-warehouse/managing-rulesets)
- [分支保护规则迁移指南](https://docs.github.com/en/repositories/configuring-your-repository/managing-rules/migrating-to-rulesets)

---

**上一篇：[GitHub Projects 项目管理](21-github-projects.md) | 下一篇：[Fork 与开源贡献](22-fork-contribute.md)**
