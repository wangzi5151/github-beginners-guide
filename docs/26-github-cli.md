# GitHub CLI (gh) 完全指南

## 概述

GitHub CLI（命令行工具 `gh`）是 GitHub 官方推出的命令行工具，让开发者能够在终端中完成几乎所有 GitHub 操作。本章将全面介绍 GitHub CLI 的安装、配置和各种实用功能，帮助你提升开发效率。

---

## 1. GitHub CLI 概述与安装

### 1.1 什么是 GitHub CLI

GitHub CLI 是 GitHub 官方的命令行界面工具，主要特性包括：

- 在终端中管理 Issues、Pull Requests、Releases
- 操作 GitHub Actions 工作流
- 管理 GitHub Codespaces
- 直接调用 GitHub API
- 支持自定义别名和扩展
- 与 Git 命令无缝配合

### 1.2 官方安装方法

**macOS 安装：**

```bash
# 使用 Homebrew（推荐）
brew install gh

# 使用 MacPorts
sudo port install gh
```

**Windows 安装：**

```powershell
# 使用 winget（推荐）
winget install --id GitHub.cli

# 使用 Scoop
scoop install gh

# 使用 Chocolatey
choco install gh

# 使用 WinGet
winget install GitHub.cli
```

**Linux 安装：**

```bash
# Ubuntu/Debian（官方源）
sudo apt update
sudo apt install gh

# CentOS/RHEL/Fedora
sudo dnf install gh

# Arch Linux
sudo pacman -S github-cli

# openSUSE
sudo zypper install gh
```

### 1.3 国内安装方案

由于网络原因，国内用户可能无法直接访问官方源，以下是几种替代方案：

**方案一：使用国内镜像源（Ubuntu/Debian）**

```bash
# 添加 GitHub CLI 官方仓库
curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null

# 如果上述地址无法访问，可以使用代理或镜像
# 方法1: 使用 ghproxy 代理
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://ghproxy.com/https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null

sudo apt update
sudo apt install gh
```

**方案二：手动下载 deb/rpm 包**

```bash
# 访问 GitHub Releases 页面下载
# https://github.com/cli/cli/releases

# 使用 ghproxy 代理下载（以 2.40.1 为例）
wget https://ghproxy.com/https://github.com/cli/cli/releases/download/v2.40.1/gh_2.40.1_linux_amd64.deb

# 安装
sudo dpkg -i gh_2.40.1_linux_amd64.deb

# 如果有依赖问题
sudo apt-get install -f
```

**方案三：使用 Go 安装**

```bash
# 确保已安装 Go 1.21+
go version

# 设置 Go 代理（国内加速）
go env -w GOPROXY=https://goproxy.cn,direct

# 安装 gh
go install github.com/cli/cli/v2/cmd/gh@latest

# 添加到 PATH
echo 'export PATH=$PATH:$(go env GOPATH)/bin' >> ~/.bashrc
source ~/.bashrc
```

**方案四：使用预编译二进制文件**

```bash
# 下载二进制文件
wget https://ghproxy.com/https://github.com/cli/cli/releases/download/v2.40.1/gh_2.40.1_linux_amd64.tar.gz

# 解压
tar -xzf gh_2.40.1_linux_amd64.tar.gz

# 移动到系统路径
sudo cp gh_2.40.1_linux_amd64/bin/gh /usr/local/bin/

# 验证安装
gh --version
```

### 1.4 Termux 安装方案

```bash
# 在 Termux 中安装
pkg update
pkg install gh

# 如果官方源不可用，尝试
pkg install -y gh

# 或者使用 Go 安装
pkg install golang
go env -w GOPROXY=https://goproxy.cn,direct
go install github.com/cli/cli/v2/cmd/gh@latest
```

### 1.5 验证安装

```bash
# 检查版本
gh --version
# 输出示例：
# gh version 2.40.1 (2023-12-01)
# https://github.com/cli/cli/releases/tag/v2.40.1

# 查看帮助
gh --help

# 查看所有可用命令
gh --help | grep -E "^  [a-z]"
```

---

## 2. gh 基础命令

### 2.1 认证管理（gh auth）

```bash
# 交互式登录（推荐）
gh auth login
# 按提示选择：
# ? What account do you want to log into? GitHub.com
# ? What is your preferred protocol for Git operations? HTTPS
# ? Authenticate Git with your GitHub credentials? Yes
# ? How would you like to authenticate? Login with a web browser

# 使用 Token 登录
gh auth login --with-token < token.txt

# 通过环境变量设置 Token
export GH_TOKEN=your_token_here
gh auth status

# 查看认证状态
gh auth status
# 输出：
# github.com
#   ✓ Logged in to github.com account username
#   - Active account: true
#   - Git operations protocol: https
#   - Token: gho_****

# 刷新 Token
gh auth refresh

# 刷新特定权限
gh auth refresh -s admin:org -s delete_repo

# 退出登录
gh auth logout

# 在浏览器中打开 GitHub 设置 Token 页面
gh auth login --web

# 设置 Token 到配置文件
gh auth setup-git
```

