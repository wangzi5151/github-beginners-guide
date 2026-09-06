# GitHub CLI 命令行工具

## 什么是 GitHub CLI？

GitHub CLI (`gh`) 是 GitHub 官方的命令行工具，可以直接在终端操作 GitHub。

## 安装

### macOS
```bash
brew install gh
```

### Windows
```powershell
winget install GitHub.cli
# 或
choco install gh
```

### Linux
```bash
# Ubuntu/Debian
sudo apt install gh

# Fedora
sudo dnf install gh

# Arch Linux
sudo pacman -S github-cli
```

## 认证

```bash
# 登录
gh auth login

# 查看状态
gh auth status
```

## 常用命令

### 仓库操作

```bash
# 创建仓库
gh repo create my-repo --public

# 克隆仓库
gh repo clone user/repo

# 查看仓库
gh repo view

# 列出仓库
gh repo list
```

### Issue 操作

```bash
# 创建 Issue
gh issue create --title "Bug report" --body "描述..."

# 列出 Issue
gh issue list

# 查看 Issue
gh issue view 42

# 关闭 Issue
gh issue close 42

# 重新打开 Issue
gh issue reopen 42
```

### Pull Request 操作

```bash
# 创建 PR
gh pr create --title "feat: 新功能" --body "描述..."

# 列出 PR
gh pr list

# 查看 PR
gh pr view 42

# 检出 PR
gh pr checkout 42

# 合并 PR
gh pr merge 42

# 审查 PR
gh pr review 42 --approve
```

### Release 操作

```bash
# 创建 Release
gh release create v1.0.0 --title "Release v1.0.0" --notes "发布说明"

# 列出 Release
gh release list

# 下载 Release 资产
gh release download v1.0.0
```

### Gist 操作

```bash
# 创建 Gist
gh gist create file.txt

# 列出 Gist
gh gist list

# 查看 Gist
gh gist view <gist-id>
```

## 别名配置

```bash
# 添加别名
gh alias set prc 'pr create'

# 查看别名
gh alias list

# 删除别名
gh alias delete prc
```

## 使用技巧

### 管道操作

```bash
# 筛选 Issue
gh issue list --label "bug" --state open

# JSON 输出
gh issue list --json number,title,state

# 获取特定字段
gh issue list --json number,title --jq '.[].title'
```

### 扩展

```bash
# 安装扩展
gh extension install dlvhdr/gh-dash

# 列出扩展
gh extension list

# 删除扩展
gh extension remove dlvhdr/gh-dash
```

## 环境变量

```bash
# 使用 token
export GITHUB_TOKEN=ghp_xxxx

# 使用 OAuth
export GH_TOKEN=ghp_xxxx
```

## 下一步

[Git LFS 大文件管理 →](27-git-lfs.md)
