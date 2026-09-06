# 撤销操作

## 修改最后一次提交

```bash
# 修改提交信息
git commit --amend -m "新的提交信息"

# 添加遗漏的文件到上次提交
git add forgotten-file.js
git commit --amend --no-edit

# 修改上次提交的内容
git add staged-changes.js
git commit --amend
```

## 撤销工作区修改

```bash
# 撤销单个文件的修改
git checkout -- filename.js

# 撤销所有修改（危险！）
git checkout -- .
```

## 取消暂存

```bash
# 取消暂存文件（保留修改）
git reset HEAD filename.js

# 取消暂存所有文件
git reset HEAD
```

## 回退提交

### 保留修改（软回退）

```bash
# 回退到上一次提交，保留修改在暂存区
git reset --soft HEAD~1

# 回退到指定提交
git reset --soft commit-hash
```

### 保留修改（混合回退，默认）

```bash
# 回退到上一次提交，保留修改在工作区
git reset HEAD~1

# 回退到指定提交
git reset commit-hash
```

### 丢弃修改（硬回退，危险！）

```bash
# 回退到上一次提交，丢弃所有修改
git reset --hard HEAD~1

# 回退到指定提交
git reset --hard commit-hash
```

## 回退远程提交

```bash
# 回退远程提交（危险！需要强制推送）
git revert HEAD
git push origin main

# 或者使用 reset（更危险！）
git reset --hard HEAD~1
git push --force origin main
```

**警告：不要对公共分支使用 `git push --force`**

## 重置文件到某次提交状态

```bash
# 将某个文件恢复到某次提交的状态
git checkout commit-hash -- filename.js
```

## 查看引用日志

```bash
# 查看所有操作历史
git reflog

# 恢复到某次操作
git reflog
# 找到想要恢复的 commit hash
git checkout commit-hash
```

## 恢复已删除的分支

```bash
# 通过 reflog 找到分支的最后提交
git reflog
# 找到类似 branch@{n} 的记录
git checkout -b recovered-branch commit-hash
```

## 常用场景

### 场景1：提交后发现遗漏文件
```bash
git add missing-file.js
git commit --amend --no-edit
```

### 场景2：提交了错误的文件
```bash
git reset HEAD wrong-file.js
git add correct-file.js
git commit -m "正确的提交"
```

### 场景3：需要完全重做上次提交
```bash
git reset --soft HEAD~1
# 重新修改文件
git add .
git commit -m "重做的提交"
```

## 下一步

[创建和管理仓库 →](14-create-repo.md)
