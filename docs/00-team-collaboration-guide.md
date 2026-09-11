# 第六章：团队协作与开源贡献

## 6.1 团队协作基础

### 团队协作流程

```
1. 克隆仓库
   ↓
2. 创建功能分支
   ↓
3. 开发和测试
   ↓
4. 推送分支
   ↓
5. 创建 Pull Request
   ↓
6. 代码审查
   ↓
7. 合并到主分支
   ↓
8. 部署
```

### 分支策略

**Git Flow：**
```
main (生产分支)
├── develop (开发分支)
│   ├── feature/xxx (功能分支)
│   └── release/xxx (发布分支)
└── hotfix/xxx (紧急修复)
```

**GitHub Flow：**
```
main (主分支)
└── feature/xxx (功能分支)
```

**Trunk-Based Development：**
```
main (主分支，频繁提交)
```

### 提交信息规范

**格式：**
```
类型(范围): 描述

详细描述（可选）

关联 Issue（可选）
```

**类型说明：**

| 类型 | 说明 | 示例 |
|------|------|------|
| `feat` | 新功能 | `feat: 添加用户登录功能` |
| `fix` | Bug 修复 | `fix: 修复登录页面样式问题` |
| `docs` | 文档更新 | `docs: 更新 README 安装说明` |
| `style` | 代码格式 | `style: 格式化代码` |
| `refactor` | 重构 | `refactor: 重构用户服务` |
| `test` | 测试 | `test: 添加单元测试` |
| `chore` | 构建/工具 | `chore: 更新依赖` |
| `perf` | 性能优化 | `perf: 优化查询性能` |

**示例：**
```bash
git commit -m "feat: 添加用户登录功能"
git commit -m "fix: 修复登录页面样式问题"
git commit -m "docs: 更新 README 安装说明"
```

## 6.2 代码审查

### 审查流程

**第 1 步：** 打开 PR 的 **Files changed** 页面

**第 2 步：** 查看代码差异

**第 3 步：** 添加行内评论

**第 4 步：** 提交审查

### 审查要点

| 类别 | 检查项 |
|------|--------|
| **功能** | 逻辑是否正确，边界情况 |
| **设计** | 架构是否合理 |
| **可读性** | 命名、注释、结构 |
| **性能** | 有无性能问题 |
| **安全** | 有无安全漏洞 |
| **测试** | 测试是否充分 |
| **文档** | 是否需要更新文档 |

### 提供好的反馈

**好的反馈：**
- 具体指出问题位置
- 解释为什么有问题
- 提供改进建议
- 使用代码建议

**示例：**
```
这里的类名 "new" 不够描述性，建议改为 "primary"
以表示这是一个主要按钮。

```suggestion
className="primary"
```
```

### 审查结果

| 选项 | 说明 | 何时使用 |
|------|------|----------|
| **Comment** | 仅评论 | 不反对合并，只是提建议 |
| **Approve** | 批准 | 代码质量好，可以合并 |
| **Request changes** | 要求修改 | 有问题需要修复 |

## 6.3 Fork 与开源贡献

### 什么是 Fork？

Fork 是将别人的仓库复制到你的 GitHub 账号下，这样你可以自由修改而不影响原仓库。

### Fork 工作流程

```
1. Fork 仓库（网页操作）
   ↓
2. 克隆 Fork 的仓库（命令行）
   ↓
3. 添加上游仓库（命令行）
   ↓
4. 创建功能分支（命令行）
   ↓
5. 开发和提交（命令行）
   ↓
6. 推送到 Fork 的仓库（命令行）
   ↓
7. 创建 Pull Request（网页操作）
```

### Fork 仓库

**第 1 步：** 打开源仓库页面

**第 2 步：** 点击右上角的 **Fork** 按钮

```
┌─────────────────────────────────────────────┐
│  owner/repo                                  │
│                                             │
│  [Fork]  ← 点击这个按钮                      │
│  ⭐ 1.2k  👁 345                             │
└─────────────────────────────────────────────┘
```

**第 3 步：** 选择 Fork 的目标（通常是你的账号）

**第 4 点：** 等待 Fork 完成

### 克隆 Fork 的仓库

```bash
# 克隆你 Fork 的仓库
git clone https://github.com/your-username/repo.git

# 进入仓库目录
cd repo
```

### 添加上游仓库

```bash
# 添加原仓库为上游
git remote add upstream https://github.com/owner/repo.git

# 查看远程仓库
git remote -v
```

**输出示例：**
```
origin    https://github.com/your-username/repo.git (fetch)
origin    https://github.com/your-username/repo.git (push)
upstream  https://github.com/owner/repo.git (fetch)
upstream  https://github.com/owner/repo.git (push)
```

### 同步 Fork

```bash
# 获取上游更新
git fetch upstream

# 合并上游 main 分支
git checkout main
git merge upstream/main

# 推送到你的 Fork
git push origin main
```

### 创建功能分支

```bash
# 创建功能分支
git checkout -b feature-your-feature
```

### 开发和提交

```bash
# 修改文件...
git add .
git commit -m "feat: 添加新功能"
```

### 推送到 Fork

```bash
git push origin feature-your-feature
```

### 创建 Pull Request

**第 1 步：** 打开你的 Fork 页面

**第 2 步：** 点击 **Compare & pull request**

