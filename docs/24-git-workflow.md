# Git 工作流完全指南

> 本章详细介绍各种 Git 工作流模式，帮助团队选择最适合的协作方式。

## 目录

- [什么是 Git 工作流](#什么是-git-工作流)
- [集中式工作流](#集中式工作流)
- [功能分支工作流](#功能分支工作流)
- [Git Flow 工作流详解](#git-flow-工作流详解)
- [GitHub Flow 工作流详解](#github-flow-工作流详解)
- [GitLab Flow 工作流详解](#gitlab-flow-工作流详解)
- [Trunk-Based Development](#trunk-based-development)
- [Forking Workflow](#forking-workflow)
- [各工作流对比与选择指南](#各工作流对比与选择指南)
- [分支命名规范](#分支命名规范)
- [提交信息规范](#提交信息规范)
- [版本号规范](#版本号规范)
- [Release 管理策略](#release-管理策略)
- [多人协作的冲突预防](#多人协作的冲突预防)
- [中国互联网公司的 Git 工作流实践](#中国互联网公司的-git-工作流实践)
- [各工作流的决策流程图](#各工作流的决策流程图)

---

## 什么是 Git 工作流

Git 工作流（Git Workflow）是一套基于 Git 版本控制系统的协作规范和最佳实践。它定义了团队成员如何创建分支、合并代码、发布版本以及处理紧急修复等操作的标准流程。

### 为什么需要 Git 工作流

在多人协作开发中，如果没有统一的工作流规范，会导致以下问题：

1. **代码冲突频繁**：多人同时修改同一文件，冲突难以解决
2. **版本混乱**：无法清晰区分开发版本、测试版本和生产版本
3. **发布困难**：不知道哪个版本可以发布，哪个版本正在测试
4. **回溯困难**：出现问题时难以快速定位和回滚
5. **协作效率低**：团队成员各自为政，协作成本高

### Git 工作流的核心要素

一个完整的 Git 工作流通常包含以下要素：

| 要素 | 说明 |
|------|------|
| 分支策略 | 如何创建、命名和管理分支 |
| 合并策略 | 如何将分支代码合并到主分支 |
| 发布策略 | 如何管理版本发布 |
| 冲突解决 | 如何预防和解决代码冲突 |
| 代码审查 | 如何进行代码评审 |
| 持续集成 | 如何与 CI/CD 工具集成 |

### Git 工作流的演进历史

Git 工作流经历了从简单到复杂的演进过程：

```
2005年 ─── Git 诞生
    │
2008年 ─── GitHub 上线，Fork 工作流出现
    │
2010年 ─── Git Flow 发布（Vincent Driessen）
    │
2011年 ─── GitHub Flow 提出（Scott Chacon）
    │
2014年 ─── GitLab Flow 提出
    │
2016年 ─── Trunk-Based Development 流行
    │
2020年 ─── 主流工作流成熟定型
```

---

## 集中式工作流

集中式工作流（Centralized Workflow）是最简单的 Git 工作流，适合小型团队或从 SVN 迁移的团队。

### 工作原理

所有开发者都在同一个分支（通常是 `main` 或 `master`）上工作，直接提交和拉取代码。

```
开发者A ──→ ┌─────────┐ ──→ 开发者A
             │  main   │
开发者B ──→ │  分支   │ ──→ 开发者B
             └─────────┘
开发者C ──→              ──→ 开发者C
```

### 基本操作流程

```bash
# 1. 克隆仓库
git clone https://github.com/team/project.git

# 2. 开始工作前，先拉取最新代码
git pull origin main

# 3. 修改文件
# ... 编辑代码 ...

# 4. 提交更改
git add .
git commit -m "feat: 添加用户登录功能"

# 5. 推送到远程仓库
git push origin main

# 6. 如果推送失败（别人有新提交），先拉取再推送
git pull --rebase origin main
git push origin main
```

### 优缺点分析

**优点：**
- 简单易懂，学习成本低
- 适合小型团队（2-3人）
- 与 SVN 工作方式相似，迁移成本低
- 不需要复杂的分支管理

**缺点：**
- 无法并行开发多个功能
- 直接在主分支提交，容易引入 bug
- 无法进行代码审查
- 不适合持续发布

### 适用场景

- 个人项目
- 2-3 人的小型团队
- 从 SVN 迁移到 Git 的初期阶段
- 简单的内部工具开发

### 常见问题与解决方案

#### 问题1：推送被拒绝

当多人同时推送代码时，可能会遇到推送被拒绝的情况：

```bash
# 错误信息
! [rejected]        main -> main (fetch first)
error: failed to push some refs to 'origin'

# 解决方案：先拉取再推送
git pull --rebase origin main
git push origin main
```

#### 问题2：误提交了大文件

如果不小心提交了大文件，需要从历史记录中清除：

```bash
# 查找大文件
git rev-list --objects --all | git cat-file --batch-check='%(objecttype) %(objectname) %(objectsize) %(rest)' | sed -n 's/^blob //p' | sort -rnk2 | head -10

# 使用 BFG 清理大文件
java -jar bfg.jar --strip-blobs-bigger-than 10M repo.git
```

#### 问题3：提交了敏感信息

如果提交了密码、密钥等敏感信息：

```bash
# 从历史记录中移除文件
git filter-branch --force --index-filter \
  'git rm --cached --ignore-unmatch PATH-TO-FILE' \
  --prune-empty --tag-name-filter cat -- --all

# 使用 git-filter-repo（推荐）
git-filter-repo --path PATH-TO-FILE --invert-paths
```

### 集中式工作流的迁移策略

从 SVN 迁移到 Git 时，可以采用渐进式迁移策略：

```
阶段一：并行运行（1-2周）
    │
    ├─ Git 仓库创建完成
    ├─ 团队成员学习 Git 基本操作
    └─ SVN 和 Git 并行使用

阶段二：主分支切换（1周）
    │
    ├─ 停止 SVN 提交
    ├─ 所有新代码提交到 Git
    └─ 保留 SVN 只读访问

阶段三：完全迁移（1周后）
    │
    ├─ SVN 仓库设为只读
    ├─ Git 成为唯一版本控制系统
    └─ 建立 Git 工作流规范
```

---

## 功能分支工作流

功能分支工作流（Feature Branch Workflow）是集中式工作流的改进版本，每个新功能都在独立的分支上开发。

### 工作原理

所有新功能、bug 修复都在独立的功能分支上开发，完成后通过 Pull Request 合并到主分支。

```
main ─────●───────●───────●───────●────→
           \     /         \     /
feature-1   ●───●           ●───●  feature-2
            功能开发         功能开发
```

### 分支命名规范

```bash
# 功能分支
feature/用户登录
feature/user-login
feature/20240101-user-login

# Bug 修复分支
bugfix/修复登录问题
bugfix/fix-login-error

# 热修复分支
hotfix/紧急修复支付漏洞
```

### 完整工作流程

```bash
# 1. 从主分支创建功能分支
git checkout main
git pull origin main
git checkout -b feature/user-login

# 2. 在功能分支上开发
# ... 编写代码 ...
git add .
git commit -m "feat: 实现用户登录功能"

# 3. 推送功能分支到远程
git push origin feature/user-login

# 4. 创建 Pull Request（PR）
# 在 GitHub/GitLab 上创建 PR，请求代码审查

# 5. 代码审查通过后合并
# 可以通过 GitHub/GitLab 界面合并，或命令行：
git checkout main
git pull origin main
git merge --no-ff feature/user-login
git push origin main

# 6. 删除功能分支
git branch -d feature/user-login
git push origin --delete feature/user-login
```

### Pull Request 工作流

Pull Request（PR）是功能分支工作流的核心，它实现了：

1. **代码审查**：团队成员可以审查代码质量
2. **讨论交流**：针对代码进行讨论和改进
3. **自动化测试**：集成 CI/CD 进行自动化测试
4. **文档记录**：PR 记录了功能的实现过程

### PR 模板示例

```markdown
## 功能描述
实现用户登录功能，支持用户名/邮箱登录。

## 实现方案
- 使用 JWT 进行身份验证
- 密码使用 bcrypt 加密
- 支持记住登录状态

## 测试情况
- [x] 单元测试通过
- [x] 集成测试通过
- [x] 手动测试通过

## 相关 Issue
Closes #123
```

### 优缺点分析

**优点：**
- 每个功能独立开发，互不影响
- 支持代码审查，提高代码质量
- 支持并行开发多个功能
- 便于持续集成和部署

**缺点：**
- 分支管理相对复杂
- 如果功能分支生命周期过长，合并时冲突多
- 需要团队成员遵守分支规范

### 适用场景

- 中小型团队（3-10人）
- 需要代码审查的项目
- 持续集成/持续部署的项目
- 大部分 Web 应用开发

### 功能分支工作流的最佳实践

#### 1. 保持分支短生命周期

功能分支应该在 1-3 天内完成，避免长期存在的分支：

```bash
# 不好的做法：分支存在数周
git checkout -b feature/massive-refactor
# ... 开发数周 ...

# 好的做法：拆分为多个小功能
git checkout -b feature/user-login      # 1-2天完成
git checkout -b feature/user-profile    # 1-2天完成
git checkout -b feature/user-settings   # 1-2天完成
```

#### 2. 频繁同步主分支

定期将主分支的最新代码合并到功能分支，减少最终合并时的冲突：

```bash
# 在功能分支上定期同步主分支
git checkout feature/my-feature
git fetch origin
git rebase origin/main

# 或者使用 merge
git merge origin/main
```

#### 3. 使用 Draft Pull Request

对于尚未完成的功能，可以创建 Draft PR 进行早期反馈：

```bash
# 在 GitHub 上创建 Draft PR
# 1. 推送功能分支
git push origin feature/my-feature

# 2. 在 GitHub 上创建 PR，选择 "Create draft pull request"

# 3. 标记为 Ready for review 完成后
```

#### 4. 自动化分支清理

使用脚本自动清理已合并的分支：

```bash
#!/bin/bash
# clean-merged-branches.sh

# 删除已合并的本地分支
git branch --merged main | grep -v "main" | xargs -n 1 git branch -d

# 删除已合并的远程分支
git branch -r --merged main | grep -v "main" | sed 's/origin\///' | xargs -n 1 git push origin --delete
```

### 功能分支工作流的常见陷阱

| 陷阱 | 问题描述 | 解决方案 |
|------|----------|----------|
| 分支生命周期过长 | 合并时冲突多，代码审查困难 | 拆分功能，短周期开发 |
| 不同步主分支 | 最终合并时大量冲突 | 定期 rebase 或 merge 主分支 |
| 缺少代码审查 | 代码质量无法保证 | 强制 PR 审查流程 |
| 分支命名混乱 | 难以识别分支用途 | 统一命名规范 |
| 不删除已合并分支 | 分支列表混乱 | 合并后立即删除分支 |

---

## Git Flow 工作流详解

Git Flow 是由 Vincent Driessen 在 2010 年提出的分支管理模型，是最经典的 Git 工作流之一。

### 分支类型

Git Flow 定义了五种分支类型：

| 分支类型 | 命名规则 | 生命周期 | 用途 |
|----------|----------|----------|------|
| master | master | 永久 | 生产环境代码 |
| develop | develop | 永久 | 开发主分支 |
| feature/* | feature/xxx | 临时 | 新功能开发 |
| release/* | release/x.x.x | 临时 | 版本发布准备 |
| hotfix/* | hotfix/xxx | 临时 | 紧急修复 |

### 分支结构图

```
master    ──●───────────────●───────────────●──→
              \             ↑               ↑
               \           /               /
hotfix          \         /               /
                 \       /               /
release           ●─────●               /
                  ↑     \             /
                  │      \           /
develop    ───────●───────●─────●────●──────→
                  ↑       ↑     ↑
                  │       │     │
feature-1         ●───────●     │
                                 │
feature-2                ●───────●
```

### 详细工作流程

#### 1. 初始化 Git Flow

```bash
# 安装 git-flow 工具
# macOS
brew install git-flow

# Linux (Ubuntu/Debian)
sudo apt-get install git-flow

# 初始化 Git Flow
git flow init

# 按提示设置分支名称：
# Production branch: master
# Development branch: develop
# Feature branches: feature/
# Release branches: release/
# Hotfix branches: hotfix/
# Support branches: support/
# Version tag prefix: v
```

#### 2. 功能开发流程

```bash
# 开始新功能
git flow feature start user-login

# 这会自动创建并切换到 feature/user-login 分支

# 在功能分支上开发
# ... 编写代码 ...
git add .
git commit -m "feat: 实现用户登录功能"

# 完成功能，合并到 develop
git flow feature finish user-login

# 这会自动：
# 1. 将 feature/user-login 合并到 develop
# 2. 删除 feature/user-login 分支
# 3. 切换回 develop 分支

# 如果需要协作，可以发布功能分支
git flow feature publish user-login

# 或者拉取远程功能分支
git flow feature pull origin user-login
```

#### 3. 版本发布流程

```bash
# 开始版本发布
git flow release start 1.0.0

# 这会自动创建 release/1.0.0 分支

# 在发布分支上进行最后的修复和调整
# ... 修复 bug ...
git add .
git commit -m "fix: 修复登录页面样式问题"

# 更新版本号
echo "1.0.0" > VERSION

# 完成版本发布
git flow release finish 1.0.0

# 这会自动：
# 1. 将 release/1.0.0 合并到 master
# 2. 在 master 上创建标签 v1.0.0
# 3. 将 release/1.0.0 合并到 develop
# 4. 删除 release/1.0.0 分支
```

#### 4. 紧急修复流程

```bash
# 开始紧急修复
git flow hotfix start fix-payment-bug

# 这会自动从 master 创建 hotfix/fix-payment-bug 分支

# 修复 bug
# ... 修复代码 ...
git add .
git commit -m "fix: 修复支付验证漏洞"

# 完成紧急修复
git flow hotfix finish fix-payment-bug

# 这会自动：
# 1. 将 hotfix/fix-payment-bug 合并到 master
# 2. 在 master 上创建标签（如 v1.0.1）
# 3. 将 hotfix/fix-payment-bug 合并到 develop
# 4. 删除 hotfix/fix-payment-bug 分支
```

### Git Flow 的优缺点

**优点：**
- 分支角色清晰，职责明确
- 适合有明确发布周期的项目
- 支持多版本并行维护
- 详细的文档和工具支持

**缺点：**
- 分支数量多，管理复杂
- 合并流程繁琐
- 不适合持续部署
- 对于小型项目可能过于复杂

### 适用场景

- 有固定发布周期的软件产品
- 需要维护多个版本的企业应用
- 大型团队协作开发
- 传统软件开发模式

### Git Flow 的实际应用案例

#### 案例1：企业级 ERP 系统

某企业 ERP 系统采用 Git Flow 工作流，版本发布周期为 2 周：

```
时间线：
Week 1-2: 功能开发
    │
    ├─ feature/user-management 开发
    ├─ feature/report-export 开发
    └─ feature/data-visualization 开发

Week 3: 版本发布准备
    │
    ├─ 创建 release/2.1.0 分支
    ├─ 测试和 Bug 修复
    └─ 文档更新

Week 4: 正式发布
    │
    ├─ 合并到 master
    ├─ 创建标签 v2.1.0
    ├─ 部署到生产环境
    └─ 合并回 develop
```

#### 案例2：移动应用开发

某移动应用采用 Git Flow，每个版本对应一个应用商店版本：

```bash
# 版本发布流程
git flow release start 3.2.0

# 修复测试发现的问题
git commit -am "fix: 修复 iOS 16 兼容性问题"

# 更新版本号
echo "3.2.0" > VERSION
git commit -am "chore: 更新版本号到 3.2.0"

# 完成发布
git flow release finish 3.2.0

# 推送到远程
git push origin master --tags
git push origin develop
```

### Git Flow 工具推荐

#### 1. git-flow 工具

```bash
# 安装 git-flow
# macOS
brew install git-flow

# Linux
apt-get install git-flow

# Windows (通过 Chocolatey)
choco install gitflow

# 初始化
git flow init

# 常用命令
git flow feature start <name>
git flow feature finish <name>
git flow release start <version>
git flow release finish <version>
git flow hotfix start <name>
git flow hotfix finish <name>
```

#### 2. GitKraken

GitKraken 是一款图形化 Git 客户端，内置 Git Flow 支持：

- 可视化分支结构
- 一键创建 Git Flow 分支
- 拖拽合并分支
- 冲突可视化解决

#### 3. SourceTree

SourceTree 是 Atlassian 出品的 Git 客户端：

- 内置 Git Flow 支持
- 可视化分支管理
- 与 Jira 集成
- 支持 Windows 和 macOS

### Git Flow 的变体

#### 1. 简化 Git Flow

对于小型团队，可以简化 Git Flow：

```
main    ──●───────●───────●──→
           \     /         \
develop     ●───●───────●───●──→
             ↑       ↑       ↑
             │       │       │
            feat    feat    feat
```

#### 2. 带支持分支的 Git Flow

对于需要长期支持多个版本的项目：

```
main    ──●───────●───────●───────●──→
           \               ↑       ↑
support-1   ●───────●──────●       │
            ↑       ↑               │
            │       │               │
hotfix     fix     fix             │
                                    │
main    ────────────────────────────●──→
                                    ↑
support-2   ●───────●───────●──────●
            ↑       ↑       ↑
            │       │       │
           feat    feat    feat
```

---

## GitHub Flow 工作流详解

GitHub Flow 是由 GitHub 提出的轻量级工作流，强调简洁和持续部署。

### 核心原则

1. **主分支始终可部署**：`main` 分支的代码随时可以发布到生产环境
2. **所有工作在功能分支进行**：新功能、bug 修复都在独立分支
3. **通过 PR 合并**：所有代码变更通过 Pull Request 合并
4. **合并后立即部署**：PR 合并后应尽快部署到生产环境

### 工作流程图

```
main    ──●───────●───────●───────●───────●──→
           \     /         \     /         \
feature-1   ●───●           │               │
                            │               │
feature-2           ●───────●               │
                                            │
bugfix                              ●───────●
```

### 完整工作流程

```bash
# 1. 从 main 创建功能分支
git checkout main
git pull origin main
git checkout -b feature/add-search

# 2. 开发并提交
# ... 编写代码 ...
git add .
git commit -m "feat: 添加搜索功能"

# 3. 推送到远程
git push origin feature/add-search

# 4. 创建 Pull Request
# 在 GitHub 上创建 PR，填写描述信息

# 5. 代码审查和讨论
# 团队成员审查代码，提出修改建议

# 6. 修复审查意见
# ... 修改代码 ...
git add .
git commit -m "fix: 根据审查意见修改搜索逻辑"
git push origin feature/add-search

# 7. 审查通过后合并
# 在 GitHub 上点击 "Merge pull request"

# 8. 部署到生产环境
# 合并后自动或手动部署

# 9. 删除功能分支
git branch -d feature/add-search
git push origin --delete feature/add-search
```

### GitHub Flow 的特点

**持续部署友好：**
- 合并后立即部署
- 小批量发布，降低风险
- 快速反馈，快速迭代

**代码审查为核心：**
- 所有变更都经过 PR
- 支持行级评论
- 支持建议修改
- 自动化检查集成

### 优缺点分析

**优点：**
- 简单易懂，学习成本低
- 适合持续部署
- 代码质量有保障
- 快速迭代

**缺点：**
- 不适合需要多版本维护的项目
- 对主分支稳定性要求高
- 需要完善的自动化测试支持

### 适用场景

- SaaS 产品
- 持续部署的 Web 应用
- 中小型团队
- 敏捷开发项目

### GitHub Flow 的自动化配置

#### 1. GitHub Actions 配置

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      - name: Install dependencies
        run: npm ci
      - name: Run tests
        run: npm test
      - name: Run linting
        run: npm run lint

  deploy:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v3
      - name: Deploy to production
        run: |
          echo "Deploying to production..."
          # 部署脚本
```

#### 2. 自动化部署脚本

```bash
#!/bin/bash
# deploy.sh

set -e

echo "开始部署..."

# 拉取最新代码
git pull origin main

# 安装依赖
npm ci

# 运行测试
npm test

# 构建项目
npm run build

# 部署到服务器
rsync -avz --delete dist/ user@server:/var/www/app/

# 重启服务
ssh user@server "sudo systemctl restart nginx"

echo "部署完成！"
```

#### 3. 分支保护配置

```yaml
# .github/settings.yml
branches:
  - name: main
    protection:
      required_pull_request_reviews:
        required_approving_review_count: 2
        dismiss_stale_reviews: true
        require_code_owner_reviews: true
      required_status_checks:
        strict: true
        contexts:
          - ci/test
          - ci/build
      enforce_admins: true
      restrictions:
        users: []
        teams:
          - maintainers
```

### GitHub Flow 的团队协作技巧

#### 1. 使用 Issue 追踪任务

```markdown
## Issue 模板

### 问题描述
简要描述问题或需求。

### 复现步骤
1. 步骤一
2. 步骤二
3. 步骤三

### 期望行为
描述期望的行为。

### 实际行为
描述实际的行为。

### 环境信息
- 操作系统：
- 浏览器：
- 版本：
```

#### 2. 使用 Project 管理任务

GitHub Projects 可以帮助团队管理任务：

- 看板视图：To Do → In Progress → Done
- 自动化：PR 合并后自动移动任务
- 筛选和排序：按优先级、负责人筛选

#### 3. 使用 CODEOWNERS

```bash
# .github/CODEOWNERS

# 默认所有者
*       @team-lead

# 前端代码
/src/frontend/    @frontend-team

# 后端代码
/src/backend/     @backend-team

# 文档
/docs/            @doc-team

# CI/CD 配置
/.github/         @devops-team
```

### GitHub Flow 与 GitHub 功能的集成

| GitHub 功能 | 用途 | 配置位置 |
|-------------|------|----------|
| Issues | 任务追踪 | Repository → Issues |
| Pull Requests | 代码审查 | Repository → Pull Requests |
| Projects | 项目管理 | Repository → Projects |
| Actions | CI/CD | Repository → Actions |
| Discussions | 团队讨论 | Repository → Discussions |
| Wiki | 文档管理 | Repository → Wiki |
| Security | 安全扫描 | Repository → Security |

---

## GitLab Flow 工作流详解

GitLab Flow 是 GitLab 提出的工作流，结合了 GitHub Flow 的简洁和 Git Flow 的版本管理能力。

### 核心理念

GitLab Flow 的核心理念是"上游优先"（Upstream First）：代码只能从上游流向下游，不能反向流动。

```
feature → main → pre-production → production
  ↑         ↑           ↑             ↑
  │         │           │             │
 功能      开发        预发布         生产
 分支      分支        分支           分支
```

### 环境分支模式

#### 1. 生产环境分支模式

适用于只有一个生产环境的项目：

```bash
# 创建功能分支
git checkout -b feature/add-user-profile main

# 开发完成后合并到 main
git checkout main
git merge feature/add-user-profile

# 部署到生产环境时，创建标签
git tag -a v1.0.0 -m "Release 1.0.0"
git push origin v1.0.0
```

#### 2. 多环境分支模式

适用于有多个环境（开发、测试、生产）的项目：

```
main ──────●───────●───────●───────●───────●──→
            \               \               \
staging       ●───────●───────●               │
              \               \               \
production             ●───────●───────●──────●──→
```

```bash
# 1. 功能开发
git checkout -b feature/new-feature main
# ... 开发 ...
git checkout main
git merge feature/new-feature

# 2. 部署到测试环境
git checkout staging
git merge main
git push origin staging

# 3. 测试通过后部署到生产环境
git checkout production
git merge staging
git push origin production

# 4. 创建版本标签
git tag -a v1.2.0 -m "Release 1.2.0"
git push origin v1.2.0
```

#### 3. 发布分支模式

适用于需要同时维护多个版本的项目：

```
main ──────●───────●───────●───────●───────●──→
            \               \
release-1.0  ●───────●───────●───────●───────●──→
              \               \               \
release-2.0           ●───────●───────●───────●──→
```

```bash
# 从 main 创建发布分支
git checkout -b release-2.0 main

# 在发布分支上修复 bug
git checkout release-2.0
# ... 修复 ...
git commit -am "fix: 修复发布版本的 bug"

# 将修复合并回 main
git checkout main
git cherry-pick <commit-hash>

# 创建版本标签
git checkout release-2.0
git tag -a v2.0.0 -m "Release 2.0.0"
```

### Issue 追踪集成

GitLab Flow 强调与 Issue 追踪系统的集成：

```bash
# 在提交信息中引用 Issue
git commit -m "feat: 实现用户搜索功能

Closes #123"

# 创建分支时引用 Issue
git checkout -b 123-user-search main
```

### 优缺点分析

**优点：**
- 灵活适应不同项目需求
- 与 CI/CD 深度集成
- 支持多环境部署
- Issue 追踪集成完善

**缺点：**
- 需要根据项目选择合适的模式
- 多环境模式分支管理复杂
- 需要团队理解上游优先原则

### 适用场景

- 使用 GitLab 的团队
- 需要多环境部署的项目
- 需要版本管理的项目
- DevOps 实践的团队

### GitLab CI/CD 配置示例

#### 1. 基础配置

```yaml
# .gitlab-ci.yml
stages:
  - build
  - test
  - deploy

variables:
  NODE_ENV: production

# 构建阶段
build:
  stage: build
  image: node:18
  script:
    - npm ci
    - npm run build
  artifacts:
    paths:
      - dist/

# 测试阶段
test:
  stage: test
  image: node:18
  script:
    - npm ci
    - npm test
  coverage: '/All files\s*\|\s*([\d\.]+)/'

# 部署到测试环境
deploy_staging:
  stage: deploy
  image: ruby:latest
  script:
    - apt-get update -qy
    - apt-get install -y ruby-dev
    - gem install dpl
    - dpl --provider=heroku --app=$HEROKU_APP_STAGING --api-key=$HEROKU_API_KEY
  environment:
    name: staging
    url: https://staging.example.com
  only:
    - develop

# 部署到生产环境
deploy_production:
  stage: deploy
  image: ruby:latest
  script:
    - apt-get update -qy
    - apt-get install -y ruby-dev
    - gem install dpl
    - dpl --provider=heroku --app=$HEROKU_APP_PRODUCTION --api-key=$HEROKU_API_KEY
  environment:
    name: production
    url: https://example.com
  only:
    - main
  when: manual
```

#### 2. 多环境部署配置

```yaml
# .gitlab-ci.yml（多环境版本）
stages:
  - build
  - test
  - deploy_staging
  - deploy_pre_production
  - deploy_production

# 构建
build:
  stage: build
  script:
    - npm ci
    - npm run build
  artifacts:
    paths:
      - dist/

# 测试
test:
  stage: test
  script:
    - npm ci
    - npm test

# 部署到测试环境
deploy_staging:
  stage: deploy_staging
  script:
    - ./deploy.sh staging
  environment:
    name: staging
    url: https://staging.example.com
  only:
    - develop

# 部署到预发布环境
deploy_pre_production:
  stage: deploy_pre_production
  script:
    - ./deploy.sh pre-production
  environment:
    name: pre-production
    url: https://pre-prod.example.com
  only:
    - /^release-.*$/

# 部署到生产环境
deploy_production:
  stage: deploy_production
  script:
    - ./deploy.sh production
  environment:
    name: production
    url: https://example.com
  only:
    - main
  when: manual
```

### GitLab Flow 的环境管理

#### 环境分支策略

```
main ──────●───────●───────●───────●───────●──→
            \               \               \
staging       ●───────●───────●               │
              \               \               \
pre-production  ●───────●───────●───────●──────●──→
                                \               \
production                               ●──────●──→
```

#### 环境变量管理

```yaml
# GitLab CI/CD 环境变量配置
# Settings → CI/CD → Variables

# 数据库配置
DATABASE_URL: postgresql://user:pass@host:5432/db

# API 密钥
API_KEY: your-api-key
API_SECRET: your-api-secret

# 部署配置
DEPLOY_SERVER: production-server.com
DEPLOY_USER: deploy
DEPLOY_KEY: |
  -----BEGIN RSA PRIVATE KEY-----
  ...
  -----END RSA PRIVATE KEY-----
```

### GitLab Flow 的高级特性

#### 1. Merge Request 模板

```markdown
<!-- .gitlab/merge_request_templates/default.md -->

## 描述

简要描述此 MR 的目的。

## 相关 Issue

Closes #

## 变更类型

- [ ] 新功能
- [ ] Bug 修复
- [ ] 重构
- [ ] 文档更新
- [ ] 其他

## 测试

- [ ] 单元测试通过
- [ ] 集成测试通过
- [ ] 手动测试通过

## 截图（如适用）

## 审查清单

- [ ] 代码风格符合规范
- [ ] 无安全隐患
- [ ] 文档已更新
- [ ] CHANGELOG 已更新
```

#### 2. 自动化代码质量检查

```yaml
# .gitlab-ci.yml（代码质量检查）
code_quality:
  stage: test
  image: docker:stable
  variables:
    DOCKER_DRIVER: overlay2
  services:
    - docker:dind
  script:
    - docker run
      --env SOURCE_CODE="$PWD"
      --volume "$PWD":/code
      --volume /var/run/docker.sock:/var/run/docker.sock
      "registry.gitlab.com/gitlab-org/ci-codescan:latest" /code
  artifacts:
    reports:
      codequality: gl-code-quality-report.json
```

#### 3. 安全扫描

```yaml
# .gitlab-ci.yml（安全扫描）
include:
  - template: Security/SAST.gitlab-ci.yml
  - template: Security/Dependency-Scanning.gitlab-ci.yml
  - template: Security/Secret-Detection.gitlab-ci.yml
  - template: Security/Container-Scanning.gitlab-ci.yml
```

---

## Trunk-Based Development

Trunk-Based Development（TBD，主干开发）是一种强调在主分支上频繁集成的开发模式。

### 核心原则

1. **所有人在主分支上工作**：开发者频繁地将小批量代码合并到主分支
2. **短生命周期分支**：功能分支存活时间不超过 1-2 天
3. **功能开关**：使用功能开关控制未完成功能的可见性
4. **持续集成**：频繁集成，快速反馈

### 工作模式

#### 1. 直接在主分支提交

```
main ──●──●──●──●──●──●──●──●──●──●──→
       ↑  ↑  ↑  ↑  ↑  ↑  ↑  ↑  ↑  ↑
      所有开发者的提交
```

```bash
# 直接在 main 分支工作
git checkout main
git pull origin main

# 小批量修改
# ... 修改代码 ...
git add .
git commit -m "feat: 添加用户名称字段"
git push origin main
```

#### 2. 短生命周期功能分支

```
main ──●─────●─────●─────●─────●──→
        \   /       \   /
feat-1   ●─●         │
                      │
feat-2          ●─────●
           (1-2天)
```

```bash
# 创建短生命周期分支
git checkout main
git pull origin main
git checkout -b feature/quick-fix

# 快速开发（1-2天内完成）
# ... 编写代码 ...
git add .
git commit -m "feat: 快速实现某功能"

# 立即合并回主分支
git checkout main
git pull origin main
git merge feature/quick-fix
git push origin main

# 删除分支
git branch -d feature/quick-fix
```

### 功能开关（Feature Flags）

功能开关是 TBD 的关键实践，用于控制未完成功能的可见性：

```javascript
// 功能开关示例
function renderNewFeature() {
  if (featureFlags.isEnabled('new-search')) {
    return <NewSearchComponent />;
  }
  return <OldSearchComponent />;
}
```

```yaml
# 功能开关配置
feature_flags:
  new-search:
    enabled: true
    rollout_percentage: 10%
    allowed_users:
      - user1
      - user2
```

### 优缺点分析

**优点：**
- 减少合并冲突
- 持续集成，快速反馈
- 简化分支管理
- 适合持续部署

**缺点：**
- 需要强大的自动化测试支持
- 需要功能开关机制
- 对团队纪律要求高
- 主分支不稳定风险

### 适用场景

- 高性能 DevOps 团队
- 持续部署的 SaaS 产品
- 有完善自动化测试的项目
- 大型互联网公司

### Trunk-Based Development 的实践要点

#### 1. 建立完善的自动化测试

TBD 要求有高覆盖率的自动化测试：

```javascript
// 单元测试示例
describe('UserService', () => {
  describe('login', () => {
    it('应该成功登录有效用户', async () => {
      const user = await userService.login('test@example.com', 'password');
      expect(user).toBeDefined();
      expect(user.email).toBe('test@example.com');
    });

    it('应该拒绝无效密码', async () => {
      await expect(
        userService.login('test@example.com', 'wrong-password')
      ).rejects.toThrow('Invalid credentials');
    });

    it('应该处理不存在的用户', async () => {
      await expect(
        userService.login('nonexistent@example.com', 'password')
      ).rejects.toThrow('User not found');
    });
  });
});
```

```javascript
// 集成测试示例
describe('User API', () => {
  describe('POST /api/login', () => {
    it('应该返回 JWT token', async () => {
      const response = await request(app)
        .post('/api/login')
        .send({ email: 'test@example.com', password: 'password' })
        .expect(200);

      expect(response.body.token).toBeDefined();
    });

    it('应该返回 401 对于无效凭据', async () => {
      await request(app)
        .post('/api/login')
        .send({ email: 'test@example.com', password: 'wrong' })
        .expect(401);
    });
  });
});
```

#### 2. 实现功能开关系统

```javascript
// 功能开关服务
class FeatureFlagService {
  constructor() {
    this.flags = {};
  }

  async loadFlags() {
    // 从配置中心加载功能开关
    this.flags = await configCenter.getFeatureFlags();
  }

  isEnabled(flagName, userId = null) {
    const flag = this.flags[flagName];
    if (!flag) return false;

    // 检查是否全局启用
    if (!flag.enabled) return false;

    // 检查灰度比例
    if (flag.rolloutPercentage < 100) {
      const hash = this.hashUserId(userId);
      if (hash > flag.rolloutPercentage) return false;
    }

    // 检查白名单
    if (flag.whitelist && flag.whitelist.includes(userId)) {
      return true;
    }

    return true;
  }

  hashUserId(userId) {
    // 简单的哈希函数，确保同一用户总是进入同一组
    let hash = 0;
    for (let i = 0; i < userId.length; i++) {
      hash = ((hash << 5) - hash) + userId.charCodeAt(i);
      hash = hash & hash;
    }
    return Math.abs(hash) % 100;
  }
}

// 使用示例
const featureFlags = new FeatureFlagService();

function renderSearchComponent(userId) {
  if (featureFlags.isEnabled('new-search', userId)) {
    return <NewSearchComponent />;
  }
  return <OldSearchComponent />;
}
```

#### 3. 建立快速回滚机制

```bash
#!/bin/bash
# rollback.sh - 快速回滚脚本

set -e

echo "开始回滚..."

# 获取上一个成功的部署版本
PREVIOUS_VERSION=$(git describe --tags --abbrev=0 HEAD~1)

# 回滚代码
git checkout $PREVIOUS_VERSION

# 重新部署
./deploy.sh

echo "回滚完成，当前版本: $PREVIOUS_VERSION"
```

### TBD 的团队协作规范

#### 1. 提交频率

```
理想的提交频率：
    │
    ├─ 每天至少 1-2 次提交
    ├─ 每个提交只做一件事
    ├─ 提交大小控制在 100-200 行代码
    └─ 提交后立即推送

避免的行为：
    │
    ├─ 一周只提交一次
    ├─ 一次性提交数千行代码
    ├─ 提交半成品代码
    └─ 长时间不推送
```

#### 2. 代码审查流程

```markdown
## TBD 代码审查规范

### 审查时机
- 提交后 1 小时内完成审查
- 紧急修复 30 分钟内完成审查

### 审查要点
- 代码质量
- 测试覆盖
- 性能影响
- 安全风险

### 审查流程
1. 提交 PR/MR
2. 自动化检查运行
3. 至少 1 人审查通过
4. 合并到主分支
5. 自动部署到测试环境
```

#### 3. 主分支保护策略

```yaml
# 主分支保护配置
branch_protection:
  main:
    required_reviews: 1
    dismiss_stale_reviews: true
    require_status_checks: true
    required_checks:
      - ci/test
      - ci/build
      - security/scan
    enforce_admins: false
    allow_force_pushes: false
```

### TBD 的挑战与解决方案

| 挑战 | 解决方案 |
|------|----------|
| 主分支不稳定 | 完善的自动化测试、快速回滚机制 |
| 功能未完成就合并 | 功能开关、特性标志 |
| 团队纪律要求高 | 代码审查、自动化检查 |
| 需要强大的 CI/CD | 投资 CI/CD 基础设施 |
| 协作冲突频繁 | 小批量提交、频繁集成 |

---

## Forking Workflow

Forking Workflow（Fork 工作流）是开源项目中最常用的工作流，也适用于企业内部的跨团队协作。

### 工作原理

每个开发者都有自己的远程仓库副本（Fork），在自己的副本上开发，然后通过 Pull Request 贡献代码。

```
┌─────────────────┐     Fork     ┌─────────────────┐
│   原始仓库       │ ──────────→ │   开发者A的Fork  │
│   (upstream)    │             │   (origin)       │
└─────────────────┘             └─────────────────┘
         ↑                              │
         │         Pull Request         │
         └──────────────────────────────┘
```

### 完整工作流程

#### 1. Fork 仓库

在 GitHub/GitLab 上点击 "Fork" 按钮，创建自己的副本。

#### 2. 克隆并配置

```bash
# 克隆自己的 Fork
git clone https://github.com/your-username/project.git
cd project

# 添加原始仓库为 upstream
git remote add upstream https://github.com/original-owner/project.git

# 查看远程仓库配置
git remote -v
# origin    https://github.com/your-username/project.git (fetch)
# origin    https://github.com/your-username/project.git (push)
# upstream  https://github.com/original-owner/project.git (fetch)
# upstream  https://github.com/original-owner/project.git (push)
```

#### 3. 保持同步

```bash
# 获取上游最新代码
git fetch upstream

# 切换到 main 分支
git checkout main

# 合并上游代码
git merge upstream/main

# 推送到自己的 Fork
git push origin main
```

#### 4. 开发新功能

```bash
# 从最新的 main 创建功能分支
git checkout main
git pull origin main
git checkout -b feature/new-feature

# 开发并提交
# ... 编写代码 ...
git add .
git commit -m "feat: 实现新功能"

# 推送到自己的 Fork
git push origin feature/new-feature
```

#### 5. 创建 Pull Request

在 GitHub/GitLab 上创建 PR，从自己的 Fork 的功能分支指向原始仓库的 main 分支。

#### 6. 处理审查意见

```bash
# 根据审查意见修改代码
# ... 修改代码 ...
git add .
git commit -m "fix: 根据审查意见修改"
git push origin feature/new-feature

# PR 会自动更新
```

### Fork 工作流的优势

1. **权限隔离**：贡献者不需要原始仓库的写权限
2. **自由实验**：可以在自己的 Fork 上自由实验
3. **代码质量**：通过 PR 进行代码审查
4. **适合开源**：开源项目的标准协作方式

### 企业内部的 Fork 工作流

在企业内部，Fork 工作流可以用于跨团队协作：

```bash
# 团队A的成员
git clone https://github.com/company/project.git
git checkout -b feature/team-a-feature
# ... 开发 ...
git push origin feature/team-a-feature
# 创建 PR 到主仓库

# 团队B的成员
git clone https://github.com/company/project.git
git checkout -b feature/team-b-feature
# ... 开发 ...
git push origin feature/team-b-feature
# 创建 PR 到主仓库
```

### 优缺点分析

**优点：**
- 适合开源项目
- 权限管理灵活
- 支持跨团队协作
- 代码质量有保障

**缺点：**
- 工作流程相对复杂
- 需要管理多个远程仓库
- 同步操作频繁
- 对新手不够友好

### 适用场景

- 开源项目
- 跨团队协作
- 需要严格权限控制的项目
- 外部贡献者参与的项目

### Forking Workflow 的详细操作指南

#### 1. 首次设置

```bash
# 1. Fork 仓库（在 GitHub/GitLab 界面操作）

# 2. 克隆你的 Fork
git clone https://github.com/your-username/project.git
cd project

# 3. 添加原始仓库为 upstream
git remote add upstream https://github.com/original-owner/project.git

# 4. 验证远程仓库配置
git remote -v
# origin    https://github.com/your-username/project.git (fetch)
# origin    https://github.com/your-username/project.git (push)
# upstream  https://github.com/original-owner/project.git (fetch)
# upstream  https://github.com/original-owner/project.git (push)

# 5. 配置 Git 用户信息
git config user.name "Your Name"
git config user.email "your.email@example.com"
```

#### 2. 日常工作流程

```bash
# 1. 同步上游最新代码
git fetch upstream
git checkout main
git merge upstream/main
git push origin main

# 2. 创建功能分支
git checkout -b feature/my-feature

# 3. 开发并提交
# ... 编写代码 ...
git add .
git commit -m "feat: 实现新功能"

# 4. 推送到你的 Fork
git push origin feature/my-feature

# 5. 创建 Pull Request

# 6. 处理审查意见
# ... 修改代码 ...
git add .
git commit -m "fix: 根据审查意见修改"
git push origin feature/my-feature

# 7. PR 合并后，清理分支
git checkout main
git pull origin main
git branch -d feature/my-feature
git push origin --delete feature/my-feature
```

#### 3. 处理上游冲突

```bash
# 1. 获取上游最新代码
git fetch upstream

# 2. 切换到功能分支
git checkout feature/my-feature

# 3. Rebase 到上游主分支
git rebase upstream/main

# 4. 解决冲突（如果有）
# ... 编辑冲突文件 ...
git add .
git rebase --continue

# 5. 强制推送到你的 Fork
git push origin feature/my-feature --force-with-lease
```

### 开源项目的贡献流程

#### 1. 寻找可贡献的项目

```markdown
## 寻找开源项目的途径

### GitHub
- Good First Issues: https://github.com/topics/good-first-issue
- Help Wanted: https://github.com/topics/help-wanted
- Up For Grabs: https://up-for-grabs.net

### 平台
- GitHub Explore: https://github.com/explore
- GitLab Explore: https://gitlab.com/explore
- Gitee 探索: https://gitee.com/explore

### 选择标准
- 项目活跃度
- 社区友好度
- 文档完善度
- 技术栈匹配度
```

#### 2. 贡献前的准备

```markdown
## 贡献前检查清单

### 了解项目
- [ ] 阅读 README.md
- [ ] 阅读 CONTRIBUTING.md
- [ ] 阅读 CODE_OF_CONDUCT.md
- [ ] 了解项目架构

### 环境准备
- [ ] Fork 项目
- [ ] 克隆到本地
- [ ] 配置开发环境
- [ ] 运行测试通过

### 沟通交流
- [ ] 查看现有 Issue
- [ ] 查看现有 PR
- [ ] 在 Issue 中表达贡献意愿
- [ ] 确认贡献方向
```

#### 3. 提交高质量的 PR

```markdown
## 高质量 PR 的要素

### 标题
- 简洁明了
- 使用 Conventional Commits 格式
- 示例: "feat: add user authentication"

### 描述
- 说明做了什么
- 为什么这么做
- 如何测试
- 相关 Issue

### 代码
- 符合项目代码风格
- 包含必要的测试
- 文档已更新
- 无 lint 错误

### 示例
## 描述
实现用户认证功能，支持 JWT token 认证。

## 变更
- 添加 JWT 认证中间件
- 实现登录接口
- 添加认证测试

## 测试
- [x] 单元测试通过
- [x] 集成测试通过
- [x] 手动测试通过

Closes #123
```

### Forking Workflow 的权限管理

#### 1. 仓库权限设置

```yaml
# GitHub 仓库权限配置
permissions:
  # 读权限
  read:
    - 外部贡献者
    - 观察者

  # 写权限
  write:
    - 核心开发者
    - 维护者

  # 管理权限
  admin:
    - 项目负责人
    - 架构师

  # 禁止的操作
  restrictions:
    - force push to main
    - delete main branch
    - modify protected branches
```

#### 2. 分支保护规则

```yaml
# 分支保护配置
branch_protection:
  main:
    required_pull_request_reviews:
      required_approving_review_count: 2
      dismiss_stale_reviews: true
      require_code_owner_reviews: true
    required_status_checks:
      strict: true
      contexts:
        - ci/test
        - ci/build
        - security/scan
    enforce_admins: true
    restrictions:
      users: []
      teams:
        - core-team
        - maintainers
```

### Forking Workflow 的自动化

#### 1. 自动同步上游

```yaml
# .github/workflows/sync-upstream.yml
name: Sync Upstream

on:
  schedule:
    - cron: '0 0 * * *'  # 每天执行一次
  workflow_dispatch:  # 手动触发

jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0

      - name: Sync upstream
        run: |
          git remote add upstream https://github.com/original-owner/project.git
          git fetch upstream
          git checkout main
          git merge upstream/main
          git push origin main
```

#### 2. 自动化 PR 检查

```yaml
# .github/workflows/pr-check.yml
name: PR Check

on:
  pull_request:
    branches: [ main ]

jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Check PR title
        uses: actions/github-script@v6
        with:
          script: |
            const title = context.payload.pull_request.title;
            const regex = /^(feat|fix|docs|style|refactor|perf|test|build|ci|chore|revert)(\(.+\))?: .{1,50}/;
            if (!regex.test(title)) {
              core.setFailed('PR title does not follow Conventional Commits format');
            }

      - name: Run tests
        run: |
          npm ci
          npm test

      - name: Run linting
        run: |
          npm run lint
```

---

## 各工作流对比与选择指南

### 对比表格

| 工作流 | 复杂度 | 分支数量 | 适合团队规模 | 持续部署 | 版本维护 | 学习成本 |
|--------|--------|----------|--------------|----------|----------|----------|
| 集中式 | 低 | 1 | 1-3人 | ✓ | ✗ | 低 |
| 功能分支 | 中低 | 中等 | 3-10人 | ✓ | ✗ | 低 |
| Git Flow | 高 | 多 | 10+人 | ✗ | ✓ | 高 |
| GitHub Flow | 低 | 少 | 3-10人 | ✓ | ✗ | 低 |
| GitLab Flow | 中 | 中等 | 5-20人 | ✓ | ✓ | 中 |
| TBD | 中 | 极少 | 5-20人 | ✓ | ✗ | 中 |
| Forking | 中高 | 多 | 不限 | ✓ | ✗ | 中高 |

### 选择决策树

```
开始选择工作流
    │
    ├─ 是否是开源项目？
    │   ├─ 是 → Forking Workflow
    │   └─ 否 ↓
    │
    ├─ 团队规模？
    │   ├─ 1-3人 → 集中式工作流
    │   ├─ 3-10人 ↓
    │   └─ 10+人 → Git Flow
    │
    ├─ 是否需要持续部署？
    │   ├─ 是 ↓
    │   └─ 否 → Git Flow
    │
    ├─ 是否需要多版本维护？
    │   ├─ 是 → GitLab Flow（发布分支模式）
    │   └─ 否 ↓
    │
    ├─ 是否有完善的自动化测试？
    │   ├─ 是 → Trunk-Based Development
    │   └─ 否 → GitHub Flow
    │
    └─ 使用 GitLab 平台？
        ├─ 是 → GitLab Flow
        └─ 否 → GitHub Flow
```

### 不同场景的推荐

#### 场景1：初创公司 Web 应用

- **推荐**：GitHub Flow
- **原因**：简单、适合持续部署、学习成本低
- **团队规模**：3-8 人
- **发布频率**：每天多次

#### 场景2：企业级 ERP 系统

- **推荐**：Git Flow
- **原因**：需要多版本维护、有固定发布周期
- **团队规模**：10-50 人
- **发布频率**：每月或每季度

#### 场景3：大型互联网公司

- **推荐**：Trunk-Based Development
- **原因**：快速迭代、持续部署、强大的自动化测试支持
- **团队规模**：50+ 人
- **发布频率**：每天多次

#### 场景4：开源项目

- **推荐**：Forking Workflow
- **原因**：权限隔离、支持外部贡献者
- **团队规模**：不限
- **发布频率**：根据项目

#### 场景5：使用 GitLab 的团队

- **推荐**：GitLab Flow
- **原因**：与 GitLab 平台深度集成
- **团队规模**：5-20 人
- **发布频率**：每周或每天

### 工作流迁移指南

#### 从集中式迁移到功能分支工作流

```bash
# 1. 创建 develop 分支
git checkout main
git pull origin main
git checkout -b develop
git push origin develop

# 2. 设置分支保护
# 在 GitHub/GitLab 上设置 main 分支保护

# 3. 培训团队
# 组织培训，讲解功能分支工作流

# 4. 制定规范
# 制定分支命名规范、提交信息规范

# 5. 逐步迁移
# 新功能使用功能分支，旧代码逐步迁移
```

#### 从 Git Flow 迁移到 GitHub Flow

```bash
# 1. 简化分支结构
# 删除不必要的分支
git branch -d feature/old-feature
git branch -d release/old-release

# 2. 合并 develop 到 main
git checkout main
git merge develop

# 3. 删除 develop 分支
git branch -d develop

# 4. 更新工作流程
# 所有功能从 main 创建分支
# 完成后通过 PR 合并回 main

# 5. 建立持续部署
# 配置 CI/CD，合并后自动部署
```

### 工作流选择的常见误区

| 误区 | 正确做法 |
|------|----------|
| 盲目选择复杂工作流 | 根据团队规模和项目需求选择 |
| 不考虑团队经验 | 选择团队能够理解和执行的工作流 |
| 忽视自动化测试 | 先建立自动化测试，再选择工作流 |
| 一成不变 | 随着项目发展调整工作流 |
| 缺乏规范 | 制定明确的分支、提交、审查规范 |

---

## 分支命名规范

### 常用命名规范

#### 1. 功能分支

```bash
# 格式：feature/简短描述
feature/user-login
feature/search-function
feature/payment-integration

# 格式：feature/Issue编号-简短描述
feature/123-user-login
feature/456-search-function
```

#### 2. Bug 修复分支

```bash
# 格式：bugfix/简短描述
bugfix/fix-login-error
bugfix/fix-search-pagination

# 格式：bugfix/Issue编号-简短描述
bugfix/789-fix-login-error
```

#### 3. 热修复分支

```bash
# 格式：hotfix/简短描述
hotfix/fix-security-vulnerability
hotfix/fix-payment-error
```

#### 4. 发布分支

```bash
# 格式：release/版本号
release/1.0.0
release/2.1.0
```

#### 5. 其他分支

```bash
# 文档分支
docs/update-readme
docs/add-api-documentation

# 测试分支
test/add-unit-tests
test/integration-tests

# 重构分支
refactor/user-module
refactor/database-layer
```

### 命名最佳实践

1. **使用小写字母和连字符**：`feature/user-login` 而不是 `feature/UserLogin`
2. **简洁明了**：分支名应能清楚表达用途
3. **包含 Issue 编号**：便于追踪和关联
4. **避免特殊字符**：不要使用空格、中文等特殊字符
5. **保持一致性**：团队统一命名规范

### 分支命名的组织策略

#### 1. 按团队/模块组织

```bash
# 用户团队
user/feature-login
user/bugfix-auth
user/refactor-profile

# 订单团队
order/feature-checkout
order/bugfix-payment
order/refactor-cart

# 支付团队
payment/feature-alipay
payment/bugfix-refund
payment/refactor-wechat
```

#### 2. 按优先级组织

```bash
# 紧急修复
hotfix/critical-security-fix
hotfix/payment-error

# 常规功能
feature/user-login
feature/search-function

# 低优先级
chore/update-dependencies
docs/readme-update
```

#### 3. 按版本组织

```bash
# 版本相关
release/v1.0.0
release/v1.1.0
release/v2.0.0

# 版本修复
hotfix/v1.0.1
hotfix/v1.0.2
```

### 分支命名自动化工具

#### 1. Git 别名配置

```bash
# 配置 Git 别名
git config --global alias.feature-start '!f() { git checkout main && git pull && git checkout -b "feature/$1"; }; f'
git config --global alias.bugfix-start '!f() { git checkout main && git pull && git checkout -b "bugfix/$1"; }; f'
git config --global alias.hotfix-start '!f() { git checkout main && git pull && git checkout -b "hotfix/$1"; }; f'

# 使用示例
git feature-start user-login    # 创建 feature/user-login
git bugfix-start fix-auth       # 创建 bugfix/fix-auth
git hotfix-start security-fix   # 创建 hotfix/security-fix
```

#### 2. 分支命名检查脚本

```bash
#!/bin/bash
# check-branch-name.sh

BRANCH_NAME=$(git rev-parse --abbrev-ref HEAD)

# 检查分支命名规范
if [[ ! $BRANCH_NAME =~ ^(main|develop|(feature|bugfix|hotfix|release|docs|test|refactor)/[a-z0-9-]+)$ ]]; then
  echo "错误: 分支名称不符合规范"
  echo "规范格式: feature/xxx, bugfix/xxx, hotfix/xxx, release/xxx"
  echo "当前分支: $BRANCH_NAME"
  exit 1
fi

echo "分支名称符合规范: $BRANCH_NAME"
```

#### 3. Husky 钩子配置

```json
// package.json
{
  "husky": {
    "hooks": {
      "pre-push": "bash check-branch-name.sh"
    }
  }
}
```

### 分支命名的常见问题

| 问题 | 示例 | 正确做法 |
|------|------|----------|
| 使用大写字母 | `Feature/User-Login` | `feature/user-login` |
| 使用中文 | `feature/用户登录` | `feature/user-login` |
| 使用空格 | `feature user login` | `feature/user-login` |
| 过于简短 | `feature/fix` | `feature/fix-login-error` |
| 过于冗长 | `feature/fix-the-login-error-that-occurs-when-user-enter-wrong-password` | `feature/fix-login-validation` |

---

## 提交信息规范

### Conventional Commits 规范

Conventional Commits 是一种用于标准化提交信息的规范，被广泛采用。

#### 基本格式

```
<type>(<scope>): <subject>

<body>

<footer>
```

#### Type 类型

| 类型 | 说明 | 示例 |
|------|------|------|
| feat | 新功能 | feat: 添加用户登录功能 |
| fix | Bug 修复 | fix: 修复登录验证错误 |
| docs | 文档更新 | docs: 更新 API 文档 |
| style | 代码格式（不影响功能） | style: 格式化代码 |
| refactor | 重构（既不是新功能也不是修复） | refactor: 重构用户模块 |
| perf | 性能优化 | perf: 优化查询性能 |
| test | 测试相关 | test: 添加单元测试 |
| build | 构建系统或外部依赖 | build: 更新 webpack 配置 |
| ci | CI 配置 | ci: 添加 GitHub Actions |
| chore | 其他杂项 | chore: 更新依赖版本 |
| revert | 回滚 | revert: 回滚上次提交 |

#### 示例

```bash
# 简单格式
git commit -m "feat: 添加用户登录功能"

# 带作用域
git commit -m "feat(auth): 添加 JWT 认证"

# 带详细描述
git commit -m "feat(auth): 添加用户登录功能

实现用户名/邮箱登录，支持记住登录状态。

Closes #123"

# 破坏性变更
git commit -m "feat(api)!: 更改用户接口

BREAKING CHANGE: 用户接口返回格式变更"
```

### Angular 提交规范

Angular 项目使用的提交规范，是 Conventional Commits 的一个实现：

```
<type>(<scope>): <short summary>
│       │             │
│       │             └─> 简短描述，不超过 50 个字符
│       │
│       └─> 影响范围（可选）
│
└─> 类型：feat|fix|docs|style|refactor|perf|test|build|ci|chore|revert
```

#### Angular 提交示例

```bash
# 功能
git commit -m "feat(user): add user registration"

# 修复
git commit -m "fix(auth): fix token expiration issue"

# 文档
git commit -m "docs(readme): update installation guide"

# 重构
git commit -m "refactor(api): simplify error handling"

# 破坏性变更
git commit -m "feat(api)!: change response format

BREAKING CHANGE: The response format has changed from XML to JSON."
```

### 提交信息验证工具

#### 1. commitlint

```bash
# 安装
npm install --save-dev @commitlint/cli @commitlint/config-conventional

# 配置 commitlint.config.js
module.exports = {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'type-enum': [
      2,
      'always',
      [
        'feat', 'fix', 'docs', 'style', 'refactor',
        'perf', 'test', 'build', 'ci', 'chore', 'revert'
      ]
    ],
    'scope-case': [2, 'always', 'lower-case'],
    'subject-case': [2, 'never', ['start-case', 'pascal-case', 'upper-case']],
    'subject-empty': [2, 'never'],
    'subject-full-stop': [2, 'never', '.'],
    'header-max-length': [2, 'always', 100]
  }
};
```

#### 2. Husky Git Hooks

```bash
# 安装 husky
npm install --save-dev husky

# 初始化 husky
npx husky install

# 添加 commit-msg hook
npx husky add .husky/commit-msg 'npx --no -- commitlint --edit $1'
```

### 提交信息最佳实践

1. **使用祈使语气**：`feat: add feature` 而不是 `feat: added feature`
2. **第一行不超过 50 字符**：简洁明了
3. **空行后添加详细描述**：解释为什么做这个修改
4. **引用相关 Issue**：使用 `Closes #123` 或 `Fixes #123`
5. **一个提交做一件事**：保持原子性

### 提交信息的高级技巧

#### 1. 多行提交信息

```bash
# 多行提交信息示例
git commit -m "feat(auth): implement JWT authentication

- Add JWT token generation and validation
- Implement login endpoint with email/password
- Add middleware for protected routes
- Include refresh token mechanism

Closes #123
Fixes #456"
```

#### 2. 使用脚本自动化

```bash
#!/bin/bash
# commit.sh - 自动化提交脚本

# 获取提交类型
echo "选择提交类型:"
echo "1. feat - 新功能"
echo "2. fix - Bug 修复"
echo "3. docs - 文档更新"
echo "4. style - 代码格式"
echo "5. refactor - 重构"
echo "6. perf - 性能优化"
echo "7. test - 测试相关"
echo "8. build - 构建系统"
echo "9. ci - CI 配置"
echo "10. chore - 其他杂项"
read -p "输入编号: " type_num

case $type_num in
  1) type="feat" ;;
  2) type="fix" ;;
  3) type="docs" ;;
  4) type="style" ;;
  5) type="refactor" ;;
  6) type="perf" ;;
  7) type="test" ;;
  8) type="build" ;;
  9) type="ci" ;;
  10) type="chore" ;;
  *) echo "无效选择"; exit 1 ;;
esac

# 获取作用域
read -p "输入作用域（可选）: " scope

# 获取描述
read -p "输入提交描述: " description

# 构建提交信息
if [ -z "$scope" ]; then
  commit_msg="$type: $description"
else
  commit_msg="$type($scope): $description"
fi

# 执行提交
git add .
git commit -m "$commit_msg"
```

#### 3. 使用 Commitizen

```bash
# 安装 Commitizen
npm install -g commitizen cz-conventional-changelog

# 初始化
echo '{ "path": "cz-conventional-changelog" }' > ~/.czrc

# 使用 cz 命令代替 git commit
cz
```

### 提交信息的错误示例与纠正

| 错误示例 | 问题 | 正确示例 |
|----------|------|----------|
| `fixed bug` | 缺少类型、过于模糊 | `fix: resolve user login validation error` |
| `update code` | 过于模糊 | `refactor: simplify authentication logic` |
| `add feature` | 缺少具体描述 | `feat: add user profile editing feature` |
| `修复bug` | 使用中文 | `fix: resolve user authentication issue` |
| `feat: 添加功能` | 使用中文 | `feat: add user registration feature` |
| `FEAT: ADD FEATURE` | 使用大写 | `feat: add user login feature` |
| `feat:add feature` | 缺少空格 | `feat: add user login feature` |

### 提交信息与自动化

#### 1. 自动生成 CHANGELOG

```bash
# 使用 conventional-changelog
npm install -g conventional-changelog-cli

# 生成 CHANGELOG
conventional-changelog -p angular -i CHANGELOG.md -s

# 配置 package.json
{
  "scripts": {
    "changelog": "conventional-changelog -p angular -i CHANGELOG.md -s",
    "release": "standard-version"
  }
}
```

#### 2. 自动确定版本号

```bash
# 使用 standard-version
npm install -g standard-version

# 自动确定版本号并生成 CHANGELOG
npm run release

# 根据提交信息自动确定版本号
# feat: Minor +1
# fix: Patch +1
# BREAKING CHANGE: Major +1
```

#### 3. 提交信息验证工具对比

| 工具 | 特点 | 适用场景 |
|------|------|----------|
| commitlint | 灵活、可配置 | 所有项目 |
| Husky | Git Hooks 管理 | 配合 commitlint 使用 |
| Commitizen | 交互式提交 | 新手友好的团队 |
| semantic-release | 自动化发布 | 持续部署的项目 |
| standard-version | 版本管理 | 需要版本管理的项目 |

---

## 版本号规范

### Semantic Versioning（语义化版本）

Semantic Versioning（SemVer）是最流行的版本号规范，格式为：`MAJOR.MINOR.PATCH`

#### 版本号格式

```
v1.2.3
│ │ │
│ │ └── Patch: 向后兼容的 Bug 修复
│ └──── Minor: 向后兼容的新功能
└────── Major: 不兼容的 API 变更
```

#### 版本号规则

| 变更类型 | 版本号变化 | 示例 |
|----------|------------|------|
| Bug 修复 | Patch +1 | 1.0.0 → 1.0.1 |
| 新功能 | Minor +1 | 1.0.0 → 1.1.0 |
| 破坏性变更 | Major +1 | 1.0.0 → 2.0.0 |

#### 预发布版本

```bash
# Alpha 版本（内部测试）
v1.0.0-alpha.1
v1.0.0-alpha.2

# Beta 版本（外部测试）
v1.0.0-beta.1
v1.0.0-beta.2

# Release Candidate（发布候选）
v1.0.0-rc.1
v1.0.0-rc.2
```

#### 版本号示例

```bash
# 初始版本
v0.1.0

# 开发阶段
v0.1.0 → v0.2.0 → v0.3.0

# 第一个正式版本
v1.0.0

# Bug 修复
v1.0.0 → v1.0.1 → v1.0.2

# 新功能
v1.0.2 → v1.1.0

# 破坏性变更
v1.1.0 → v2.0.0
```

### 其他版本号规范

#### 1. CalVer（日历版本）

基于日期的版本号，常用于 Ubuntu、Python 等项目：

```
# 格式：YY.MM.PATCH
Ubuntu: 22.04, 23.10

# 格式：YYYY.MM.DD
Python: 2024.1.15
```

#### 2. 自定义版本号

一些项目使用自定义的版本号格式：

```
# 语义化 + 构建号
v1.2.3-build.456

# 日期 + 序号
release-20240115-001
```

### Git 标签管理

```bash
# 创建轻量标签
git tag v1.0.0

# 创建附注标签
git tag -a v1.0.0 -m "Release version 1.0.0"

# 推送标签到远程
git push origin v1.0.0

# 推送所有标签
git push origin --tags

# 删除本地标签
git tag -d v1.0.0

# 删除远程标签
git push origin --delete v1.0.0

# 查看所有标签
git tag

# 查看特定标签信息
git show v1.0.0
```

### 版本号自动管理

#### 使用 standard-version

```bash
# 安装
npm install --save-dev standard-version

# 添加到 package.json scripts
{
  "scripts": {
    "release": "standard-version"
  }
}

# 运行自动版本发布
npm run release

# 根据提交信息自动确定版本号
# feat: Minor +1
# fix: Patch +1
# BREAKING CHANGE: Major +1
```

#### 使用 semantic-release

```bash
# 安装
npm install --save-dev semantic-release

# 配置 .releaserc.json
{
  "branches": ["main"],
  "plugins": [
    "@semantic-release/commit-analyzer",
    "@semantic-release/release-notes-generator",
    "@semantic-release/changelog",
    "@semantic-release/npm",
    "@semantic-release/github"
  ]
}
```

### 版本号管理的最佳实践

#### 1. 版本号与发布流程

```markdown
## 版本发布流程

### 1. 准备阶段
- 确定版本号类型（Major/Minor/Patch）
- 更新 CHANGELOG
- 更新版本号文件

### 2. 测试阶段
- 运行完整测试套件
- 性能测试
- 安全扫描

### 3. 发布阶段
- 创建 Git 标签
- 推送到远程仓库
- 部署到生产环境

### 4. 验证阶段
- 验证生产环境
- 监控错误日志
- 通知团队成员
```

#### 2. 版本号文件管理

```json
// package.json
{
  "name": "my-app",
  "version": "1.2.3",
  "scripts": {
    "version": "npm version"
  }
}
```

```yaml
# VERSION 文件
1.2.3
```

```python
# Python: __init__.py
__version__ = "1.2.3"
```

```java
// Java: pom.xml
<project>
  <version>1.2.3</version>
</project>
```

#### 3. 版本号与依赖管理

```json
// package.json（语义化版本范围）
{
  "dependencies": {
    "lodash": "^4.17.21",    // 兼容 4.x.x
    "react": "^18.2.0",      // 兼容 18.x.x
    "express": "~4.18.2",    // 兼容 4.18.x
    "axios": "1.6.0"         // 精确版本
  }
}
```

### 版本号管理的常见问题

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 版本号冲突 | 多人同时更新版本号 | 使用自动化工具 |
| 版本号回退 | 需要撤销发布 | 使用 npm unpublish 或 Git 标签 |
| 版本号不一致 | 不同文件版本号不同 | 使用统一的版本号管理脚本 |
| 预发布版本管理 | Alpha/Beta/RC 版本 | 使用预发布版本标识 |

### 版本号管理工具对比

| 工具 | 特点 | 适用场景 |
|------|------|----------|
| npm version | Node.js 内置 | Node.js 项目 |
| standard-version | 自动化版本管理 | 需要 CHANGELOG 的项目 |
| semantic-release | 全自动发布 | 持续部署的项目 |
| lerna | 多包管理 | Monorepo 项目 |
| changesets | 变更集管理 | 多包协作项目 |

---

## Release 管理策略

### 发布流程标准化

#### 1. 发布检查清单

```markdown
## 发布检查清单 v1.2.0

### 代码准备
- [ ] 所有功能分支已合并
- [ ] 所有测试通过
- [ ] 代码审查完成
- [ ] 无未解决的严重 Bug

### 文档更新
- [ ] CHANGELOG 更新
- [ ] API 文档更新
- [ ] 用户手册更新

### 测试验证
- [ ] 单元测试通过
- [ ] 集成测试通过
- [ ] 性能测试通过
- [ ] 安全测试通过

### 部署准备
- [ ] 数据库迁移脚本准备
- [ ] 配置文件更新
- [ ] 回滚方案准备

### 发布执行
- [ ] 创建发布分支
- [ ] 更新版本号
- [ ] 创建 Git 标签
- [ ] 部署到测试环境
- [ ] 测试验证
- [ ] 部署到生产环境
- [ ] 监控验证
```

#### 2. 发布流程图

```
┌─────────────┐
│ 功能开发完成 │
└──────┬──────┘
       ↓
┌─────────────┐
│ 代码审查    │
└──────┬──────┘
       ↓
┌─────────────┐
│ 自动化测试  │
└──────┬──────┘
       ↓
┌─────────────┐
│ 创建发布分支│
└──────┬──────┘
       ↓
┌─────────────┐
│ 测试环境验证│
└──────┬──────┘
       ↓
┌─────────────┐
│ 生产环境部署│
└──────┬──────┘
       ↓
┌─────────────┐
│ 监控和验证  │
└──────┬──────┘
       ↓
┌─────────────┐
│ 发布完成    │
└─────────────┘
```

### CHANGELOG 管理

#### CHANGELOG 格式

```markdown
# Changelog

## [1.2.0] - 2024-01-15

### Added
- 添加用户搜索功能 (#123)
- 添加数据导出功能 (#124)

### Changed
- 优化查询性能 (#125)
- 更新用户界面 (#126)

### Fixed
- 修复登录验证错误 (#127)
- 修复分页显示问题 (#128)

### Security
- 更新依赖版本修复安全漏洞 (#129)

## [1.1.0] - 2024-01-01

### Added
- 添加用户注册功能 (#120)
...
```

#### 自动生成 CHANGELOG

```bash
# 使用 conventional-changelog
npm install --save-dev conventional-changelog-cli

# 生成 CHANGELOG
conventional-changelog -p angular -i CHANGELOG.md -s

# 配置 package.json
{
  "scripts": {
    "changelog": "conventional-changelog -p angular -i CHANGELOG.md -s"
  }
}
```

### 热修复流程

```bash
# 1. 从生产版本创建热修复分支
git checkout v1.0.0
git checkout -b hotfix/fix-critical-bug

# 2. 修复 Bug
# ... 修复代码 ...
git add .
git commit -m "fix: 修复支付验证漏洞"

# 3. 更新版本号
npm version patch  # 1.0.0 → 1.0.1

# 4. 合并到主分支
git checkout main
git merge hotfix/fix-critical-bug

# 5. 创建标签
git tag -a v1.0.1 -m "Hotfix: 修复支付验证漏洞"

# 6. 部署到生产环境
git push origin main --tags

# 7. 清理分支
git branch -d hotfix/fix-critical-bug
```

### Release 管理的最佳实践

#### 1. 自动化发布流程

```yaml
# .github/workflows/release.yml
name: Release

on:
  push:
    tags:
      - 'v*'

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run tests
        run: npm test
      
      - name: Build
        run: npm run build
      
      - name: Create Release
        uses: actions/create-release@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          tag_name: ${{ github.ref }}
          release_name: Release ${{ github.ref }}
          body: |
            ## Changes
            - See CHANGELOG.md for details
          draft: false
          prerelease: false
      
      - name: Deploy to Production
        run: |
          echo "Deploying to production..."
          # 部署脚本
```

#### 2. 蓝绿部署策略

```bash
#!/bin/bash
# blue-green-deploy.sh

set -e

CURRENT_ENV=$(curl -s http://localhost/current-env)
NEW_ENV=""

if [ "$CURRENT_ENV" = "blue" ]; then
  NEW_ENV="green"
else
  NEW_ENV="blue"
fi

echo "当前环境: $CURRENT_ENV"
echo "部署到: $NEW_ENV"

# 部署到新环境
./deploy.sh $NEW_ENV

# 运行健康检查
./health-check.sh $NEW_ENV

# 切换流量
./switch-traffic.sh $NEW_ENV

echo "部署完成！当前环境: $NEW_ENV"
```

#### 3. 灰度发布策略

```yaml
# 灰度发布配置
canary_release:
  stages:
    - name: 内部测试
      percentage: 1%
      duration: 1h
    
    - name: 小规模灰度
      percentage: 5%
      duration: 2h
    
    - name: 中等规模灰度
      percentage: 20%
      duration: 4h
    
    - name: 大规模灰度
      percentage: 50%
      duration: 8h
    
    - name: 全量发布
      percentage: 100%
      duration: 0
  
  rollback:
    error_rate_threshold: 1%
    latency_threshold: 500ms
```

### Release 管理工具对比

| 工具 | 特点 | 适用场景 |
|------|------|----------|
| GitHub Releases | 简单易用 | GitHub 项目 |
| GitLab Releases | 功能丰富 | GitLab 项目 |
| semantic-release | 全自动 | 持续部署项目 |
| release-it | 灵活配置 | 各类项目 |
| lerna | 多包管理 | Monorepo 项目 |

---

## 多人协作的冲突预防

### 冲突产生的原因

1. **同一文件的同一位置被修改**
2. **文件被删除但有人修改了它**
3. **二进制文件冲突**
4. **长时间不合并导致的分歧**

### 预防策略

#### 1. 频繁拉取和合并

```bash
# 开始工作前先拉取最新代码
git checkout main
git pull origin main

# 功能分支定期合并主分支
git checkout feature/my-feature
git merge main

# 或者使用 rebase
git checkout feature/my-feature
git rebase main
```

#### 2. 小批量提交

```bash
# 不好的做法：一次性提交大量修改
git add .
git commit -m "完成所有功能"

# 好的做法：小批量、频繁提交
git add src/user/login.js
git commit -m "feat: 实现用户登录"

git add src/user/register.js
git commit -m "feat: 实现用户注册"
```

#### 3. 模块化代码组织

```
project/
├── src/
│   ├── user/          # 用户模块
│   │   ├── login.js
│   │   └── register.js
│   ├── order/         # 订单模块
│   │   ├── create.js
│   │   └── query.js
│   └── payment/       # 支付模块
│       ├── alipay.js
│       └── wechat.js
```

#### 4. 使用 .gitignore

```gitignore
# IDE 配置
.idea/
.vscode/
*.swp
*.swo

# 系统文件
.DS_Store
Thumbs.db

# 依赖目录
node_modules/
vendor/

# 构建产物
dist/
build/

# 环境配置
.env
.env.local
```

### 冲突解决方法

#### 1. 手动解决冲突

```bash
# 合并时出现冲突
git merge feature-branch
# CONFLICT (content): Merge conflict in src/app.js

# 查看冲突文件
git status

# 编辑冲突文件
<<<<<<< HEAD
// 当前分支的代码
const url = 'https://api.example.com';
=======
// 合并分支的代码
const url = 'https://api.new.com';
>>>>>>> feature-branch

# 手动选择或合并代码
const url = 'https://api.new.com';

# 标记冲突已解决
git add src/app.js

# 完成合并
git commit -m "merge: 合并 feature-branch，解决冲突"
```

#### 2. 使用合并工具

```bash
# 配置合并工具
git config --global merge.tool vscode
git config --global mergetool.vscode.cmd 'code --wait --merge $REMOTE $LOCAL $BASE $MERGED'

# 使用合并工具解决冲突
git mergetool
```

#### 3. 使用 rebase

```bash
# 使用 rebase 代替 merge
git checkout feature/my-feature
git rebase main

# 解决冲突后继续 rebase
git add .
git rebase --continue

# 或者放弃 rebase
git rebase --abort
```

### 团队协作规范

#### 1. 代码所有权文件（CODEOWNERS）

```bash
# .github/CODEOWNERS 或 docs/CODEOWNERS

# 默认所有者
*       @team-lead

# 用户模块
/src/user/    @user-team

# 订单模块
/src/order/   @order-team

# 文档
/docs/        @doc-team
```

#### 2. 分支保护规则

```yaml
# GitHub 分支保护配置示例
branches:
  main:
    protection:
      required_pull_request_reviews:
        required_approving_review_count: 2
        dismiss_stale_reviews: true
      required_status_checks:
        strict: true
        contexts:
          - ci/build
          - ci/test
      enforce_admins: true
      restrictions:
        users: []
        teams:
          - core-team
```

#### 3. 代码审查清单

```markdown
## 代码审查清单

### 代码质量
- [ ] 代码风格符合规范
- [ ] 命名清晰有意义
- [ ] 无重复代码
- [ ] 适当的注释

### 功能实现
- [ ] 功能实现正确
- [ ] 边界条件处理
- [ ] 错误处理完善
- [ ] 性能考虑

### 测试覆盖
- [ ] 单元测试完整
- [ ] 测试用例覆盖边界情况
- [ ] 测试通过

### 安全考虑
- [ ] 输入验证
- [ ] 权限检查
- [ ] 敏感信息处理

### 文档更新
- [ ] API 文档更新
- [ ] README 更新
- [ ] CHANGELOG 更新
```

### 冲突解决的高级技巧

#### 1. 使用 rerere（Reuse Recorded Resolution）

Git 的 rerere 功能可以记住冲突解决方式，下次遇到相同冲突时自动应用：

```bash
# 启用 rerere
git config --global rerere.enabled true

# 查看 rerere 记录
git rerere status

# 清除 rerere 记录
git rerere forget <path>
```

#### 2. 使用交互式 rebase

```bash
# 交互式 rebase，合并多个提交
git rebase -i HEAD~3

# 编辑器会显示：
# pick abc1234 第一个提交
# pick def5678 第二个提交
# pick ghi9012 第三个提交

# 修改为：
# pick abc1234 第一个提交
# squash def5678 第二个提交
# squash ghi9012 第三个提交
```

#### 3. 使用 cherry-pick

```bash
# 从其他分支选择特定提交
git cherry-pick <commit-hash>

# 选择多个提交
git cherry-pick <commit1> <commit2>

# 选择提交范围
git cherry-pick <start-commit>..<end-commit>
```

### 冲突预防的团队协作规范

```markdown
## 团队协作规范

### 沟通规范
- 每天站会同步进度
- 大功能提前沟通
- 代码审查及时完成
- 问题及时反馈

### 代码规范
- 统一代码风格
- 使用 ESLint/Prettier
- 遵循项目架构
- 模块化开发

### 提交规范
- 小批量提交
- 频繁集成
- 使用功能开关
- 保持主分支稳定

### 审查规范
- 至少 2 人审查
- 24 小时内完成
- 关注代码质量
- 提供建设性反馈
```

---

## 中国互联网公司的 Git 工作流实践

### 阿里巴巴

阿里巴巴主要采用 **Aone Flow** 工作流，这是基于 Trunk-Based Development 的改进版本：

#### 核心特点

1. **主干开发**：所有开发在主干（master）上进行
2. **特性分支短生命周期**：功能分支不超过 1-2 天
3. **发布分支**：从主干拉取发布分支到生产环境
4. **环境分支**：测试、预发、生产环境分离

#### 工作流程

```
master    ──●──●──●──●──●──●──●──●──●──→
              \         \         \
release-test   ●───●     │         │
                \         \         \
release-pre      ●─────●   │         │
                  \         \         \
release-prod       ●───────●───●──────●──→
```

```bash
# 1. 从 master 创建功能分支
git checkout master
git pull origin master
git checkout -b feature/user-login

# 2. 开发完成，合并回 master
git checkout master
git merge feature/user-login

# 3. 从 master 创建测试分支
git checkout -b release-test master

# 4. 测试通过后，创建预发分支
git checkout -b release-pre release-test

# 5. 预发验证通过后，创建生产分支
git checkout -b release-prod release-pre
```

### 腾讯

腾讯内部使用 **工蜂（工蜂 Git）** 平台，主要采用 **Git Flow** 的变体：

#### 核心特点

1. **主分支保护**：master 分支需要代码审查才能合并
2. **功能分支**：每个功能在独立分支开发
3. **集成分支**：使用 develop 分支进行功能集成
4. **发布分支**：从 develop 创建发布分支

#### 分支策略

```
master    ──●───────────────●───────────────●──→
              ↑               ↑               ↑
              │               │               │
release-1.0   ●───────●───────●               │
              ↑               ↑               │
              │               │               │
develop  ─────●───────●───────●───────●───────●──→
              ↑       ↑       ↑       ↑
              │       │       │       │
feature-1     ●───────●       │       │
                              │       │
feature-2             ●───────●       │
                                      │
feature-3                     ●───────●
```

### 字节跳动

字节跳动采用 **Trunk-Based Development** 为主的工作流：

#### 核心特点

1. **主干开发**：所有代码提交到主分支
2. **短生命周期分支**：功能分支不超过 24 小时
3. **功能开关**：使用功能开关控制未完成功能
4. **自动化测试**：完善的自动化测试体系

#### 实践要点

```javascript
// 功能开关配置
const featureFlags = {
  'new-search': {
    enabled: true,
    rollout: 10,  // 灰度 10%
    whitelist: ['user1', 'user2']
  },
  'new-payment': {
    enabled: false,
    rollout: 0
  }
};

// 使用功能开关
if (featureFlags.isEnabled('new-search')) {
  // 新搜索功能
} else {
  // 旧搜索功能
}
```

### 美团

美团采用 **Git Flow** 与 **GitHub Flow** 结合的方式：

#### 核心特点

1. **主分支保护**：master 分支需要 CR（Code Review）才能合并
2. **功能分支**：每个功能在独立分支开发
3. **发布分支**：从 master 创建发布分支
4. **热修复分支**：紧急修复从发布分支创建

#### 工作流程

```bash
# 1. 创建功能分支
git checkout master
git pull origin master
git checkout -b feature/new-feature

# 2. 开发并提交
git add .
git commit -m "feat: 实现新功能"

# 3. 创建 MR（Merge Request）
# 在 GitLab 上创建 MR，请求代码审查

# 4. CR 通过后合并
git checkout master
git merge feature/new-feature

# 5. 部署到测试环境
git checkout test
git merge master

# 6. 测试通过后部署到生产
git checkout production
git merge test
```

### 华为

华为采用 **DevOps + Git Flow** 的工作流：

#### 核心特点

1. **代码门禁**：提交前必须通过自动化检查
2. **代码审查**：强制代码审查制度
3. **分支管理**：严格的分支管理策略
4. **安全扫描**：集成安全扫描工具

#### 代码门禁配置

```yaml
# 代码门禁检查项
code_gate:
  checks:
    - name: unit_test
      command: "npm test"
      threshold: 80%  # 测试覆盖率要求

    - name: lint
      command: "npm run lint"
      threshold: 0  # 无 lint 错误

    - name: security_scan
      command: "npm audit"
      threshold: 0  # 无高危漏洞

    - name: code_review
      required: true
      min_reviewers: 2

  blocking: true  # 门禁检查失败则阻断提交
```

### 中国互联网公司的通用实践

#### 1. 代码审查制度

```markdown
## 代码审查流程

1. 开发者提交 MR/PR
2. 自动化检查（CI/CD）
3. 至少 2 人审查通过
4. 必须解决所有审查意见
5. 合并到主分支

## 审查要点

- 代码质量
- 性能影响
- 安全风险
- 测试覆盖
- 文档更新
```

#### 2. 分支保护策略

```yaml
# 通用分支保护配置
branch_protection:
  master:
    required_reviews: 2
    dismiss_stale_reviews: true
    require_status_checks: true
    required_checks:
      - ci/build
      - ci/test
      - security/scan
    restrict_pushes: true
    allowed_pushers:
      - @team-lead
      - @devops
```

#### 3. 发布流程

```bash
# 通用发布流程
# 1. 代码冻结
git checkout master
git tag -a code-freeze-v1.2.0 -m "代码冻结 v1.2.0"

# 2. 创建发布分支
git checkout -b release-v1.2.0

# 3. 测试和修复
# ... 修复 bug ...
git commit -am "fix: 修复发布版本 bug"

# 4. 部署到预发环境
git checkout pre-release
git merge release-v1.2.0

# 5. 预发验证通过后部署到生产
git checkout production
git merge pre-release
git tag -a v1.2.0 -m "Release v1.2.0"

# 6. 合并回主分支
git checkout master
git merge release-v1.2.0
```

### 中国互联网公司的工具生态

#### 1. 代码托管平台

| 平台 | 特点 | 适用场景 |
|------|------|----------|
| Gitee（码云） | 国内访问快、中文界面 | 国内团队、开源项目 |
| CODING | 腾讯云旗下、DevOps 工具链 | 企业级开发 |
| 阿里云 Codeup | 阿里云旗下、与阿里云集成 | 阿里云用户 |
| GitLab 中国版 | 私有部署、功能完整 | 大型企业 |

#### 2. CI/CD 工具

| 工具 | 特点 | 适用场景 |
|------|------|----------|
| Jenkins | 开源、插件丰富 | 各类项目 |
| 云效 | 阿里云旗下、一站式 DevOps | 阿里云用户 |
| CODING CI | 腾讯云旗下、简单易用 | 腾讯云用户 |
| GitLab CI | 与 GitLab 集成 | GitLab 用户 |
| GitHub Actions | 与 GitHub 集成 | GitHub 用户 |

#### 3. 代码质量工具

```yaml
# 代码质量检查配置
code_quality:
  tools:
    - name: ESLint
      purpose: JavaScript/TypeScript 代码检查
    
    - name: Prettier
      purpose: 代码格式化
    
    - name: SonarQube
      purpose: 代码质量分析
    
    - name: CodeQL
      purpose: 安全漏洞扫描
  
  integration:
    - 与 CI/CD 集成
    - 自动化检查
    - 质量门禁
```

### 中国互联网公司的最佳实践总结

#### 1. 分支管理策略

```markdown
## 分支管理最佳实践

### 主分支保护
- 强制代码审查
- 自动化测试通过
- 禁止直接推送

### 功能分支规范
- 短生命周期（1-3天）
- 频繁同步主分支
- 及时清理已合并分支

### 发布分支管理
- 从主分支创建
- 只修复 bug
- 测试通过后合并
```

#### 2. 代码审查流程

```markdown
## 代码审查最佳实践

### 审查时机
- 提交后 24 小时内完成
- 紧急修复 2 小时内完成

### 审查要点
- 代码质量
- 性能影响
- 安全风险
- 测试覆盖

### 审查反馈
- 具体、可操作
- 建设性意见
- 及时响应修改
```

#### 3. 持续集成实践

```yaml
# 持续集成最佳实践
ci_cd:
  pipeline:
    - stage: 代码检查
      tools:
        - ESLint
        - Prettier
        - TypeScript
    
    - stage: 单元测试
      coverage_threshold: 80%
    
    - stage: 集成测试
      environment: test
    
    - stage: 安全扫描
      tools:
        - npm audit
        - Snyk
    
    - stage: 构建
      artifacts:
        - dist/
        - build/
    
    - stage: 部署
      environments:
        - staging
        - production
```

---

## 各工作流的决策流程图

### 工作流选择决策图

```
                    开始选择工作流
                          │
                          ▼
              ┌─────────────────────┐
              │ 是否是开源项目？     │
              └─────────────────────┘
                    │           │
                   是           否
                    │           │
                    ▼           ▼
          ┌─────────────┐  ┌─────────────────────┐
          │ Forking      │  │ 团队规模是多少？     │
          │ Workflow     │  └─────────────────────┘
          └─────────────┘        │         │         │
                              1-3人     3-10人     10+人
                                │         │         │
                                ▼         ▼         ▼
                      ┌───────────┐ ┌───────────┐ ┌───────────┐
                      │ 集中式    │ │ 功能分支   │ │ Git Flow  │
                      │ 工作流    │ │ 工作流     │ │ 工作流    │
                      └───────────┘ └───────────┘ └───────────┘
```

### 持续部署决策图

```
                    是否需要持续部署？
                          │
                    ┌─────┴─────┐
                   是           否
                    │           │
                    ▼           ▼
          ┌─────────────────┐  ┌─────────────────┐
          │ 自动化测试是否   │  │ 是否需要多版本   │
          │ 完善？          │  │ 维护？          │
          └─────────────────┘  └─────────────────┘
                │       │           │       │
               是       否         是       否
                │       │           │       │
                ▼       ▼           ▼       ▼
      ┌───────────┐ ┌───────────┐ ┌───────────┐ ┌───────────┐
      │ Trunk-    │ │ GitHub    │ │ GitLab    │ │ Git Flow  │
      │ Based     │ │ Flow      │ │ Flow      │ │ 工作流    │
      │ Dev       │ │           │ │           │ │           │
      └───────────┘ └───────────┘ └───────────┘ └───────────┘
```

### 分支策略决策图

```
                    如何选择分支策略？
                          │
                          ▼
              ┌─────────────────────┐
              │ 是否需要发布分支？   │
              └─────────────────────┘
                    │           │
                   是           否
                    │           │
                    ▼           ▼
          ┌─────────────────┐  ┌─────────────────┐
          │ 是否需要维护     │  │ 是否需要功能     │
          │ 多个版本？       │  │ 开关？          │
          └─────────────────┘  └─────────────────┘
                │       │           │       │
               是       否         是       否
                │       │           │       │
                ▼       ▼           ▼       ▼
      ┌───────────┐ ┌───────────┐ ┌───────────┐ ┌───────────┐
      │ GitLab    │ │ Git Flow  │ │ Trunk-    │ │ GitHub    │
      │ Flow      │ │           │ │ Based     │ │ Flow      │
      │ (发布分支)│ │           │ │ Dev       │ │           │
      └───────────┘ └───────────┘ └───────────┘ └───────────┘
```

### 冲突解决决策图

```
                    遇到代码冲突
                          │
                          ▼
              ┌─────────────────────┐
              │ 冲突类型是什么？     │
              └─────────────────────┘
                    │       │       │
                 内容冲突  删除冲突  二进制冲突
                    │       │       │
                    ▼       ▼       ▼
          ┌─────────────────────────────────────┐
          │ 冲突复杂程度？                       │
          └─────────────────────────────────────┘
                    │               │
                 简单冲突         复杂冲突
                    │               │
                    ▼               ▼
          ┌─────────────────┐  ┌─────────────────┐
          │ 手动编辑解决     │  │ 使用合并工具     │
          │ git add .       │  │ git mergetool   │
          │ git commit      │  │                 │
          └─────────────────┘  └─────────────────┘
```

### 代码审查决策图

```
                    代码审查流程
                          │
                          ▼
              ┌─────────────────────┐
              │ 提交 PR/MR          │
              └─────────────────────┘
                          │
                          ▼
              ┌─────────────────────┐
              │ 自动化检查通过？     │
              └─────────────────────┘
                    │           │
                   是           否
                    │           │
                    ▼           ▼
          ┌─────────────────┐  ┌─────────────────┐
          │ 人工代码审查     │  │ 修复自动化       │
          │                 │  │ 检查问题         │
          └─────────────────┘  └─────────────────┘
                    │
                    ▼
              ┌─────────────────────┐
              │ 审查通过？          │
              └─────────────────────┘
                    │           │
                   是           否
                    │           │
                    ▼           ▼
          ┌─────────────────┐  ┌─────────────────┐
          │ 合并到主分支     │  │ 根据审查意见     │
          │                 │  │ 修改代码         │
          └─────────────────┘  └─────────────────┘
```

---

## 总结

### 选择工作流的关键因素

1. **团队规模**：小团队选择简单的工作流，大团队选择规范的工作流
2. **发布频率**：持续部署选择 GitHub Flow，固定周期选择 Git Flow
3. **平台选择**：使用 GitLab 选择 GitLab Flow，使用 GitHub 选择 GitHub Flow
4. **项目类型**：开源项目选择 Forking Workflow，内部项目选择其他工作流
5. **团队经验**：新手团队选择简单工作流，经验丰富的团队可以选择复杂工作流

### 最佳实践总结

1. **统一规范**：团队统一使用相同的分支命名、提交信息规范
2. **代码审查**：所有代码变更都应经过代码审查
3. **自动化测试**：建立完善的自动化测试体系
4. **持续集成**：集成 CI/CD 工具，自动化构建和部署
5. **文档记录**：记录工作流规范和最佳实践
6. **定期回顾**：定期回顾和优化工作流

### 推荐学习路径

```
入门阶段
    │
    ├─ 学习 Git 基本操作
    ├─ 理解分支概念
    └─ 尝试集中式工作流
        │
        ▼
进阶阶段
    │
    ├─ 学习功能分支工作流
    ├─ 掌握 Pull Request
    └─ 了解代码审查流程
        │
        ▼
高级阶段
    │
    ├─ 学习 Git Flow
    ├─ 了解 GitHub Flow / GitLab Flow
    └─ 掌握版本管理和发布流程
        │
        ▼
专家阶段
    │
    ├─ 实践 Trunk-Based Development
    ├─ 建立完善的 CI/CD 流程
    └─ 优化团队协作效率
```

---

## 附录

### 常用 Git 命令速查表

```bash
# 分支操作
git branch                    # 查看本地分支
git branch -a                 # 查看所有分支
git branch -d <branch>        # 删除本地分支
git checkout -b <branch>      # 创建并切换分支
git switch -c <branch>        # 创建并切换分支（新语法）

# 合并操作
git merge <branch>            # 合并分支
git rebase <branch>           # 变基
git cherry-pick <commit>      # 摘取提交

# 标签操作
git tag                       # 查看标签
git tag -a v1.0.0 -m "msg"   # 创建标签
git push origin --tags        # 推送标签

# 远程操作
git remote -v                 # 查看远程仓库
git remote add <name> <url>   # 添加远程仓库
git fetch <remote>            # 获取远程更新
git pull <remote> <branch>    # 拉取并合并
git push <remote> <branch>    # 推送

# 暂存操作
git stash                     # 暂存当前工作
git stash pop                 # 恢复暂存的工作
git stash list                # 查看暂存列表

# 日志操作
git log                       # 查看提交日志
git log --oneline             # 简洁日志
git log --graph               # 图形化日志
git log --author=<name>       # 按作者查看
```

### 推荐工具

| 工具 | 用途 | 官网 |
|------|------|------|
| Git | 版本控制 | https://git-scm.com |
| GitHub | 代码托管 | https://github.com |
| GitLab | 代码托管 | https://gitlab.com |
| SourceTree | Git GUI | https://www.sourcetreeapp.com |
| GitKraken | Git GUI | https://www.gitkraken.com |
| VS Code | 代码编辑器 | https://code.visualstudio.com |
| Husky | Git Hooks | https://typicode.github.io/husky |
| commitlint | 提交检查 | https://commitlint.js.org |
| standard-version | 版本管理 | https://github.com/conventional-changelog/standard-version |

### 参考资料

1. [Git 官方文档](https://git-scm.com/doc)
2. [GitHub Flow](https://docs.github.com/en/get-started/quickstart/github-flow)
3. [GitLab Flow](https://docs.gitlab.com/ee/topics/gitlab_flow.html)
4. [A Successful Git Branching Model](https://nvie.com/posts/a-successful-git-branching-model/)
5. [Conventional Commits](https://www.conventionalcommits.org/)
6. [Semantic Versioning](https://semver.org/)
7. [Trunk-Based Development](https://trunkbaseddevelopment.com/)

---

> 本文档最后更新：2024年1月
> 
> 如有疑问或建议，欢迎提交 Issue 或 Pull Request。