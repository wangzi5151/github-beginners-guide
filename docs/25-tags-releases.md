# Git 标签与 GitHub Releases 完全指南

## 概述

在软件开发中，版本管理是至关重要的一环。Git 标签（Tag）和 GitHub Releases 是标记项目里程碑、管理发布版本的核心工具。本章将深入讲解 Git 标签的各种用法、GitHub Releases 的完整操作流程，以及如何构建自动化发布体系。

---

## 1. Git 标签概念

### 1.1 什么是标签

标签是 Git 中用于标记特定提交（commit）的引用，通常用于标识版本发布点。与分支不同，标签指向的提交是固定的，不会随新的提交而移动。

标签的本质是一个指向特定 commit 的不可变指针，它为该 commit 起一个人类可读的名字，方便日后回溯和引用。

### 1.2 轻量标签（Lightweight Tag）

轻量标签是最简单的标签形式，它只是一个指向特定提交的指针，不包含任何额外信息。

```bash
# 创建轻量标签
git tag v1.0.0
```

轻量标签的特点：
- 不存储标签创建者信息
- 不存储创建时间
- 不存储标签消息
- 不支持 GPG 签名
- 本质上就是一个不会移动的分支引用

### 1.3 附注标签（Annotated Tag）

附注标签是 Git 中推荐使用的标签类型，它存储在 Git 数据库中的完整对象。

```bash
# 创建附注标签
git tag -a v1.0.0 -m "正式发布 1.0.0 版本"
```

附注标签的特点：
- 存储标签创建者姓名和邮箱
- 存储创建日期和时间
- 存储标签消息（说明文字）
- 支持 GPG 签名验证
- 可以被校验和验证完整性

### 1.4 两种标签的对比

| 特性 | 轻量标签 | 附注标签 |
|------|---------|---------|
| 创建方式 | `git tag <name>` | `git tag -a <name> -m "msg"` |
| 存储为 Git 对象 | 否 | 是 |
| 包含作者信息 | 否 | 是 |
| 包含时间戳 | 否 | 是 |
| 包含消息 | 否 | 是 |
| 支持 GPG 签名 | 否 | 是 |
| 推荐用于发布 | 否 | 是 |
| 占用空间 | 极小 | 较小 |

**最佳实践**：在正式项目中，始终使用附注标签来标记版本发布，轻量标签仅用于临时标记或个人用途。

---

## 2. 创建、列出、删除标签

### 2.1 创建标签

```bash
# 创建轻量标签
git tag v1.0.0

# 创建附注标签
git tag -a v1.0.0 -m "Release version 1.0.0"

# 为历史提交创建标签
git tag -a v0.9.0 abc1234 -m "Beta version 0.9.0"

# 使用完整的提交哈希创建标签
git tag -a v0.8.0 9fceb02d0ae7906dd5e9f9ef49e8c01e51a1b33b

# 在特定提交上创建标签
git log --oneline
# 输出：
# a1b2c3d 完成登录功能
# e4f5g6h 添加用户模块
# i7j8k9l 初始提交

git tag -a v0.1.0 i7j8k9l -m "项目初始化"
```

### 2.2 列出标签

```bash
# 列出所有标签
git tag
# 输出：
# v0.1.0
# v0.9.0
# v1.0.0

# 按模式筛选标签
git tag -l "v1.*"
# 输出：
# v1.0.0
# v1.0.1
# v1.1.0

git tag -l "v*.*.*"
# 输出所有语义化版本标签

# 查看标签详细信息（附注标签）
git show v1.0.0
# 输出：
# tag v1.0.0
# Tagger: 张三 <zhangsan@example.com>
# Date:   Mon Jan 1 12:00:00 2024 +0800
#
# Release version 1.0.0
#
# commit a1b2c3d...
# Author: 张三 <zhangsan@example.com>
# ...

# 查看轻量标签信息
git show v1.0.0-lightweight
# 只显示 commit 信息，没有 tag 元数据

# 按版本排序列出标签
git tag -l --sort=-version:refname
# 输出：
# v2.0.0
# v1.2.0
# v1.1.0
# v1.0.0

# 查看标签指向的提交
git rev-parse v1.0.0
# 输出完整的 commit hash
```

### 2.3 删除标签

```bash
# 删除本地标签
git tag -d v1.0.0
# 输出：Deleted tag 'v1.0.0'

# 删除远程标签（旧方法）
git push origin :refs/tags/v1.0.0

# 删除远程标签（推荐方法）
git push origin --delete v1.0.0

# 批量删除本地标签
git tag -l "v0.*" | xargs git tag -d

# 批量删除远程标签
git tag -l "v0.*" | xargs -I {} git push origin --delete {}
```

### 2.4 重命名标签

Git 没有直接重命名标签的命令，需要先删除再创建：

```bash
# 重命名标签的步骤
# 1. 获取旧标签指向的提交
COMMIT=$(git rev-list -n 1 old-tag-name)

# 2. 删除旧标签
git tag -d old-tag-name
git push origin --delete old-tag-name

# 3. 在同一提交上创建新标签
git tag -a new-tag-name $COMMIT -m "重命名自 old-tag-name"

# 4. 推送新标签
git push origin new-tag-name
```