### 2.2 仓库操作（gh repo）

```bash
# 克隆仓库
gh repo clone owner/repo

# 克隆到指定目录
gh repo clone owner/repo ~/projects/my-repo

# 创建新仓库（交互式）
gh repo create

# 创建公开仓库
gh repo create my-project --public --description "我的新项目"

# 创建私有仓库
gh repo create my-project --private

# 基于模板创建仓库
gh repo create my-project --template owner/template-repo

# 创建并克隆
gh repo create my-project --public --clone

# 查看仓库信息
gh repo view owner/repo

# 在浏览器中打开仓库
gh repo view --web

# 列出自己的仓库
gh repo list

# 列出指定用户的仓库
gh repo list username

# 列出组织的仓库
gh repo list my-org --limit 50

# 只列出自己拥有的仓库
gh repo list --source

# 只列出 fork 的仓库
gh repo list --fork

# 查看仓库的 Topics
gh repo view owner/repo --json name,description,homepageUrl

# 编辑仓库设置
gh repo edit --description "新的描述"
gh repo edit --homepage "https://example.com"
gh repo edit --visibility private
gh repo edit --default-branch main

# 归档仓库
gh repo archive owner/repo --yes

# 删除仓库（谨慎使用）
gh repo delete owner/repo --yes

# Fork 仓库
gh repo fork owner/repo

# Fork 并克隆
gh repo fork owner/repo --clone

# 同步 Fork
gh repo sync owner/repo
```

### 2.3 浏览器操作（gh browse）

```bash
# 在浏览器中打开当前仓库
gh browse

# 打开指定仓库
gh browse owner/repo

# 打开仓库的 Issues 页面
gh browse --issues

# 打开仓库的 Pull Requests 页面
gh browse --pulls

# 打开仓库的 Wiki
gh browse --wiki

# 打开仓库的 Settings
gh browse --settings

# 打开指定文件
gh browse src/main.go

# 打开指定文件的特定行
gh browse src/main.go:42

# 打开指定分支的文件
gh browse src/main.go --branch develop

# 只输出 URL，不打开浏览器
gh browse --no-browser

# 打开 GitHub Actions 页面
gh browse --actions
```

---

## 3. Issue 管理（gh issue）

### 3.1 查看 Issue

```bash
# 列出当前仓库的 Issue
gh issue list

# 列出指定仓库的 Issue
gh issue list --repo owner/repo

# 按状态筛选
gh issue list --state open
gh issue list --state closed
gh issue list --state all

# 按标签筛选
gh issue list --label "bug"
gh issue list --label "bug,priority:high"

# 按作者筛选
gh issue list --author username

# 按指派人筛选
gh issue list --assignee username

# 按里程碑筛选
gh issue list --milestone "v1.0"

# 限制数量
gh issue list --limit 20

# 按创建时间排序
gh issue list --sort created
gh issue list --sort updated
gh issue list --sort comments

# 查看特定 Issue
gh issue view 123

# 在浏览器中打开 Issue
gh issue view 123 --web

# 查看 Issue 的评论
gh issue view 123 --comments

# 以 JSON 格式查看
gh issue view 123 --json title,body,state,labels,assignees
```

### 3.2 创建 Issue

```bash
# 交互式创建 Issue
gh issue create

# 快速创建 Issue
gh issue create --title "修复登录页面的 Bug" --body "描述问题详情..."

# 带标签创建
gh issue create \
  --title "修复登录页面的 Bug" \
  --body "用户无法正常登录" \
  --label "bug,priority:high"

# 带指派人创建
gh issue create \
  --title "修复登录页面的 Bug" \
  --body "用户无法正常登录" \
  --assignee username

# 从文件读取 body
gh issue create \
  --title "功能请求" \
  --body-file issue-template.md

# 使用模板创建
gh issue create \
  --title "Bug 报告" \
  --template bug_report.md
```

### 3.3 管理 Issue

```bash
# 关闭 Issue
gh issue close 123

# 关闭并添加评论
gh issue close 123 --comment "已修复，请验证"

# 重新打开 Issue
gh issue reopen 123

# 重新打开并添加评论
gh issue reopen 123 --comment "问题仍然存在"

# 添加评论
gh issue comment 123 --body "正在处理这个问题..."

# 添加标签
gh issue edit 123 --add-label "bug,priority:high"

# 移除标签
gh issue edit 123 --remove-label "priority:low"

# 添加指派人
gh issue edit 123 --add-assignee username

# 移除指派人
gh issue edit 123 --remove-assignee username

# 修改标题
gh issue edit 123 --title "新的标题"

# 修改里程碑
gh issue edit 123 --milestone "v2.0"

# 转移 Issue 到其他仓库
gh issue transfer 123 owner/other-repo

# 锁定 Issue
gh issue lock 123

# 解锁 Issue
gh issue unlock 123

# 删除 Issue（需要权限）
gh issue delete 123 --yes

# 批量操作 Issue
gh issue list --label "stale" --json number --jq '.[].number' | \
  xargs -I {} gh issue close {} --comment "自动关闭：长期无活动"
```

