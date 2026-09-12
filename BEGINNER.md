# 🎯 GitHub 新手学习路线

> 一份为中文开发者量身定制的 GitHub 学习路线图。按照你的水平选择起点，循序渐进，从零基础到高级实践，每一步都有清晰的目标和配套教程。

---

## 📊 你的水平在哪里？

不确定从哪里开始？花 1 分钟做个自我评估：

| 问题 | 如果你的回答是"否" |
|------|-------------------|
| 你知道什么是版本控制吗？ | 👉 从 **Level 1** 开始 |
| 你能在本地使用 `git add`、`git commit` 吗？ | 👉 从 **Level 2** 开始 |
| 你能在 GitHub 上创建仓库并推送代码吗？ | 👉 从 **Level 3** 开始 |
| 你用过 Pull Request 和 Code Review 吗？ | 👉 从 **Level 4** 开始 |
| 你配置过 GitHub Actions 自动化流程吗？ | 👉 从 **Level 5** 开始 |
| 你了解 Copilot、K8s、Terraform 等高级工具吗？ | 👉 进入 **Level 6** |

> 💡 **建议**：即使你有一定基础，也建议快速浏览前面的级别，查漏补缺。很多看似简单的概念，深入理解后会让你事半功倍。

---

## 🗺️ 学习路线总览

```
Level 1 ──→ Level 2 ──→ Level 3 ──→ Level 4 ──→ Level 5 ──→ Level 6
 零基础       Git操作      GitHub入门    团队协作      自动化CI/CD    高级主题
 1-2天        3-5天        1周          1-2周         1-2周         持续学习
```

---

## Level 1：完全零基础 — 什么是 Git 和 GitHub？

**🎯 目标**：理解版本控制的核心概念，完成 GitHub 账号注册，搭建 Git 环境

**⏱️ 预计时间**：1-2 天

**📋 前置要求**：无需任何编程经验，只需要一台电脑和学习的热情

### 你将学到什么

- 版本控制是什么，为什么每个开发者都需要它
- Git 和 GitHub 的区别与联系
- 如何注册 GitHub 账号并完成基本设置
- 如何在自己的电脑上安装和配置 Git

### 📖 教程列表

| 序号 | 教程 | 说明 |
|------|------|------|
| 0 | [前言](docs/00-preface.md) | 了解本教程的定位、目标读者和使用方法，帮助你制定合理的学习计划 |
| 1 | [什么是版本控制](docs/00-what-is-version-control.md) | 从零理解版本控制的概念，明白为什么它能极大提升开发效率和团队协作能力 |
| 2 | [什么是 GitHub](docs/01-what-is-github.md) | 了解 GitHub 的核心功能、生态系统以及它在现代软件开发中的重要地位 |
| 3 | [Git 安装指南](docs/00-git-installation-guide.md) | 在 Windows、macOS、Linux 三大操作系统上安装 Git 的详细图文教程 |
| 4 | [安装 Git](docs/02-install-git.md) | 更多安装细节和常见安装问题的解决方案，确保你的 Git 环境正常运行 |
| 5 | [GitHub 注册指南](docs/00-github-signup-guide.md) | 手把手教你注册 GitHub 账号，完成邮箱验证、个人资料设置等初始化步骤 |
| 6 | [注册 GitHub](docs/03-signup-github.md) | 注册过程中的注意事项和常见问题解答，帮你顺利迈出第一步 |

### ✅ 完成标准

- [ ] 能用自己的话解释"什么是版本控制"
- [ ] 能区分 Git（工具）和 GitHub（平台）的不同
- [ ] 已在电脑上成功安装 Git（运行 `git --version` 能看到版本号）
- [ ] 已成功注册 GitHub 账号并登录

### 🇨🇳 国内用户特别提示

如果你在国内网络环境下使用 GitHub，建议提前阅读：

- [国内加速指南](docs/Q-china-acceleration.md) — 配置镜像源和代理，提升访问速度
- [国内直连指南](docs/Q2-china-direct-access.md) — 多种方案解决国内访问 GitHub 的网络问题

---

## Level 2：学会基本 Git 操作

**🎯 目标**：掌握 Git 的核心操作，能够在本地独立管理代码版本

**⏱️ 预计时间**：3-5 天

**📋 前置要求**：完成 Level 1，Git 已安装并配置完成

### 你将学到什么