**第 3 步：** 确认 PR 信息

```
┌─────────────────────────────────────────────┐
│  Open a pull request                         │
│                                             │
│  base: owner:main  ← compare: your:feature  │
│                                             │
│  Title: [feat: 添加新功能           ]         │
│                                             │
│  Description:                                │
│  ┌─────────────────────────────────────┐    │
│  │ ## 变更说明                         │    │
│  │ 添加了一个新功能                     │    │
│  │                                     │    │
│  │ ## 测试                             │    │
│  │ - [x] 已通过所有测试                │    │
│  └─────────────────────────────────────┘    │
│                                             │
│        [Create pull request]                │
└─────────────────────────────────────────────┘
```

**第 4 步：** 点击 **Create pull request**

### 保持 Fork 同步

```bash
# 获取上游更新
git fetch upstream

# 切换到 main 分支
git checkout main

# 合并上游更新
git merge upstream/main

# 推送到你的 Fork
git push origin main
```

## 6.4 开源项目运营

### 创建开源项目

**准备工作：**
- [ ] 编写 README
- [ ] 添加 LICENSE
- [ ] 创建 CONTRIBUTING.md
- [ ] 创建 CODE_OF_CONDUCT.md
- [ ] 设置 Issue 模板
- [ ] 设置 PR 模板
- [ ] 配置 CI/CD

### README 模板

```markdown
# 项目名称

> 简短描述

## 特性

- 特性 1
- 特性 2
- 特性 3

## 快速开始

### 安装

```bash
npm install your-package
```

### 使用

```javascript
import { yourFunction } from 'your-package';

yourFunction();
```

## 文档

- [文档链接](docs/)

## 贡献指南

欢迎贡献！请查看 [CONTRIBUTING.md](CONTRIBUTING.md)。

## 许可证

[MIT](LICENSE)
```

### CONTRIBUTING.md 模板

```markdown
# 贡献指南

感谢你对项目的关注！

## 如何贡献

### 报告 Bug

1. 搜索现有的 Issues
2. 创建新的 Issue
3. 使用 Bug Report 模板

### 提交代码

1. Fork 仓库
2. 创建功能分支
3. 提交更改
4. 推送到 Fork
5. 创建 Pull Request

### 代码规范

- 遵循现有代码风格
- 添加必要的注释
- 确保测试通过

### 提交信息规范

使用 Conventional Commits 规范：

```
类型(范围): 描述

详细描述（可选）
```
```

### 社区建设

**渠道：**
- GitHub Issues：问题追踪
- GitHub Discussions：社区讨论
- Discord/Slack：即时通讯
- Twitter：项目动态

**活动：**
- 定期发布更新
- 回复 Issue 和 PR
- 组织线上活动
- 撰写博客文章

### 项目推广

**方法：**
- 在社交媒体分享
- 写博客介绍
- 参与开源活动
- 与其他项目合作
- 提交到 awesome 列表

## 6.5 开源许可证

### 常用许可证

| 许可证 | 说明 | 限制 |
|--------|------|------|
| **MIT** | 最宽松，允许任何使用 | 需要保留版权声明 |
| **Apache 2.0** | 宽松，需要注明修改 | 需要保留版权声明 |
| **GPL** | 需要开源衍生作品 | 衍生作品必须开源 |
| **LGPL** | 允许库被闭源使用 | 库本身需要开源 |
| **BSD** | 类似 MIT，有附加限制 | 需要保留版权声明 |

### 选择许可证

**推荐：**
- 个人项目：MIT
- 企业项目：Apache 2.0
- 希望衍生作品开源：GPL

### 添加许可证

**方法一：网页添加**

1. 进入仓库页面
2. 点击 **Add file** → **Create new file**
3. 输入文件名 `LICENSE`
4. 选择模板
5. 点击 **Commit changes**

**方法二：命令行添加**

```bash
# 使用 GitHub CLI
gh api repos/{owner}/{repo}/license \
  --method PUT \
  -f license='mit'
```

## 6.6 最佳实践

### 团队协作最佳实践

1. **使用清晰的分支命名**
   - `feature/user-login`
   - `bugfix/fix-crash`
   - `hotfix/security-patch`

2. **编写好的提交信息**
   - 使用 Conventional Commits 规范
   - 描述清晰，说明做了什么

3. **保持 PR 小而专注**
   - 一个 PR 只做一件事
   - 便于审查和理解

4. **及时响应审查**
   - 不要让 PR 放太久
   - 认真考虑每条反馈

### 开源贡献最佳实践

1. **从简单任务开始**
   - 查找 `good first issue` 标签
   - 修复文档错误

2. **阅读贡献指南**
   - 了解项目规范
   - 遵循代码风格

3. **保持沟通**
   - 在 Issue 中讨论想法
   - 及时回复反馈

4. **保持耐心**
   - 审查需要时间
   - 被拒绝不要气馁

## 6.7 本章小结

本章详细介绍了团队协作和开源贡献的内容，包括：

- 团队协作流程
- 代码审查方法
- Fork 与开源贡献
- 开源项目运营
- 开源许可证

**关键要点：**
- 良好的协作习惯是团队成功的基础
- 参与开源可以提升技术能力
- 尊重他人，保持友好

**下一步：**
[安全与 DevOps →](28-security-permissions.md)
