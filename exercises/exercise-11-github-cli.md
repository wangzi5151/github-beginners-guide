# 练习 11：GitHub CLI 深入使用

## 学习目标

- 掌握 GitHub CLI 的高级功能
- 学会使用 `gh` 管理仓库、Issue、PR
- 了解 `gh` 的扩展和别名

## 前置条件

确保已安装 GitHub CLI 并登录：

```bash
# 安装（macOS）
brew install gh

# 安装（Windows）
winget install GitHub.cli

# 安装（Linux）
sudo apt install gh

# 登录
gh auth login
```

## 基础操作

### 仓库管理

```bash
# 创建仓库
gh repo create my-project --public --description "我的项目"

# 克隆仓库
gh repo clone owner/repo

# 列出仓库
gh repo list

# 查看仓库信息
gh repo view owner/repo

# 设置仓库默认分支
gh repo edit owner/repo --default-branch main
```

### Issue 管理

```bash
# 创建 Issue
gh issue create --title "功能请求" --body "描述你的需求"

# 从模板创建
gh issue create --template "bug_report.md"

# 列出 Issue
gh issue list

# 查看 Issue
gh issue view 123

# 添加评论
gh issue comment 123 --body "评论内容"

# 关闭 Issue
gh issue close 123 --reason "completed"

# 重新打开
gh issue reopen 123

# 分配 Issue
gh issue edit 123 --add-assignee username
```

### Pull Request 管理

```bash
# 创建 PR
gh pr create --title "feat: 新功能" --body "描述"

# 列出 PR
gh pr list

# 查看 PR
gh pr view 456

# 检出 PR
gh pr checkout 456

# 合并 PR
gh pr merge 456 --merge

# Squash 合并
gh pr merge 456 --squash

# Rebase 合并
gh pr merge 456 --rebase

# 审查 PR
gh pr review 456 --approve

# 请求更改
gh pr review 456 --request-changes --body "需要修改..."

# 添加标签
gh pr edit 456 --add-label "enhancement"
```

## 高级功能

### 使用 JSON 输出

```bash
# 获取 Issue 列表（JSON 格式）
gh issue list --json number,title,state

# 使用 jq 处理
gh issue list --json number,title | jq '.[] | "\(.number): \(.title)"'

# 获取 PR 信息
gh pr view 456 --json title,body,author
```

### 使用 API

```bash
# 调用 API
gh api repos/owner/repo/issues

# 创建 Issue
gh api repos/owner/repo/issues \
  --method POST \
  -f title="Issue 标题" \
  -f body="Issue 内容"

# 更新 Issue
gh api repos/owner/repo/issues/123 \
  --method PATCH \
  -f state="closed"
```

### 别名配置

```bash
# 创建别名
gh alias set prc 'pr create --fill'
gh alias set prl 'pr list --state all'
gh alias set iss 'issue list'

# 查看所有别名
gh alias list

# 删除别名
gh alias delete prc
```

## 实战任务

### 任务 1：管理仓库

1. 创建一个新仓库
2. 添加 README 文件
3. 设置仓库描述
4. 添加话题标签

```bash
gh repo create practice-project --public
gh repo clone practice-project
echo "# Practice" > README.md
git add . && git commit -m "docs: 添加 README"
git push
gh repo edit practice-project --description "练习项目" --add-topic "git,github,practice"
```

### 任务 2：管理 Issue

1. 创建 3 个 Issue
2. 为 Issue 添加标签
3. 分配 Issue
4. 关闭一个 Issue

```bash
gh issue create --title "功能 1" --label "enhancement"
gh issue create --title "功能 2" --label "enhancement"
gh issue create --title "Bug 1" --label "bug"

gh issue close 1 --reason "completed"
```

### 任务 3：管理 PR

1. 创建一个新分支
2. 修改文件并提交
3. 创建 PR
4. 添加评论
5. 合并 PR

```bash
git checkout -b feature/new-feature
echo "新功能" > feature.txt
git add . && git commit -m "feat: 添加新功能"
git push -u origin feature/new-feature

gh pr create --title "feat: 添加新功能" --body "实现了新功能"
gh pr comment 1 --body "看起来不错"
gh pr merge 1 --squash
```

### 任务 4：使用扩展

```bash
# 安装扩展
gh extension install dlvhdr/gh-dash

# 查看已安装扩展
gh extension list

# 使用 dash
gh dash
```

## 高级脚本

### 批量操作脚本

```bash
#!/bin/bash
# 批量关闭所有已合并的 PR
gh pr list --state merged --json number | \
  jq -r '.[].number' | \
  xargs -I {} gh pr close {}

# 批量添加标签
for issue in $(gh issue list --label "needs-triage" --json number | jq -r '.[].number'); do
  gh issue edit $issue --add-label "triaged"
done
```

### 自动化工作流

```yaml
# .github/workflows/auto-label.yml
name: Auto Label

on:
  issues:
    types: [opened]

jobs:
  label:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/labeler@v4
      with:
        repo-token: ${{ secrets.GITHUB_TOKEN }}
```

## 验证清单

- [ ] 能够使用 `gh repo` 命令
- [ ] 能够使用 `gh issue` 命令
- [ ] 能够使用 `gh pr` 命令
- [ ] 能够使用 JSON 输出
- [ ] 能够使用 `gh api` 调用 API
- [ ] 能够配置和使用别名
- [ ] 能够安装和使用扩展

## 常见问题

### Q: 如何更新 GitHub CLI？
A: 使用 `gh extension upgrade` 更新扩展，使用包管理器更新 `gh` 本身。

### Q: 如何配置代理？
A: 设置环境变量：
```bash
export HTTP_PROXY=http://proxy.example.com:8080
export HTTPS_PROXY=http://proxy.example.com:8080
```

### Q: 如何查看 API 速率限制？
A: 使用 `gh api rate_limit` 查看。

## 下一步

完成本练习后，你已经掌握了 GitHub CLI 的高级用法。可以继续探索 [GitHub Actions 自动化](exercise-7-github-actions.md) 或 [参与开源项目](exercise-6-open-source.md)
