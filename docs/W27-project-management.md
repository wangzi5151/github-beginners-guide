# GitHub 项目管理

## GitHub Projects

### 项目视图

| 视图 | 说明 |
|------|------|
| Board | 看板视图 |
| Table | 表格视图 |
| Roadmap | 路线图视图 |
| Calendar | 日历视图 |

### 创建项目

```bash
# 使用 CLI 创建项目
gh project create --title "项目名称" --owner your-org

# 添加到项目
gh project item-add 1 --owner your-org --url https://github.com/your-org/your-repo/issues/1
```

### 自动化

```yaml
# .github/workflows/project-automation.yml
name: Project Automation

on:
  issues:
    types: [opened, closed]
  pull_request:
    types: [opened, closed, ready_for_review]

jobs:
  auto-add:
    runs-on: ubuntu-latest
    steps:
    - name: Add to project
      uses: actions/add-to-project@v0.5.0
      with:
        project-url: https://github.com/orgs/your-org/projects/1
        github-token: ${{ secrets.GITHUB_TOKEN }}
```

## Issue 管理

### Issue 模板

```yaml
# .github/ISSUE_TEMPLATE/bug.yml
name: Bug Report
description: 报告问题
labels: ["bug"]
body:
  - type: textarea
    id: description
    attributes:
      label: 问题描述
      description: 描述你遇到的问题
    validations:
      required: true
  - type: textarea
    id: steps
    attributes:
      label: 复现步骤
      description: 如何复现这个问题
    validations:
      required: true
  - type: dropdown
    id: priority
    attributes:
      label: 优先级
      options:
        - P0 - 紧急
        - P1 - 高
        - P2 - 中
        - P3 - 低
    validations:
      required: true
```

### Issue 标签

```yaml
标签管理：
  类型:
    - bug: Bug 修复
    - feature: 新功能
    - docs: 文档
    - enhancement: 增强
  
  优先级:
    - P0: 紧急
    - P1: 高
    - P2: 中
    - P3: 低
  
  状态:
    - needs-triage: 待分类
    - needs-review: 待审查
    - in-progress: 进行中
    - done: 完成
```

## 里程碑管理

### 创建里程碑

```bash
# 创建里程碑
gh api repos/{org}/{repo}/milestones \
  --method POST \
  -f title="v1.0.0" \
  -f description="第一个正式版本" \
  -f due_on="2024-12-31T00:00:00Z"
```

### 里程碑规划

```markdown
# 里程碑规划

## v1.0.0（2024-12-31）
### 目标
- 完成核心功能
- 通过安全审计
- 发布正式版本

### 任务
- [ ] 功能开发
- [ ] 测试编写
- [ ] 文档完善
- [ ] 安全审查
```

## 发布管理

### 发布流程

```yaml
发布流程：
  准备:
    - 功能冻结
    - 测试通过
    - 文档更新
  
  发布:
    - 创建 release 分支
    - 更新版本号
    - 创建 tag
    - 生成 Release Notes
  
  部署:
    - 部署到 staging
    - 验证通过
    - 部署到 production
  
  监控:
    - 监控错误率
    - 监控性能
    - 收集反馈
```

### Release Notes

```yaml
# .github/release.yml
changelog:
  categories:
    - title: 🚀 New Features
      labels:
        - enhancement
    - title: 🐛 Bug Fixes
      labels:
        - bug
    - title: 📝 Documentation
      labels:
        - documentation
    - title: 🔒 Security
      labels:
        - security
    - title: ⚡ Performance
      labels:
        - performance
```

## 进度跟踪

### 仪表板

```markdown
# 项目仪表板

## 进度概览
- 总任务: 50
- 已完成: 30 (60%)
- 进行中: 10 (20%)
- 待开始: 10 (20%)

## 本周完成
- 完成 5 个任务
- 合并 3 个 PR
- 修复 2 个 Bug

## 风险
- 依赖库有安全漏洞
- 性能测试未通过
```

### 自动化报告

```yaml
# .github/workflows/weekly-report.yml
name: Weekly Report

on:
  schedule:
    - cron: '0 0 * * 1'  # 每周一

jobs:
  report:
    runs-on: ubuntu-latest
    steps:
    - name: Generate report
      run: |
        # 获取本周 PR 统计
        gh pr list --state merged --json mergedAt \
          --jq '[.[] | select(.mergedAt >= (now - 604800))] | length'
        
        # 获取本周 Issue 统计
        gh issue list --state closed --json closedAt \
          --jq '[.[] | select(.closedAt >= (now - 604800))] | length'
```

## 最佳实践

1. **清晰的目标**：确保每个里程碑有清晰的目标
2. **合理的估算**：对任务进行合理的时间估算
3. **定期更新**：定期更新任务状态
4. **及时沟通**：有问题及时沟通
5. **持续改进**：定期回顾和改进流程

## 相关资源

- [GitHub Projects 文档](https://docs.github.com/en/issues/organizing-your-work-with-project-boards)
- [Issue 管理](https://docs.github.com/en/issues/tracking-your-work-with-issues)
- [Release 管理](https://docs.github.com/en/repositories/releasing-projects-on-github)

---

**上一篇：[团队协作规范](W26-team-collaboration.md) | 下一篇：[GitHub 性能优化](W28-performance.md)**
