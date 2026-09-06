# 中文 Git/GitHub 术语对照表

## 基础概念

| 英文 | 中文 | 说明 |
|------|------|------|
| Repository | 仓库 | 项目的存储空间 |
| Clone | 克隆 | 复制远程仓库到本地 |
| Commit | 提交 | 保存更改到本地仓库 |
| Branch | 分支 | 独立的开发线路 |
| Merge | 合并 | 将两个分支的修改合并 |
| Stage | 暂存 | 将修改标记为准备提交 |
| Remote | 远程 | 托管在服务器上的仓库 |
| Working Directory | 工作区 | 正在编辑文件的目录 |
| Index | 索引 | 暂存区的另一个名称 |

## Git 操作

| 英文 | 中文 | 说明 |
|------|------|------|
| Rebase | 变基 | 将当前分支的提交重放到目标分支 |
| Reset | 回退 | 撤销提交或暂存 |
| Revert | 反转 | 创建新提交来撤销之前的修改 |
| Checkout | 检出 | 切换分支或恢复文件 |
| Stash | 暂存 | 临时保存未提交的修改 |
| Cherry-pick | 挑选 | 从其他分支选取特定提交 |
| Squash | 压缩 | 将多个提交合并为一个 |
| Amend | 修改 | 修改最近一次提交 |
| Diff | 差异 | 查看代码变更 |
| Log | 日志 | 查看提交历史 |
| Tag | 标签 | 标记特定提交 |

## GitHub 功能

| 英文 | 中文 | 说明 |
|------|------|------|
| Pull Request (PR) | 拉取请求 | 请求合并代码的审核流程 |
| Issue | 问题 | 追踪 bug、功能请求等 |
| Fork | 分叉 | 复制别人的仓库到自己的账户 |
| Code Review | 代码审查 | 审查代码质量和安全性 |
| Actions | 动作 | GitHub 的 CI/CD 自动化平台 |
| Pages | 页面 | 静态网站托管服务 |
| Projects | 项目 | 看板式项目管理工具 |
| Discussions | 讨论 | 社区讨论区 |
| Packages | 包 | 包管理服务 |
| Release | 发布 | 软件版本发布 |
| Marketplace | 市场 | 工具和服务市场 |
| Sponsors | 赞助 | 开发者赞助平台 |
| Copilot | 副驾驶 | AI 编程助手 |

## 安全相关

| 英文 | 中文 | 说明 |
|------|------|------|
| SSH | 安全外壳 | 安全的远程连接协议 |
| Key | 密钥 | 用于身份验证的加密凭证 |
| Token | 令牌 | 用于 API 认证的字符串 |
| 2FA | 两步验证 | 两因素身份验证 |
| Dependencies | 依赖项 | 项目依赖的其他软件包 |
| Vulnerability | 漏洞 | 安全漏洞 |
| CODEOWNERS | 代码所有者 | 定义代码审查责任人的文件 |

## 工作流术语

| 英文 | 中文 | 说明 |
|------|------|------|
| Trunk-Based Development | 主干开发 | 所有人直接向主分支提交 |
| Feature Flags | 功能开关 | 运行时控制功能启用/禁用 |
| CI | 持续集成 | 自动化构建和测试 |
| CD | 持续部署 | 自动化部署到生产环境 |
| Pipeline | 管道 | CI/CD 的工作流程 |
| Workflow | 工作流 | GitHub Actions 的自动化配置 |
| Runner | 运行器 | 执行 GitHub Actions 的服务器 |

## 版本控制

| 英文 | 中文 | 说明 |
|------|------|------|
| Commit Message | 提交信息 | 描述提交内容的文字 |
| Hash | 哈希 | 提交的唯一标识符 |
| HEAD | 头指针 | 指向当前分支最新提交的指针 |
| History | 历史 | 提交的有序列表 |
| Conflict | 冲突 | 合并时无法自动解决的差异 |
| Base | 基准 | 比较或合并的基础分支 |
| Upstream | 上游 | 原始仓库（相对于 Fork） |
| Downstream | 下游 | Fork 出的仓库 |

## 常用缩写

| 英文 | 中文 | 说明 |
|------|------|------|
| PR | 拉取请求 | Pull Request |
| MR | 合并请求 | Merge Request（GitLab 用法） |
| CI | 持续集成 | Continuous Integration |
| CD | 持续部署 | Continuous Deployment |
| PT | PR 模板 | Pull Request Template |
| LGTM | 看起来不错 | Looks Good To Me |
| WIP | 进行中 | Work In Progress |
| TBD | 待定 | To Be Determined |
| N/A | 不适用 | Not Applicable |

## Git 命令对照

| 英文命令 | 中文说明 | 示例 |
|----------|----------|------|
| init | 初始化 | `git init` |
| clone | 克隆 | `git clone <url>` |
| add | 添加 | `git add <file>` |
| commit | 提交 | `git commit -m "msg"` |
| push | 推送 | `git push origin main` |
| pull | 拉取 | `git pull origin main` |
| fetch | 获取 | `git fetch origin` |
| branch | 分支 | `git branch <name>` |
| checkout | 检出 | `git checkout <branch>` |
| merge | 合并 | `git merge <branch>` |
| rebase | 变基 | `git rebase <branch>` |
| stash | 暂存 | `git stash` |
| log | 日志 | `git log --oneline` |
| diff | 差异 | `git diff` |
| status | 状态 | `git status` |
| remote | 远程 | `git remote -v` |
| tag | 标签 | `git tag <name>` |

## GitHub 操作对照

| 英文 | 中文 | 说明 |
|------|------|------|
| Create Repository | 创建仓库 | 创建新的代码仓库 |
| Fork | 分叉 | 复制仓库到自己的账户 |
| Star | 收藏 | 收藏感兴趣的项目 |
| Watch | 关注 | 关注项目动态 |
| Clone | 克隆 | 复制仓库到本地 |
| Pull Request | 拉取请求 | 请求合并代码 |
| Issue | 问题 | 报告问题或请求功能 |
| Release | 发布 | 发布软件版本 |
| Action | 动作 | 自动化工作流 |
| Page | 页面 | 静态网站托管 |
| Project | 项目 | 项目管理看板 |
| Discussion | 讨论 | 社区讨论区 |

## 常见状态对照

| 英文 | 中文 | 说明 |
|------|------|------|
| Open | 打开 | Issue/PR 状态 |
| Closed | 已关闭 | Issue/PR 状态 |
| Merged | 已合并 | PR 状态 |
| Opened | 已打开 | PR 状态 |
| Approved | 已批准 | 审查状态 |
| Changes Requested | 请求修改 | 审查状态 |
| Commented | 已评论 | 审查状态 |

## 提交类型对照

| 英文 | 中文 | 说明 |
|------|------|------|
| feat | 新功能 | Feature |
| fix | 修复 | Bug Fix |
| docs | 文档 | Documentation |
| style | 格式 | Style |
| refactor | 重构 | Refactor |
| test | 测试 | Test |
| chore | 杂务 | Chore |
| perf | 性能 | Performance |
| ci | CI | Continuous Integration |
| build | 构建 | Build |
| revert | 回退 | Revert |