### 2.5 检出标签

```bash
# 检出标签对应的代码（进入分离头指针状态）
git checkout v1.0.0

# 基于标签创建新分支（推荐用于修复旧版本）
git checkout -b hotfix-v1.0.0 v1.0.0

# 切换回主分支
git checkout main
```

---

## 3. 标签签名（GPG Signing）

### 3.1 为什么需要签名

GPG 签名可以验证标签的真实性和完整性，确保标签确实由项目的维护者创建，且内容未被篡改。这对于开源项目的安全性至关重要。

### 3.2 配置 GPG 密钥

```bash
# 检查是否已有 GPG 密钥
gpg --list-keys

# 生成新的 GPG 密钥
gpg --full-generate-key

# 按提示选择：
# 1. 密钥类型：RSA and RSA
# 2. 密钥长度：4096
# 3. 有效期：0（永不过期）或指定时间
# 4. 姓名、邮箱（必须与 Git 配置一致）
# 5. 设置密码

# 查看 GPG 密钥 ID
gpg --list-secret-keys --keyid-format=long
# 输出：
# sec   rsa4096/ABCDEF1234567890 2024-01-01 [SC]
#       1234567890ABCDEF1234567890ABCDEF12345678
# uid                 [ultimate] 张三 <zhangsan@example.com>
# ssb   rsa4096/1234567890ABCDEF 2024-01-01 [E]

# 导出公钥（用于上传到 GitHub）
gpg --armor --export ABCDEF1234567890
```

### 3.3 配置 Git 使用 GPG

```bash
# 设置 Git 使用指定的 GPG 密钥
git config --global user.signingkey ABCDEF1234567890

# 设置自动签名所有标签
git config --global tag.forceSignAnnotated true

# 设置 GPG 程序路径（如使用 GPG2）
git config --global gpg.program /usr/bin/gpg2
```

### 3.4 创建签名标签

```bash
# 创建签名标签（交互式输入密码）
git tag -s v1.0.0 -m "Signed release v1.0.0"

# 创建签名标签并指定密钥
git tag -u ABCDEF1234567890 v1.0.0 -m "Signed release v1.0.0"

# 验证签名标签
git tag -v v1.0.0
# 输出包含：
# gpg: Good signature from "张三 <zhangsan@example.com>"
```

### 3.5 在 GitHub 上添加 GPG 公钥

```bash
# 步骤：
# 1. 导出公钥
gpg --armor --export ABCDEF1234567890

# 2. 复制输出内容（包含 BEGIN 和 END 行）

# 3. 在 GitHub 上添加：
#    Settings -> SSH and GPG keys -> New GPG key
#    粘贴公钥内容并保存

# 4. 验证标签在 GitHub 上显示为 "Verified"
```

---

## 4. GitHub Releases 详解

### 4.1 什么是 GitHub Release

GitHub Release 是基于 Git 标签构建的发布管理功能，它提供了：

- 一个正式的版本发布页面
- Release Notes（发布说明）
- 二进制文件附件（Assets）
- 预发布版本标记
- 草稿发布功能
- 社区参与（讨论、反馈）

### 4.2 Release 与 Tag 的关系

- 每个 Release 都必须基于一个 Tag
- 一个 Tag 只能对应一个 Release
- Release 包含 Tag 之外的额外信息（说明、附件等）
- 可以在没有 Release 的情况下使用 Tag，但反过来不行

### 4.3 Release 的生命周期

```
创建草稿 → 编辑内容 → 发布（正式/预发布）→ 编辑/删除
    ↓
 Draft   →  Edit  →  Publish    →  Update/Delete
```

---

## 5. 创建 Release 和 Release Notes

### 5.1 通过网页界面创建 Release

**步骤详解：**

1. 进入 GitHub 仓库页面
2. 点击右侧边栏的 "Releases" 链接
3. 点击 "Draft a new release" 或 "Create a new release"
4. 填写以下信息：
   - **Choose a tag**: 选择现有标签或创建新标签
   - **Target**: 选择目标分支（创建新标签时）
   - **Release title**: 发布标题（通常是标签名）
   - **Describe this release**: 详细的发布说明
   - **Attach binaries**: 上传二进制文件
   - **This is a pre-release**: 标记为预发布
   - **Create a discussion**: 为发布创建讨论区
5. 点击 "Publish release" 或 "Save draft"

### 5.2 通过网页界面编写 Release Notes

**编写模板：**

```markdown
## 🚀 新功能 (What's New)

- 新增用户登录功能
- 支持 OAuth 2.0 第三方登录
- 添加深色模式主题

## 🐛 问题修复 (Bug Fixes)

- 修复首页加载缓慢的问题 (#123)
- 修复移动端显示错位 (#124)

## ⚡ 性能优化 (Performance)

- 优化图片加载速度，减少 50% 加载时间
- 数据库查询优化

## 📦 依赖更新 (Dependencies)

- 升级 React 到 18.2.0
- 升级 Node.js 到 20.x

## 💔 破坏性变更 (Breaking Changes)

- API 接口 /api/v1/users 改为 /api/v2/users
- 移除已废弃的 legacy 模块

## 🙏 致谢 (Contributors)

感谢以下贡献者的参与：
@contributor1, @contributor2, @contributor3

**完整更新日志**: https://github.com/user/repo/compare/v1.0.0...v1.1.0
```

