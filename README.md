# GitHub 新手完全指南

> 从零开始，全面掌握 GitHub 的使用方法。无论你是程序员、设计师还是任何需要协作的人，这份指南都能帮助你快速上手 GitHub。

---

## 目录

### 第一部分：基础准备
- [什么是 GitHub？](docs/01-what-is-github.md)
- [安装与配置 Git](docs/02-install-git.md)
- [注册 GitHub 账号](docs/03-signup-github.md)
- [配置 SSH 密钥](docs/04-ssh-keys.md)
- [Git 基本配置](docs/05-git-config.md)

### 第二部分：Git 核心命令
- [Git 工作原理](docs/06-how-git-works.md)
- [创建与克隆仓库](docs/07-init-clone.md)
- [暂存与提交](docs/08-add-commit.md)
- [查看历史与差异](docs/09-log-diff.md)
- [分支操作](docs/10-branching.md)
- [合并与变基](docs/11-merge-rebase.md)
- [解决冲突](docs/12-resolve-conflicts.md)
- [撤销操作](docs/13-undo.md)

### 第三部分：GitHub 核心功能
- [创建和管理仓库](docs/14-create-repo.md)
- [README 与文档](docs/15-readme-docs.md)
- [Issue 问题追踪](docs/16-issues.md)
- [Pull Request 协作](docs/17-pull-requests.md)
- [Code Review 代码审查](docs/18-code-review.md)
- [GitHub Actions 自动化](docs/19-github-actions.md)
- [GitHub Pages 静态网站](docs/20-github-pages.md)
- [GitHub Projects 项目管理](docs/21-github-projects.md)

### 第四部分：协作与进阶
- [Fork 与开源贡献](docs/22-fork-contribute.md)
- [团队协作最佳实践](docs/23-team-collaboration.md)
- [Git 工作流](docs/24-git-workflow.md)
- [标签与发布](docs/25-tags-releases.md)
- [GitHub CLI 命令行工具](docs/26-github-cli.md)
- [Git LFS 大文件管理](docs/27-git-lfs.md)
- [安全与权限管理](docs/28-security-permissions.md)
- [GitHub API 与 Webhooks](docs/29-api-webhooks.md)

### 第五部分：GitHub 生态系统
- [GitHub Copilot 介绍](docs/H-github-copilot.md)
- [GitHub Discussions 介绍](docs/I-github-discussions.md)
- [GitHub Packages 介绍](docs/J-github-packages.md)
- [GitHub Mobile 介绍](docs/K-github-mobile.md)
- [GitHub Sponsors 介绍](docs/L-github-sponsors.md)
- [GitHub Marketplace](docs/N-marketplace.md)

### 第六部分：安全与最佳实践
- [GitHub 安全最佳实践](docs/O-security-best-practices.md)
- [GitHub API 使用指南](docs/P-github-api.md)

### 第七部分：实战练习
- [练习 1：创建你的第一个仓库](exercises/exercise-1-create-repo.md)
- [练习 2：分支与合并练习](exercises/exercise-2-branch-merge.md)
- [练习 3：Pull Request 工作流](exercises/exercise-3-pull-request.md)
- [练习 4：修复 Merge Conflict](exercises/exercise-4-fix-conflict.md)
- [练习 5：使用 GitHub Pages 部署网站](exercises/exercise-5-github-pages.md)
- [练习 6：参与开源项目](exercises/exercise-6-open-source.md)
- [练习 7：使用 GitHub Actions](exercises/exercise-7-github-actions.md)
- [练习 8：设置 GitHub Discussions](exercises/exercise-8-discussions.md)

### 附录
- [常用命令速查表](docs/A-common-commands.md)
- [Git 别名配置](docs/B-git-aliases.md)
- [常见问题解答](docs/C-faq.md)
- [推荐学习资源](docs/D-resources.md)
- [Git 与 GitHub 术语表](docs/E-glossary.md)
- [GitHub 快捷键指南](docs/F-shortcuts.md)
- [Git 可视化指南（图解）](docs/G-visual-guide.md)
- [常见错误排查指南](docs/M-troubleshooting.md)

---

## 快速开始

### 新手入门

如果你是完全的新手，建议按顺序阅读第一部分和第二部分：

1. 了解 [什么是 GitHub](docs/01-what-is-github.md)
2. [安装与配置 Git](docs/02-install-git.md)
3. [注册 GitHub 账号](docs/03-signup-github.md)
4. [配置 SSH 密钥](docs/04-ssh-keys.md)
5. 开始 [实战练习](exercises/exercise-1-create-repo.md)

### 有基础的开发者

如果你已经了解 Git 基础，可以直接跳到：
- [GitHub 核心功能](docs/14-create-repo.md)（第三部分）
- [GitHub 生态系统](docs/H-github-copilot.md)（第五部分）
- [安全与最佳实践](docs/O-security-best-practices.md)（第六部分）

### 快速克隆本仓库

```bash
# 克隆本仓库到本地
git clone https://github.com/wangzi5151/github-beginners-guide.git

# 进入仓库目录
cd github-beginners-guide
```

---

## 内容概览

| 部分 | 内容 | 适合人群 |
|------|------|----------|
| 第一部分 | 基础准备 | 完全新手 |
| 第二部分 | Git 核心命令 | 所有用户 |
| 第三部分 | GitHub 核心功能 | 所有用户 |
| 第四部分 | 协作与进阶 | 进阶用户 |
| 第五部分 | GitHub 生态系统 | 所有用户 |
| 第六部分 | 安全与最佳实践 | 团队负责人 |
| 第七部分 | 实战练习 | 所有用户 |
| 附录 | 速查表和参考 | 所有用户 |

---

## 贡献

欢迎贡献！请查看 [贡献指南](CONTRIBUTING.md) 了解如何参与。

## 许可证

本项目采用 [MIT 许可证](LICENSE)，欢迎自由使用和分享。

## 反馈

如果你有任何问题或建议，请在 [Issues](https://github.com/wangzi5151/github-beginners-guide/issues) 中提出。