### 3.4 搜索 Issue

```bash
# 搜索 Issue
gh search issues "登录 bug"

# 在指定仓库中搜索
gh search issues "登录 bug" --repo owner/repo

# 搜索打开的 Issue
gh search issues "登录" --state open

# 按标签搜索
gh search issues --label "bug"

# 按作者搜索
gh search issues --author username

# 组合搜索
gh search issues "登录" --label "bug" --state open --sort created
```

---

## 4. Pull Request 管理（gh pr）

### 4.1 查看 PR

```bash
# 列出当前仓库的 PR
gh pr list

# 列出指定仓库的 PR
gh pr list --repo owner/repo

# 按状态筛选
gh pr list --state open
gh pr list --state closed
gh pr list --state merged
gh pr list --state all

# 按分支筛选
gh pr list --head feature-branch
gh pr list --base main

# 按作者筛选
gh pr list --author username

# 按标签筛选
gh pr list --label "needs-review"

# 查看特定 PR
gh pr view 456

# 在浏览器中打开 PR
gh pr view 456 --web

# 查看 PR 的评论
gh pr view 456 --comments

# 查看 PR 的 diff
gh pr diff 456

# 查看 PR 的检查状态
gh pr checks 456

# 以 JSON 格式查看
gh pr view 456 --json title,body,state,mergeable,reviews
```

### 4.2 创建 PR

```bash
# 交互式创建 PR
gh pr create

# 快速创建 PR
gh pr create \
  --title "feat: 添加用户登录功能" \
  --body "## 变更内容\n\n- 实现用户登录\n- 添加登录页面"

# 指定 base 分支
gh pr create --base main --head feature-branch

# 设置为草稿 PR
gh pr create --draft

# 添加审阅者
gh pr create --reviewer reviewer1,reviewer2

# 添加指派人
gh pr create --assignee username

# 添加标签
gh pr create --label "feature,needs-review"

# 添加到项目
gh pr create --project "My Project"

# 设置里程碑
gh pr create --milestone "v1.0"

# 自动填充提交信息
gh pr create --fill

# 从 Issue 创建 PR（关联 Issue）
gh pr create --body "Closes #123"

# 从文件读取 body
gh pr create --body-file pr-template.md
```

### 4.3 管理 PR

```bash
# 检出 PR 到本地
gh pr checkout 456

# 检出 PR 到指定分支
gh pr checkout 456 --branch my-branch

# 合并 PR
gh pr merge 456

# 使用 squash 合并
gh pr merge 456 --squash

# 使用 merge commit 合并
gh pr merge 456 --merge

# 使用 rebase 合并
gh pr merge 456 --rebase

# 合并并删除分支
gh pr merge 456 --squash --delete-branch

# 合并并自动填充合并信息
gh pr merge 456 --squash --auto

# 关闭 PR
gh pr close 456

# 关闭并添加评论
gh pr close 456 --comment "暂时不合并"

# 重新打开 PR
gh pr reopen 456

# 编辑 PR
gh pr edit 456 --title "新的标题"
gh pr edit 456 --body "新的描述"
gh pr edit 456 --add-reviewer reviewer1
gh pr edit 456 --remove-label "wip"
gh pr edit 456 --add-assignee username

# 将 PR 标记为就绪
gh pr ready 456

# 将 PR 标记为草稿
gh pr ready 456 --undo

# 锁定 PR
gh pr lock 456

# 解锁 PR
gh pr unlock 456

# 添加审阅意见
gh pr review 456 --approve
gh pr review 456 --request-changes --body "请修改以下问题..."
gh pr review 456 --comment --body "看起来不错"
```

### 4.4 PR 状态和检查

```bash
# 查看 PR 状态
gh pr status

# 查看所有 PR 的状态
gh pr status --repo owner/repo

# 查看 PR 的检查结果
gh pr checks 456

# 等待检查完成
gh pr checks 456 --watch

# 查看 PR 的合并状态
gh pr view 456 --json mergeable,mergeStateStatus

# 查看 PR 的审阅状态
gh pr view 456 --json reviews
```

### 4.5 搜索 PR

```bash
# 搜索 PR
gh search prs "login feature"

# 在指定仓库中搜索
gh search prs "login" --repo owner/repo

# 搜索打开的 PR
gh search prs "login" --state open

# 按作者搜索
gh search prs --author username

# 按审阅者搜索
gh search prs --reviewer username

# 按标签搜索
gh search prs --label "needs-review"
```

---

## 5. Release 管理（gh release）

### 5.1 查看 Release

```bash
# 列出所有 Release
gh release list

# 列出指定仓库的 Release
gh release list --repo owner/repo

# 限制数量
gh release list --limit 10

# 查看最新 Release
gh release view

# 查看指定 Release
gh release view v1.0.0

# 在浏览器中打开 Release
gh release view v1.0.0 --web

# 以 JSON 格式查看
gh release view v1.0.0 --json tagName,name,body,assets

# 只查看 tag 名
gh release view v1.0.0 --json tagName --jq '.tagName'

# 查看 Release 的 Assets
gh release view v1.0.0 --json assets
```

