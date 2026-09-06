# GitHub 团队协作规范

## 协作流程

### 标准工作流

```
1. 从 main 创建功能分支
2. 在功能分支上开发
3. 提交 PR
4. 代码审查
5. CI 检查通过
6. 合并到 main
7. 部署
```

### 分支命名规范

```yaml
分支命名：
  功能开发: feature/xxx
  Bug 修复: bugfix/xxx
  紧急修复: hotfix/xxx
  文档更新: docs/xxx
  实验功能: experiment/xxx
```

### 提交信息规范

```
<type>(<scope>): <subject>

<body>

<footer>
```

类型说明：

| 类型 | 说明 |
|------|------|
| feat | 新功能 |
| fix | Bug 修复 |
| docs | 文档更新 |
| style | 代码格式（不影响功能） |
| refactor | 重构 |
| test | 测试 |
| chore | 构建/工具 |
| perf | 性能优化 |
| ci | CI 配置 |
| build | 构建系统 |

## 代码审查

### 审查清单

```markdown
## 代码质量
- [ ] 代码是否清晰易懂
- [ ] 是否遵循编码规范
- [ ] 是否有不必要的复杂性

## 功能正确性
- [ ] 是否实现了需求
- [ ] 边界情况是否处理
- [ ] 错误处理是否完善

## 测试
- [ ] 是否有单元测试
- [ ] 测试覆盖率是否足够
- [ ] 测试用例是否合理

## 安全性
- [ ] 是否有安全漏洞
- [ ] 是否有敏感信息泄露
- [ ] 权限控制是否正确

## 性能
- [ ] 是否有性能问题
- [ ] 是否有内存泄漏
- [ ] 是否有不必要的计算

## 文档
- [ ] 是否需要更新文档
- [ ] 注释是否清晰
- [ ] README 是否需要更新
```

### 审查反馈

```markdown
## 反馈格式

### 必须修改
- [ ] 问题描述
- [ ] 建议方案

### 建议修改
- [ ] 问题描述
- [ ] 建议方案

### 问题
- [ ] 问题描述

### 优点
- [ ] 亮点描述
```

## 团队协作

### 团队结构

```yaml
团队结构：
  技术负责人:
    - 架构设计
    - 技术决策
    - 代码审查
  
  开发工程师:
    - 功能开发
    - Bug 修复
    - 测试编写
  
  DevOps 工程师:
    - CI/CD 维护
    - 部署管理
    - 基础设施
  
  产品经理:
    - 需求管理
    - 优先级排序
    - 进度跟踪
```

### 协作工具

| 工具 | 用途 |
|------|------|
| GitHub Issues | 任务管理 |
| GitHub Projects | 项目看板 |
| GitHub Discussions | 技术讨论 |
| GitHub Actions | 自动化 |
| Slack/Teams | 即时通讯 |
| Notion/Confluence | 文档协作 |

### 沟通机制

```markdown
## 每日站会（15分钟）
- 昨天做了什么
- 今天计划做什么
- 有什么阻碍

## 周会（1小时）
- 上周回顾
- 本周计划
- 技术分享

## 月度复盘（2小时）
- 目标回顾
- 问题分析
- 改进计划
```

## 知识共享

### 代码审查学习

```markdown
## 学习方式
1. 参与他人代码审查
2. 学习审查反馈
3. 分享审查经验
4. 建立审查指南
```

### 技术分享

```markdown
## 分享机制
1. 每周技术分享
2. 月度技术博客
3. 季度技术分享会
4. 年度技术总结
```

### 文档管理

```markdown
## 文档类型
- README.md: 项目介绍
- CONTRIBUTING.md: 贡献指南
- docs/: 详细文档
- ADR/: 架构决策记录
```

## 冲突解决

### 冲突类型

| 类型 | 解决方式 |
|------|----------|
| 代码冲突 | 手动解决 |
| 观点冲突 | 讨论达成共识 |
| 优先级冲突 | 产品负责人决定 |
| 技术方案冲突 | 技术负责人决定 |

### 解决流程

```markdown
## 解决步骤
1. 识别冲突
2. 分析原因
3. 讨论方案
4. 达成共识
5. 记录决策
6. 执行方案
```

## 最佳实践

1. **清晰的沟通**：保持沟通清晰、及时
2. **尊重他人**：尊重团队成员的意见
3. **文档化**：重要的决策和约定要文档化
4. **持续改进**：定期回顾和改进协作流程
5. **知识共享**：积极分享知识和经验

## 相关资源

- [GitHub 协作文档](https://docs.github.com/en/collaborating)
- [团队管理最佳实践](https://docs.github.com/en/organizations/managing-user-access-to-your-organizations-repositories)
- [代码审查指南](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/reviewing-changes-in-pull-requests/about-pull-request-reviews)

---

**上一篇：[GitHub 企业治理](W25-enterprise-governance.md) | 下一篇：[GitHub 项目管理](W27-project-management.md)**
