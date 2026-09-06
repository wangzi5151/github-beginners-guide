# 练习 2：分支与合并练习

## 目标

掌握 Git 分支创建、切换、合并和删除操作。

## 步骤

### 1. 创建练习仓库

```bash
mkdir branch-practice
cd branch-practice
git init
echo "# Branch Practice" > README.md
git add README.md
git commit -m "Initial commit"
```

### 2. 创建多个分支

```bash
# 创建功能分支
git checkout -b feature-1
echo "Feature 1" > feature1.txt
git add feature1.txt
git commit -m "feat: 添加功能1"

# 回到主分支
git checkout main

# 创建另一个功能分支
git checkout -b feature-2
echo "Feature 2" > feature2.txt
git add feature2.txt
git commit -m "feat: 添加功能2"

# 查看所有分支
git branch -a
```

### 3. 合并分支

```bash
# 合并 feature-1 到 main
git checkout main
git merge feature-1

# 查看历史
git log --oneline --graph --all
```

### 4. 处理合并冲突

```bash
# 在 main 分支修改文件
echo "Main changes" > shared.txt
git add shared.txt
git commit -m "main: 修改 shared.txt"

# 在 feature-2 分支修改同一文件
git checkout feature-2
echo "Feature 2 changes" > shared.txt
git add shared.txt
git commit -m "feat: 修改 shared.txt"

# 尝试合并（会有冲突）
git checkout main
git merge feature-2
```

### 5. 解决冲突

1. 打开 `shared.txt`
2. 选择保留的内容
3. 删除冲突标记
4. 提交

```bash
# 手动编辑文件后
git add shared.txt
git commit -m "解决合并冲突"
```

### 6. 删除分支

```bash
# 删除已合并的分支
git branch -d feature-1
git branch -d feature-2

# 查看剩余分支
git branch
```

### 7. 推送到远程

```bash
git remote add origin git@github.com:你的用户名/branch-practice.git
git push -u origin main
```

## 挑战任务

1. 创建一个名为 `hotfix` 的分支
2. 在 `hotfix` 分支上修复一个 bug
3. 将 `hotfix` 分支合并到 `main`
4. 删除 `hotfix` 分支

## 知识点

- `git branch`：查看和创建分支
- `git checkout`：切换分支
- `git merge`：合并分支
- `git branch -d`：删除分支
- `git log --graph`：查看分支图

## 下一步

[练习 3：Pull Request 工作流 →](exercise-3-pull-request.md)
