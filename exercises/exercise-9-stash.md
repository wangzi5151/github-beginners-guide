# 练习 9：Git Stash 实战

## 学习目标

- 掌握 git stash 的基本操作
- 理解 stash 的使用场景
- 学会管理多个 stash

## 场景描述

你正在开发一个新功能，突然需要切换到另一个分支修复紧急 bug。使用 stash 临时保存你的工作。

## 步骤

### 步骤 1：创建测试仓库

```bash
mkdir stash-practice
cd stash-practice
git init
```

### 步骤 2：创建初始文件

```bash
echo "# 项目说明" > README.md
echo "console.log('Hello');" > app.js
git add .
git commit -m "feat: 初始提交"
```

### 步骤 3：开始新功能开发

```bash
echo "// 新功能代码" >> app.js
echo "// 更多代码" >> app.js
git status
```

### 步骤 4：使用 stash 保存工作

```bash
# 保存当前工作
git stash save "开发新功能中"

# 查看 stash 列表
git stash list
```

### 步骤 5：切换分支修复 bug

```bash
# 创建并切换到修复分支
git checkout -b fix/urgent-bug

# 修复 bug
echo "// 修复的代码" > bugfix.js
git add .
git commit -m "fix: 修复紧急 bug"

# 切换回原分支
git checkout main
```

### 步骤 6：恢复 stash 的内容

```bash
# 恢复最近的 stash
git stash pop

# 查看文件内容
cat app.js
```

## 高级操作

### 查看 stash 详情

```bash
# 查看 stash 的详细内容
git stash show -p stash@{0}

# 查看特定 stash
git stash show -p stash@{1}
```

### 管理多个 stash

```bash
# 保存多个 stash
git stash save "功能 1"
git stash save "功能 2"
git stash save "功能 3"

# 查看所有 stash
git stash list

# 应用特定 stash（不删除）
git stash apply stash@{1}

# 删除特定 stash
git stash drop stash@{1}
```

### 清除所有 stash

```bash
# 删除所有 stash
git stash clear

# 或者删除特定 stash
git stash drop stash@{0}
git stash drop stash@{1}
```

## 实战任务

### 任务 1：创建并管理 stash

1. 创建一个新文件 `feature.js`
2. 添加一些内容但不提交
3. 使用 stash 保存更改
4. 创建另一个文件 `temp.js`
5. 再次使用 stash 保存
6. 查看 stash 列表
7. 恢复第一个 stash

### 任务 2：处理 stash 冲突

1. 修改文件 A 并 stash
2. 在另一个分支修改文件 A 并提交
3. 切换回原分支
4. 尝试 stash pop
5. 解决冲突

### 任务 3：使用 -u 参数

```bash
# 保存未跟踪文件
git stash save -u "包含新文件"
```

1. 创建新文件（未跟踪）
2. 使用 `-u` 参数 stash
3. 验证新文件被 stash 保存

## 验证清单

- [ ] 能够使用 `git stash save`
- [ ] 能够使用 `git stash list`
- [ ] 能够使用 `git stash pop`
- [ ] 能够使用 `git stash apply`
- [ ] 能够使用 `git stash show`
- [ ] 能够管理多个 stash
- [ ] 能够处理 stash 冲突

## 常见问题

### Q: stash 和 commit 的区别？
A: stash 是临时保存，不会创建新的提交；commit 是永久保存到历史记录。

### Q: stash 会保存未跟踪的文件吗？
A: 默认不会，需要使用 `-u` 参数。

### Q: 如何恢复已删除的 stash？
A: 使用 `git stash` 的 reflog 功能：
```bash
git fsck --no-reflogs | grep commit
git stash apply <commit-hash>
```

## 下一步

完成本练习后，请继续进行 [练习 10：Interactive Rebase 实战](exercise-10-interactive-rebase.md)
