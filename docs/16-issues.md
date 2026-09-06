# Issue 问题追踪

## 什么是 Issue？

Issue 是 GitHub 的问题追踪系统，用于：
- 报告 bug
- 提出功能请求
- 讨论问题
- 跟踪任务

## 创建 Issue（图文详解）

### 第 1 步：进入 Issues 页面

1. 打开仓库页面
2. 点击 **Issues** 标签
3. 点击绿色的 **New issue** 按钮

```
┌─────────────────────────────────────────────┐
│  Issues                                      │
│                                             │
│  [New issue]  ← 点击这个按钮                 │
│                                             │
│  Filters: [Open ▼] [Labels ▼] [Assignee ▼] │
│                                             │
│  No issues found                             │
└─────────────────────────────────────────────┘
```

### 第 2 步：填写 Issue 信息

```
┌─────────────────────────────────────────────┐
│  New issue                                   │
│                                             │
│  Title: [Bug: 首页加载失败              ]     │
│                                             │
│  Leave a comment:                            │
│  ┌─────────────────────────────────────┐    │
│  │ ## 问题描述                         │    │
│  │ 首页在某些情况下无法正常加载           │    │
│  │                                     │    │
│  │ ## 复现步骤                         │    │
│  │ 1. 打开首页                         │    │
│  │ 2. 点击登录按钮                      │    │
│  │ 3. 页面显示空白                      │    │
│  │                                     │    │
│  │ ## 期望行为                         │    │
│  │ 应该显示登录表单                     │    │
│  │                                     │    │
│  │ ## 环境信息                         │    │
│  │ - OS: Windows 11                    │    │
│  │ - Browser: Chrome 120               │    │
│  └─────────────────────────────────────┘    │
│                                             │
│  ☑ Assignees: [选择负责人]                  │
│  ☑ Labels: [bug] [help wanted]              │
│  ☑ Milestone: [v1.0]                       │
│                                             │
│        [Submit new issue]                   │
└─────────────────────────────────────────────┘
```

**填写说明：**

| 字段 | 说明 | 建议 |
|------|------|------|
| **Title** | Issue 标题 | 简洁描述问题 |
| **Comment** | 详细描述 | 提供复现步骤 |
| **Assignees** | 负责人 | 选择处理此 Issue 的人 |
| **Labels** | 标签 | 分类管理 |
| **Milestone** | 里程碑 | 关联版本计划 |

### 第 3 步：提交 Issue

1. 填写完成后
2. 点击绿色的 **Submit new issue** 按钮
3. Issue 创建成功

## 使用 Issue 模板

很多仓库提供 Issue 模板，可以更快地创建标准 Issue：

```
┌─────────────────────────────────────────────┐
│  Choose a template                           │
│                                             │
│  ┌─────────────┐  ┌─────────────┐          │
│  │ 🐛 Bug     │  │ ✨ Feature  │          │
│  │ Report      │  │ Request     │          │
│  │             │  │             │          │
│  │ [Use        │  │ [Use        │          │
│  │  template]  │  │  template]  │          │
│  └─────────────┘  └─────────────┘          │
│                                             │
│  ┌─────────────┐  ┌─────────────┐          │
│  │ ❓ Question │  │ 📝 Docs     │          │
│  │             │  │             │          │
│  │ [Use        │  │ [Use        │          │
│  │  template]  │  │  template]  │          │
│  └─────────────┘  └─────────────┘          │
└─────────────────────────────────────────────┘
```

## Issue 标签 (Labels)

### 默认标签

| 标签 | 颜色 | 说明 |
|------|------|------|
| `bug` | 红色 | Bug 报告 |
| `enhancement` | 蓝色 | 功能增强 |
| `documentation` | 浅蓝 | 文档相关 |
| `good first issue` | 绿色 | 适合新手 |
| `help wanted` | 绿色 | 需要帮助 |
| `question` | 紫色 | 问题 |
| `wontfix` | 灰色 | 不修复 |
| `duplicate` | 灰色 | 重复 |

### 添加标签到 Issue

**方法一：创建时添加**
1. 创建 Issue 时
2. 点击 **Labels** 下拉框
3. 选择或创建标签