### 5.2 创建 Release

```bash
# 创建简单 Release
gh release create v1.0.0

# 创建带标题和说明的 Release
gh release create v1.0.0 \
  --title "v1.0.0 正式版" \
  --notes "首个正式发布版本"

# 从文件读取说明
gh release create v1.0.0 \
  --title "v1.0.0" \
  --notes-file RELEASE_NOTES.md

# 自动生成 Release Notes
gh release create v1.0.0 --generate-notes

# 创建草稿 Release
gh release create v1.0.0 \
  --title "v1.0.0" \
  --notes "准备发布" \
  --draft

# 创建预发布版本
gh release create v1.0.0-beta.1 \
  --title "v1.0.0 Beta 1" \
  --notes "测试版本，请勿在生产环境使用" \
  --prerelease

# 创建 Release 并上传文件
gh release create v1.0.0 \
  --title "v1.0.0" \
  --notes "正式发布" \
  ./dist/app-linux \
  ./dist/app-macos \
  ./dist/app-windows.exe

# 使用通配符上传文件
gh release create v1.0.0 ./dist/*

# 指定目标分支
gh release create v1.0.0 --target main

# 为 Release 创建讨论
gh release create v1.0.0 --discussion-category "Announcements"

# 使用 notes 模板
gh release create v1.0.0 \
  --notes "**🚀 新功能**\n\n- 功能1\n- 功能2\n\n**🐛 修复**\n\n- Bug1"
```

### 5.3 管理 Release

```bash
# 编辑 Release
gh release edit v1.0.0 --title "v1.0.0 - 正式版"
gh release edit v1.0.0 --notes "更新的说明"
gh release edit v1.0.0 --notes-file NEW_NOTES.md

# 将草稿发布为正式版
gh release edit v1.0.0 --draft=false

# 标记为预发布
gh release edit v1.0.0 --prerelease

# 取消预发布标记
gh release edit v1.0.0 --prerelease=false

# 向 Release 添加文件
gh release upload v1.0.0 ./new-asset.zip

# 上传多个文件
gh release upload v1.0.0 ./file1.zip ./file2.tar.gz

# 替换现有文件
gh release upload v1.0.0 ./app-linux --clobber

# 删除 Release 的文件
gh release delete-asset v1.0.0 old-file.zip --yes

# 下载 Release 文件
gh release download v1.0.0

# 下载到指定目录
gh release download v1.0.0 --dir ./downloads

# 下载特定文件
gh release download v1.0.0 --pattern "*.zip"

# 下载特定平台文件
gh release download v1.0.0 --pattern "*linux*"

# 删除 Release
gh release delete v1.0.0 --yes
```

### 5.4 下载最新 Release

```bash
# 下载最新 Release
gh release download --pattern "*.tar.gz"

# 下载最新 Release 到指定目录
gh release download --dir ./latest --pattern "app-*"

# 只下载校验和文件
gh release download --pattern "checksums*"
```

---

## 6. GitHub Actions 管理

### 6.1 查看工作流（gh workflow）

```bash
# 列出所有工作流
gh workflow list

# 查看工作流详情
gh workflow view deploy.yml

# 查看工作流的 runs
gh workflow view deploy.yml --yaml

# 启用工作流
gh workflow enable deploy.yml

# 禁用工作流
gh workflow disable deploy.yml

# 手动触发工作流
gh workflow run deploy.yml

# 手动触发并传递参数
gh workflow run deploy.yml --ref main -f environment=production -f version=1.0.0

# 在浏览器中打开工作流
gh workflow view deploy.yml --web
```

### 6.2 查看运行记录（gh run）

```bash
# 列出最近的 runs
gh run list

# 列出指定工作流的 runs
gh run list --workflow=deploy.yml

# 按状态筛选
gh run list --status=completed
gh run list --status=failure
gh run list --status=in_progress

# 限制数量
gh run list --limit 20

# 查看特定 run
gh run view 1234567890

# 查看 run 的 jobs
gh run view 1234567890 --json jobs

# 查看 run 的日志
gh run view 1234567890 --log

# 查看特定 job 的日志
gh run view 1234567890 --job=job-id --log

# 在浏览器中打开 run
gh run view 1234567890 --web

# 查看 run 的状态
gh run status 1234567890

# 等待 run 完成
gh run watch 1234567890

# 重新运行失败的 jobs
gh run rerun 1234567890 --failed

# 重新运行整个 workflow
gh run rerun 1234567890

# 取消正在运行的 workflow
gh run cancel 1234567890

# 下载 run 的 artifacts
gh run download 1234567890

# 下载特定 artifact
gh run download 1234567890 --name my-artifact

# 下载到指定目录
gh run download 1234567890 --dir ./artifacts
```

### 6.3 查看 Secrets 和 Variables