### 5.3 通过 GitHub CLI 创建 Release

```bash
# 创建简单的 Release
gh release create v1.0.0 --title "v1.0.0" --notes "首次正式发布"

# 创建带详细说明的 Release
gh release create v1.0.0 \
  --title "v1.0.0 - 正式版" \
  --notes-file CHANGELOG.md

# 创建带附件的 Release
gh release create v1.0.0 \
  --title "v1.0.0" \
  --notes "正式发布" \
  ./dist/app-linux.tar.gz \
  ./dist/app-macos.zip \
  ./dist/app-windows.exe

# 创建草稿 Release
gh release create v1.0.0 \
  --title "v1.0.0" \
  --notes "待发布" \
  --draft

# 创建预发布版本
gh release create v1.0.0-beta.1 \
  --title "v1.0.0 Beta 1" \
  --notes "测试版本" \
  --prerelease
```

### 5.4 通过 GitHub API 创建 Release

```bash
# 使用 curl 调用 API
curl -X POST \
  -H "Authorization: token YOUR_TOKEN" \
  -H "Accept: application/vnd.github.v3+json" \
  https://api.github.com/repos/OWNER/REPO/releases \
  -d '{
    "tag_name": "v1.0.0",
    "target_commitish": "main",
    "name": "v1.0.0",
    "body": "## 新功能\n\n- 功能1\n- 功能2",
    "draft": false,
    "prerelease": false,
    "generate_release_notes": true
  }'
```

---

## 6. 自动生成 Release Notes

### 6.1 GitHub 内置自动生成功能

GitHub 提供了基于 PR 和 commit 的自动 Release Notes 生成：

```bash
# 使用 CLI 启用自动生成
gh release create v1.0.0 --generate-notes

# 使用 API 启用自动生成
curl -X POST \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/OWNER/REPO/releases \
  -d '{
    "tag_name": "v1.0.0",
    "generate_release_notes": true
  }'
```

### 6.2 配置自动 Release Notes 模板

在仓库根目录创建 `.github/release.yml` 文件：

```yaml
# .github/release.yml
changelog:
  exclude:
    labels:
      - ignore-for-release
    authors:
      - dependabot
      - github-actions
  categories:
    - title: 🚀 新功能
      labels:
        - enhancement
        - feature
    - title: 🐛 问题修复
      labels:
        - bug
        - fix
        - bugfix
    - title: ⚡ 性能优化
      labels:
        - performance
        - optimization
    - title: 📚 文档更新
      labels:
        - documentation
        - docs
    - title: 🔧 维护工作
      labels:
        - chore
        - maintenance
        - dependencies
    - title: 💔 破坏性变更
      labels:
        - breaking-change
        - major
    - title: 🆕 其他变更
      labels:
        - "*"
```

### 6.3 使用 Release Drafter

Release Drafter 是一个 GitHub Action，可以在每次合并 PR 时自动更新草稿 Release。

**配置文件 `.github/release-drafter.yml`：**

```yaml
# .github/release-drafter.yml
name-template: 'v$RESOLVED_VERSION 🌈'
tag-template: 'v$RESOLVED_VERSION'

categories:
  - title: '🚀 新功能'
    labels:
      - 'feature'
      - 'enhancement'
  - title: '🐛 Bug 修复'
    labels:
      - 'fix'
      - 'bugfix'
      - 'bug'
  - title: '🧰 维护'
    labels:
      - 'chore'
      - 'dependencies'
  - title: '📖 文档'
    labels:
      - 'documentation'
      - 'docs'

change-template: '- $TITLE @$AUTHOR (#$NUMBER)'
change-title-escapes: '\<*_&'
no-changes-template: '无变更。'

version-resolver:
  major:
    labels:
      - 'major'
      - 'breaking-change'
  minor:
    labels:
      - 'minor'
      - 'feature'
      - 'enhancement'
  patch:
    labels:
      - 'patch'
      - 'fix'
      - 'bugfix'
      - 'bug'
  default: patch

template: |
  ## 变更内容

  $CHANGES

  ## 安装

  ```bash
  npm install my-package@$RESOLVED_VERSION
  ```

  **完整更新日志**: https://github.com/$OWNER/$REPOSITORY/compare/$PREVIOUS_TAG...v$RESOLVED_VERSION

autolabeler:
  - label: 'feature'
    title:
      - '/^feat/i'
  - label: 'bug'
    title:
      - '/^fix/i'
  - label: 'documentation'
    title:
      - '/^docs/i'
  - label: 'dependencies'
    title:
      - '/^deps/i'
```

**GitHub Action 配置 `.github/workflows/release-drafter.yml`：**

