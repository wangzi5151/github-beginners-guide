# 团队协作最佳实践

## 分支策略

### Git Flow
```
main (生产)
  ↑
develop (开发)
  ↑
feature/* (功能分支)
```

### GitHub Flow
```
main
  ↑
feature-branch → PR → 合并
```

### Trunk-Based Development
```
main
  ↑
短命分支 → 快速合并
```

## 提交信息规范

### Conventional Commits

```
<type>(<scope>): <subject>

<body>

<footer>
```

### 类型
| 类型 | 说明 |
|------|------|
| feat | 新功能 |
| fix | 修复 bug |
| docs | 文档 |
| style | 格式 |
| refactor | 重构 |
| test | 测试 |
| chore | 构建/工具 |

### 示例
```
feat(auth): 添加用户登录功能

实现了基于 JWT 的用户认证系统。
支持邮箱和密码登录。

Closes #42
```

## 代码审查流程

1. **提交 PR**：描述清晰
2. **自动化检查**：CI/CD 通过
3. **人工审查**：至少一人批准
4. **修改反馈**：及时响应
5. **合并部署**：保持主分支健康

## Issue 管理

### 使用标签分类
- `bug`：Bug
- `enhancement`：功能
- `documentation`：文档
- `priority/high`：高优先级
- `status/in-progress`：进行中

### 使用里程碑
- v1.0.0
- v1.1.0
- Sprint 1

## 文档管理

### 必要文档
- README.md：项目说明
- CONTRIBUTING.md：贡献指南
- CODE_OF_CONDUCT.md：行为准则
- CHANGELOG.md：更新日志

### 文档更新时机
- 代码变更时
- 功能新增时
- 配置变化时

## 沟通协作

### Issue 讨论
- 技术方案讨论
- 问题反馈
- 功能请求

### PR 讨论
- 代码审查
- 设计讨论
- 实现细节

## 团队角色

| 角色 | 职责 |
|------|------|
| Maintainer | 维护项目，审查 PR |
| Contributor | 提交代码，报告问题 |
| Reviewer | 审查代码质量 |

## 下一步

[Git 工作流 →](24-git-workflow.md)
