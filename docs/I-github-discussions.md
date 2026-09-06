# GitHub Discussions 介绍

## 什么是 GitHub Discussions？

GitHub Discussions 是一个面向社区的问答和讨论平台，类似于论坛。它允许团队、社区和用户围绕项目进行讨论。

## Discussions vs Issues

| 特性 | Discussions | Issues |
|------|-------------|--------|
| 目的 | 问答、讨论、分享 | 追踪 bug、功能请求 |
| 结构 | 分类、标签、答案 | 标签、里程碑、指派 |
| 答案 | 可标记最佳答案 | 无答案概念 |
| 社交 | 点赞、评论、关注 | 评论、订阅 |
| 关闭 | 无需关闭 | 可以关闭 |

## 启用 Discussions

1. 进入仓库 **Settings**
2. 在 **Features** 部分勾选 **Discussions**
3. 点击 **Set up discussions**

## 分类设置

### 默认分类

| 分类 | 用途 |
|------|------|
| 📣 Announcements | 官方公告 |
| 💬 General | 一般讨论 |
| 💡 Ideas | 想法和建议 |
| 🙋 Q&A | 问答 |
| 📝 Show and tell | 展示和分享 |

### 自定义分类

1. 进入 **Discussions** 页面
2. 点击 **Categories**
3. 点击 **New category**
4. 设置名称、描述和图标

## 使用场景

### 1. 项目问答

```
标题：如何配置数据库连接？

我在尝试连接 PostgreSQL 数据库时遇到错误...
[错误信息]

有人知道怎么解决吗？
```

### 2. 功能讨论

```
标题：建议：添加暗黑模式支持

我觉得这个项目可以添加暗黑模式功能...
[详细描述]

大家觉得怎么样？
```

### 3. 社区展示

```
标题：我用这个项目做了一个 XXX

大家好，我用这个项目开发了一个应用...
[截图和说明]

欢迎提意见！
```

## 最佳答案

1. 在讨论中回复
2. 点击回复旁边的 **...** 菜单
3. 选择 **Mark as answer**

最佳答案会突出显示，方便后来者快速找到解决方案。

## 管理 Discussion

### 锁定讨论
防止进一步评论：
1. 点击 **Lock discussion**
2. 选择锁定原因

### 转换为 Issue
如果讨论发现是 bug：
1. 点击 **Convert to issue**
2. Issue 会链接到原讨论

### 删除 Discussion
1. 点击 **Delete discussion**
2. 确认删除

## 团队协作

### 代码所有者

在 `.github/CODEOWNERS` 中定义：
```
# Discussions 由社区团队管理
.github/discussions/ @community-team
```

### 指派负责人

在 Discussion 中添加负责人：
1. 点击右侧 **Assignees**
2. 选择负责人

## 最佳实践

1. **设置清晰的分类**：帮助用户找到正确的讨论区
2. **及时回复**：保持社区活跃
3. **标记最佳答案**：方便后来者
4. **定期清理**：删除无意义的内容
5. **鼓励参与**：感谢贡献者

## 集成其他功能

### 关联 Issue

```
相关 Issue：#123
```

### 关联 PR

```
相关 PR：#456
```

### 添加标签

使用 Issue 标签来分类讨论。

## 相关资源

- [GitHub Discussions 官方文档](https://docs.github.com/en/discussions)
- [使用讨论进行社区管理](https://docs.github.com/en/discussions/collaborating-with-your-community-using-discussions)