- Git 的核心工作原理（工作区、暂存区、仓库的概念）
- 如何创建仓库、暂存文件、提交更改
- 如何查看提交历史和文件差异
- 如何使用分支进行并行开发
- 如何合并分支、解决代码冲突
- 如何撤销错误的操作

### 📖 教程列表

| 序号 | 教程 | 说明 |
|------|------|------|
| 7 | [SSH 密钥](docs/04-ssh-keys.md) | 配置 SSH 密钥实现免密推送，这是使用 GitHub 的必备技能，也是安全最佳实践 |
| 8 | [Git 配置](docs/05-git-config.md) | 设置用户名、邮箱、编辑器等基础配置，让你的 Git 环境个性化且高效 |
| 9 | [Git 工作原理](docs/06-how-git-works.md) | 深入理解工作区、暂存区、本地仓库的概念，掌握这些会让后续学习事半功倍 |
| 10 | [Git 基础指南](docs/00-git-basics-guide.md) | Git 基础操作的综合概述，帮你建立完整的知识框架 |
| 11 | [创建与克隆仓库](docs/07-init-clone.md) | 学会 `git init` 初始化新仓库和 `git clone` 克隆已有仓库的两种方式 |
| 12 | [暂存与提交](docs/08-add-commit.md) | 掌握 `git add` 和 `git commit` 的使用方法，理解暂存区的作用和提交的最佳实践 |
| 13 | [查看历史与差异](docs/09-log-diff.md) | 使用 `git log`、`git diff` 等命令追踪代码变更，成为代码侦探 |
| 14 | [分支操作](docs/10-branching.md) | 学会创建、切换、删除分支，理解分支是 Git 最强大的特性之一 |
| 15 | [合并与变基](docs/11-merge-rebase.md) | 掌握 `git merge` 和 `git rebase` 的区别与使用场景，选择合适的整合方式 |
| 16 | [解决冲突](docs/12-resolve-conflicts.md) | 学会识别和解决合并冲突，这是团队协作中不可避免但可以优雅处理的技能 |
| 17 | [撤销操作](docs/13-undo.md) | 掌握 `git reset`、`git revert`、`git checkout` 等撤销操作，知道什么时候用哪个 |

### ✅ 完成标准

- [ ] 能独立创建 Git 仓库并进行多次提交
- [ ] 能使用 `git log` 和 `git diff` 查看历史和差异
- [ ] 能创建分支、切换分支、合并分支
- [ ] 能解决简单的合并冲突
- [ ] 理解 `git reset` 和 `git revert` 的区别

### 💡 学习建议

> Git 的学习曲线在初期会比较陡峭，不要急于求成。建议每学一个命令就动手实践一遍，用一个测试仓库反复练习。记住：**犯错是学习的一部分**，Git 几乎所有的操作都是可以撤销的。

---

## Level 3：开始使用 GitHub

**🎯 目标**：熟练使用 GitHub 平台的核心功能，能够独立管理远程仓库

**⏱️ 预计时间**：1 周

**📋 前置要求**：完成 Level 2，熟悉本地 Git 操作

### 你将学到什么

- 在 GitHub 上创建和管理远程仓库
- 编写高质量的 README 和项目文档
- 使用 Issue 追踪问题和管理任务
- 掌握 GitHub 的核心协作功能

### 📖 教程列表

| 序号 | 教程 | 说明 |
|------|------|------|
| 18 | [GitHub 功能指南](docs/00-github-features-guide.md) | 全面了解 GitHub 的核心功能模块，建立对平台的整体认知 |
| 19 | [创建仓库](docs/14-create-repo.md) | 在 GitHub 上创建远程仓库，设置可见性、许可证、.gitignore 等选项 |
| 20 | [README 与文档](docs/15-readme-docs.md) | 学会编写专业的 README.md，让项目一目了然，提升项目的专业度和吸引力 |
| 21 | [Issue 问题追踪](docs/16-issues.md) | 使用 Issue 管理任务、报告 Bug、讨论功能需求，这是项目管理的基础工具 |
| 22 | [GitHub Projects](docs/21-github-projects.md) | 使用看板管理项目进度，将 Issue 组织成可视化的项目管理流程 |

### ✅ 完成标准

- [ ] 能在 GitHub 上创建仓库并推送本地代码
- [ ] 能编写结构清晰、内容完整的 README
- [ ] 能创建和管理 Issue，使用标签进行分类
- [ ] 能使用 GitHub Projects 看板管理任务进度

### 🔧 实践项目

完成本级别后，建议你做一个小练习：