```yaml
# .github/workflows/release-drafter.yml
name: Release Drafter

on:
  push:
    branches:
      - main
  pull_request:
    types:
      - opened
      - reopened
      - synchronize

permissions:
  contents: read
  pull-requests: write

jobs:
  update_release_draft:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
    steps:
      - uses: release-drafter/release-drafter@v5
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### 6.4 使用 Conventional Changelog

```bash
# 安装 conventional-changelog-cli
npm install -g conventional-changelog-cli

# 生成完整的 changelog
conventional-changelog -p angular -i CHANGELOG.md -s

# 从上一个 tag 生成
conventional-changelog -p angular -i CHANGELOG.md -s -r 1

# 使用 conventional-github-releaser 直接创建 Release
npm install -g conventional-github-releaser
conventional-github-releaser -p angular -t $GITHUB_TOKEN
```

**配置文件 `.conventional-changelog.json`：**

```json
{
  "types": [
    { "type": "feat", "section": "🚀 新功能" },
    { "type": "fix", "section": "🐛 问题修复" },
    { "type": "perf", "section": "⚡ 性能优化" },
    { "type": "docs", "section": "📖 文档" },
    { "type": "chore", "hidden": true },
    { "type": "style", "hidden": true },
    { "type": "refactor", "section": "♻️ 代码重构" },
    { "type": "test", "hidden": true },
    { "type": "ci", "hidden": true }
  ]
}
```

---

## 7. Release Assets（附件管理）

### 7.1 什么是 Release Assets

Release Assets 是附加到 Release 上的二进制文件，常见用途包括：

- 编译好的程序（可执行文件、安装包）
- 压缩的源代码包
- 文档（PDF、HTML）
- 配置文件模板
- Docker 镜像的 tar 包

### 7.2 通过网页上传 Assets

1. 编辑或创建 Release
2. 在 "Attach binaries" 区域拖拽文件或点击选择
3. 等待上传完成
4. 保存 Release

### 7.3 通过 CLI 管理 Assets

```bash
# 创建 Release 并上传文件
gh release create v1.0.0 \
  ./dist/app-linux-amd64 \
  ./dist/app-linux-arm64 \
  ./dist/app-darwin-amd64 \
  ./dist/app-windows-amd64.exe \
  --title "v1.0.0" \
  --notes "跨平台发布"

# 向现有 Release 添加文件
gh release upload v1.0.0 ./new-asset.zip

# 替换现有同名文件
gh release upload v1.0.0 ./app-linux --clobber

# 删除 Asset
gh release delete-asset v1.0.0 old-file.zip --yes

# 下载 Asset
gh release download v1.0.0 --pattern "*.zip"

# 下载到指定目录
gh release download v1.0.0 --dir ./downloads --pattern "app-*"

# 只下载特定文件
gh release download v1.0.0 --pattern "app-linux-*"
```

### 7.4 通过 API 管理 Assets

```bash
# 上传 Asset
curl -X POST \
  -H "Authorization: token $GITHUB_TOKEN" \
  -H "Content-Type: application/octet-stream" \
  "https://uploads.github.com/repos/OWNER/REPO/releases/RELEASE_ID/assets?name=myfile.zip" \
  --data-binary @myfile.zip

# 列出 Assets
curl -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/OWNER/REPO/releases/RELEASE_ID/assets

# 删除 Asset
curl -X DELETE \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/OWNER/REPO/releases/assets/ASSET_ID
```

### 7.5 Asset 命名最佳实践

```
推荐命名格式：
<项目名>-<版本号>-<平台>-<架构>[.<扩展名>]

示例：
myapp-1.0.0-linux-amd64.tar.gz
myapp-1.0.0-linux-arm64.tar.gz
myapp-1.0.0-darwin-amd64.tar.gz
myapp-1.0.0-darwin-arm64.tar.gz (Apple Silicon)
myapp-1.0.0-windows-amd64.zip
myapp-1.0.0-windows-arm64.zip

包含校验和文件：
myapp-1.0.0-checksums.txt
myapp-1.0.0-checksums.sha256
```

---

## 8. 预发布版本（Pre-release）

### 8.1 预发布版本的概念

预发布版本用于在正式发布前让用户测试新功能，常见的预发布标识：

| 标识 | 含义 | 示例 |
|------|------|------|
| alpha | 内部测试版本 | v1.0.0-alpha.1 |
| beta | 公开测试版本 | v1.0.0-beta.1 |
| rc | 候选发布版本 | v1.0.0-rc.1 |
| dev | 开发版本 | v1.0.0-dev.1 |
| canary | 金丝雀版本 | v1.0.0-canary.1 |

### 8.2 创建预发布版本

```bash
# 通过 CLI 创建
gh release create v2.0.0-beta.1 \
  --title "v2.0.0 Beta 1" \
  --notes "## ⚠️ 这是测试版本\n\n请勿在生产环境使用。" \
  --prerelease

# 通过网页创建
# 1. 创建 Release
# 2. 勾选 "This is a pre-release" 选项
# 3. 发布

