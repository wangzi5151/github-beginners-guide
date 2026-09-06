# Git 与 GitHub 术语表

## 基础概念

| 术语 | 英文 | 说明 |
|------|------|------|
| 仓库 | Repository | 项目的存储空间，包含所有文件和历史记录 |
| 克隆 | Clone | 将远程仓库复制到本地 |
| 提交 | Commit | 保存更改到本地仓库 |
| 分支 | Branch | 独立的开发线路 |
| 合并 | Merge | 将两个分支的修改合并 |
| 暂存 | Stage | 将修改标记为准备提交 |
| 远程 | Remote | 托管在服务器上的仓库副本 |
| 工作区 | Working Directory | 你正在编辑文件的目录 |
| 索引 | Index | 暂存区的另一个名称 |

## Git 操作

| 术语 | 英文 | 说明 |
|------|------|------|
| 变基 | Rebase | 将当前分支的提交重放到目标分支上 |
| 回退 | Reset | 撤销提交或暂存 |
| 反转 | Revert | 创建新提交来撤销之前的修改 |
| 检出 | Checkout | 切换分支或恢复文件 |
| 暂存 | Stash | 临时保存未提交的修改 |
| 挑选 | Cherry-pick | 从其他分支选取特定提交 |
| 压缩 | Squash | 将多个提交合并为一个 |

## GitHub 功能

| 术语 | 英文 | 说明 |
|------|------|------|
| 拉取请求 | Pull Request (PR) | 请求合并代码的审核流程 |
| 问题 | Issue | 用于追踪 bug、功能请求等 |
| 分叉 | Fork | 复制别人的仓库到自己的账户 |
| 代码审查 | Code Review | 审查代码质量和安全性 |
| 动作 | Actions | GitHub 的 CI/CD 自动化平台 |
| 页面 | Pages | 静态网站托管服务 |
| 项目 | Projects | 看板式项目管理工具 |
| 讨论 | Discussions | 社区讨论区 |
| 包 | Packages | 包管理服务 |
| 发布 | Release | 软件版本发布 |
| 标签 | Tag | 标记特定提交（通常用于版本） |

## 安全相关

| 术语 | 英文 | 说明 |
|------|------|------|
| SSH | Secure Shell | 安全的远程连接协议 |
| 密钥 | Key | 用于身份验证的加密凭证 |
| 令牌 | Token | 用于 API 认证的字符串 |
| 二步验证 | 2FA | 两因素身份验证 |
| 依赖项 | Dependencies | 项目依赖的其他软件包 |
| 漏洞 | Vulnerability | 安全漏洞 |
| 代码所有者 | CODEOWNERS | 定义代码审查责任人的文件 |

## 工作流术语

| 术语 | 英文 | 说明 |
|------|------|------|
| 主干开发 | Trunk-Based Development | 所有人直接向主分支提交 |
| 功能开关 | Feature Flags | 运行时控制功能启用/禁用 |
| 持续集成 | CI (Continuous Integration) | 自动化构建和测试 |
| 持续部署 | CD (Continuous Deployment) | 自动化部署到生产环境 |
| 管道 | Pipeline | CI/CD 的工作流程 |
| 工作流 | Workflow | GitHub Actions 的自动化配置 |
| 运行器 | Runner | 执行 GitHub Actions 的服务器 |

## 版本控制

| 术语 | 英文 | 说明 |
|------|------|------|
| 提交信息 | Commit Message | 描述提交内容的文字 |
| 哈希 | Hash | 提交的唯一标识符（SHA-1） |
| HEAD | HEAD | 指向当前分支最新提交的指针 |
| 历史 | History | 提交的有序列表 |
| 冲突 | Conflict | 合并时无法自动解决的差异 |
| 基准 | Base | 比较或合并的基础分支 |
| 上游 | Upstream | 原始仓库（相对于 Fork） |
| 下游 | Downstream | Fork 出的仓库 |

## 常用缩写

| 缩写 | 全称 | 说明 |
|------|------|------|
| PR | Pull Request | 拉取请求 |
| MR | Merge Request | 合并请求（GitLab 用法） |
| CI | Continuous Integration | 持续集成 |
| CD | Continuous Deployment | 持续部署 |
| PT | Pull Request Template | PR 模板 |
| LGTM | Looks Good To Me | 审查通过 |
| WIP | Work In Progress | 进行中 |
| TBD | To Be Determined | 待定 |
| N/A | Not Applicable | 不适用 |