```bash
# 列出仓库 Secrets
gh secret list

# 列出环境 Secrets
gh secret list --env production

# 列出组织 Secrets
gh secret list --org my-org

# 设置仓库 Secret
gh secret set MY_SECRET --body "secret-value"

# 从文件设置 Secret
gh secret set MY_SECRET --body-file secret.txt

# 设置环境 Secret
gh secret set MY_SECRET --env production --body "value"

# 删除 Secret
gh secret delete MY_SECRET

# 列出 Variables
gh variable list

# 设置 Variable
gh variable set MY_VAR --body "variable-value"

# 删除 Variable
gh variable delete MY_VAR
```

---

## 7. GitHub Codespaces 管理（gh codespace）

### 7.1 基础操作

```bash
# 列出 Codespaces
gh codespace list

# 创建 Codespace
gh codespace create --repo owner/repo

# 创建并指定机器类型
gh codespace create --repo owner/repo --machine largePremiumLinux

# 创建并指定分支
gh codespace create --repo owner/repo --branch develop

# 查看 Codespace 信息
gh codespace view codespace-name

# 在浏览器中打开 Codespace
gh codespace codespace codespace-name

# 删除 Codespace
gh codespace delete codespace-name --yes

# 停止 Codespace
gh codespace stop codespace-name
```

### 7.2 远程连接

```bash
# 通过 SSH 连接到 Codespace
gh codespace ssh codespace-name

# 使用 VS Code 连接
gh codespace code codespace-name

# 使用 Jupyter Notebook 连接
gh codespace jupyter codespace-name

# 转发端口
gh codespace ports forward 8080:80 codespace-name

# 查看端口列表
gh codespace ports list codespace-name

# 设置端口可见性
gh codespace ports visibility 8080:public codespace-name
```

### 7.3 文件操作

```bash
# 从 Codespace 复制文件到本地
gh codespace cp codespace-name:/path/to/file ./local-file

# 从本地复制文件到 Codespace
gh codespace cp ./local-file codespace-name:/path/to/file
```

---

## 8. gh api 高级用法

### 8.1 基础 API 调用

```bash
# GET 请求
gh api repos/owner/repo

# POST 请求
gh api repos/owner/repo/issues -f title="Bug" -f body="问题描述"

# PATCH 请求
gh api repos/owner/repo/issues/123 -f state="closed"

# DELETE 请求
gh api repos/owner/repo/issues/123 -X DELETE

# 指定 API 版本
gh api --method GET repos/owner/repo
```

### 8.2 请求参数

```bash
# 添加查询参数
gh api "repos/owner/repo/issues?state=open&per_page=10"

# 添加请求体字段
gh api repos/owner/repo/issues \
  -f title="Issue Title" \
  -f body="Issue Body" \
  -f "labels[]=bug" \
  -f "labels[]=priority:high"

# 添加请求头
gh api repos/owner/repo \
  -H "Accept: application/vnd.github.v3+json"

# 从文件读取请求体
gh api repos/owner/repo/issues --input issue.json

# 使用 JSON 格式
gh api repos/owner/repo/issues \
  --method POST \
  --input - <<EOF
{
  "title": "Bug Report",
  "body": "描述问题",
  "labels": ["bug"]
}
EOF
```

### 8.3 响应处理

```bash
# 以 JSON 格式输出
gh api repos/owner/repo --jq '.name'

# 提取特定字段
gh api repos/owner/repo --jq '.full_name, .description'

# 格式化输出
gh api repos/owner/repo --jq '{name: .name, desc: .description}'

# 列表处理
gh api repos/owner/repo/issues --jq '.[].title'

# 使用 @ghjson 处理
gh api repos/owner/repo/issues --jq '.[] | "\(.number): \(.title)"'

# 获取分页数据
gh api --paginate repos/owner/repo/issues

# 限制分页数量
gh api --paginate repos/owner/repo/issues --per-page 5

# 获取所有页面（自动处理分页）
gh api --paginate "repos/owner/repo/contributors" --jq '.[].login'
```

### 8.4 GraphQL 查询

```bash
# GraphQL 查询
gh api graphql -f query='
{
  repository(owner: "owner", name: "repo") {
    name
    description
    stargazerCount
  }
}
'

# 带变量的 GraphQL
gh api graphql -f query='
query($owner: String!, $repo: String!) {
  repository(owner: $owner, name: $repo) {
    name
    pullRequests(first: 10, states: OPEN) {
      nodes {
        title
        author {
          login
        }
      }
    }
  }
}
' -f owner="owner" -f repo="repo"

# 获取当前用户信息
gh api graphql -f query='{ viewer { login name email } }'

# 获取仓库的 Issues（使用 GraphQL）
gh api graphql -f query='
{
  repository(owner: "owner", name: "repo") {
    issues(first: 10, states: OPEN) {
      nodes {
        number
        title
        createdAt
      }
    }
  }
}
'
```

### 8.5 常用 API 端点