# 通过 API 创建
curl -X POST \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/OWNER/REPO/releases \
  -d '{
    "tag_name": "v2.0.0-beta.1",
    "name": "v2.0.0 Beta 1",
    "prerelease": true
  }'
```

### 8.3 预发布版本的显示

- 在 Releases 页面，预发布版本会有明显标识
- 默认不显示在最新 Release 中
- npm 等包管理器可以通过 `@beta` 标签安装

```bash
# npm 安装预发布版本
npm install my-package@beta
npm install my-package@2.0.0-beta.1

# 查看所有预发布版本
npm versions --json
```

---

## 9. Draft Release（草稿发布）

### 9.1 Draft 的用途

- 提前准备 Release 内容，等时机成熟再发布
- 配合 CI/CD 自动填充 Release Notes
- 团队协作审核 Release 内容
- 批量准备多个版本的 Release

### 9.2 管理 Draft Release

```bash
# 创建 Draft Release
gh release create v1.1.0 \
  --title "v1.1.0" \
  --notes "开发中..." \
  --draft

# 列出所有 Release（包括 Draft）
gh release list --limit 20

# 查看 Draft Release 详情
gh release view v1.1.0

# 编辑 Draft Release
gh release edit v1.1.0 \
  --title "v1.1.0 - 新版本" \
  --notes-file RELEASE_NOTES.md

# 发布 Draft Release（取消草稿状态）
gh release edit v1.1.0 --draft=false

# 删除 Draft Release
gh release delete v1.1.0 --yes
```

### 9.3 自动化 Draft Release 工作流

```yaml
# .github/workflows/auto-release-draft.yml
name: Auto Release Draft

on:
  push:
    tags:
      - 'v*'

jobs:
  create-release:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Generate Changelog
        id: changelog
        run: |
          # 获取上一个 tag
          PREV_TAG=$(git describe --tags --abbrev=0 HEAD^ 2>/dev/null || echo "")
          if [ -z "$PREV_TAG" ]; then
            CHANGES=$(git log --oneline --no-merges)
          else
            CHANGES=$(git log --oneline --no-merges ${PREV_TAG}..HEAD)
          fi
          echo "changes<<EOF" >> $GITHUB_OUTPUT
          echo "$CHANGES" >> $GITHUB_OUTPUT
          echo "EOF" >> $GITHUB_OUTPUT

      - name: Create Draft Release
        uses: softprops/action-gh-release@v1
        with:
          draft: true
          generate_release_notes: true
          body: |
            ## 变更内容

            ${{ steps.changelog.outputs.changes }}

            ## 安装说明

            请从下方 Assets 下载对应平台的安装包。
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

---

## 10. Semantic Versioning（语义化版本）详解

### 10.1 语义化版本规范

语义化版本（Semantic Versioning，简称 SemVer）是一套版本号命名规范，格式为：

```
MAJOR.MINOR.PATCH[-PRERELEASE][+BUILD]

例如：
1.0.0
2.1.3
1.0.0-alpha.1
1.0.0-beta.2+build.123
```

| 组成部分 | 说明 | 何时递增 |
|---------|------|---------|
| MAJOR | 主版本号 | 不兼容的 API 变更 |
| MINOR | 次版本号 | 向下兼容的新功能 |
| PATCH | 修订号 | 向下兼容的问题修复 |
| PRERELEASE | 预发布标识 | 测试版本 |
| BUILD | 构建元数据 | 构建信息（不影响优先级） |

### 10.2 版本号递增规则

```bash
# 初始开发阶段：0.y.z
# 任何变更都可以在 0.y.z 中发生
0.1.0 → 0.2.0 (可能有破坏性变更)

# 正式发布：1.0.0
# 之后严格遵守 SemVer 规则

# PATCH 修订：1.0.0 → 1.0.1
# - Bug 修复
# - 性能优化
# - 文档修正

# MINOR 次版本：1.0.0 → 1.1.0
# - 新增功能
# - 废弃功能（不删除）
# - 内部重构（不影响公共 API）

# MAJOR 主版本：1.0.0 → 2.0.0
# - 删除功能
# - 改变现有功能行为
# - 改变公共 API 签名
```

### 10.3 版本比较规则

```
1.0.0-alpha < 1.0.0-alpha.1 < 1.0.0-alpha.beta
< 1.0.0-beta < 1.0.0-beta.2 < 1.0.0-beta.11
< 1.0.0-rc.1 < 1.0.0
```

### 10.4 使用工具管理版本号

```bash
# 使用 npm version 自动递增版本号
npm version patch   # 1.0.0 → 1.0.1
npm version minor   # 1.0.0 → 1.1.0
npm version major   # 1.0.0 → 2.0.0

# 创建预发布版本
npm version prepatch --preid=beta   # 1.0.0 → 1.0.1-beta.0
npm version preminor --preid=alpha  # 1.0.0 → 1.1.0-alpha.0
npm version premajor --preid=rc     # 1.0.0 → 2.0.0-rc.0

# 使用 standard-version
npm install -g standard-version
standard-version  # 自动分析 commits 并递增版本号

# 使用 semantic-release
npx semantic-release  # 全自动版本管理
```

---

