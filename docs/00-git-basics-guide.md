# 第四章：Git 基础命令详解

## 4.1 仓库操作

### 创建新仓库

#### 方法一：在 GitHub 网页创建

**第 1 步：** 登录 GitHub，点击右上角的 **+** 号，选择 **New repository**

**第 2 步：** 填写仓库信息
- Repository name：仓库名称
- Description：仓库描述（可选）
- Public/Private：选择可见性
- 勾选 **Add a README file**

**第 3 步：** 点击 **Create repository**

#### 方法二：在本地创建

```bash
# 创建新目录
mkdir my-project
cd my-project

# 初始化 Git 仓库
git init
```

**git init 的作用：**
- 创建 `.git` 目录
- 初始化仓库结构
- 创建默认分支（通常是 main）

#### 方法三：克隆现有仓库

```bash
# 克隆仓库
git clone https://github.com/user/repo.git

# 克隆到指定目录
git clone https://github.com/user/repo.git my-folder

# 克隆特定分支
git clone -b develop https://github.com/user/repo.git
```

### 查看仓库状态

```bash
git status
```

**输出示例：**

```
On branch main
Your branch is up to date with 'origin/main'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   README.md
        modified:   src/index.js

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        src/utils.js

no changes added to commit (use "git add" and/or "git commit -a")
```

**状态说明：**
- `Changes not staged for commit`：已修改但未暂存
- `Untracked files`：新文件，未被 Git 跟踪
- `Changes to be committed`：已暂存，等待提交

## 4.2 文件操作

### 添加文件到暂存区

```bash
# 添加单个文件
git add index.html

# 添加多个文件
git add index.html style.css

# 添加所有更改
git add .

# 添加所有 .js 文件
git add *.js

# 添加 src 目录下的所有文件
git add src/
```

**什么是暂存区？**

暂存区（Staging Area）是一个中间区域，用于准备下次提交的内容。你可以选择性地添加文件，而不是一次性提交所有更改。

```
工作区 → 暂存区 → 仓库
       git add    git commit
```

### 提交更改

```bash
# 提交暂存区的内容
git commit -m "描述信息"

# 提交并添加详细描述
git commit -m "描述信息" -m "详细描述"

# 添加所有更改并提交（跳过 git add）
git commit -am "描述信息"

# 修改最近一次提交
git commit --amend -m "新的描述信息"
```

**提交信息规范：**

```
类型(范围): 描述

详细描述（可选）

关联 Issue（可选）
```

**类型说明：**
- `feat`: 新功能
- `fix`: Bug 修复
- `docs`: 文档更新
- `style`: 代码格式（不影响功能）
- `refactor`: 重构
- `test`: 测试
- `chore`: 构建/工具

**示例：**
```bash
git commit -m "feat: 添加用户登录功能"
git commit -m "fix: 修复登录页面样式问题"
git commit -m "docs: 更新 README 安装说明"
```

### 查看文件差异

```bash
# 查看工作区与暂存区的差异
git diff

# 查看暂存区与仓库的差异
git diff --staged

# 查看两个提交之间的差异
git diff abc1234 def5678

# 查看某个文件的差异
git diff index.html
```

## 4.3 分支操作

### 查看分支

```bash
# 查看本地分支
git branch

# 查看所有分支（含远程）
git branch -a

# 查看分支及最后一次提交
git branch -v

# 查看已合并的分支
git branch --merged

# 查看未合并的分支
git branch --no-merged
```

**输出示例：**
```
* main          abc1234 最新提交信息
  feature-login def5678 添加登录功能
  feature-cart  ghi9012 购物车功能
```

带 `*` 号的是当前分支。

### 创建分支

```bash
# 创建新分支
git branch feature-login

# 创建并切换到新分支（推荐）
git checkout -b feature-login

# 或使用新语法
git switch -c feature-login

# 基于特定提交创建分支
git checkout -b feature-login abc1234

# 基于远程分支创建本地分支
git checkout -b feature-login origin/feature-login
```

### 切换分支

```bash
# 切换到已有分支
git checkout feature-login

# 或使用新语法
git switch feature-login
```

### 合并分支

```bash
# 切换到目标分支
git checkout main

# 合并功能分支
git merge feature-login

# 合并并创建合并提交
git merge --no-ff feature-login

# 取消合并
git merge --abort
```

**合并类型：**

1. **快进合并（Fast-forward）**
```
合并前：
main:    A --- B --- C
                      \
feature:               D --- E

合并后：
main:    A --- B --- C --- D --- E
```

2. **三方合并（Three-way merge）**
```
合并前：
main:    A --- B --- C --- F
                      \
feature:               D --- E

合并后：
main:    A --- B --- C --- F --- M (merge commit)
                      \         /
feature:               D --- E
```

### 删除分支

```bash
# 删除已合并的分支
git branch -d feature-login

# 强制删除分支
git branch -D feature-login

# 删除远程分支
git push origin --delete feature-login
```

### 重命名分支

```bash
# 重命名当前分支
git branch -m new-name

# 重命名指定分支
git branch -m old-name new-name
```

## 4.4 远程操作

### 查看远程仓库

```bash
# 查看远程仓库列表
git remote -v

# 查看远程仓库详细信息
git remote show origin
```

### 添加远程仓库

```bash
# 添加远程仓库
git remote add origin https://github.com/user/repo.git

# 添加多个远程仓库
git remote add upstream https://github.com/owner/repo.git
```

### 推送到远程