```bash
# 获取当前用户
gh api user

# 获取用户的仓库
gh api users/username/repos

# 获取仓库的 Issues
gh api repos/owner/repo/issues

# 获取仓库的 PRs
gh api repos/owner/repo/pulls

# 获取仓库的 Releases
gh api repos/owner/repo/releases

# 获取仓库的 Tags
gh api repos/owner/repo/tags

# 获取仓库的 Contributors
gh api repos/owner/repo/contributors

# 获取仓库的 Languages
gh api repos/owner/repo/languages

# 获取仓库的 Workflows
gh api repos/owner/repo/actions/workflows

# 获取 Workflow 的 Runs
gh api repos/owner/repo/actions/workflows/deploy.yml/runs

# 获取用户的 Notifications
gh api notifications

# 获取用户的 Organizations
gh api user/orgs

# 获取 Organization 的 Members
gh api orgs/my-org/members

# 获取团队信息
gh api orgs/my-org/teams
```

---

## 9. gh alias 自定义别名

### 9.1 创建别名

```bash
# 创建简单的命令别名
gh alias set pv 'pr view'

# 使用别名
gh pv 123  # 等同于 gh pr view 123

# 创建带参数的别名
gh alias set prs 'pr list --state=open --author=@me'

# 使用别名
gh prs  # 列出自己创建的打开状态 PR

# 创建复杂的别名（使用 shell 命令）
gh alias set my-repos 'api user/repos --jq ".[].full_name"'

# 创建多行别名
gh alias set my-status '!gh pr status && echo "---" && gh issue list --assignee=@me'

# 创建带参数替换的别名
gh alias set pr-info '!f() { gh pr view $1 --json title,body,state; }; f'

# 使用环境变量
gh alias set whoami 'api user --jq ".login"'
```

### 9.2 管理别名

```bash
# 列出所有别名
gh alias list

# 删除别名
gh alias delete pv

# 编辑别名配置文件
gh alias edit

# 导出别名
gh alias list > aliases.txt

# 导入别名（从配置文件）
# 编辑 ~/.config/gh/config.yml
```

### 9.3 实用别名示例

```bash
# 快速查看自己的 PR
gh alias set my-prs 'pr list --author=@me --state=open'

# 快速查看需要审阅的 PR
gh alias set review-prs 'pr list --reviewer=@me --state=open'

# 快速查看分配给自己的 Issue
gh alias set my-issues 'issue list --assignee=@me --state=open'

# 快速合并 PR（squash）
gh alias set pr-merge-squash '!f() { gh pr merge $1 --squash --delete-branch; }; f'

# 快速创建 Issue
gh alias set bug '!f() { gh issue create --title "$1" --body "$2" --label bug; }; f'

# 快速查看仓库信息
gh alias set repo-info 'api repos/{owner}/{repo} --jq "{name: .name, stars: .stargazers_count, forks: .forks_count}"'

# 批量关闭 Issue
gh alias set close-all '!gh issue list --state=open --json number --jq ".[].number" | xargs -I {} gh issue close {}'

# 快速查看 workflow 状态
gh alias set wf-status 'run list --limit=5 --json name,status,conclusion --jq ".[] | \"\(.name): \(.status) - \(.conclusion)\""'

# 快速 fork 并 clone
gh alias set fork-clone '!f() { gh repo fork $1 --clone; }; f'

# 查看仓库贡献者
gh alias set contributors 'api repos/{owner}/{repo}/contributors --jq ".[] | \"\(.login): \(.contributions) commits\""'
```

---

## 10. gh 扩展系统

### 10.1 什么是 gh 扩展

gh 扩展是用任何编程语言编写的命令行工具，可以无缝集成到 gh 中。扩展命名格式为 `gh-<name>`。

### 10.2 管理扩展

```bash
# 浏览可用扩展
gh extension list

# 安装扩展
gh extension install owner/gh-extension-name

# 从特定分支安装
gh extension install owner/gh-extension-name --branch main

# 升级扩展
gh extension upgrade gh-extension-name

# 升级所有扩展
gh extension upgrade --all

# 卸载扩展
gh extension remove gh-extension-name

# 查看扩展信息
gh extension browse gh-extension-name
```

### 10.3 常用扩展推荐

```bash
# gh-dash - 终端仪表板
gh extension install dlvhdr/gh-dash

# gh-copilot - AI 助手
gh extension install github/gh-copilot

# gh-poi - 清理已合并的分支
gh extension install seachicken/gh-poi

# gh-notify - 通知管理
gh extension install meiji163/gh-notify

# gh-stars - 星标仓库管理
gh extension install gaowei2/gh-stars

# gh-user-status - 用户状态
gh extension install vilmibm/gh-user-status

# gh-repo-explore - 仓库浏览
gh extension install samcoe/gh-repo-explore

# gh-actions-cache - Actions 缓存管理
gh extension install actions/gh-actions-cache

# gh-eco - 生态系统浏览
gh extension install github/gh-eco
```

### 10.4 创建自己的扩展

```bash
# 创建 bash 扩展
cat > gh-hello << 'EOF'
#!/bin/bash
# gh hello - 打招呼扩展
echo "Hello from gh extension!"
echo "Arguments: $@"
EOF
chmod +x gh-hello

# 安装本地扩展
gh extension install .

# 测试扩展
gh hello world

# 创建 Go 扩展（使用 gh 的库）
# 参考：https://github.com/cli/go-gh

# 发布扩展
# 1. 创建名为 gh-<name> 的公开仓库
# 2. 推送代码
# 3. 其他用户可以通过 gh extension install owner/gh-<name> 安装
```