## 11. 自动化发布流程（GitHub Actions）

### 11.1 基础自动化发布工作流

```yaml
# .github/workflows/release.yml
name: Release

on:
  push:
    tags:
      - 'v*'

permissions:
  contents: write

jobs:
  build:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        include:
          - os: linux
            arch: amd64
          - os: linux
            arch: arm64
          - os: darwin
            arch: amd64
          - os: darwin
            arch: arm64
          - os: windows
            arch: amd64

    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Setup Go
        uses: actions/setup-go@v5
        with:
          go-version: '1.21'

      - name: Build
        env:
          GOOS: ${{ matrix.os }}
          GOARCH: ${{ matrix.arch }}
        run: |
          EXTENSION=""
          if [ "$GOOS" = "windows" ]; then
            EXTENSION=".exe"
          fi
          go build -ldflags="-s -w" -o myapp-${{ matrix.os }}-${{ matrix.arch }}${EXTENSION} .

      - name: Upload Artifact
        uses: actions/upload-artifact@v4
        with:
          name: myapp-${{ matrix.os }}-${{ matrix.arch }}
          path: myapp-*

  release:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Download Artifacts
        uses: actions/download-artifact@v4
        with:
          path: artifacts

      - name: Generate Checksums
        run: |
          cd artifacts
          find . -type f -name "myapp-*" -exec sha256sum {} \; > ../checksums.txt

      - name: Create Release
        uses: softprops/action-gh-release@v1
        with:
          generate_release_notes: true
          files: |
            artifacts/myapp-*/myapp-*
            checksums.txt
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### 11.2 基于 conventional commits 的自动发布

```yaml
# .github/workflows/semantic-release.yml
name: Semantic Release

on:
  push:
    branches:
      - main

jobs:
  release:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      issues: write
      pull-requests: write

    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 'lts/*'

      - name: Install Dependencies
        run: npm ci

      - name: Build
        run: npm run build

      - name: Run Tests
        run: npm test

      - name: Release
        run: npx semantic-release
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```

**`.releaserc.json` 配置文件：**

```json
{
  "branches": ["main"],
  "plugins": [
    "@semantic-release/commit-analyzer",
    "@semantic-release/release-notes-generator",
    "@semantic-release/changelog",
    "@semantic-release/npm",
    "@semantic-release/github",
    "@semantic-release/git"
  ]
}
```

### 11.3 多平台 Docker 镜像发布

```yaml
# .github/workflows/docker-release.yml
name: Docker Release

on:
  push:
    tags:
      - 'v*'

jobs:
  docker:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Login to DockerHub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Login to GHCR
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: |
            myuser/myapp
            ghcr.io/${{ github.repository }}
          tags: |
            type=semver,pattern={{version}}
            type=semver,pattern={{major}}.{{minor}}
            type=semver,pattern={{major}}

      - name: Build and Push
        uses: docker/build-push-action@v5
        with:
          context: .
          platforms: linux/amd64,linux/arm64
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

---

## 12. Changelog 自动生成

### 12.1 基于 PR Label 的 Changelog

```yaml
# .github/workflows/changelog.yml
name: Generate Changelog

on:
  release:
    types: [published]

jobs:
  changelog:
    runs-on: ubuntu-latest
    permissions:
      contents: write

    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Generate Changelog
        run: |
          # 获取当前版本和上一个版本
          CURRENT_TAG=${{ github.event.release.tag_name }}
          PREV_TAG=$(git describe --tags --abbrev=0 HEAD^ 2>/dev/null || echo "")

          if [ -z "$PREV_TAG" ]; then
            echo "首次发布，生成完整 changelog"
            RANGE="HEAD"
          else
            RANGE="${PREV_TAG}..${CURRENT_TAG}"
          fi

          # 生成 changelog
          {
            echo "# Changelog"
            echo ""
            echo "## ${CURRENT_TAG} ($(date +%Y-%m-%d))"
            echo ""
            echo "### 🚀 新功能"
            git log $RANGE --oneline --grep="^feat" | sed 's/^[a-z0-9]* /- /'
            echo ""
            echo "### 🐛 问题修复"
            git log $RANGE --oneline --grep="^fix" | sed 's/^[a-z0-9]* /- /'
            echo ""
            echo "### ⚡ 性能优化"
            git log $RANGE --oneline --grep="^perf" | sed 's/^[a-z0-9]* /- /'
            echo ""
            echo "### 📖 文档"
            git log $RANGE --oneline --grep="^docs" | sed 's/^[a-z0-9]* /- /'
          } > NEW_CHANGELOG.md

          # 合并到现有 changelog
          if [ -f CHANGELOG.md ]; then
            tail -n +2 CHANGELOG.md >> NEW_CHANGELOG.md
          fi
          mv NEW_CHANGELOG.md CHANGELOG.md

      - name: Commit Changelog
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"
          git add CHANGELOG.md
          git commit -m "docs: update changelog for ${CURRENT_TAG}" || true
          git push
```

### 12.2 使用 git-cliff 生成 Changelog

