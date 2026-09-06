# 安全与权限管理

## 个人账户安全

### 两步验证 (2FA)

1. 进入 **Settings** → **Password and authentication**
2. 启用 **Two-factor authentication**
3. 扫描二维码或保存恢复代码

### SSH 密钥管理

```bash
# 查看已添加的密钥
gh auth setup-git

# 管理 SSH 密钥
# Settings → SSH and GPG keys
```

### 访问令牌 (Personal Access Tokens)

1. **Settings** → **Developer settings** → **Personal access tokens**
2. 创建新令牌
3. 选择权限范围

```bash
# 使用令牌
git clone https://<token>@github.com/user/repo.git
```

## 仓库权限

### 协作者权限

| 权限 | 说明 |
|------|------|
| Read | 只读，可克隆和查看 |
| Triage | 管理 Issue 和 PR |
| Write | 可推送代码 |
| Maintain | 管理仓库设置 |
| Admin | 完全控制 |

### 添加协作者

```bash
# 使用 CLI
gh repo add-collaborator user/repo username --permission write
```

### 分支保护

1. **Settings** → **Branches**
2. 点击 **Add rule**
3. 配置规则：
   - 要求 PR 审查
   - 要求状态检查通过
   - 要求分支最新
   - 禁止强制推送

## 组织权限

### 团队管理
- **Team**：创建团队
- **Role**：设置角色（Member, Maintainer, Owner）
- **Repository access**：控制仓库访问

### 要求计划 (CODEOWNERS)

创建 `CODEOWNERS` 文件：

```
# 默认所有者
* @team-leads

# 前端代码
/src/frontend/ @frontend-team

# 后端代码
/src/backend/ @backend-team

# 文档
/docs/ @docs-team
```

## 仓库安全功能

### Dependabot

自动检查依赖漏洞：
1. **Insights** → **Dependency graph**
2. 启用 **Dependabot alerts**

### Secret Scanning

扫描代码中的敏感信息：
- API 密钥
- 访问令牌
- 密码

### Code Scanning

静态代码分析：
- **Settings** → **Code security**
- 启用 CodeQL 分析

## 安全最佳实践

1. **启用 2FA**：保护账户安全
2. **使用 SSH**：避免密码泄露
3. **最小权限原则**：只授予必要权限
4. **定期审查**：检查协作者和密钥
5. **不提交敏感信息**：使用 .gitignore 和 secrets

## 下一步

[GitHub API 与 Webhooks →](29-api-webhooks.md)