**方法二：创建后添加**
1. 打开 Issue 页面
2. 在右侧找到 **Labels**
3. 点击齿轮图标
4. 选择标签

```
┌─────────────────────────────────────┐
│  Labels  ⚙️                          │
│                                     │
│  ☐ bug                              │
│  ☑ enhancement  ← 选中这个          │
│  ☐ documentation                    │
│  ☐ good first issue                 │
│                                     │
│  Filter: [搜索标签...]              │
│                                     │
│  [New label]  ← 创建新标签          │
└─────────────────────────────────────┘
```

### 创建自定义标签

1. 在 Issues 页面点击 **Labels**
2. 点击 **New label**
3. 填写信息：
   - **Label name**：标签名称
   - **Description**：描述
   - **Color**：颜色（点击选择）
4. 点击 **Add label**

```
┌─────────────────────────────────────┐
│  New label                           │
│                                     │
│  Label name: [priority: high   ]    │
│                                     │
│  Description: [高优先级问题      ]    │
│                                     │
│  Color: [🔴]  ← 点击选择颜色         │
│                                     │
│     [Add label]                     │
└─────────────────────────────────────┘
```

## 指派 (Assignees)

将 Issue 分配给特定人员：

1. 打开 Issue 页面
2. 在右侧找到 **Assignees**
3. 点击 **Assign yourself** 或搜索添加

```
┌─────────────────────────────────────┐
│  Assignees                           │
│                                     │
│  No one assigned                    │
│                                     │
│  [Assign yourself]  ← 分配给自己    │
│  [Assign others]   ← 分配给他人     │
└─────────────────────────────────────┘
```

## 里程碑 (Milestones)

将 Issue 归入版本计划：

### 创建里程碑

1. 在 Issues 页面点击 **Milestones**
2. 点击 **New milestone**
3. 填写标题和描述
4. 设置截止日期
5. 点击 **Create milestone**

### 关联 Issue

1. 编辑 Issue
2. 在右侧选择 **Milestone**
3. 选择里程碑

```
┌─────────────────────────────────────┐
│  Milestone                           │
│                                     │
│  ○ No milestone                     │
│  ○ v1.0 - 首次发布                   │
│  ● v2.0 - 功能增强  ← 选择这个      │
│                                     │
└─────────────────────────────────────┘
```

## 关闭 Issue

### 方法一：在 Issue 页面关闭

1. 打开 Issue 页面
2. 点击底部的 **Close issue** 按钮

### 方法二：使用关键词自动关闭

在提交信息或 PR 描述中使用关键词：

```bash
# 关闭单个 Issue
git commit -m "fix: 修复登录问题，closes #42"

# 关闭多个 Issue
git commit -m "fix: 修复多个问题，fixes #42, fixes #43"
```

**关键词：**
- `closes #42`
- `fixes #42`
- `resolves #42`

### 方法三：在 PR 中自动关闭

1. 在 PR 描述中输入 `Closes #42`
2. 合并 PR 时会自动关闭关联的 Issue

## Issue 最佳实践

1. **使用清晰的标题**：简明描述问题
2. **提供复现步骤**：让别人能重现问题
3. **附加信息**：截图、日志、环境信息
4. **使用标签**：分类管理
5. **及时回复**：回应评论和问题

---

## 实践练习

### 练习：创建和管理 Issue

**任务 1：创建 Issue**
1. 创建一个仓库（如果没有）
2. 点击 **Issues** → **New issue**
3. 标题：`Bug: 测试问题`
4. 描述：填写问题详情
5. 添加标签 `bug`
6. 点击 **Submit new issue**

**任务 2：使用模板创建 Issue**
1. 点击 **New issue**
2. 选择 Bug Report 模板
3. 填写模板内容
4. 提交 Issue

**任务 3：管理 Issue**
1. 给 Issue 添加标签
2. 分配负责人
3. 关联里程碑
4. 关闭 Issue

**验证方法：**
- Issue 列表显示创建的 Issue
- 标签、指派、里程碑设置正确

## 下一步

[Pull Request 协作 →](17-pull-requests.md)