```yaml
# .github/workflows/git-cliff.yml
name: Git Cliff Changelog

on:
  release:
    types: [published]

jobs:
  changelog:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Generate a changelog
        uses: orhun/git-cliff-action@v3
        with:
          config: cliff.toml
          args: --latest --strip header
        env:
          OUTPUT: CHANGES.md
          GITHUB_REPO: ${{ github.repository }}

      - name: Upload changelog to release
        uses: softprops/action-gh-release@v1
        with:
          body_path: CHANGES.md
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

**`cliff.toml` 配置：**

```toml
# cliff.toml
[changelog]
header = """
# Changelog\n
"""
body = """
{% if version %}\
    ## [{{ version | trim_start_matches(pat="v") }}] - {{ timestamp | date(format="%Y-%m-%d") }}
{% else %}\
    ## [unreleased]
{% endif %}\
{% for group, commits in commits | group_by(attribute="group") %}
    ### {{ group | upper_first }}
    {% for commit in commits %}
        - {% if commit.scope %}**{{ commit.scope }}:** {% endif %}\
            {{ commit.message | upper_first }}\
            {% if commit.links %} ({{ commit.links | join(sep=", ") }}){% endif %}\
    {% endfor %}
{% endfor %}\n
"""
trim = true

[git]
conventional_commits = true
filter_unconventional = true
split_commits = false

commit_parsers = [
  { message = "^feat", group = "🚀 新功能" },
  { message = "^fix", group = "🐛 问题修复" },
  { message = "^doc", group = "📖 文档" },
  { message = "^perf", group = "⚡ 性能优化" },
  { message = "^refactor", group = "♻️ 代码重构" },
  { message = "^style", group = "💄 样式" },
  { message = "^test", group = "✅ 测试" },
  { message = "^chore", group = "🔧 杂项" },
]
```

---

## 13. 多平台发布策略

### 13.1 跨平台构建矩阵

```yaml
# .github/workflows/cross-platform.yml
name: Cross Platform Build

on:
  push:
    tags:
      - 'v*'

jobs:
  build:
    runs-on: ${{ matrix.os }}
    strategy:
      fail-fast: false
      matrix:
        include:
          - os: ubuntu-latest
            target: linux-amd64
          - os: ubuntu-latest
            target: linux-arm64
          - os: macos-latest
            target: darwin-amd64
          - os: macos-14
            target: darwin-arm64
          - os: windows-latest
            target: windows-amd64

    steps:
      - uses: actions/checkout@v4

      - name: Build
        run: |
          echo "Building for ${{ matrix.target }}"
          # 根据 target 执行不同的构建命令
```

### 13.2 多包管理器发布

```yaml
# .github/workflows/multi-registry.yml
name: Publish to Multiple Registries

on:
  release:
    types: [published]

jobs:
  publish-npm:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 'lts/*'
          registry-url: 'https://registry.npmjs.org'
      - run: npm ci
      - run: npm publish
        env:
          NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}

  publish-gpr:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 'lts/*'
          registry-url: 'https://npm.pkg.github.com'
      - run: npm ci
      - run: npm publish
        env:
          NODE_AUTH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

---

## 14. 实战：设置自动化版本发布

### 14.1 完整的发布流程设计

```
开发者推送代码
       ↓
PR 触发 CI 测试
       ↓
合并到 main 分支
       ↓
自动分析 conventional commits
       ↓
自动生成 Release Notes 草稿
       ↓
手动审核并发布（或自动发布）
       ↓
触发构建流水线
       ↓
构建多平台二进制文件
       ↓
上传到 Release Assets
       ↓
发布到包管理器（npm/pypi/crates.io）
       ↓
更新 Docker 镜像
       ↓
通知社区（邮件、Discord、Slack）
```

### 14.2 完整的自动化配置

**步骤一：配置 `package.json`**

```json
{
  "name": "my-awesome-project",
  "version": "1.0.0",
  "scripts": {
    "build": "tsc && webpack --mode production",
    "test": "jest --coverage",
    "lint": "eslint src/ --ext .ts,.tsx",
    "release": "standard-version",
    "release:minor": "standard-version --release-as minor",
    "release:major": "standard-version --release-as major",
    "release:patch": "standard-version --release-as patch"
  },
  "devDependencies": {
    "standard-version": "^9.5.0",
    "@commitlint/cli": "^18.0.0",
    "@commitlint/config-conventional": "^18.0.0",
    "husky": "^9.0.0"
  }
}
```

**步骤二：配置 Commitlint**

```javascript
// commitlint.config.js
module.exports = {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'type-enum': [
      2,
      'always',
      [
        'feat',     // 新功能
        'fix',      // 修复
        'docs',     // 文档
        'style',    // 格式
        'refactor', // 重构
        'perf',     // 性能
        'test',     // 测试
        'build',    // 构建
        'ci',       // CI
        'chore',    // 杂项
        'revert',   // 回退
      ],
    ],
    'type-case': [2, 'always', 'lower-case'],
    'type-empty': [2, 'never'],
    'subject-empty': [2, 'never'],
    'subject-full-stop': [2, 'never', '.'],
    'header-max-length': [2, 'always', 100],
  },
};
```