```bash
# 推送当前分支
git push

# 推送到指定分支
git push origin main

# 首次推送并设置上游分支
git push -u origin main

# 推送所有分支
git push --all

# 推送标签
git push --tags
```

### 从远程拉取

```bash
# 拉取远程更改（不合并）
git fetch

# 拉取并合并
git pull

# 拉取并变基
git pull --rebase

# 拉取特定远程分支
git pull origin main
```

### 查看远程分支

```bash
# 查看远程分支
git branch -r

# 查看所有分支（本地+远程）
git branch -a

# 查看远程分支的详细信息
git branch -vv
```

## 4.5 查看历史

### 查看提交历史

```bash
# 查看所有提交
git log

# 查看简洁版历史
git log --oneline

# 查看图形化历史
git log --graph --oneline

# 查看最近 5 次提交
git log -5

# 查看某个文件的历史
git log index.html

# 查看某个作者的提交
git log --author="Zhang"

# 按时间筛选
git log --since="2024-01-01"
git log --until="2024-12-31"
```

**输出示例：**
```
* abc1234 (HEAD -> main) feat: 添加用户登录
* def5678 fix: 修复样式问题
* ghi9012 docs: 更新 README
* jkl3456 feat: 初始化项目
```

### 查看提交详情

```bash
# 查看某次提交的详细信息
git show abc1234

# 查看某次提交的文件变更
git show --stat abc1234
```

### 查看文件变更

```bash
# 查看工作区与暂存区的差异
git diff

# 查看暂存区与仓库的差异
git diff --staged

# 查看某次提交的变更
git diff abc1234^ abc1234

# 查看某文件的变更
git diff index.html
```

## 4.6 撤销操作

### 撤销工作区的修改

```bash
# 丢弃工作区的修改
git restore index.html

# 丢弃所有修改
git restore .
```

### 撤销暂存区的修改

```bash
# 将文件从暂存区移除（保留工作区修改）
git restore --staged index.html

# 将所有文件从暂存区移除
git restore --staged .
```

### 修改最近一次提交

```bash
# 修改提交信息
git commit --amend -m "新的提交信息"

# 添加文件到最近一次提交
git add forgotten-file.js
git commit --amend --no-edit
```

### 回退提交

```bash
# 回退到某次提交（保留修改在工作区）
git reset abc1234

# 回退到某次提交（保留修改在暂存区）
git reset --soft abc1234

# 回退到某次提交（丢弃所有修改）
git reset --hard abc1234

# 回退最近一次提交
git reset HEAD~1
```

**reset 的三种模式：**

| 模式 | 说明 |
|------|------|
| `--soft` | 保留工作区和暂存区的修改 |
| `--mixed`（默认） | 保留工作区的修改，清空暂存区 |
| `--hard` | 丢弃所有修改 |

### 创建反向提交

```bash
# 创建一个新提交来撤销之前的提交
git revert abc1234
```

## 4.7 标签操作

### 查看标签

```bash
# 列出所有标签
git tag

# 查看标签详情
git show v1.0.0

# 按模式搜索标签
git tag -l "v1.*"
```

### 创建标签

```bash
# 创建轻量标签
git tag v1.0.0

# 创建附注标签（推荐）
git tag -a v1.0.0 -m "Release version 1.0.0"

# 为某次提交创建标签
git tag -a v1.0.0 abc1234 -m "Release version 1.0.0"
```

### 推送标签

```bash
# 推送单个标签
git push origin v1.0.0

# 推送所有标签
git push origin --tags
```

### 删除标签

```bash
# 删除本地标签
git tag -d v1.0.0

# 删除远程标签
git push origin --delete v1.0.0
```

### 检出标签

```bash
# 检出标签（只读）
git checkout v1.0.0

# 基于标签创建分支
git checkout -b release-1.0.0 v1.0.0
```

## 4.8 暂存操作

### 暂存当前修改

```bash
# 暂存所有修改
git stash

# 暂存并添加描述
git stash save "正在开发登录功能"

# 暂存未跟踪的文件
git stash -u

# 暂存所有文件（包括忽略的）
git stash -a
```

### 查看暂存列表

```bash
git stash list
```

**输出示例：**
```
stash@{0}: On main: 正在开发登录功能
stash@{1}: WIP on main: abc1234 some commit
```

### 恢复暂存

```bash
# 恢复最近的暂存（保留暂存记录）
git stash apply

# 恢复最近的暂存（删除暂存记录）
git stash pop

# 恢复特定暂存
git stash apply stash@{2}

# 恢复特定暂存并删除记录
git stash pop stash@{2}
```

### 删除暂存

```bash
# 删除最近的暂存
git stash drop

# 删除特定暂存
git stash drop stash@{0}

# 删除所有暂存
git stash clear
```

### 查看暂存详情

```bash
# 查看最近暂存的详细内容
git stash show -p

# 查看特定暂存的详细内容
git stash show -p stash@{2}
```

## 4.9 本章小结

本章详细介绍了 Git 的基础命令，包括：

- 仓库操作：创建、克隆、状态查看
- 文件操作：添加、提交、差异查看
- 分支操作：创建、切换、合并、删除
- 远程操作：添加、推送、拉取
- 历史查看：日志、差异、详情
- 撤销操作：修改、回退、反向提交
- 标签操作：创建、推送、删除
- 暂存操作：暂存、恢复、删除

**关键要点：**
- 熟练掌握这些命令是使用 Git 的基础
- 多练习，在实践中加深理解
- 遇到问题不要慌，大多数操作都可以撤销

**下一步：**
[GitHub 核心功能 →](14-create-repo.md)
