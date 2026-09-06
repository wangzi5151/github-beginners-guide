# Issue 问题追踪

## 什么是 Issue？

Issue 是 GitHub 的问题追踪系统，用于：
- 报告 bug
- 提出功能请求
- 讨论问题
- 跟踪任务

## 创建 Issue

### 网页创建
1. 进入仓库 → **Issues** 标签
2. 点击 **New issue**
3. 填写标题和描述
4. 点击 **Submit new issue**

### 使用模板
很多仓库提供 Issue 模板：
- Bug Report
- Feature Request
- Question

### 命令行创建

```bash
# 使用 GitHub CLI
gh issue create --title "Bug: 首页加载失败" --body "描述问题..."
```

## Issue 标签 (Labels)

默认标签：
- `bug`：bug 报告
- `enhancement`：功能增强
- `documentation`：文档
- `good first issue`：适合新手
- `help wanted`：需要帮助
- `question`：问题

自定义标签：
1. 进入 **Issues** → **Labels**
2. 点击 **New label**
3. 设置名称、颜色和描述

## 指派 (Assignees)

将 Issue 分配给特定人员：
1. 编辑 Issue
2. 在右侧 **Assignees** 选择人员

## 里程碑 (Milestones)

将 Issue 归入版本计划：
1. 创建 Milestone
2. 将 Issue 关联到 Milestone

## 关闭 Issue

### 自动关闭
在提交信息或 PR 描述中使用关键词：

```bash
# 关闭单个 Issue
git commit -m "fix: 修复登录问题，closes #42"

# 关闭多个 Issue
git commit -m "fix: 修复多个问题，fixes #42, fixes #43"
```

### 关键词
- `closes #42`
- `fixes #42`
- `resolves #42`

### 手动关闭
在 Issue 页面点击 **Close issue**

## Issue 最佳实践

1. **使用清晰的标题**：简明描述问题
2. **提供复现步骤**：让别人能重现问题
3. **附加信息**：截图、日志、环境信息
4. **使用标签**：分类管理
5. **及时回复**：回应评论和问题

## Issue 模板

创建 `.github/ISSUE_TEMPLATE/bug_report.md`：

```markdown
---
name: Bug Report
about: 报告一个 bug
labels: bug
---

## 描述
简要描述问题

## 复现步骤
1. 访问 '...'
2. 点击 '...'
3. 看到错误

## 期望行为
描述期望的行为

## 实际行为
描述实际的行为

## 环境
- OS: 
- Browser: 
- Version: 
```

## 下一步

[Pull Request 协作 →](17-pull-requests.md)
