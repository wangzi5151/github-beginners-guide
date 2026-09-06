# 中国开发者常见问题

## 账号相关

### Q: GitHub 账号注册收不到验证码？

**解决方案**：
1. 使用国外邮箱（Gmail、Outlook）
2. 检查垃圾邮件文件夹
3. 尝试使用不同的浏览器
4. 稍后再试

### Q: GitHub 账号被限制怎么办？

**解决方案**：
1. 联系 GitHub 支持
2. 提供身份验证
3. 等待解封

### Q: 如何注销 GitHub 账号？

**步骤**：
1. 进入 Settings
2. 滚动到底部
3. 点击 Delete account
4. 确认删除

---

## 网络相关

### Q: 无法访问 GitHub？

**解决方案**：
1. 修改 hosts 文件
2. 使用加速代理
3. 配置 VPN/代理
4. 使用国内镜像

### Q: git push 很慢？

**解决方案**：
```bash
# 使用 SSH 代替 HTTPS
git config --global url."git@github.com:".insteadOf "https://github.com/"

# 使用浅克隆
git clone --depth 1

# 配置代理
git config --global http.proxy http://127.0.0.1:7890
```

### Q: GitHub Pages 无法访问？

**解决方案**：
1. 等待几分钟（首次部署需要时间）
2. 检查仓库设置
3. 使用自定义域名
4. 配置 DNS

---

## Git 相关

### Q: Git 中文文件名显示乱码？

**解决方案**：
```bash
git config --global core.quotepath false
```

### Q: Git 提交信息如何写中文？

**解决方案**：
```bash
git commit -m "feat: 添加新功能"
```

### Q: 如何忽略文件权限变更？

**解决方案**：
```bash
git config --global core.fileMode false
```

### Q: 如何设置默认分支为 main？

**解决方案**：
```bash
git config --global init.defaultBranch main
```

---

## 协作相关

### Q: 如何参与国际开源项目？

**建议**：
1. 先从文档贡献开始
2. 修复简单的 bug
3. 使用英文沟通
4. 遵循项目规范

### Q: 英语不好如何参与 GitHub？

**建议**：
1. 使用翻译工具
2. 学习常用术语
3. 从国内项目开始
4. 参与中文社区

### Q: 如何在 GitHub 上展示自己的项目？

**建议**：
1. 写好 README
2. 添加文档
3. 编写测试
4. 使用标签
5. 定期更新

---

## 安全相关

### Q: 如何保护 GitHub 账号？

**建议**：
1. 启用两步验证
2. 使用 SSH 密钥
3. 定期更换密码
4. 不要分享凭证

### Q: 如何检测代码中的敏感信息？

**解决方案**：
```bash
# 使用 git-secrets
git secrets --install
git secrets --scan

# 使用 truffleHog
trufflehog git file://.
```

### Q: 如何防止意外提交敏感信息？

**解决方案**：
1. 使用 .gitignore
2. 使用 pre-commit hook
3. 定期检查代码

---

## 性能相关

### Q: 如何加速 GitHub 访问？

**方案**：
1. 修改 hosts
2. 使用代理
3. 使用镜像
4. 使用 SSH

### Q: 如何优化大仓库克隆？

**解决方案**：
```bash
# 浅克隆
git clone --depth 1

# 只克隆特定分支
git clone --single-branch --branch main

# 使用 sparse checkout
git sparse-checkout init
git sparse-checkout set dir1 dir2
```

### Q: 如何清理 Git 仓库？

**解决方案**：
```bash
# 垃圾回收
git gc

# 清理未跟踪文件
git clean -fd

# 压缩历史
git repack -a -d
```

---

## 工具相关

### Q: 推荐哪些 Git 客户端？

**推荐**：
| 工具 | 平台 | 特点 |
|------|------|------|
| GitKraken | 跨平台 | 功能强大 |
| SourceTree | Windows/Mac | 免费 |
| GitHub Desktop | 跨平台 | 简单易用 |
| VS Code | 跨平台 | 集成开发 |

### Q: 如何配置 Git 别名？

**示例**：
```bash
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.lg "log --oneline --graph --all"
```

### Q: 如何使用 GitHub CLI？

**安装**：
```bash
# macOS
brew install gh

# Windows
winget install GitHub.cli

# Linux
sudo apt install gh
```

**使用**：
```bash
# 登录
gh auth login

# 创建仓库
gh repo create my-repo --public

# 创建 PR
gh pr create --title "New feature" --body "Description"
```

---

## 更多问题

如果以上没有解决你的问题，可以：

1. 搜索 [GitHub Discussions](https://github.com/wangzi5151/chinese-use-github-helper/discussions)
2. 提交 [Issue](https://github.com/wangzi5151/chinese-use-github-helper/issues)
3. 搜索 [Stack Overflow](https://stackoverflow.com/questions/tagged/git)
4. 查看 [GitHub 文档](https://docs.github.com)