1. 在 GitHub 上创建一个个人项目仓库
2. 编写一个专业的 README（包含项目介绍、安装方法、使用示例）
3. 创建 3-5 个 Issue 作为项目的待办事项
4. 用 Projects 看板管理这些 Issue

---

## Level 4：团队协作与开源贡献

**🎯 目标**：掌握现代团队协作的核心工作流，能够参与开源项目贡献

**⏱️ 预计时间**：1-2 周

**📋 前置要求**：完成 Level 3，熟悉 GitHub 基本操作

### 你将学到什么

- Pull Request 的完整工作流程
- Code Review 的方法论和最佳实践
- Fork 工作流和开源贡献的标准流程
- 团队协作中的 Git 工作流策略
- 标签管理与版本发布

### 📖 教程列表

| 序号 | 教程 | 说明 |
|------|------|------|
| 23 | [Pull Request](docs/17-pull-requests.md) | 掌握 PR 的创建、审查、合并全流程，这是 GitHub 协作的核心机制 |
| 24 | [Code Review](docs/18-code-review.md) | 学会进行有效的代码审查，提升代码质量，这也是高级开发者的必备技能 |
| 25 | [Fork 与开源贡献](docs/22-fork-contribute.md) | 掌握 Fork 工作流，学会向开源项目提交贡献的标准流程和礼仪 |
| 26 | [团队协作](docs/00-team-collaboration-guide.md) | 了解团队协作的综合指南，包括权限管理、沟通规范等 |
| 27 | [团队协作实战](docs/23-team-collaboration.md) | 更深入的团队协作实践，包括代码所有权、评审策略等 |
| 28 | [Git 工作流](docs/24-git-workflow.md) | 学习 Git Flow、GitHub Flow、Trunk Based 等主流工作流，选择适合团队的方案 |
| 29 | [标签与发布](docs/25-tags-releases.md) | 使用 Git 标签和 GitHub Releases 管理版本发布，建立规范的版本管理体系 |
| 30 | [高级功能指南](docs/00-advanced-features-guide.md) | 探索 GitHub 的高级协作功能，进一步提升团队效率 |

### ✅ 完成标准

- [ ] 能创建 PR 并与 reviewer 进行有效的代码讨论
- [ ] 能对同事的代码进行有建设性的 Code Review
- [ ] 能 Fork 开源项目并成功提交 PR
- [ ] 理解至少两种 Git 工作流的区别和适用场景
- [ ] 能使用标签和 Releases 管理项目版本

### 🌍 开源贡献指南

参与开源项目是提升技能的最佳方式之一。建议你：

1. 先在 GitHub 上找到感兴趣的项目，阅读其 `CONTRIBUTING.md`
2. 从简单的任务开始（修复文档错误、补充测试用例等）
3. 遵循项目的代码规范和提交规范
4. 耐心等待维护者 review，积极回应反馈

---

## Level 5：自动化与 CI/CD

**🎯 目标**：使用 GitHub Actions 实现自动化构建、测试和部署

**⏱️ 预计时间**：1-2 周

**📋 前置要求**：完成 Level 4，熟悉 PR 和团队协作流程

### 你将学到什么

- GitHub Actions 的核心概念和工作原理
- 编写自动化工作流（构建、测试、部署）
- 使用 GitHub Pages 部署静态网站
- 使用 GitHub CLI 提升命令行效率
- 大文件管理与 Git LFS

### 📖 教程列表

| 序号 | 教程 | 说明 |
|------|------|------|
| 31 | [GitHub Actions](docs/19-github-actions.md) | 从零开始学习 CI/CD 自动化，编写你的第一个 GitHub Actions 工作流 |
| 32 | [GitHub Pages](docs/20-github-pages.md) | 使用 GitHub Pages 免费部署静态网站，搭建个人博客或项目主页 |
| 33 | [GitHub CLI](docs/26-github-cli.md) | 安装和使用 GitHub 命令行工具，在终端中完成几乎所有 GitHub 操作 |
| 34 | [Git LFS](docs/27-git-lfs.md) | 使用 Git Large File Storage 管理大型文件（图片、视频、数据集等） |
| 35 | [安全与 DevOps 指南](docs/00-security-devops-guide.md) | 了解 DevOps 实践中安全自动化的重要性，为后续高级主题打下基础 |

### ✅ 完成标准

- [ ] 能编写一个完整的 GitHub Actions 工作流
- [ ] 能使用 GitHub Pages 部署一个静态网站
- [ ] 能使用 GitHub CLI 在命令行中操作仓库和 PR
- [ ] 了解 Git LFS 的使用场景和基本操作