---

## 11. gh 与 Git 命令的配合

### 11.1 仓库克隆与初始化

```bash
# 使用 gh 克隆（支持简写）
gh clone owner/repo  # 等同于 git clone https://github.com/owner/repo.git

# Fork 后克隆
gh repo fork owner/repo --clone

# 同步 fork
gh repo sync

# 创建仓库并推送
gh repo create my-project --public --source=. --push
```

### 11.2 分支管理配合

```bash
# 创建分支并推送
git checkout -b feature/new-feature
git push -u origin feature/new-feature
gh pr create  # 直接创建 PR

# 检出 PR 分支
gh pr checkout 123
# 这会自动创建本地分支并切换

# PR 合并后清理
gh pr merge 123 --squash --delete-branch
# 自动删除远程和本地分支
```

### 11.3 提交与 PR 关联

```bash
# 在提交信息中引用 Issue
git commit -m "fix: 修复登录问题

Closes #123"

# 推送并创建 PR
git push origin feature/fix-login
gh pr create --body "Closes #123"

# 使用 gh 自动关联
gh pr create --fill-first  # 使用第一个 commit 作为 PR 标题
```

### 11.4 工作流示例

```bash
# 完整的 feature 分支工作流

# 1. 同步主分支
git checkout main
git pull origin main

# 2. 创建 feature 分支
git checkout -b feature/user-auth

# 3. 开发和提交
git add .
git commit -m "feat: 实现用户认证模块"

# 4. 推送分支
git push -u origin feature/user-auth

# 5. 创建 PR
gh pr create \
  --title "feat: 用户认证功能" \
  --body "## 变更内容\n\n- 实现登录/注册\n- 添加 JWT 验证\n\nCloses #45" \
  --reviewer team-lead \
  --label "feature,needs-review"

# 6. 查看 PR 状态
gh pr status

# 7. 等待审阅和 CI
gh pr checks 456 --watch

# 8. 合并 PR
gh pr merge 456 --squash --delete-branch

# 9. 更新本地
git checkout main
git pull origin main
git branch -d feature/user-auth
```

---

## 12. 实用场景与技巧

### 12.1 快速创建 Issue 模板

```bash
# 创建 Issue 模板
gh alias set bug-report '!f() {
  gh issue create \
    --title "Bug: $1" \
    --body "## 问题描述\n\n$1\n\n## 复现步骤\n\n1. \n2. \n3. \n\n## 期望行为\n\n\n\n## 实际行为\n\n\n\n## 环境信息\n\n- OS: \n- Browser: \n- Version: " \
    --label "bug"
}; f'

# 使用
gh bug-report "登录页面无法加载"
```

### 12.2 批量操作

```bash
# 批量关闭过期 Issue（超过 30 天无活动）
gh issue list --state=open --json number,updatedAt --jq '
  .[] | select((now - (.updatedAt | fromdate)) > 2592000) | .number
' | xargs -I {} gh issue close {} --comment "自动关闭：超过 30 天无活动"

# 批量添加标签
gh issue list --state=open --json number --jq '.[].number' | \
  xargs -I {} gh issue edit {} --add-label "triage"

# 批量下载 Release Assets
gh release list --json tagName --jq '.[].tagName' | \
  xargs -I {} gh release download {} --pattern "*.zip" --dir ./downloads/{}
```

### 12.3 监控和通知

```bash
# 创建 PR 状态检查脚本
#!/bin/bash
echo "=== 我的 PR 状态 ==="
gh pr list --author=@me --json number,title,statusCheckRollup --jq '
  .[] | "\(.number) \(.title) - \(if .statusCheckRollup | length > 0 then .statusCheckRollup[0].conclusion else "pending" end)"
'

echo ""
echo "=== 需要审阅的 PR ==="
gh pr list --reviewer=@me --json number,title --jq '.[] | "\(.number) \(.title)"'

echo ""
echo "=== 我的 Issue ==="
gh issue list --assignee=@me --json number,title --jq '.[] | "\(.number) \(.title)"'
```

### 12.4 自动化脚本

```bash
# 每日报告脚本
#!/bin/bash
DATE=$(date +%Y-%m-%d)
REPORT="daily-report-$DATE.md"

{
  echo "# 每日开发报告 - $DATE"
  echo ""
  echo "## 今日合并的 PR"
  gh search prs --merged=$(date +%Y-%m-%d) --author=@me --json number,title --jq '.[] | "- #\(.number) \(.title)"'
  echo ""
  echo "## 今日创建的 Issue"
  gh search issues --created=$(date +%Y-%m-%d) --author=@me --json number,title --jq '.[] | "- #\(.number) \(.title)"'
  echo ""
  echo "## 待处理的 PR"
  gh pr list --reviewer=@me --state=open --json number,title --jq '.[] | "- #\(.number) \(.title)"'
} > $REPORT

echo "报告已生成: $REPORT"
```

### 12.5 配置文件

