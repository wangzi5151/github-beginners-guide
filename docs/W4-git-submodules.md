# Git Submodules 使用指南

## 什么是 Submodule？

Git Submodule 允许你将一个 Git 仓库作为另一个 Git 仓库的子目录。它能让你将一个仓库作为另一个仓库的子模块来引用。

## 基本操作

### 添加 Submodule

```bash
# 添加子模块
git submodule add https://github.com/user/repo.git path/to/submodule

# 添加指定分支的子模块
git submodule add -b main https://github.com/user/repo.git path/to/submodule
```

### 克隆包含 Submodule 的仓库

```bash
# 方法 1：克隆时初始化
git clone --recurse-submodules https://github.com/user/repo.git

# 方法 2：克隆后初始化
git clone https://github.com/user/repo.git
cd repo
git submodule init
git submodule update

# 方法 3：自动初始化
git clone --recurse-submodules --remote-submodules https://github.com/user/repo.git
```

### 更新 Submodule

```bash
# 更新到子模块的最新提交
git submodule update --remote

# 更新并合并
git submodule update --remote --merge

# 更新并变基
git submodule update --remote --rebase
```

## 查看 Submodule 信息

```bash
# 查看子模块列表
git submodule

# 查看子模块详情
git submodule summary

# 查看子模块状态
git submodule status
```

## 修改 Submodule

### 在子模块中工作

```bash
# 进入子模块目录
cd path/to/submodule

# 切换分支
git checkout main

# 进行修改并提交
git add .
git commit -m "修改子模块"
git push
```

### 更新父仓库的子模块引用

```bash
# 在子模块目录中进行修改后
cd path/to/submodule
git add .
git commit -m "更新子模块"
git push

# 返回父仓库
cd ../..
git add path/to/submodule
git commit -m "更新子模块引用"
git push
```

## 删除 Submodule

```bash
# 步骤 1：从 .gitmodules 中删除
git submodule deinit -f path/to/submodule

# 步骤 2：从 .git/modules 中删除
git rm -f path/to/submodule

# 步骤 3：从工作目录中删除
rm -rf .git/modules/path/to/submodule
```

## 高级用法

### 指定 Submodule 版本

```bash
# 切换到特定提交
cd path/to/submodule
git checkout v1.0.0
cd ../..
git add path/to/submodule
git commit -m "使用子模块的 v1.0.0 版本"
```

### 使用 .gitmodules 配置

```ini
[submodule "path/to/submodule"]
    path = path/to/submodule
    url = https://github.com/user/repo.git
    branch = main
```

### 自动更新 Submodule

创建 GitHub Actions 工作流：

```yaml
name: Update Submodules

on:
  schedule:
    - cron: '0 0 * * *'  # 每天运行
  workflow_dispatch:

jobs:
  update:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
      with:
        submodules: true
        fetch-depth: 0
    
    - name: Update submodules
      run: |
        git submodule update --remote
        git config user.name "github-actions[bot]"
        git config user.email "github-actions[bot]@users.noreply.github.com"
        git add .
        git commit -m "chore: update submodules" || exit 0
        git push
```

## 常见问题

### Q: Submodule 和 Subtree 的区别？

| 特性 | Submodule | Subtree |
|------|-----------|---------|
| 复杂度 | 较高 | 较低 |
| 历史记录 | 分离的 | 合并的 |
| 离线工作 | 不支持 | 支持 |
| 推送修改 | 需要两个仓库 | 直接推送 |

### Q: 如何处理 Submodule 冲突？

1. 进入子模块目录解决冲突
2. 提交子模块的更改
3. 返回父仓库提交子模块引用更新

### Q: Submodule 不显示修改？

```bash
# 检查子模块状态
git submodule status

# 强制更新
git submodule update --init --force
```

## 最佳实践

1. **使用 HTTPS URL**：避免 SSH 密钥配置问题
2. **指定分支**：明确使用哪个分支
3. **文档化子模块**：在 README 中说明子模块用途
4. **定期更新**：保持子模块为最新版本
5. **考虑替代方案**：对于简单场景，考虑使用 subtree 或包管理器

## 相关资源

- [Git Submodule 官方文档](https://git-scm.com/book/en/v2/Git-Tools-Submodules)
- [Subtree 替代方案](https://www.atlassian.com/git/tutorials/git-subtree)

---

**上一篇：[GitHub CLI 命令行工具](26-github-cli.md) | 下一篇：[安全与权限管理](28-security-permissions.md)**
