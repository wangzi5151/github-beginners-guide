# Git 工作流

## Git Flow

适合有明确发布周期的项目。

### 分支结构
```
main ─────────────────────────────────── 生产环境
  ↑
develop ──────────────────────────────── 开发分支
  ↑
feature/* ───── 功能开发
  ↑
release/* ───── 发布准备
  ↑
hotfix/* ────── 紧急修复
```

### 工作流程

```bash
# 开始新功能
git checkout develop
git checkout -b feature/new-feature

# 完成功能
git checkout develop
git merge --no-ff feature/new-feature
git branch -d feature/new-feature

# 准备发布
git checkout develop
git checkout -b release/1.0.0

# 发布
git checkout main
git merge --no-ff release/1.0.0
git tag -a v1.0.0 -m "Release 1.0.0"
git checkout develop
git merge --no-ff release/1.0.0

# 紧急修复
git checkout main
git checkout -b hotfix/fix-bug
git merge --no-ff hotfix/fix-bug
git checkout develop
git merge --no-ff hotfix/fix-bug
```

## GitHub Flow

简单的工作流，适合持续部署的项目。

### 流程
1. 从 main 创建分支
2. 在分支上开发
3. 创建 PR
4. 代码审查
5. 合并到 main
6. 部署

```bash
# 开始工作
git checkout main
git pull
git checkout -b feature/add-login

# 开发
git add .
git commit -m "feat: 添加登录页面"

# 推送并创建 PR
git push -u origin feature/add-login
# 在 GitHub 上创建 PR
```

## Trunk-Based Development

主干开发，适合快速迭代的团队。

### 特点
- 所有人直接向 main 提交
- 使用短命分支（< 1天）
- 功能开关控制发布

```bash
# 快速开发
git checkout main
git pull
git checkout -b quick-fix
# 小改动
git add .
git commit -m "fix: 小修复"
git push -u origin quick-fix
# 立即创建 PR 并合并
```

## 选择建议

| 工作流 | 适用场景 |
|--------|---------|
| Git Flow | 传统软件发布 |
| GitHub Flow | 持续部署 |
| Trunk-Based | 快速迭代团队 |

## 下一步

[标签与发布 →](25-tags-releases.md)