### 🚀 CI/CD 实践建议

> 自动化是现代软件开发的核心。建议你从一个简单的项目开始：
> 1. 配置一个自动运行测试的 Actions 工作流
> 2. 添加自动部署到 GitHub Pages 的流程
> 3. 尝试添加代码质量检查（lint、格式化等）
> 4. 逐步完善，最终实现完整的 CI/CD 流水线

---

## Level 6：AI、DevOps 与高级主题

**🎯 目标**：掌握前沿工具和高级实践，成为全能型开发者

**⏱️ 预计时间**：持续学习

**📋 前置要求**：完成 Level 5，有扎实的 Git 和 GitHub 基础

### 你将学到什么

- 使用 GitHub Copilot 进行 AI 辅助编程
- 掌握 Kubernetes、Terraform 等 DevOps 工具
- 了解 Git 内部机制和性能优化
- 学习各领域的专业工作流（前端、后端、移动端、机器学习等）

### 📖 AI 辅助编程

| 序号 | 教程 | 说明 |
|------|------|------|
| 36 | [Copilot 完全指南](docs/X1-github-copilot-complete-guide.md) | 全面掌握 GitHub Copilot 的使用方法，让 AI 成为你的编程搭档 |
| 37 | [Copilot AI Agent](docs/X14-github-copilot-workspace-agents.md) | 了解 Copilot Workspace 和 AI Agent，体验下一代开发方式 |

### 📖 DevOps 与基础设施

| 序号 | 教程 | 说明 |
|------|------|------|
| 38 | [Kubernetes 实战](docs/X5-kubernetes-github-actions.md) | 使用 GitHub Actions 部署和管理 Kubernetes 集群，实现容器化应用的自动化运维 |
| 39 | [Terraform IaC](docs/X6-terraform-iac-github.md) | 使用 Terraform 进行基础设施即代码（IaC）管理，自动化云资源的创建和配置 |
| 40 | [安全与权限](docs/28-security-permissions.md) | 深入了解 GitHub 的安全机制，包括权限管理、安全扫描、依赖漏洞检测等 |
| 41 | [API 与 Webhooks](docs/29-api-webhooks.md) | 使用 GitHub API 和 Webhooks 构建自定义集成和自动化工具 |

### 📖 Git 进阶

| 序号 | 教程 | 说明 |
|------|------|------|
| 42 | [Git 内部机制](docs/X11-git-internals-deep-dive.md) | 深入理解 Git 的对象模型、引用机制和内部数据结构，成为 Git 专家 |
| 43 | [Git 性能优化](docs/X12-git-performance-optimization.md) | 优化大型仓库的 Git 性能，包括浅克隆、稀疏检出等高级技巧 |

### 📖 各领域专业工作流

| 序号 | 教程 | 说明 |
|------|------|------|
| 44 | [机器学习工作流](docs/X2-ml-workflow-github.md) | 使用 GitHub 管理机器学习项目，包括数据版本控制、模型管理等 |
| 45 | [前端开发工作流](docs/X3-frontend-github-workflow.md) | 前端项目的 GitHub 最佳实践，包括组件库管理、预览部署等 |
| 46 | [后端 CI/CD 指南](docs/X4-backend-cicd-github.md) | 后端项目的持续集成与持续部署完整方案 |
| 47 | [移动端开发指南](docs/X9-mobile-app-github.md) | iOS 和 Android 项目的 GitHub 工作流和自动化实践 |
| 48 | [微服务管理](docs/X10-microservices-github.md) | 使用 GitHub 管理微服务架构，包括服务发现、配置管理等 |
| 49 | [技术写作指南](docs/X8-technical-writing-github.md) | 使用 GitHub 编写和管理技术文档，打造专业的文档体系 |
| 50 | [开源协议指南](docs/X7-open-source-licenses-guide.md) | 了解各种开源协议的区别和选择方法，避免法律风险 |
| 51 | [项目案例分析](docs/X13-real-world-project-case-studies.md) | 分析真实项目的 GitHub 实践，从成功案例中学习经验 |

### ✅ 完成标准

- [ ] 能使用 Copilot 辅助编写代码并理解其工作原理
- [ ] 了解 Kubernetes 和 Terraform 的基本使用场景
- [ ] 理解 Git 的内部工作机制
- [ ] 至少深入学习一个专业领域的工作流

### 📚 持续学习