```bash
# 查看配置
gh config list

# 设置默认编辑器
gh config set editor vim
gh config set editor code  # VS Code

# 设置默认浏览器
gh config set browser firefox

# 设置默认协议
gh config set git_protocol https

# 设置默认提示
gh config set prompt enabled

# 设置默认别名
gh config set aliases.my-alias 'pr list --author=@me'

# 配置文件位置
# ~/.config/gh/config.yml
# ~/.config/gh/hosts.yml
```

---

## 13. 国内使用注意事项

### 13.1 网络问题解决方案

**方案一：使用代理**

```bash
# 设置 HTTP 代理
export HTTP_PROXY=http://127.0.0.1:7890
export HTTPS_PROXY=http://127.0.0.1:7890

# 设置 gh 代理
gh config set http_proxy http://127.0.0.1:7890
gh config set https_proxy http://127.0.0.1:7890

# 或者在配置文件中设置
# ~/.config/gh/config.yml
# http_proxy: http://127.0.0.1:7890
# https_proxy: http://127.0.0.1:7890
```

**方案二：使用 ghproxy**

```bash
# 克隆时使用 ghproxy
gh repo clone owner/repo -- --config url."https://ghproxy.com/https://github.com/".insteadOf="https://github.com/"

# 或者全局配置
git config --global url."https://ghproxy.com/https://github.com/".insteadOf "https://github.com/"
```

**方案三：配置 Git 使用镜像**

```bash
# 使用 gitee 镜像（如果仓库已镜像）
git config --global url."https://gitee.com/".insteadOf "https://github.com/"

# 只对特定域名配置
git config --global url."https://ghproxy.com/https://github.com/".insteadOf "https://github.com/"
```

### 13.2 Token 配置技巧

```bash
# 手动配置 Token（避免网络问题）
# 1. 在 GitHub 网站创建 Personal Access Token
#    Settings → Developer settings → Personal access tokens → Tokens (classic)
#    权限：repo, read:org, workflow

# 2. 使用 Token 登录
echo "ghp_your_token_here" | gh auth login --with-token

# 3. 或者设置环境变量
echo 'export GH_TOKEN=ghp_your_token_here' >> ~/.bashrc
source ~/.bashrc

# 4. 验证登录状态
gh auth status
```

### 13.3 常见问题排查

```bash
# 问题1: 网络连接超时
# 解决: 使用代理或 ghproxy

# 问题2: 权限不足
# 解决: 刷新 Token 权限
gh auth refresh -s admin:org,repo,workflow

# 问题3: API 速率限制
# 解决: 使用认证的 Token（认证用户限制更高）
gh api rate_limit

# 问题4: SSH 连接问题
# 解决: 使用 HTTPS 协议
gh config set git_protocol https

# 问题5: 证书问题
# 解决: 配置 Git 跳过 SSL 验证（不推荐生产环境）
git config --global http.sslVerify false
```

### 13.4 国内替代方案

如果 gh 实在无法使用，可以考虑：

```bash
# 1. 使用 GitHub API 直接调用
curl -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/owner/repo

# 2. 使用 GitLab CLI (glab)
# GitLab 在国内访问相对稳定
brew install glab

# 3. 使用 Gitee API
# 如果项目托管在 Gitee
curl https://gitee.com/api/v5/repos/owner/repo

# 4. 使用 FastGit 镜像
# https://hub.fastgit.xyz/
```

### 13.5 最佳实践

```bash
# 1. 始终使用 Token 认证（比密码更安全）
gh auth login --with-token < token.txt

# 2. 定期刷新 Token
gh auth refresh

# 3. 使用 ghproxy 加速
git config --global url."https://ghproxy.com/https://github.com/".insteadOf "https://github.com/"

# 4. 配置代理
export HTTPS_PROXY=http://127.0.0.1:7890

# 5. 使用 SSH 协议（如果 SSH 可用）
gh config set git_protocol ssh

# 6. 缓存认证信息
gh auth setup-git

# 7. 使用 gh 的 --json 输出，减少 API 调用
gh pr list --json number,title,state

# 8. 使用缓存减少 API 调用
gh api repos/owner/repo --cache 1h
```

---

## 总结

本章全面介绍了 GitHub CLI (gh) 的使用方法：

1. **安装配置**：官方和国内多种安装方案
2. **基础命令**：认证、仓库、浏览器操作
3. **Issue 管理**：创建、查看、更新、搜索 Issue
4. **PR 管理**：完整的 Pull Request 工作流
5. **Release 管理**：创建和管理发布版本
6. **Actions 管理**：工作流和运行监控
7. **Codespaces 管理**：云端开发环境操作
8. **API 高级用法**：REST 和 GraphQL 调用
9. **自定义别名**：提升效率的快捷命令
10. **扩展系统**：安装和创建扩展
11. **Git 配合**：与 Git 命令的协作
12. **实用技巧**：批量操作、自动化脚本
13. **国内注意事项**：网络问题和替代方案

掌握 GitHub CLI，你可以在终端中完成几乎所有 GitHub 操作，大幅提升开发效率，告别频繁切换浏览器的烦恼。
