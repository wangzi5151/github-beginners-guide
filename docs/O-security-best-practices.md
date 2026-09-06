# GitHub 安全最佳实践

## 账户安全

### 1. 启用两步验证 (2FA)

**设置步骤**：
1. 进入 **Settings** → **Password and authentication**
2. 点击 **Enable two-factor authentication**
3. 选择验证方式：
   - 认证器应用（推荐）
   - 短信
   - 安全密钥

**推荐使用**：
- **认证器应用**：Google Authenticator、Authy、1Password
- **安全密钥**：YubiKey、Titan Security Key

### 2. 使用 SSH 密钥

**生成 SSH 密钥**：
```bash
# 推荐使用 Ed25519
ssh-keygen -t ed25519 -C "your@email.com"

# 或使用 RSA
ssh-keygen -t rsa -b 4096 -C "your@email.com"
```

**安全建议**：
- 为不同设备使用不同密钥
- 为密钥设置密码
- 定期轮换密钥

### 3. 使用个人访问令牌 (PAT)

**创建令牌**：
1. **Settings** → **Developer settings** → **Personal access tokens**
2. 点击 **Generate new token**
3. 选择最小权限
4. 设置过期时间

**安全建议**：
- 使用 fine-grained token
- 定期轮换
- 不要在代码中硬编码

## 仓库安全

### 1. .gitignore 配置

确保敏感信息不被提交：

```gitignore
# 环境变量
.env
.env.local
.env.*.local

# 密钥文件
*.pem
*.key
id_rsa
id_ed25519

# 配置文件
config.json
credentials.json
```

### 2. 分支保护

**设置步骤**：
1. **Settings** → **Branches**
2. 点击 **Add rule**
3. 配置保护规则：
   - 要求 PR 审查
   - 要求状态检查通过
   - 要求分支最新
   - 禁止强制推送
   - 限制推送权限

### 3. 代码所有者 (CODEOWNERS)

定义代码审查责任人：

```
# 默认所有者
* @team-leads

# 安全相关代码
/security/ @security-team

# 配置文件
*.yml @devops-team
```

### 4. Secret Scanning

启用 Secret Scanning：
1. **Settings** → **Code security**
2. 启用 **Secret scanning**
3. 启用 **Push protection**

### 5. Dependabot

启用 Dependabot：
1. **Settings** → **Code security**
2. 启用 **Dependabot alerts**
3. 启用 **Dependabot security updates**
4. 配置自动更新

## CI/CD 安全

### 1. 使用 Secrets

在 GitHub Actions 中使用 Secrets：

```yaml
- name: Deploy
  env:
    API_KEY: ${{ secrets.API_KEY }}
  run: ./deploy.sh
```

### 2. 限制 Actions 权限

```yaml
permissions:
  contents: read
  pages: write
  id-token: write
```

### 3. 验证 Action 来源

```yaml
# 使用完整的 commit SHA
- uses: actions/checkout@b4ffde65f46336ab88eb53be808477a3936bae11

# 或使用标签
- uses: actions/checkout@v4
```

### 4. 审查第三方 Action

在使用第三方 Action 前：
- 检查 Action 的源代码
- 查看 Action 的权限需求
- 验证 Action 的维护状态

## 依赖安全

### 1. 定期更新依赖

```bash
# npm
npm update

# pip
pip install --upgrade 包名

# composer
composer update
```

### 2. 使用 lock 文件

确保使用 lock 文件：
- `package-lock.json`
- `yarn.lock`
- `composer.lock`
- `Pipfile.lock`

### 3. 扫描依赖漏洞

```bash
# npm
npm audit

# pip
pip audit

# bundler
bundle audit
```

## 团队安全

### 1. 最小权限原则

- 只授予必要的权限
- 定期审查权限
- 及时移除不再需要的权限

### 2. 安全培训

- 定期进行安全培训
- 分享安全最佳实践
- 演练安全事件响应

### 3. 代码审查

- 审查所有代码变更
- 关注安全相关问题
- 使用安全检查工具

## 安全检查清单

### 账户安全
- [ ] 启用两步验证
- [ ] 使用 SSH 密钥
- [ ] 配置个人访问令牌
- [ ] 定期审查登录活动

### 仓库安全
- [ ] 配置 .gitignore
- [ ] 设置分支保护
- [ ] 启用 Secret Scanning
- [ ] 启用 Dependabot
- [ ] 配置 CODEOWNERS

### CI/CD 安全
- [ ] 使用 Secrets
- [ ] 限制 Actions 权限
- [ ] 验证 Action 来源
- [ ] 审查第三方 Action

### 依赖安全
- [ ] 定期更新依赖
- [ ] 使用 lock 文件
- [ ] 扫描依赖漏洞
- [ ] 监控安全公告

## 相关资源

- [GitHub 安全文档](https://docs.github.com/en/security)
- [GitHub 安全最佳实践](https://docs.github.com/en/code-security)
- [GitHub 安全公告](https://github.com/advisories)
