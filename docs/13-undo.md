# Git 撤销操作完全指南

> **"在 Git 中，几乎没有什么操作是不可撤销的。"**

Git 最强大的特性之一就是它给了你"后悔药"。无论是误删文件、错误提交，还是需要回退到之前的版本，Git 都提供了多种撤销操作的方式。本章将系统性地介绍 Git 中所有的撤销操作，帮助你在遇到问题时能够从容应对。

作为一名开发者，在日常工作中难免会犯错。也许你不小心删除了一个重要的文件，也许你提交了包含错误代码的版本，也许你需要回退到之前的某个稳定版本。在没有版本控制系统的时代，这些操作可能会让你花费大量时间来修复，甚至可能导致无法挽回的损失。但是有了 Git，这些操作都变得简单而安全。

本章将从最基础的撤销操作开始，逐步深入到更复杂的场景，帮助你全面掌握 Git 的撤销功能。无论你是 Git 新手还是有一定经验的开发者，都能在这里找到你需要的知识。

---

## 目录

- [撤销工作区修改](#撤销工作区修改)
- [撤销暂存](#撤销暂存)
- [撤销最后一次提交](#撤销最后一次提交)
- [git reset 三种模式详解与对比](#git-reset-三种模式详解与对比)
- [git revert vs git reset 的区别与使用场景](#git-revert-vs-git-reset-的区别与使用场景)
- [修改历史提交](#修改历史提交)
- [恢复已删除的分支](#恢复已删除的分支)
- [恢复已删除的文件](#恢复已删除的文件)
- [git clean 清理未跟踪文件](#git-clean-清理未跟踪文件)
- [撤销已推送的提交](#撤销已推送的提交)
- [git restore 多种用法](#git-restore-多种用法)
- [安全操作原则](#安全操作原则)
- [危险操作警告与安全网](#危险操作警告与安全网)
- [撤销操作决策树](#撤销操作决策树)
- [常见撤销场景实战](#常见撤销场景实战)

---

## 撤销工作区修改

### 场景说明

当你修改了工作区的文件，但还没有执行 `git add`，发现改错了，想要丢弃这些修改，恢复到上次提交时的状态。

### 使用 git restore（推荐，Git 2.23+）

```bash
# 撤销单个文件的修改
git restore filename.js

# 撤销多个文件的修改
git restore file1.js file2.js file3.js

# 撤销所有已修改文件的修改（危险！）
git restore .
```

**命令详解：** `git restore` 命令是 Git 2.23 版本引入的新命令，专门用于恢复工作区的文件。它的语义更加清晰明确，相比于旧的 `git checkout` 命令，更容易理解和使用。当你执行 `git restore filename.js` 时，Git 会从暂存区（如果文件已暂存）或最近一次提交中读取文件的原始内容，并将其覆盖到工作区中对应的位置。

### 使用 git checkout（旧版语法）

```bash
# 撤销单个文件的修改
git checkout -- filename.js

# 撤销所有修改（危险！）
git checkout -- .
```

### 实际操作演示

```bash
# 1. 查看当前状态
git status
# 输出：modified: app.js

# 2. 查看修改内容
git diff app.js

# 3. 确认要撤销后执行
git restore app.js

# 4. 再次查看状态
git status
# 输出：nothing to commit, working tree clean
```

> **⚠️ 警告：** 此操作会**永久丢弃**工作区中未提交的修改，无法恢复。请在执行前确认是否需要备份。

### 撤销指定目录的修改

```bash
# 撤销 src 目录下所有修改
git restore src/

# 撤销所有 .js 文件的修改
git restore '*.js'
```

---

## 撤销暂存

### 场景说明

当你执行了 `git add` 将文件添加到暂存区，但又不想在下次提交中包含该文件，需要将其从暂存区移除（但保留工作区的修改）。这种情况在实际开发中非常常见，比如你可能不小心添加了错误的文件，或者想要将一个大的提交拆分成多个小的提交。理解如何正确地撤销暂存操作，对于保持清晰的提交历史非常重要。

### 使用 git restore --staged（推荐，Git 2.23+）

```bash
# 将单个文件从暂存区移除
git restore --staged filename.js

# 将多个文件从暂存区移除
git restore --staged file1.js file2.js

# 将所有文件从暂存区移除
git restore --staged .
```

### 使用 git reset（旧版语法）

```bash
# 将单个文件从暂存区移除
git reset HEAD filename.js

# 将所有文件从暂存区移除
git reset HEAD
```

### 实际操作演示

```bash
# 1. 假设你不小心 add 了错误的文件
git add secret-keys.txt

# 2. 查看状态
git status
# 输出：Changes to be committed: new file: secret-keys.txt

# 3. 将文件从暂存区移除
git restore --staged secret-keys.txt

# 4. 再次查看状态
git status
# 输出：Untracked files: secret-keys.txt
# 文件回到了未跟踪状态，修改仍然保留
```

### 从暂存区移除后保留文件

如果你想把文件从暂存区移除，同时在工作区也删除该文件：

```bash
# 使用 git rm --cached 仅从暂存区移除（保留工作区文件）
git rm --cached filename.js

# 使用 git rm 从暂存区和工作区都删除（危险！）
git rm filename.js
```

---

## 撤销最后一次提交

### 场景说明

你刚刚执行了 `git commit`，但发现提交信息写错了、或者遗漏了文件、或者提交了不该提交的内容，需要撤销这次提交。

### 方法一：git commit --amend（修改最近一次提交）

```bash
# 修改提交信息
git commit --amend -m "正确的提交信息"

# 添加遗漏文件并修改提交（保留原提交信息）
git add forgotten-file.js
git commit --amend --no-edit

# 添加遗漏文件并修改提交信息
git add forgotten-file.js
git commit --amend -m "更新后的提交信息"
```

### 方法二：git reset 回退提交

```bash
# 软回退：撤销提交，保留修改在暂存区
git reset --soft HEAD~1

# 混合回退：撤销提交，保留修改在工作区（默认模式）
git reset HEAD~1

# 硬回退：撤销提交，丢弃所有修改（危险！）
git reset --hard HEAD~1
```

### 方法三：git revert 创建撤销提交

```bash
# 创建一个新的提交来撤销上次提交的更改
git revert HEAD

# 创建撤销提交但不自动提交（仅放入暂存区）
git revert --no-commit HEAD
```

---

## git reset 三种模式详解与对比

`git reset` 是 Git 中最强大也最常用的撤销命令之一。它有三种模式：`--soft`、`--mixed`（默认）和 `--hard`。理解这三种模式的区别对于正确使用 Git 至关重要。很多 Git 初学者在使用撤销命令时感到困惑，主要原因就是没有真正理解这三种模式的工作原理和适用场景。

### 三种模式的工作原理

在深入了解三种模式之前，首先需要理解 Git 的三个核心区域：工作区、暂存区和仓库。工作区是你实际编辑代码的地方，暂存区是你准备提交的内容的临时存储区域，仓库则是 Git 保存所有提交历史的地方。`git reset` 命令的作用就是移动 HEAD 指针，并根据不同的模式决定是否修改暂存区和工作区。

```
工作区 (Working Directory)  ←→  暂存区 (Staging Area)  ←→  仓库 (Repository)
     你的代码文件               git add 后的快照            git commit 后的记录
```

| 模式 | 工作区 | 暂存区 | 仓库 | 用途 |
|------|--------|--------|------|------|
| `--soft` | ✅ 保留 | ✅ 保留 | ↩️ 回退 | 修改提交信息、合并多个提交 |
| `--mixed` | ✅ 保留 | ❌ 清除 | ↩️ 回退 | 重新选择要提交的文件 |
| `--hard` | ❌ 清除 | ❌ 清除 | ↩️ 回退 | 完全丢弃修改 |

### --soft 模式详解

**作用：** 仅回退提交，保留暂存区和工作区的所有修改。

```bash
git reset --soft HEAD~1
```

**使用场景：**
- 修改最近一次提交的内容
- 将多个提交合并为一个
- 提交后发现需要做小调整

```bash
# 示例：合并最近三次提交为一个
git reset --soft HEAD~3
git commit -m "合并后的提交信息"

# 示例：提交后添加遗漏文件
git commit -m "添加功能A"
git add forgotten-file.js
git reset --soft HEAD~1
git commit -m "添加功能A（完整版）"
```

### --mixed 模式详解（默认模式）

**作用：** 回退提交和暂存区，但保留工作区的修改。

```bash
git reset --mixed HEAD~1
# 等同于
git reset HEAD~1
```

**使用场景：**
- 需要重新组织提交内容
- 暂存了错误的文件，想要重新选择
- 将一个大提交拆分为多个小提交

```bash
# 示例：将一个提交拆分为多个小提交
git reset HEAD~1
# 现在所有修改都在工作区
git add feature-a.js
git commit -m "添加功能A"
git add feature-b.js
git commit -m "添加功能B"
```

### --hard 模式详解

**作用：** 回退提交、暂存区和工作区，完全丢弃所有修改。

```bash
git reset --hard HEAD~1
```

**使用场景：**
- 完全丢弃最近的修改，回到之前的状态
- 本地分支需要强制回退到某个版本

```bash
# 示例：完全回退到上一个版本
git reset --hard HEAD~1

# 示例：回退到指定的提交
git reset --hard abc1234

# 示例：强制与远程分支同步
git fetch origin
git reset --hard origin/main
```

> **🚨 危险警告：** `--hard` 模式会**永久删除**未提交的修改！执行前请确保已经备份重要内容。

### 三种模式对比图

```
初始状态（三个提交后）：
  仓库: A ← B ← C (HEAD)
  暂存区: C 的内容
  工作区: C 的内容 + 可能的修改

执行 git reset --soft HEAD~1 后：
  仓库: A ← B (HEAD)
  暂存区: C 的内容（保留）
  工作区: C 的内容 + 可能的修改（保留）

执行 git reset --mixed HEAD~1 后：
  仓库: A ← B (HEAD)
  暂存区: B 的内容（清除 C 的暂存）
  工作区: C 的内容 + 可能的修改（保留）

执行 git reset --hard HEAD~1 后：
  仓库: A ← B (HEAD)
  暂存区: B 的内容（清除）
  工作区: B 的内容（清除所有修改）
```

---

## git revert vs git reset 的区别与使用场景

在 Git 中，`git revert` 和 `git reset` 都可以用来撤销提交，但它们的工作方式和适用场景有着本质的区别。理解这两个命令的差异，是成为 Git 高手的必经之路。很多团队协作中的问题，都是因为错误地选择了撤销命令导致的。

### 核心区别

`git reset` 的工作方式是移动 HEAD 指针，直接删除提交历史，这会改变项目的历史记录。而 `git revert` 则是通过创建一个新的提交来撤销指定提交的更改，它不会修改历史，而是在历史记录中添加一个新的"撤销"操作。这种差异决定了它们的使用场景完全不同。

| 特性 | git reset | git revert |
|------|-----------|------------|
| **工作方式** | 移动 HEAD 指针，删除提交 | 创建新提交来撤销更改 |
| **历史记录** | 修改历史（删除提交） | 保留历史（新增撤销提交） |
| **安全性** | 可能丢失工作 | 安全，不丢失任何提交 |
| **适用场景** | 本地未推送的提交 | 已推送到远程的提交 |
| **是否改变历史** | 是 | 否 |

### git revert 详解

`git revert` 通过创建一个新的提交来撤销指定提交的更改，不会修改历史记录。

```bash
# 撤销最近一次提交
git revert HEAD

# 撤销指定提交
git revert abc1234

# 撤销多个提交
git revert HEAD~3..HEAD

# 撤销合并提交
git revert -m 1 merge-commit-hash

# 创建撤销提交但不自动提交
git revert --no-commit HEAD
```

### 使用场景对比

#### 场景一：本地开发中发现错误

**推荐使用 git reset：**

```bash
# 刚提交，还没推送，发现代码有 bug
git reset --soft HEAD~1
# 修复 bug
git add .
git commit -m "修复 bug 后的提交"
```

#### 场景二：已推送的提交需要撤销

**必须使用 git revert：**

```bash
# 代码已经推送到远程，其他同事已经拉取
git revert HEAD
git push origin main

# 其他同事拉取后会看到一个"撤销"提交，历史记录保持清晰
```

#### 场景三：撤销中间某次提交

```bash
# 使用 revert（推荐，安全）
git revert abc1234

# 使用 reset（危险，会丢失后续提交）
git reset --hard abc1234
```

### revert 多个提交

```bash
# 逐个撤销（每个撤销创建一个提交）
git revert HEAD~3..HEAD

# 合并为一个撤销提交
git revert --no-commit HEAD~3..HEAD
git commit -m "撤销最近三次提交"
```

---

## 修改历史提交

### 使用 git commit --amend

`git commit --amend` 只能修改最近一次提交。这个命令在日常开发中使用频率非常高，主要用途包括：修改提交信息、添加遗漏的文件到上次提交、修改上次提交的内容。需要注意的是，`git commit --amend` 实际上是创建了一个新的提交来替换原来的提交，而不是真正地"修改"原来的提交。因此，如果已经推送到远程仓库，使用 `--amend` 后需要强制推送。

```bash
# 修改提交信息
git commit --amend -m "新的提交信息"

# 不修改提交信息，只添加文件
git add new-file.js
git commit --amend --no-edit

# 交互式修改（打开编辑器）
git commit --amend
```

### 使用 git rebase -i 修改历史提交

交互式变基可以修改任意历史提交，功能非常强大。它是 Git 中最灵活的提交修改工具，可以实现提交重排序、合并多个提交、修改提交信息、删除不需要的提交等操作。虽然交互式变基的学习曲线相对陡峭，但一旦掌握，它将成为你日常开发中不可或缺的工具。需要注意的是，交互式变基会修改提交历史，因此只适用于尚未推送到远程仓库的本地提交。

```bash
# 修改最近 3 次提交
git rebase -i HEAD~3
```

执行后会打开编辑器，显示类似内容：

```
pick abc1234 第一次提交
pick def5678 第二次提交
pick ghi9012 第三次提交

# Rebase aaa1111..ghi9012 onto bbb2222 (3 commands)
#
# Commands:
# p, pick = use commit
# r, reword = use commit, but edit the commit message
# e, edit = use commit, but stop for amending
# s, squash = use commit, but meld into previous commit
# f, fixup = like "squash", but discard this commit's log message
# d, drop = remove commit
```

#### 常用操作示例

**修改提交信息（reword）：**

```
reword abc1234 第一次提交
pick def5678 第二次提交
pick ghi9012 第三次提交
```

**合并多个提交（squash）：**

```
pick abc1234 第一次提交
squash def5678 第二次提交
squash ghi9012 第三次提交
```

**删除某次提交（drop）：**

```
pick abc1234 第一次提交
drop def5678 不想要的提交
pick ghi9012 第三次提交
```

**修改某次提交的内容（edit）：**

```
pick abc1234 第一次提交
edit def5678 需要修改的提交
pick ghi9012 第三次提交
```

当 rebase 停在 `edit` 标记的提交时：

```bash
# 修改文件
vim need-fix.js

# 添加修改
git add need-fix.js

# 继续 rebase
git rebase --continue
```

> **⚠️ 注意：** 交互式变基会修改历史，只适用于尚未推送到远程的提交。

---

## 恢复已删除的分支

### 场景说明

你不小心删除了一个重要的分支，需要恢复它。这种情况在实际工作中时有发生，特别是当开发者使用 `git branch -D` 命令强制删除分支时。幸运的是，Git 提供了 `reflog` 机制来帮助我们恢复已删除的分支。只要分支被删除的时间不是太久（默认 90 天内），我们都可以通过 reflog 找到分支最后的提交记录，然后重新创建分支。

### 使用 git reflog 查找并恢复

`git reflog` 是 Git 中一个非常强大的工具，它记录了 HEAD 指针的所有移动历史，包括分支创建、切换、提交、重置等操作。即使分支被删除，reflog 中仍然保留着该分支最后指向的提交记录。通过这些记录，我们可以找到分支的最后状态并恢复它。

```bash
# 1. 查看所有操作历史（包括已删除分支的操作）
git reflog

# 输出示例：
# abc1234 HEAD@{0}: checkout: moving from feature-branch to main
# def5678 HEAD@{1}: commit: 添加新功能
# ghi9012 HEAD@{2}: branch: Created from HEAD

# 2. 找到分支最后的提交
# 从输出中找到 feature-branch 最后的 commit hash

# 3. 恢复分支
git checkout -b feature-branch def5678
# 或者使用 git switch
git switch -c feature-branch def5678
```

### 完整操作流程

```bash
# 1. 假设不小心删除了分支
git branch -D important-feature
# 输出：Deleted branch important-feature (was def5678).

# 2. 立即查看 reflog
git reflog --all | grep important-feature

# 3. 找到分支的提交记录
git reflog
# 查找类似这样的记录：
# def5678 HEAD@{5}: commit: feat: 添加重要功能

# 4. 恢复分支
git checkout -b important-feature def5678

# 5. 验证恢复成功
git log --oneline -5
```

### reflog 的有效期

- 默认保留 90 天（可通过配置修改）
- 已过期的 reflog 记录会被垃圾回收清理
- 重要分支删除后应尽快恢复

```bash
# 查看 reflog 过期时间配置
git config gc.reflogExpire
git config gc.reflogExpireUnreachable

# 设置更长的保留时间
git config gc.reflogExpire "180 days"
git config gc.reflogExpireUnreachable "90 days"
```

---

## 恢复已删除的文件

在软件开发过程中，文件被删除是常见的问题。可能是你不小心执行了 `git rm` 命令删除了重要文件，也可能是手动删除了文件后才意识到需要恢复。Git 提供了多种方式来恢复已删除的文件，具体使用哪种方式取决于文件是如何被删除的，以及删除操作是否已经提交到仓库中。掌握这些恢复方法可以帮助你在遇到文件丢失时快速找回重要内容，避免不必要的损失。

### 方法一：使用 git checkout 恢复

```bash
# 恢复单个文件到最近一次提交的状态
git checkout HEAD -- filename.js

# 恢复文件到指定提交的状态
git checkout abc1234 -- filename.js

# 恢复整个目录
git checkout HEAD -- src/
```

### 方法二：使用 git restore 恢复

```bash
# 恢复单个文件
git restore filename.js

# 恢复文件到指定提交
git restore --source=abc1234 filename.js

# 恢复整个目录
git restore src/
```

### 方法三：使用 git show 查看并恢复

```bash
# 查看某次提交中的文件内容
git show HEAD:filename.js
git show abc1234:src/app.js

# 将文件内容重定向到文件
git show HEAD:filename.js > filename.js
```

### 方法四：恢复已提交后被删除的文件

```bash
# 1. 查找文件被删除的提交
git log --diff-filter=D --summary | grep filename.js

# 2. 找到删除前的提交
git log --all --full-history -- filename.js

# 3. 从删除前的提交恢复文件
git checkout abc1234^ -- filename.js
# abc1234^ 表示 abc1234 的父提交（即删除操作之前的提交）
```

### 恢复多个文件

```bash
# 恢复所有已删除的文件
git ls-files -d | xargs git checkout HEAD --

# 恢复特定类型文件
git ls-files -d '*.js' | xargs git checkout HEAD --
```

---

## git clean 清理未跟踪文件

### 场景说明

在日常开发中，工作区中经常会积累很多未跟踪的文件，比如编译产生的临时文件、日志文件、编辑器生成的备份文件、操作系统的隐藏文件等。这些文件虽然不影响代码功能，但会让工作区变得混乱，也可能干扰 `git status` 的输出。`git clean` 命令就是专门用来清理这些未跟踪文件的工具。使用时需要特别注意，因为被 `git clean` 删除的文件无法通过 Git 恢复，因为这些文件从未被 Git 跟踪过。

### 基本用法

```bash
# 预览将要删除的文件（不实际删除）
git clean -n

# 删除未跟踪的文件
git clean -f

# 删除未跟踪的文件和目录
git clean -fd

# 删除未跟踪和忽略的文件（更彻底）
git clean -fdx
```

### 常用选项

| 选项 | 说明 |
|------|------|
| `-n` | 预览模式，只显示将要删除的文件 |
| `-f` | 强制删除文件 |
| `-d` | 同时删除未跟踪的目录 |
| `-x` | 同时删除被 .gitignore 忽略的文件 |
| `-X` | 只删除被 .gitignore 忽略的文件 |
| `-i` | 交互模式，逐个确认 |

### 安全操作流程

```bash
# 1. 先预览将要删除的内容
git clean -n
# 输出：Would remove temp-file.txt
# 输出：Would remove build/

# 2. 确认无误后执行删除
git clean -f

# 3. 如果需要删除目录
git clean -fd

# 4. 使用交互模式（最安全）
git clean -i
# 会逐个询问每个文件是否删除
```

> **⚠️ 警告：** `git clean` 删除的文件**无法恢复**，因为它们从未被 Git 跟踪。执行前请务必使用 `-n` 选项预览。

### .gitignore 配合使用

```bash
# 清理构建产物（如果在 .gitignore 中配置了）
git clean -fdX

# 只清理 node_modules
git clean -fdX node_modules/

# 清理所有未跟踪文件，但保留 .env 等配置文件
# 先在 .gitignore 中添加需要保留的文件
echo ".env" >> .gitignore
git clean -fdx
```

---

## 撤销已推送的提交

### 安全方法：git revert（推荐）

当代码已经推送到远程仓库后，撤销操作就需要特别谨慎了。因为其他开发者可能已经基于你的提交进行了开发，如果你直接修改历史记录（比如使用 `git reset` + `git push --force`），会导致其他人的工作出现问题。在这种情况下，最安全的做法是使用 `git revert` 命令创建一个新的"撤销"提交，这样既达到了撤销更改的目的，又保留了完整的历史记录，不会影响其他开发者的工作流程。

```bash
# 1. 创建撤销提交
git revert HEAD

# 2. 推送撤销提交
git push origin main

# 撤销指定提交
git revert abc1234
git push origin main

# 撤销多个提交
git revert HEAD~3..HEAD
git push origin main
```

### 不安全方法：git reset + force push（慎用）

```bash
# 1. 回退本地提交
git reset --hard HEAD~1

# 2. 强制推送到远程（危险！）
git push --force origin main
# 或使用更安全的选项
git push --force-with-lease origin main
```

### force-with-lease vs force

```bash
# --force：无条件强制推送，可能覆盖他人的提交
git push --force origin main

# --force-with-lease：只有在远程分支没有新提交时才推送
# 如果其他人已经推送了新提交，会拒绝推送
git push --force-with-lease origin main
```

> **🚨 严重警告：** 对公共分支使用 `git push --force` 可能导致其他开发者的工作丢失。只有在你完全确定后果时才使用。

### 最佳实践

1. **未推送的提交**：使用 `git reset` 或 `git commit --amend`
2. **已推送的提交**：使用 `git revert` 创建撤销提交
3. **个人分支**：可以使用 `git reset` + `git push --force-with-lease`
4. **公共分支**：必须使用 `git revert`

---

## git restore 多种用法

`git restore` 是 Git 2.23 引入的新命令，用于恢复工作区和暂存区的文件。这个命令的引入是为了解决 `git checkout` 命令职责过多的问题。在旧版本的 Git 中，`git checkout` 既要负责切换分支，又要负责恢复文件，这导致命令的语义不够清晰。`git restore` 专注于文件恢复操作，语义更加明确，使用起来也更加直观。对于 Git 新手来说，建议优先使用 `git restore` 命令，它更容易理解和记忆。

### 恢复工作区文件

```bash
# 恢复工作区文件到暂存区的状态
git restore filename.js

# 恢复工作区文件到指定提交的状态
git restore --source=HEAD~2 filename.js
git restore --source=abc1234 filename.js

# 恢复整个目录
git restore src/

# 恢复所有文件
git restore .
```

### 恢复暂存区文件

```bash
# 将暂存区的文件恢复到 HEAD 的状态（取消暂存）
git restore --staged filename.js

# 将暂存区的文件恢复到指定提交的状态
git restore --staged --source=HEAD~2 filename.js

# 取消所有文件的暂存
git restore --staged .
```

### 同时恢复工作区和暂存区

```bash
# 将工作区和暂存区都恢复到 HEAD 的状态
git restore --staged --worktree filename.js

# 恢复所有文件
git restore --staged --worktree .
```

### 常用参数组合

```bash
# 从指定分支恢复文件
git restore --source=feature-branch filename.js

# 从指定提交恢复文件
git restore --source=abc1234 filename.js

# 恢复并创建工作区不存在的文件
git restore --source=HEAD filename.js

# 确认模式（显示将要执行的操作）
git restore --dry-run filename.js
```

### git restore vs git checkout

| 功能 | git restore | git checkout |
|------|-------------|--------------|
| 恢复工作区文件 | `git restore file` | `git checkout -- file` |
| 恢复暂存区 | `git restore --staged file` | `git reset HEAD file` |
| 从指定提交恢复 | `git restore --source=commit file` | `git checkout commit -- file` |
| 切换分支 | ❌ | `git checkout branch` |

`git restore` 更专注于文件恢复操作，语义更清晰，推荐使用。

---

## 安全操作原则

在使用 Git 撤销操作时，遵循一些安全原则可以避免很多不必要的麻烦。这些原则是无数开发者在实际工作中总结出来的经验教训，掌握它们可以让你在使用 Git 时更加从容自信。记住，预防总是比补救更容易，养成良好的操作习惯可以大大减少误操作带来的损失。

### 原则一：操作前先备份

```bash
# 创建备份分支
git branch backup-before-reset

# 然后执行危险操作
git reset --hard HEAD~3

# 如果需要恢复，切换回备份分支
git checkout backup-before-reset
```

### 原则二：善用 git stash 保存临时工作

```bash
# 保存当前修改
git stash push -m "保存当前进度"

# 查看保存的列表
git stash list

# 恢复保存的修改
git stash pop

# 恢复但不删除 stash 记录
git stash apply stash@{0}
```

### 原则三：使用 reflog 作为安全网

```bash
# reflog 记录了 HEAD 的所有移动历史
git reflog

# 即使执行了 git reset --hard，也能通过 reflog 恢复
git reflog
# 找到 reset 之前的 commit hash
git reset --hard abc1234
```

### 原则四：先预览再执行

```bash
# 使用 --dry-run 预览
git clean --dry-run
git restore --dry-run filename.js

# 使用 diff 查看修改
git diff
git diff --staged

# 使用 status 查看状态
git status
```

### 原则五：小步提交，频繁提交

```bash
# 不要一次性提交大量修改
# 好的做法：小步提交
git add feature-a.js
git commit -m "feat: 实现功能A"

git add feature-b.js
git commit -m "feat: 实现功能B"

# 这样需要撤销时，影响范围更小
```

---

## 危险操作警告与安全网

在 Git 中，有些操作具有一定的风险性，可能会导致数据丢失或者影响团队协作。了解这些危险操作，并知道如何正确使用安全网机制来保护自己，是每个 Git 用户必须掌握的技能。下面我们将详细介绍 Git 中的高危操作，以及如何利用 Git 的内置安全机制来避免和恢复误操作。

### 高危操作清单

| 操作 | 危险等级 | 说明 |
|------|----------|------|
| `git reset --hard` | 🔴 高 | 丢弃所有未提交的修改 |
| `git push --force` | 🔴 高 | 可能覆盖远程他人提交 |
| `git clean -f` | 🟡 中 | 删除未跟踪文件，无法恢复 |
| `git branch -D` | 🟡 中 | 强制删除未合并的分支 |
| `git checkout -- .` | 🟡 中 | 丢弃所有工作区修改 |
| `git reset --mixed` | 🟢 低 | 仅清除暂存区，工作区保留 |

### 安全网机制

#### 1. reflog 是最后的救命稻草

```bash
# 几乎所有误操作都可以通过 reflog 恢复
git reflog

# 示例：误执行了 git reset --hard
git reset --hard HEAD~3
# 发现回退过多了

# 通过 reflog 找到原来的位置
git reflog
# 输出：
# abc1234 HEAD@{0}: reset: moving to HEAD~3
# def5678 HEAD@{1}: commit: 我需要的提交

# 恢复
git reset --hard def5678
```

#### 2. 暂存区是第二道防线

```bash
# 即使工作区的文件被修改，暂存区可能还有之前的版本
git restore --staged filename.js
```

#### 3. 备份分支永远是好习惯

```bash
# 在执行任何危险操作前创建备份
git branch backup-$(date +%Y%m%d-%H%M%S)

# 查看所有备份分支
git branch | grep backup
```

---

## 撤销操作决策树

当你需要撤销操作时，面对众多的 Git 命令可能会感到困惑。为了帮助你快速做出正确的决策，我们设计了一个详细的决策树。这个决策树覆盖了所有常见的撤销场景，通过回答几个简单的问题，就能找到最适合的命令。建议你将这个决策树保存下来，在遇到撤销需求时参考使用。随着使用经验的积累，你会逐渐形成自己的判断能力，不再需要依赖决策树。

当你需要撤销操作时，可以参考以下决策树：

```
                        需要撤销什么？
                            │
            ┌───────────────┼───────────────┐
            │               │               │
        工作区修改       暂存区内容        提交
            │               │               │
    ┌───────┴───────┐       │       ┌───────┴───────┐
    │               │       │       │               │
 未暂存         已暂存      │    已推送          未推送
    │               │       │       │               │
git restore     git restore  │   git revert    git reset
    │          --staged      │       │          (--soft/
    │               │       │       │           mixed/hard)
    │               │       │       │
    ▼               ▼       ▼       ▼
  丢弃修改      取消暂存   取消暂存  安全撤销     本地回退
                              │
                        ┌─────┴─────┐
                        │           │
                    已推送       未推送
                        │           │
                   git revert   git reset
                        │       或 amend
                        │
                        ▼
                   创建撤销提交
                   （保留历史）
```

### 按场景选择命令

在实际工作中，我们经常需要根据具体的场景来选择最合适的撤销命令。下面这个表格总结了所有常见场景及其对应的推荐命令，可以作为你的快速参考手册。记住，选择正确的命令不仅要考虑技术实现，还要考虑团队协作的影响。在公共分支上操作时，一定要选择不会破坏历史记录的安全命令。

```
场景                          推荐命令
─────────────────────────────────────────────────────
修改未暂存的文件               git restore <file>
已暂存想取消                   git restore --staged <file>
修改最后一次提交               git commit --amend
撤销本地提交                   git reset --soft HEAD~1
完全丢弃本地修改               git reset --hard HEAD~1
撤销已推送的提交               git revert HEAD
恢复删除的分支                 git reflog + git checkout -b
恢复删除的文件                 git checkout HEAD -- <file>
清理临时文件                   git clean -fd
修改历史提交                   git rebase -i
```

---

## 常见撤销场景实战

理论知识固然重要，但实际操作更能帮助我们理解和掌握 Git 撤销命令。在这一节中，我们将通过一系列真实的开发场景，演示如何正确使用各种撤销命令来解决实际问题。每个场景都包含了详细的问题描述、解决方案和操作步骤，帮助你在遇到类似问题时能够快速找到正确的处理方式。这些场景都是在日常开发中非常常见的，掌握它们将大大提高你的开发效率。

### 场景一：提交信息写错了

**情况：** 刚提交了代码，但提交信息有错别字或描述不准确。这是开发中最常见的小错误之一，虽然不影响代码功能，但会影响提交历史的可读性和团队协作体验。特别是在开源项目中，清晰准确的提交信息对于代码审查和问题追踪非常重要。使用 `git commit --amend` 命令可以轻松修复这个问题。

```bash
# 方法：修改最后一次提交信息
git commit --amend -m "正确的提交信息"

# 如果已经推送到远程
git commit --amend -m "正确的提交信息"
git push --force-with-lease origin main
```

### 场景二：提交后发现遗漏文件

**情况：** 提交后发现忘记添加某个文件。

```bash
# 方法一：使用 amend
git add forgotten-file.js
git commit --amend --no-edit

# 方法二：如果有多个遗漏文件
git add file1.js file2.js
git commit --amend -m "补充完整后的提交信息"
```

### 场景三：误提交了敏感信息

**情况：** 不小心提交了密码、API Key、数据库连接字符串等敏感信息。这是一个非常严重的问题，因为一旦敏感信息被提交到版本控制系统中，即使后续删除了文件，信息仍然会保留在提交历史中。在开源项目中，这可能导致安全漏洞和隐私泄露。发现这种情况后，应该立即采取行动，不仅要撤销提交，还要立即更换泄露的密钥和密码。

```bash
# 1. 如果还没推送
git reset --soft HEAD~1
# 从暂存区移除敏感文件
git restore --staged secret.env
# 添加到 .gitignore
echo "secret.env" >> .gitignore
git add .gitignore
git commit -m "移除敏感信息"

# 2. 如果已经推送（敏感信息已泄露，建议立即更换密钥）
git revert HEAD
git push origin main
# 然后立即更换泄露的密钥/密码
```

### 场景四：提交了错误的文件

**情况：** 把不应该提交的文件添加到了暂存区。

```bash
# 1. 从暂存区移除错误文件
git restore --staged wrong-file.js

# 2. 添加正确的文件
git add correct-file.js

# 3. 如果还没提交
git commit -m "正确的提交"

# 4. 如果已经提交但没推送
git reset --soft HEAD~1
git restore --staged wrong-file.js
git add correct-file.js
git commit -m "正确的提交"
```

### 场景五：需要完全重做最近的提交

**情况：** 最近一次提交的内容需要大幅修改，可能是因为发现了很多问题需要修复，或者想要重新组织代码结构。在这种情况下，使用 `git reset --soft HEAD~1` 命令可以回退提交但保留所有修改在暂存区，然后你可以自由地修改代码并重新提交。这种方法比创建多个修复提交更加整洁，可以让提交历史保持清晰。

```bash
# 1. 回退提交，保留修改在暂存区
git reset --soft HEAD~1

# 2. 进行修改
vim need-fix.js

# 3. 重新提交
git add .
git commit -m "重做后的提交"
```

### 场景六：需要回退到之前的版本

**情况：** 发现最近的几个提交都有问题，需要回退到更早的版本。

```bash
# 1. 查看提交历史
git log --oneline -10
# 输出：
# abc1234 (HEAD -> main) 最新提交
# def5678 有问题的提交
# ghi9012 有问题的提交
# jkl3456 这个版本是正常的

# 2. 本地回退（未推送）
git reset --hard jkl3456

# 3. 已经推送的情况（使用 revert）
git revert ghi9012
git revert def5678
git revert abc1234
git push origin main
```

### 场景七：误删了分支

**情况：** 不小心删除了一个重要的分支，可能是因为执行了 `git branch -D` 命令，或者在清理分支时误删了还在使用的分支。这种情况在团队协作中特别常见，特别是当多个开发者同时在处理多个功能分支时。幸运的是，只要分支被删除的时间不是太久，我们都可以通过 `git reflog` 命令找到分支最后指向的提交，然后重新创建分支。这个功能是 Git 的一大安全网，让开发者可以放心地管理分支而不必担心数据丢失。

```bash
# 1. 查看 reflog
git reflog
# 找到分支最后的提交，例如 def5678

# 2. 恢复分支
git checkout -b recovered-branch def5678

# 3. 验证
git log --oneline -5
```

### 场景八：误删了文件

**情况：** 执行了 `git rm` 或手动删除了文件。

```bash
# 1. 如果只是删除了工作区文件
git restore filename.js

# 2. 如果执行了 git rm（已暂存删除操作）
git restore --staged filename.js  # 取消暂存
git restore filename.js           # 恢复文件

# 3. 如果文件在之前的提交中被删除
git log --all --full-history -- filename.js
# 找到删除前的提交
git checkout abc1234^ -- filename.js
```

### 场景九：合并冲突后想放弃合并

**情况：** 合并分支时出现大量冲突，解决起来非常复杂，或者发现合并方向有误，想要放弃合并重新来过。在这种情况下，可以使用 `git merge --abort` 命令放弃正在进行的合并操作，将工作区恢复到合并前的状态。类似地，如果正在进行 rebase 或 cherry-pick 操作，也可以使用相应的 abort 命令来放弃操作。这些 abort 命令是 Git 提供的安全机制，让开发者可以在任何时候放弃正在进行的操作。

```bash
# 放弃正在进行的合并
git merge --abort

# 放弃正在进行的 rebase
git rebase --abort

# 放弃正在进行的 cherry-pick
git cherry-pick --abort
```

### 场景十：stash 的内容丢失了

**情况：** 不小心执行了 `git stash drop` 删除了某个 stash 记录，或者执行了 `git stash clear` 清空了所有 stash。`git stash` 是一个非常方便的功能，可以临时保存工作区的修改，让开发者能够快速切换到其他分支处理紧急任务。但是如果不小心删除了 stash 记录，保存的修改似乎就丢失了。实际上，Git 仍然保留着这些修改的提交对象，我们可以通过 `git fsck` 命令找到这些悬空的提交对象并恢复它们。

```bash
# 1. 查看 fsck 找到悬空的 stash 对象
git fsck --unreachable | grep commit

# 2. 查看找到的提交
git show <commit-hash>

# 3. 恢复 stash
git stash apply <commit-hash>
```

---

## 总结

通过本章的学习，我们系统性地掌握了 Git 中所有的撤销操作方法。从最基础的工作区修改撤销，到复杂的提交历史修改，再到各种安全恢复机制，这些知识将帮助你在日常开发中更加自信地使用 Git。记住，Git 的设计哲学是"安全第一"，只要正确使用这些撤销命令，几乎没有什么操作是不可逆的。在实际工作中，建议优先使用更安全的命令（如 `git restore` 和 `git revert`），只有在完全确定后果的情况下才使用高风险命令（如 `git reset --hard` 和 `git push --force`）。

### 核心要点

1. **理解工作区、暂存区、仓库的关系** 是正确使用撤销命令的基础
2. **未推送的提交** 可以安全地使用 `git reset` 修改
3. **已推送的提交** 应该使用 `git revert` 安全撤销
4. **reflog 是最后的安全网** 几乎可以恢复任何误操作
5. **操作前先备份** 养成创建备份分支的习惯

### 命令速查表

| 场景 | 命令 |
|------|------|
| 撤销工作区修改 | `git restore <file>` |
| 取消暂存 | `git restore --staged <file>` |
| 修改最近提交 | `git commit --amend` |
| 撤销本地提交 | `git reset --soft HEAD~1` |
| 完全丢弃修改 | `git reset --hard HEAD~1` |
| 安全撤销已推送提交 | `git revert HEAD` |
| 恢复删除的分支 | `git reflog` + `git checkout -b` |
| 恢复删除的文件 | `git restore <file>` |
| 清理未跟踪文件 | `git clean -fd` |
| 修改历史提交 | `git rebase -i` |

### 最后的建议

- **新手建议：** 优先使用 `git restore` 和 `git revert`，它们更安全，语义也更清晰
- **进阶用户：** 掌握 `git reset` 三种模式的区别，能够根据具体场景选择最合适的命令
- **团队协作：** 公共分支只用 `git revert`，避免使用 `git push --force` 影响其他开发者
- **养成习惯：** 操作前先执行 `git status` 和 `git diff`，确认当前工作区的状态
- **备份优先：** 在执行任何危险操作前，创建备份分支是一个非常好的习惯
- **善用 reflog：** `git reflog` 是最后的安全网，几乎可以恢复任何误操作
- **小步提交：** 频繁地进行小的提交，这样在需要撤销时影响范围更小
- **测试验证：** 在执行撤销操作后，务必验证代码是否正确，确保没有引入新的问题

---

## 下一步

[创建和管理仓库 →](14-create-repo.md)
