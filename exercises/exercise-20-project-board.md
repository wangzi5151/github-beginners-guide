# 练习 20：GitHub Projects 看板管理

## 学习目标

- 创建 GitHub Project
- 配置看板视图
- 使用自动化

## 步骤

### 步骤 1：创建项目

```bash
# 创建项目
gh project create --title "My Project" --owner your-org
```

### 步骤 2：添加 Issue 到项目

```bash
# 添加 Issue
gh project item-add 1 --owner your-org --url https://github.com/your-org/your-repo/issues/1
```

### 步骤 3：配置自动化

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

### 步骤 4：创建视图

```bash
# 使用 CLI 查看项目
gh project list --owner your-org
gh project item-list 1 --owner your-org
```

## 实战任务

1. 创建 GitHub Project
2. 添加多个 Issue 到项目
3. 配置自动化工作流
4. 创建不同视图（看板、表格、路线图）

## 验证清单

- [ ] 项目已创建
- [ ] Issue 已添加到项目
- [ ] 自动化已配置
- [ ] 视图已创建

## 完成

恭喜完成所有练习！你已经掌握了：
- Git 基础操作
- GitHub 核心功能
- CI/CD 流水线
- Docker 容器化
- 安全扫描
- 发布管理
- Monorepo 管理
- 国内环境配置
- 企业配置
- 复用工作流
- 项目管理