> 技术日新月异，学习永无止境。建议你：
> - 关注 GitHub 官方博客和 Changelog，了解最新功能
> - 定期回顾和更新自己的工作流
> - 参与技术社区讨论，分享经验
> - 将学到的知识应用到实际项目中

---

## 🧪 动手练习

理论学习固然重要，但**实践才是掌握技能的关键**。以下是精心设计的 30 个练习，从基础到高级，建议按顺序完成。

### 🟢 入门练习（Level 1-2）

| 编号 | 练习 | 对应知识点 | 难度 |
|------|------|-----------|------|
| 1 | [创建仓库](exercises/exercise-1-create-repo.md) | 创建仓库、初始化 Git、首次提交 | ⭐ |
| 2 | [分支与合并](exercises/exercise-2-branch-merge.md) | 分支操作、合并策略 | ⭐⭐ |
| 3 | [Pull Request](exercises/exercise-3-pull-request.md) | 创建 PR、描述编写、关联 Issue | ⭐⭐ |
| 4 | [解决冲突](exercises/exercise-4-fix-conflict.md) | 识别冲突、手动解决、验证结果 | ⭐⭐ |
| 9 | [Stash 暂存](exercises/exercise-9-stash.md) | git stash 的使用场景和技巧 | ⭐⭐ |
| 10 | [交互式变基](exercises/exercise-10-interactive-rebase.md) | git rebase -i 整理提交历史 | ⭐⭐⭐ |

### 🟡 中级练习（Level 3-4）

| 编号 | 练习 | 对应知识点 | 难度 |
|------|------|-----------|------|
| 5 | [GitHub Pages](exercises/exercise-5-github-pages.md) | 部署静态网站、自定义域名 | ⭐⭐ |
| 6 | [开源贡献](exercises/exercise-6-open-source.md) | Fork 工作流、贡献规范 | ⭐⭐⭐ |
| 8 | [Discussions](exercises/exercise-8-discussions.md) | GitHub Discussions 社区功能 | ⭐⭐ |
| 20 | [项目看板](exercises/exercise-20-project-board.md) | GitHub Projects 看板管理 | ⭐⭐ |
| 24 | [Issue 表单](exercises/exercise-24-issue-forms.md) | Issue 模板和表单 | ⭐⭐ |
| 25 | [子模块](exercises/exercise-25-submodules.md) | Git Submodules 管理 | ⭐⭐⭐ |
| 29 | [Git LFS 工作流](exercises/exercise-29-git-lfs-workflow.md) | 大文件存储管理 | ⭐⭐⭐ |

### 🔴 高级练习（Level 5-6）

| 编号 | 练习 | 对应知识点 | 难度 |
|------|------|-----------|------|
| 7 | [GitHub Actions](exercises/exercise-7-github-actions.md) | 编写 CI/CD 工作流 | ⭐⭐⭐ |
| 11 | [GitHub CLI](exercises/exercise-11-github-cli.md) | 命令行操作 GitHub | ⭐⭐⭐ |
| 12 | [CI/CD 流水线](exercises/exercise-12-cicd-pipeline.md) | 完整的 CI/CD 配置 | ⭐⭐⭐⭐ |
| 13 | [Docker 部署](exercises/exercise-13-docker-deploy.md) | 容器化部署自动化 | ⭐⭐⭐⭐ |
| 14 | [安全扫描](exercises/exercise-14-security-scan.md) | 代码安全扫描配置 | ⭐⭐⭐ |
| 15 | [版本发布管理](exercises/exercise-15-release-management.md) | Releases 和版本策略 | ⭐⭐⭐ |
| 16 | [Monorepo](exercises/exercise-16-monorepo.md) | 单体仓库管理策略 | ⭐⭐⭐⭐ |
| 19 | [可复用工作流](exercises/exercise-19-reusable-workflows.md) | Actions 可复用组件 | ⭐⭐⭐⭐ |
| 21 | [Copilot 基础](exercises/exercise-21-copilot-basics.md) | AI 辅助编程入门 | ⭐⭐⭐ |
| 22 | [Dependabot](exercises/exercise-22-dependabot-setup.md) | 依赖自动更新配置 | ⭐⭐⭐ |
| 23 | [Codespaces](exercises/exercise-23-codespaces-dev.md) | 云端开发环境 | ⭐⭐⭐ |
| 26 | [可复用工作流进阶](exercises/exercise-26-reusable-workflows.md) | 高级 Actions 模式 | ⭐⭐⭐⭐ |
| 27 | [GitHub Models](exercises/exercise-27-github-models.md) | AI 模型集成 | ⭐⭐⭐⭐ |
| 28 | [安全扫描进阶](exercises/exercise-28-security-scanning.md) | 高级安全策略 | ⭐⭐⭐⭐ |
| 30 | [Terraform + GitHub](exercises/exercise-30-terraform-github.md) | 基础设施即代码 | ⭐⭐⭐⭐⭐ |

