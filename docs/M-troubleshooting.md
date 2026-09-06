# 常见错误排查指南

## Git 错误

### 1. "Permission denied (publickey)"

**原因**：SSH 密钥配置错误

**解决方法**：
```bash
# 检查是否有 SSH 密钥
ls -al ~/.ssh

# 测试 SSH 连接
ssh -T git@github.com

# 如果没有密钥，生成新的
ssh-keygen -t ed25519 -C "your@email.com"

# 将公钥添加到 GitHub
cat ~/.ssh/id_ed25519.pub | pbcopy  # macOS
cat ~/.ssh/id_ed25519.pub | xclip -selection clipboard  # Linux
```

### 2. "fatal: remote origin already exists"

**原因**：已存在名为 origin 的远程仓库

**解决方法**：
```bash
# 查看远程仓库
git remote -v

# 修改远程仓库 URL
git remote set-url origin git@github.com:user/repo.git

# 或删除并重新添加
git remote remove origin
git remote add origin git@github.com:user/repo.git
```

### 3. "error: failed to push some refs"

**原因**：本地分支与远程分支不一致

**解决方法**：
```bash
# 如果确定要覆盖远程
git push --force origin branch

# 更安全的方式
git push --force-with-lease origin branch
```

### 4. "CONFLICT (content): Merge conflict"

**原因**：合并时发生冲突

**解决方法**：
```bash
# 1. 查看冲突文件
git status

# 2. 手动编辑冲突文件，解决冲突

# 3. 标记为已解决
git add .

# 4. 完成合并
git commit -m "解决合并冲突"
```

### 5. "fatal: refusing to merge unrelated histories"

**原因**：尝试合并没有共同祖先的分支

**解决方法**：
```bash
git merge --allow-unrelated-histories branch-name
```

### 6. "error: pathspec 'xxx' did not match"

**原因**：文件不存在或路径错误

**解决方法**：
```bash
# 检查文件是否存在
ls -la

# 检查当前分支
git branch

# 确保在正确的目录
pwd
```

### 7. "warning: LF will be replaced by CRLF"

**原因**：行尾符问题（Windows/Linux/Mac）

**解决方法**：
```bash
# Windows 用户
git config --global core.autocrlf true

# Linux/Mac 用户
git config --global core.autocrlf input
```

## GitHub 错误

### 1. "403 Forbidden"

**原因**：权限不足或 API 限制

**解决方法**：
```bash
# 检查认证状态
gh auth status

# 重新登录
gh auth login

# 检查 API 限制
gh api rate_limit
```

### 2. "404 Not Found"

**原因**：仓库不存在或无访问权限

**解决方法**：
```bash
# 检查仓库是否存在
gh repo view user/repo

# 检查权限
gh auth status

# 如果是私有仓库，确保已登录
gh auth login
```

### 3. "repository not found"

**原因**：仓库名称错误或无访问权限

**解决方法**：
```bash
# 确认仓库名称
gh repo list

# 检查是否已登录
gh auth status
```

### 4. "remote: Repository not found"

**原因**：远程仓库不存在

**解决方法**：
```bash
# 检查远程仓库 URL
git remote -v

# 修改为正确的 URL
git remote set-url origin git@github.com:user/repo.git
```

## GitHub Actions 错误

### 1. "Error: Process completed with exit code 1"

**原因**：工作流步骤失败

**解决方法**：
1. 查看失败步骤的日志
2. 检查命令是否正确
3. 检查环境变量是否设置

### 2. "Error: No space left on device"

**原因**：Runner 磁盘空间不足

**解决方法**：
```yaml
# 清理磁盘空间
- name: Free disk space
  uses: jlumbroso/free-disk-space@main
  with:
    tool-cache: false
    android: true
    dotnet: true
    haskell: true
    large-packages: true
    docker-images: true
    swap-storage: true
```

### 3. "Error: Timeout"

**原因**：工作流运行超时

**解决方法**：
```yaml
# 增加超时时间
jobs:
  build:
    runs-on: ubuntu-latest
    timeout-minutes: 30
```

### 4. "Error: The following actions uses node12"

**原因**：使用了过时的 Action

**解决方法**：
```yaml
# 使用最新版本的 Action
- uses: actions/checkout@v4
- uses: actions/setup-node@v4
```

## 常见问题排查步骤

### 1. 检查 Git 配置

```bash
# 查看所有配置
git config --list

# 检查用户信息
git config user.name
git config user.email
```

### 2. 检查远程仓库

```bash
# 查看远程仓库
git remote -v

# 测试连接
ssh -T git@github.com
```

### 3. 检查分支状态

```bash
# 查看当前分支
git branch

# 查看所有分支
git branch -a

# 查看状态
git status
```

### 4. 查看日志

```bash
# 查看最近提交
git log --oneline -10

# 查看详细日志
git log --stat
```

## 调试技巧

### 1. 使用 verbose 模式

```bash
# 查看详细输出
git -v push

# 查看 HTTP 详细信息
GIT_CURL_VERBOSE=1 git push
```

### 2. 检查网络连接

```bash
# 测试 GitHub 连接
ping github.com

# 测试 SSH 连接
ssh -vT git@github.com
```

### 3. 查看 Git 版本

```bash
git --version
```

## 寻求帮助

1. **GitHub 文档**：https://docs.github.com
2. **Stack Overflow**：https://stackoverflow.com/questions/tagged/git
3. **GitHub Community**：https://github.com/community
4. **Git 官方文档**：https://git-scm.com/doc
