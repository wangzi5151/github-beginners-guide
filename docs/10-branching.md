# 分支操作

## 基本概念

分支是平行开发的独立线路。主分支通常是 `main`（或 `master`）。

```
main:     A --- B --- C
                   \
feature:            D --- E
```

## 分支操作命令

### 查看分支

```bash
# 查看本地分支
git branch

# 查看所有分支（含远程）
git branch -a

# 查看分支及最后一次提交
git branch -v
```

### 创建分支

```bash
git branch feature-login
```

### 切换分支

```bash
git checkout feature-login
# 或（Git 2.23+）
git switch feature-login
```

### 创建并切换分支

```bash
git checkout -b feature-login
# 或
git switch -c feature-login
```

### 删除分支

```bash
# 删除已合并的分支
git branch -d feature-login

# 强制删除分支
git branch -D feature-login
```

### 重命名分支

```bash
git branch -m old-name new-name
```

## 分支工作流

### 1. 创建功能分支

```bash
git checkout main
git pull
git checkout -b feature-new-button
```

### 2. 在功能分支上工作

```bash
# 修改文件
git add .
git commit -m "feat: 添加新按钮"
```

### 3. 合并到主分支

```bash
git checkout main
git merge feature-new-button
```

### 4. 删除功能分支

```bash
git branch -d feature-new-button
```

## 远程分支

```bash
# 推送本地分支到远程
git push -u origin feature-login

# 查看远程分支
git branch -r

# 删除远程分支
git push origin --delete feature-login

# 清理已删除的远程分支引用
git fetch --prune
```

## 分支保护

在 GitHub 上可以设置分支保护规则：
1. 进入仓库 Settings → Branches
2. 点击 Add rule
3. 配置保护规则（如要求 PR 审查、通过检查等）

## 下一步

[合并与变基 →](11-merge-rebase.md)