### 🎯 专项练习

| 编号 | 练习 | 说明 |
|------|------|------|
| 17 | [国内环境配置](exercises/exercise-17-china-setup.md) | 配置国内镜像源，优化 GitHub 访问速度 |
| 18 | [企业级配置](exercises/exercise-18-enterprise-setup.md) | 企业级 GitHub 使用场景和配置方案 |

### 📝 练习建议

> 1. **先看再做**：每个练习都有详细的步骤说明，建议先通读一遍再动手
> 2. **不要跳级**：基础练习是高级练习的铺垫，扎实的基础会让你后续学习更轻松
> 3. **记录笔记**：遇到问题时记录下来，解决后写下心得，这会成为你宝贵的财富
> 4. **反复练习**：一次不会很正常，多练几次就熟了

---

## 📖 速查与参考

遇到问题时，这些资料会帮到你：

### 🔍 日常参考

| 文档 | 说明 | 使用场景 |
|------|------|---------|
| [常用命令速查](docs/A-common-commands.md) | Git 和 GitHub 常用命令的速查手册 | 日常开发中快速查找命令用法 |
| [术语表](docs/E-glossary.md) | Git 和 GitHub 专业术语的中英文对照 | 阅读英文文档时查阅专业术语 |
| [常见问题](docs/C-faq.md) | 新手最常遇到的问题及其解答 | 遇到问题时先来这里找答案 |
| [错误排查](docs/M-troubleshooting.md) | 常见错误信息和解决方案 | 遇到报错时快速定位和修复 |

### 🇨🇳 国内用户参考

| 文档 | 说明 | 使用场景 |
|------|------|---------|
| [国内加速指南](docs/Q-china-acceleration.md) | 配置镜像和代理加速访问 GitHub | 网络访问缓慢时的优化方案 |
| [国内直连指南](docs/Q2-china-direct-access.md) | 多种直连方案解决网络问题 | 需要稳定访问 GitHub 时参考 |

---

## ⏭️ 学完之后？

恭喜你完成了所有学习路线！🎉 但学习之路才刚刚开始：

### 🌟 立即行动

1. **⭐ Star 本仓库** — 如果这个学习路线对你有帮助，请给一个 Star 支持一下
2. **🍴 Fork 本仓库** — 添加你自己的学习笔记和心得
3. **📢 分享给朋友** — 帮助更多中文开发者学习 GitHub

### 🚀 进阶方向

- **参与开源项目** — 在 GitHub 上找到感兴趣的项目，提交你的第一个 PR
- **建立个人品牌** — 用 GitHub Pages 搭建个人博客，分享技术文章
- **持续学习** — 关注 GitHub 官方博客，了解最新功能和最佳实践
- **帮助他人** — 在 GitHub Discussions 或社区中回答新手的问题

### 💬 社区参与

> 开源精神的核心是**分享与协作**。当你掌握了这些技能后，不要忘记回馈社区：
> - 为使用的开源项目提交 Bug 报告或功能建议
> - 参与文档翻译和改进
> - 分享你的使用经验和最佳实践
> - 帮助其他新手解决问题

---

## 📌 学习路线速览表

| 级别 | 主题 | 预计时间 | 核心技能 |
|------|------|---------|---------|
| Level 1 | 零基础入门 | 1-2 天 | 版本控制概念、Git 安装、GitHub 注册 |
| Level 2 | Git 基础操作 | 3-5 天 | add/commit/branch/merge/rebase |
| Level 3 | GitHub 入门 | 1 周 | 远程仓库、README、Issue、Projects |
| Level 4 | 团队协作 | 1-2 周 | PR、Code Review、Fork、工作流 |
| Level 5 | 自动化 CI/CD | 1-2 周 | GitHub Actions、Pages、CLI |
| Level 6 | 高级主题 | 持续学习 | Copilot、K8s、Terraform、Git 内部机制 |

---

> 📖 **记住**：学习不是一场赛跑，而是一段旅程。按照自己的节奏，享受学习的过程。每一个专家都曾经是新手。加油！💪