**步骤三：配置 Husky Git Hooks**

```bash
# 初始化 husky
npx husky init

# 添加 commit-msg hook
echo 'npx --no -- commitlint --edit $1' > .husky/commit-msg

# 添加 pre-push hook
echo 'npm run lint && npm test' > .husky/pre-push
```

**步骤四：配置 `.versionrc`（standard-version）**

```json
{
  "types": [
    { "type": "feat", "section": "🚀 新功能" },
    { "type": "fix", "section": "🐛 问题修复" },
    { "type": "perf", "section": "⚡ 性能优化" },
    { "type": "refactor", "section": "♻️ 代码重构" },
    { "type": "docs", "section": "📖 文档更新" },
    { "type": "test", "section": "✅ 测试", "hidden": true },
    { "type": "chore", "section": "🔧 维护", "hidden": true },
    { "type": "style", "section": "💄 样式", "hidden": true },
    { "type": "ci", "section": "🔄 CI", "hidden": true }
  ],
  "commitUrlFormat": "https://github.com/owner/repo/commit/{{hash}}",
  "compareUrlFormat": "https://github.com/owner/repo/compare/{{previousTag}}...{{currentTag}}",
  "issueUrlFormat": "https://github.com/owner/repo/issues/{{id}}"
}
```

**步骤五：完整的 CI/CD 工作流**

```yaml
# .github/workflows/ci-cd.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main]
    tags: ['v*']
  pull_request:
    branches: [main]

jobs:
  # 持续集成
  ci:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Lint
        run: npm run lint

      - name: Test
        run: npm test

      - name: Build
        run: npm run build

      - name: Upload build artifacts
        uses: actions/upload-artifact@v4
        with:
          name: build
          path: dist/

  # 自动发布（仅在 main 分支）
  release:
    needs: ci
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    runs-on: ubuntu-latest
    permissions:
      contents: write
      issues: write
      pull-requests: write
      packages: write

    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
          token: ${{ secrets.GITHUB_TOKEN }}

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          registry-url: 'https://registry.npmjs.org'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Download build artifacts
        uses: actions/download-artifact@v4
        with:
          name: build
          path: dist/

      - name: Configure Git
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"

      - name: Release
        run: npx standard-version
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

      - name: Push changes
        run: git push --follow-tags origin main

      - name: Create GitHub Release
        uses: softprops/action-gh-release@v1
        with:
          generate_release_notes: true
          files: dist/*
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

      - name: Publish to npm
        run: npm publish
        env:
          NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
```

### 14.3 发布检查清单

```markdown
## 发布前检查清单

### 代码质量
- [ ] 所有测试通过
- [ ] 代码覆盖率达标（≥80%）
- [ ] 无 lint 错误
- [ ] TypeScript 类型检查通过

### 文档
- [ ] README 已更新
- [ ] API 文档已更新
- [ ] CHANGELOG 已生成
- [ ] 迁移指南已编写（如有破坏性变更）

### 版本
- [ ] 版本号符合 SemVer 规范
- [ ] 标签名格式正确（v1.2.3）
- [ ] 预发布版本已测试

### 发布
- [ ] Release Notes 已审核
- [ ] 二进制文件已构建并测试
- [ ] 包管理器发布成功
- [ ] Docker 镜像已推送

### 发布后
- [ ] Release 页面访问正常
- [ ] 下载链接工作正常
- [ ] 社区已通知
- [ ] 文档网站已更新
```

### 14.4 回滚发布

```bash
# 如果发布有问题，需要紧急回滚

# 1. 删除有问题的 Release
gh release delete v1.2.0 --yes

# 2. 删除标签
git tag -d v1.2.0
git push origin --delete v1.2.0

# 3. 从 npm 撤回版本（72小时内可操作）
npm unpublish my-package@1.2.0

# 4. 创建修复版本
git revert HEAD
git tag -a v1.2.1 -m "回滚 v1.2.0 的变更"
git push origin main --tags

# 5. 发布修复版本
gh release create v1.2.1 \
  --title "v1.2.1 - 紧急修复" \
  --notes "回滚 v1.2.0 中引入的问题"
```

---

## 总结

本章详细介绍了 Git 标签和 GitHub Releases 的完整知识体系：

1. **Git 标签**：轻量标签和附注标签的区别及使用场景
2. **标签管理**：创建、列出、删除、重命名标签的完整操作
3. **GPG 签名**：保障发布安全性的签名机制
4. **GitHub Releases**：从创建到管理的完整流程
5. **自动 Release Notes**：多种自动生成方案
6. **Release Assets**：二进制文件附件管理
7. **预发布版本**：alpha、beta、rc 版本管理
8. **Draft Release**：草稿发布的使用场景
9. **语义化版本**：SemVer 规范详解
10. **自动化发布**：完整的 CI/CD 发布流程
11. **Changelog 自动生成**：多种工具和方案
12. **多平台发布**：跨平台构建和多注册表发布

掌握这些知识，你就能为项目建立专业、可靠的版本发布体系，提升开发效率和用户体验。
