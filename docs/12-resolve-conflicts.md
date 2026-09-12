# 第十二章：解决 Git 合并冲突

> 本章将深入讲解 Git 合并冲突的产生原因、类型分类、解决方法以及预防策略，帮助开发者从容应对版本控制中的冲突问题。

---

## 目录

1. [什么是合并冲突](#什么是合并冲突)
2. [冲突的类型](#冲突的类型)
3. [手动解决冲突](#手动解决冲突)
4. [使用 VS Code 解决冲突](#使用-vs-code-解决冲突)
5. [使用命令行工具解决冲突](#使用命令行工具解决冲突)
6. [使用 git rerere 自动记录冲突解决方案](#使用-git-rerere-自动记录冲突解决方案)
7. [合并冲突 vs 变基冲突](#合并冲突-vs-变基冲突)
8. [复杂冲突场景](#复杂冲突场景)
9. [冲突预防策略](#冲突预防策略)
10. [放弃合并/变基操作](#放弃合并变基操作)
11. [Git 冲突解决工具对比](#git-冲突解决工具对比)
12. [团队冲突解决流程规范](#团队冲突解决流程规范)
13. [常见冲突问题排查](#常见冲突问题排查)

---

## 什么是合并冲突

### 冲突产生的根本原因

Git 合并冲突（Merge Conflict）是指当 Git 在合并两个分支时，无法自动确定应该如何合并某些更改，需要开发者手动介入解决的情况。

冲突产生的核心条件：

```
时间线示意：

分支 A (main):     a --- b --- c --- f (你修改了 file.txt)
                                    \
分支 B (feature):   x --- y --- z --- e (他人也修改了 file.txt 的同一区域)
```

当满足以下条件时，Git 无法自动合并：

1. **两个分支修改了同一文件的同一区域**
2. **一个分支删除了文件，另一个分支修改了该文件**
3. **两个分支添加了同名但内容不同的文件**

### Git 合并的底层原理

Git 使用三路合并（Three-way Merge）算法来合并分支。三路合并需要三个版本：

```
           ┌─────────────┐
           │  共同祖先 (Base)  │
           │  merge-base  │
           └───────┬─────┘
                   │
          ┌────────┴────────┐
          │                 │
   ┌──────▼──────┐   ┌─────▼───────┐
   │  当前分支 (Ours) │   │  目标分支 (Theirs) │
   │    HEAD      │   │   branch    │
   └─────────────┘   └─────────────┘
```

Git 通过对比三个版本的内容来决定如何合并：

- 如果只有 Ours 修改了某行 → 采用 Ours 的修改
- 如果只有 Theirs 修改了某行 → 采用 Theirs 的修改
- 如果两者都修改了同一行 → **产生冲突，需要手动解决**

### 冲突标记详解

当冲突发生时，Git 会在冲突文件中插入特殊的冲突标记，标识出冲突的区域：

```plaintext
这是一个正常的行，没有冲突。

<<<<<<< HEAD
这是当前分支（你的）的修改内容。
你可以在这里看到 HEAD 指向的分支所做的更改。
=======
这是要合并的分支（他们的）的修改内容。
你可以在这里看到另一个分支所做的更改。
>>>>>>> feature-branch

这是另一个正常的行，没有冲突。
```

**冲突标记说明：**

| 标记 | 含义 |
|------|------|
| `<<<<<<< HEAD` | 冲突区域开始，标记当前分支（HEAD）内容的起始位置 |
| `=======` | 分隔符，分隔当前分支和要合并分支的内容 |
| `>>>>>>> feature-branch` | 冲突区域结束，标记要合并分支内容的结束位置 |

**实际冲突示例：**

假设 `config.js` 文件在两个分支中被不同地修改：

```javascript
// main 分支的版本
const config = {
  port: 3000,
  host: 'localhost',
  debug: true
};
```

```javascript
// feature 分支的版本
const config = {
  port: 8080,
  host: '0.0.0.0',
  debug: false
};
```

合并后的冲突文件内容：

```javascript
const config = {
<<<<<<< HEAD
  port: 3000,
  host: 'localhost',
  debug: true
=======
  port: 8080,
  host: '0.0.0.0',
  debug: false
>>>>>>> feature-branch
};
```

---

## 冲突的类型

### 1. 内容冲突（Content Conflict）

最常见的冲突类型，两个分支修改了同一文件的同一区域。

```
场景示意：

main 分支:                 feature 分支:
┌──────────────┐           ┌──────────────┐
│ 第 10 行:     │           │ 第 10 行:     │
│ price = 100  │           │ price = 200  │
└──────────────┘           └──────────────┘
        │                          │
        └──────────┬───────────────┘
                   ▼
            ┌──────────────┐
            │   冲突!       │
            │ 需要手动决定   │
            │ 最终价格      │
            └──────────────┘
```

**示例：**

```bash
# 在 main 分支修改
echo "price = 100" > product.txt
git add product.txt && git commit -m "设置价格为 100"

# 切换到 feature 分支修改
git checkout feature
echo "price = 200" > product.txt
git add product.txt && git commit -m "设置价格为 200"

# 尝试合并
git checkout main
git merge feature
# 输出: CONFLICT (content): Merge conflict in product.txt
```

### 2. 删除冲突（Delete Conflict）

一个分支删除了文件，另一个分支修改了该文件。

```
场景示意：

main 分支:                 feature 分支:
┌──────────────┐           ┌──────────────┐
│ 删除 old.txt  │           │ 修改 old.txt  │
│   (rm)       │           │  (edit)      │
└──────────────┘           └──────────────┘
        │                          │
        └──────────┬───────────────┘
                   ▼
            ┌──────────────┐
            │   冲突!       │
            │ 删除还是保留？ │
            └──────────────┘
```

**示例：**

```bash
# main 分支删除文件
git rm deprecated.js
git commit -m "删除废弃文件"

# feature 分支修改同一文件
git checkout feature
echo "// 新功能代码" >> deprecated.js
git add deprecated.js
git commit -m "更新废弃文件"

# 合并时产生冲突
git checkout main
git merge feature
# 输出: CONFLICT (delete/modify): deprecated.js deleted in HEAD and modified in feature
```

### 3. 添加冲突（Add/Add Conflict）

两个分支分别添加了同名但内容不同的文件。

```
场景示意：

main 分支:                 feature 分支:
┌──────────────┐           ┌──────────────┐
│ 新建 utils.js │           │ 新建 utils.js │
│  内容: A      │           │  内容: B      │
└──────────────┘           └──────────────┘
        │                          │
        └──────────┬───────────────┘
                   ▼
            ┌──────────────┐
            │   冲突!       │
            │ 保留哪个版本？ │
            └──────────────┘
```

**示例：**

```bash
# main 分支创建文件
echo "export const mainUtil = () => {}" > utils.js
git add utils.js && git commit -m "添加工具函数"

# feature 分支创建同名文件
git checkout feature
echo "export const featureUtil = () => {}" > utils.js
git add utils.js && git commit -m "添加特性工具函数"

# 合并冲突
git checkout main
git merge feature
# 输出: CONFLICT (add/add): Merge conflict in utils.js
```

### 4. 重命名冲突（Rename Conflict）

涉及文件重命名操作时产生的冲突，包括以下子类型：

- **重命名/修改冲突**：一个分支重命名了文件，另一个分支修改了原文件
- **重命名/重命名冲突**：两个分支都将同一文件重命名为不同的名称
- **重命名/删除冲突**：一个分支重命名了文件，另一个分支删除了该文件

```
重命名/修改冲突示意：

main 分支:                 feature 分支:
┌──────────────┐           ┌──────────────┐
│ a.txt → b.txt │           │ 修改 a.txt    │
│  (rename)    │           │  (edit)      │
└──────────────┘           └──────────────┘
```

**示例：**

```bash
# main 分支重命名文件
git mv old-name.js new-name.js
git commit -m "重命名文件"

# feature 分支修改原文件
git checkout feature
echo "// 新代码" >> old-name.js
git add old-name.js
git commit -m "修改文件"

# 合并冲突
git checkout main
git merge feature
# 输出: CONFLICT (rename/modify): old-name.js renamed to new-name.js in HEAD and modified in feature
```

---

## 手动解决冲突

### 解决冲突的基本流程

手动解决冲突是最基础也是最灵活的方法。以下是完整的步骤：

```
冲突解决流程：

  ┌─────────────┐
  │  发生冲突    │
  └──────┬──────┘
         ▼
  ┌─────────────┐
  │ 查看冲突状态 │  git status
  └──────┬──────┘
         ▼
  ┌─────────────┐
  │ 打开冲突文件 │  使用编辑器查看
  └──────┬──────┘
         ▼
  ┌─────────────┐
  │ 编辑解决冲突 │  删除标记，保留正确内容
  └──────┬──────┘
         ▼
  ┌─────────────┐
  │ 暂存解决文件 │  git add <file>
  └──────┬──────┘
         ▼
  ┌─────────────┐
  │ 提交合并结果 │  git commit
  └──────┬──────┘
         ▼
  ┌─────────────┐
  │ 完成合并     │
  └─────────────┘
```

### 步骤一：查看冲突状态

```bash
git status
```

输出示例：

```
On branch main
You have unmerged paths.
  (fix conflicts and run "git commit")
  (use "git merge --abort" to abort the merge)

Unmerged paths:
  (use "git add <file>..." to mark resolution)
        both modified:   config.js
        both modified:   src/utils.js

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        new-file.js
```

### 步骤二：打开并编辑冲突文件

使用任意文本编辑器打开冲突文件，找到冲突标记：

```javascript
// config.js 文件内容
const database = {
<<<<<<< HEAD
  host: 'localhost',
  port: 3306,
  user: 'admin',
  password: 'secret123'
=======
  host: 'production-db.example.com',
  port: 5432,
  user: 'prod_user',
  password: 'secure_password_456'
>>>>>>> feature-branch
};
```

### 步骤三：选择保留内容

根据实际需求，有以下几种选择：

**选择一：保留当前分支的内容（HEAD）**

```javascript
const database = {
  host: 'localhost',
  port: 3306,
  user: 'admin',
  password: 'secret123'
};
```

**选择二：保留要合并分支的内容**

```javascript
const database = {
  host: 'production-db.example.com',
  port: 5432,
  user: 'prod_user',
  password: 'secure_password_456'
};
```

**选择三：合并两者的修改**

```javascript
const database = {
  host: 'production-db.example.com',
  port: 3306,
  user: 'prod_user',
  password: 'secure_password_456'
};
```

**选择四：完全重写**

```javascript
const database = {
  host: process.env.DB_HOST || 'localhost',
  port: parseInt(process.env.DB_PORT) || 3306,
  user: process.env.DB_USER || 'admin',
  password: process.env.DB_PASSWORD || 'secret'
};
```

### 步骤四：暂存并提交

```bash
# 标记冲突已解决
git add config.js

# 提交合并
git commit -m "解决 config.js 合并冲突"
```

### 完整实战示例

```bash
# 1. 创建实验环境
mkdir conflict-demo && cd conflict-demo
git init

# 2. 创建初始文件
echo "# 项目配置" > README.md
echo "version = 1.0" > config.txt
git add .
git commit -m "初始提交"

# 3. 创建 feature 分支并修改
git checkout -b feature
echo "version = 2.0" > config.txt
echo "new_feature = true" >> config.txt
git add config.txt
git commit -m "feature: 更新配置版本"

# 4. 回到 main 分支并修改
git checkout main
echo "version = 1.1" > config.txt
echo "hotfix = true" >> config.txt
git add config.txt
git commit -m "hotfix: 修复配置"

# 5. 尝试合并（会产生冲突）
git merge feature
# 输出：
# Auto-merging config.txt
# CONFLICT (content): Merge conflict in config.txt
# Automatic merge failed; fix conflicts and then commit the result.

# 6. 查看冲突
cat config.txt
# <<<<<<< HEAD
# version = 1.1
# hotfix = true
# =======
# version = 2.0
# new_feature = true
# >>>>>>> feature

# 7. 编辑解决冲突（手动修改文件）
echo "version = 2.0" > config.txt
echo "hotfix = true" >> config.txt
echo "new_feature = true" >> config.txt

# 8. 标记解决并提交
git add config.txt
git commit -m "合并 feature 分支，解决版本冲突"

# 9. 查看结果
git log --oneline --graph
```

---

## 使用 VS Code 解决冲突

VS Code 内置了强大的 Git 冲突解决工具，提供了可视化的操作界面。

### 冲突检测与提示

当打开包含冲突的文件时，VS Code 会自动检测并显示：

```
┌─────────────────────────────────────────────────────┐
│  config.js                              ×  ─  □    │
├─────────────────────────────────────────────────────┤
│                                                     │
│  const database = {                                 │
│  ┌─────────────────────────────────────────────┐   │
│  │ ⚠ Merge Conflict Detected                   │   │
│  │                                             │   │
│  │ Accept Current | Accept Incoming | Accept Both│   │
│  │                                             │   │
│  │ Compare Changes                             │   │
│  └─────────────────────────────────────────────┘   │
│                                                     │
│  <<<<<<< HEAD                                       │
│  │ host: 'localhost',         (绿色 - 当前更改)     │
│  │ port: 3306,                                     │
│  =======                                            │
│  │ host: 'production.example.com', (蓝色 - 传入更改) │
│  │ port: 5432,                                     │
│  >>>>>>> feature                                    │
│                                                     │
└─────────────────────────────────────────────────────┘
```

### VS Code 冲突解决按钮

VS Code 在冲突区域上方提供四个操作按钮：

| 按钮 | 功能 | 快捷键 |
|------|------|--------|
| **Accept Current Change** | 保留当前分支的修改 | `Ctrl+Shift+1` |
| **Accept Incoming Change** | 保留要合并分支的修改 | `Ctrl+Shift+2` |
| **Accept Both Changes** | 保留双方的修改 | `Ctrl+Shift+3` |
| **Compare Changes** | 对比查看双方的修改差异 | 无 |

### 使用 VS Code 解决冲突的步骤

**第一步：打开源代码管理面板**

```
┌─────────────────────────────────────────────┐
│  VS Code 左侧活动栏                          │
│                                             │
│  ┌───┐                                      │
│  │ 📁 │ 文件资源管理器                        │
│  ├───┤                                      │
│  │ 🔍 │ 搜索                                │
│  ├───┤                                      │
│  │ 🔀 │ 源代码管理  ← 点击这里                │
│  ├───┤                                      │
│  │ 🐛 │ 调试                                │
│  └───┘                                      │
└─────────────────────────────────────────────┘
```

**第二步：查看冲突文件列表**

```
┌─────────────────────────────────────────────┐
│  源代码管理                                  │
├─────────────────────────────────────────────┤
│  ▼ 合并更改 (3)                              │
│    ⚠ config.js          已修改(双方)         │
│    ⚠ src/utils.js       已修改(双方)         │
│    ⚠ src/index.js        已修改(双方)         │
│                                              │
│  ▼ 更改 (0)                                  │
└─────────────────────────────────────────────┘
```

**第三步：逐个解决冲突**

点击冲突文件，VS Code 会打开编辑器并高亮显示冲突区域：

```javascript
// 在编辑器中，冲突区域会有明显的颜色标识
const database = {
<<<<<<< HEAD                          // ← 冲突开始标记
  host: 'localhost',                  // ← 绿色背景（当前更改）
  port: 3306,
=======                               // ← 分隔符
  host: 'production.example.com',     // ← 蓝色背景（传入更改）
  port: 5432,
>>>>>>> feature-branch                // ← 冲突结束标记
};
```

**第四步：使用按钮快速解决**

点击上方的按钮选择解决方案：

- 点击 **Accept Current Change**：自动删除冲突标记，保留当前分支内容
- 点击 **Accept Incoming Change**：自动删除冲突标记，保留传入分支内容
- 点击 **Accept Both Changes**：自动删除冲突标记，保留双方内容

**第五步：手动微调（可选）**

如果选择了 "Accept Both Changes"，可能需要手动调整合并后的逻辑：

```javascript
const database = {
  host: 'production.example.com',  // 选择传入的主机
  port: 3306,                       // 保留当前的端口
  user: 'prod_user',               // 选择传入的用户
  password: 'secure_password'       // 选择传入的密码
};
```

### VS Code 的三路合并视图

VS Code 还提供三路合并视图，可以同时查看基础版本、当前版本和传入版本：

```
┌────────────────────────────────────────────────────────────┐
│  三路合并视图                                                │
├──────────────────┬──────────────────┬──────────────────────┤
│   BASE (基础)     │   CURRENT (当前)  │   INCOMING (传入)    │
├──────────────────┼──────────────────┼──────────────────────┤
│                  │                  │                      │
│ host: 'dev.db'   │ host: 'localhost'│ host: 'prod.db'      │
│ port: 3306       │ port: 3306       │ port: 5432           │
│ user: 'dev'      │ user: 'admin'    │ user: 'prod'         │
│                  │                  │                      │
└──────────────────┴──────────────────┴──────────────────────┘
```

### VS Code 扩展推荐

以下扩展可以增强 VS Code 的冲突解决能力：

| 扩展名称 | 功能描述 |
|---------|---------|
| **GitLens** | 增强 Git 功能，提供行内注释和历史查看 |
| **Merge Conflict** | 专门用于合并冲突的可视化工具 |
| **Git Graph** | 可视化 Git 分支和提交历史 |

---

## 使用命令行工具解决冲突

### git mergetool 简介

`git mergetool` 是 Git 内置的冲突解决工具启动器，它可以调用外部可视化合并工具来帮助解决冲突。

```bash
# 查看当前配置的合并工具
git config --global merge.tool

# 设置默认合并工具
git config --global merge.tool vimdiff

# 使用合并工具
git mergetool
```

### 常用命令行合并工具

#### vimdiff

Vim 的多窗口差异对比模式：

```
┌──────────────────────────────────────────────────┐
│  vimdiff 三路合并视图                              │
├───────────────┬───────────────┬──────────────────┤
│   LOCAL       │   BASE        │   REMOTE         │
│  (当前分支)    │  (共同祖先)    │  (要合并分支)     │
├───────────────┴───────────────┴──────────────────┤
│                                                  │
│  <<<<<<< HEAD                                    │
│  host: 'localhost',                              │
│  port: 3306,                                     │
│  =======                                         │
│  host: 'production.example.com',                 │
│  port: 5432,                                     │
│  >>>>>>> feature                                 │
│                                                  │
│   MERGED (合并结果)                               │
│                                                  │
└──────────────────────────────────────────────────┘
```

**vimdiff 常用快捷键：**

| 快捷键 | 功能 |
|--------|------|
| `]c` | 跳转到下一个差异 |
| `[c` | 跳转到上一个差异 |
| `do` | 获取对方的更改（diff obtain） |
| `dp` | 将当前更改推送到对方（diff put） |
| `:wqa` | 保存并退出所有窗口 |

#### 使用 vimdiff 解决冲突的步骤

```bash
# 1. 触发合并冲突
git merge feature-branch

# 2. 启动 vimdiff
git mergetool

# 3. 在 vimdiff 中：
#    - 使用 ]c 和 [c 导航差异
#    - 使用 do 获取远程更改
#    - 使用 dp 推送本地更改
#    - 直接编辑 MERGED 窗口

# 4. 保存退出
:wqa

# 5. Git 会自动标记冲突已解决
# 6. 提交合并
git commit -m "解决合并冲突"
```

#### 配置 vimdiff 为默认工具

```bash
# 设置 vimdiff 为合并工具
git config --global merge.tool vimdiff

# 设置 vimdiff 为差异对比工具
git config --global diff.tool vimdiff

# 可选：设置不自动启动工具（只在手动调用时启动）
git config --global mergetool.prompt false
```

### 其他命令行工具

#### opendiff (macOS)

macOS 自带的 FileMerge 工具：

```bash
# macOS 用户可以使用
git config --global merge.tool opendiff
git mergetool
```

#### Emacs Ediff

Emacs 用户可以使用 Ediff 模式：

```bash
git config --global merge.tool ediff
git mergetool
```

---

## 使用 git rerere 自动记录冲突解决方案

### 什么是 git rerere

`rerere` 全称 "Reuse Recorded Resolution"，是 Git 的一个高级功能，可以自动记录冲突的解决方案，当相同的冲突再次出现时自动应用之前的解决方案。

```
rerere 工作原理：

第一次遇到冲突：
┌─────────────────┐
│  冲突出现        │
└────────┬────────┘
         ▼
┌─────────────────┐
│  手动解决冲突    │
└────────┬────────┘
         ▼
┌─────────────────┐
│  rerere 记录     │  ← Git 自动保存解决方案
│  解决方案        │
└────────┬────────┘
         ▼
┌─────────────────┐
│  完成合并        │
└─────────────────┘

再次遇到相同冲突：
┌─────────────────┐
│  相同冲突出现    │
└────────┬────────┘
         ▼
┌─────────────────┐
│  rerere 自动应用 │  ← 自动使用之前的解决方案
│  已记录的方案    │
└────────┬────────┘
         ▼
┌─────────────────┐
│  冲突自动解决    │
└─────────────────┘
```

### 启用 git rerere

```bash
# 全局启用 rerere
git config --global rerere.enabled true

# 查看配置
git config --global --get rerere.enabled
```

### 使用示例

```bash
# 1. 启用 rerere
git config --global rerere.enabled true

# 2. 创建实验环境
mkdir rerere-demo && cd rerere-demo
git init

# 3. 创建初始文件
echo "line 1" > file.txt
echo "line 2" >> file.txt
echo "line 3" >> file.txt
git add file.txt && git commit -m "初始提交"

# 4. 创建分支 A 并修改
git checkout -b branch-a
sed -i 's/line 2/modified by branch A/' file.txt
git add file.txt && git commit -m "A: 修改第2行"

# 5. 回到 main 并创建分支 B
git checkout main
git checkout -b branch-b
sed -i 's/line 2/modified by branch B/' file.txt
git add file.txt && git commit -m "B: 修改第2行"

# 6. 合并分支 A（产生冲突）
git checkout main
git merge branch-a
# 手动解决冲突
echo "line 1" > file.txt
echo "resolved content" >> file.txt
echo "line 3" >> file.txt
git add file.txt
git commit -m "合并 A，解决冲突"

# 7. 查看 rerere 记录
git rerere status
# 输出类似：Resolved 'file.txt' using previous resolution.

# 8. 合并分支 B（相同冲突会自动解决）
git merge branch-b
# rerere 会自动应用之前的解决方案！
# 输出：Auto-merging file.txt
#       CONFLICT (content): Merge conflict in file.txt
#       Resolved 'file.txt' using previous resolution.

# 9. 检查结果
cat file.txt
# 内容应该已经自动解决
```

### 管理 rerere 记录

```bash
# 查看当前记录的冲突解决方案
git rerere status

# 查看 rerere 缓存的差异
git rerere diff

# 清除所有 rerere 记录
git rerere forget

# 清除特定文件的 rerere 记录
git rerere forget file.txt

# 手动 GC 清除过期的 rerere 记录
git gc
```

### rerere 的存储位置

rerere 的记录存储在 `.git/rr-cache/` 目录下：

```bash
# 查看 rerere 缓存目录
ls .git/rr-cache/

# 输出类似：
# 3f5a8b2c1d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a/
# a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0/

# 每个目录包含：
# - preimage: 冲突前的内容
# - postimage: 解决后的内容
```

### rerere 使用场景

1. **长期分支反复合并**：功能分支长期存在，需要定期从 main 分支合并更新
2. **变基操作**：变基时经常遇到重复冲突
3. **代码重构期间**：多个分支同时修改重构的代码

---

## 合并冲突 vs 变基冲突

### 合并冲突的特点

合并（Merge）创建一个新的合并提交，保留完整的分支历史：

```
合并前：
main:    A --- B --- C
                  \
feature:           D --- E

合并后：
main:    A --- B --- C ------- M (合并提交)
                  \           /
feature:           D --- E ---┘
```

**合并冲突的特点：**

- 冲突只发生一次，在最终合并时
- 历史记录保留完整分支结构
- 使用 `--abort` 可以轻松放弃

### 变基冲突的特点

变基（Rebase）将提交逐个应用到目标分支上，可能多次遇到冲突：

```
变基前：
main:    A --- B --- C
                  \
feature:           D --- E

变基后：
main:    A --- B --- C --- D' --- E'
```

**变基冲突的特点：**

- 每个被变基的提交都可能产生冲突
- 冲突可能需要多次解决
- 历史记录被改写（创建新提交）

### 变基冲突解决流程

```bash
# 1. 开始变基
git checkout feature
git rebase main

# 输出：
# Rebasing (1/3)
# Rebasing (2/3)
# CONFLICT (content): Merge conflict in config.js

# 2. 解决第一个冲突
# 编辑 config.js，解决冲突
git add config.js

# 3. 继续变基
git rebase --continue

# 4. 可能遇到更多冲突，重复步骤 2-3
# 解决第二个冲突
# 编辑 utils.js
git add utils.js
git rebase --continue

# 5. 所有冲突解决后，变基完成
```

### 变基冲突的特殊标记

变基冲突的标记与合并冲突略有不同：

```javascript
// 变基冲突标记
<<<<<<< HEAD
  // 当前分支（正在变基到的目标分支）的内容
  host: 'main-branch-value',
=======
  // 正在应用的提交的内容
  host: 'feature-branch-value',
>>>>>>> commit abc1234 (提交信息)
```

### 选择合并还是变基

```
决策树：

你是否需要保留完整的历史记录？
├── 是 → 使用合并（merge）
│         适用场景：团队协作、发布分支
│
└── 否 → 你是否想要线性的提交历史？
          ├── 是 → 使用变基（rebase）
          │         适用场景：个人分支、清理历史
          │
          └── 否 → 根据团队规范决定
```

**建议：**

| 场景 | 推荐方式 | 原因 |
|------|---------|------|
| 合并功能分支到 main | 合并 | 保留完整的功能开发历史 |
| 更新本地功能分支 | 变基 | 保持线性历史，便于后续合并 |
| 公共分支 | 合并 | 避免改写他人基于的提交 |
| 个人分支 | 变基 | 清理提交历史 |

---

## 复杂冲突场景

### 三方合并详解

三方合并（Three-way Merge）是 Git 解决冲突的核心算法。理解三方合并有助于更好地解决复杂冲突。

```
三方合并示意：

        Base (共同祖先)
        ┌───────────┐
        │ line 1    │
        │ line 2    │  ← 原始内容
        │ line 3    │
        └───────────┘
              │
     ┌────────┴────────┐
     ▼                 ▼
┌───────────┐    ┌───────────┐
│ Ours      │    │ Theirs    │
│ line 1    │    │ line 1    │
│ line 2-A  │    │ line 2-B  │  ← 两边都修改了 line 2
│ line 3    │    │ line 3    │
└───────────┘    └───────────┘

分析：
- line 1: 三者相同 → 自动采用，无冲突
- line 2: Base → Ours 改为 2-A, Base → Theirs 改为 2-B → 冲突！
- line 3: 三者相同 → 自动采用，无冲突
```

**实际案例：**

```bash
# 创建三方合并场景
mkdir three-way && cd three-way
git init

# 创建基础版本
cat > style.css << 'EOF'
.header {
  background-color: blue;
  font-size: 16px;
  margin: 10px;
}
EOF
git add style.css
git commit -m "基础样式"

# 分支 A 修改
git checkout -b branch-a
cat > style.css << 'EOF'
.header {
  background-color: red;
  font-size: 16px;
  margin: 10px;
}
EOF
git add style.css && git commit -m "A: 改为红色背景"

# 回到 main，分支 B 修改
git checkout main
git checkout -b branch-b
cat > style.css << 'EOF'
.header {
  background-color: green;
  font-size: 20px;
  margin: 10px;
}
EOF
git add style.css && git commit -m "B: 改为绿色背景和更大字体"

# 合并分支 A
git checkout main
git merge branch-a
git commit -m "合并分支 A"

# 合并分支 B（产生冲突）
git merge branch-b
# 冲突！需要手动决定最终的背景色和字体大小
```

### 多文件冲突处理

当多个文件同时出现冲突时，需要系统地逐个解决：

```bash
# 查看所有冲突文件
git status

# 输出：
# Unmerged paths:
#   both modified:   src/config.js
#   both modified:   src/utils.js
#   both modified:   src/index.js
#   both modified:   tests/test.js
#   both modified:   package.json
```

**处理策略：**

```bash
# 方法一：逐个解决
# 先解决简单/重要的文件
git add src/config.js    # 标记已解决
git add src/utils.js     # 标记已解决
# ... 继续处理其他文件

# 方法二：使用工具批量查看
git diff --name-only --diff-filter=U  # 列出所有未解决的冲突

# 方法三：按优先级排序处理
# 1. 配置文件（可能影响构建）
# 2. 核心业务代码
# 3. 测试文件
# 4. 其他文件
```

### 二进制文件冲突

二进制文件（图片、PDF、编译文件等）无法进行文本合并：

```bash
# 二进制文件冲突
git merge feature-branch
# 输出：
# CONFLICT (content): Merge conflict in images/logo.png
# Auto-merging images/logo.png
# warning: Cannot merge binary files: images/logo.png (HEAD vs feature-branch)

# 解决方案：选择保留哪个版本
git checkout --ours images/logo.png    # 保留当前分支版本
# 或
git checkout --theirs images/logo.png  # 保留要合并分支版本

# 标记已解决
git add images/logo.png
git commit -m "解决二进制文件冲突"
```

### 子模块冲突

当子模块（submodule）在不同分支指向不同提交时会产生冲突：

```bash
# 子模块冲突示例
git merge feature-branch
# 输出：
# CONFLICT (submodule): Merge conflict in lib/vendor-library
# Automatic merge failed; fix conflicts and then commit the result.

# 解决方案：
cd lib/vendor-library

# 查看两个分支指向的提交
git log --oneline HEAD  # 当前分支指向的提交
git log --oneline feature-branch  # 要合并分支指向的提交

# 选择要使用的版本
git checkout main  # 使用 main 分支的版本
# 或
git checkout feature-branch  # 使用 feature 分支的版本
# 或
git checkout <specific-commit>  # 使用特定提交

cd ../..
git add lib/vendor-library
git commit -m "解决子模块冲突"
```

### 空白符冲突

有时 Git 会因为换行符或空白字符差异产生冲突：

```bash
# 配置 Git 忽略空白符差异
git merge -Xignore-all-space feature-branch

# 或者只忽略行尾空白
git merge -Xignore-space-at-eol feature-branch

# 全局配置
git config --global core.autocrlf true  # Windows
git config --global core.autocrlf input # macOS/Linux
```

---

## 冲突预防策略

### 1. 频繁合并（Sync Frequently）

定期将主分支的更改合并到功能分支，减少累积差异：

```
不良做法（长期不合并）：

main:    A --- B --- C --- D --- E --- F --- G
                  \
feature:           X --- Y --- Z (差异太大，冲突多)

推荐做法（频繁合并）：

main:    A --- B --- C --- D --- E --- F --- G
                  \       \       \
feature:           X --- Y-+--- Z-+--- W (定期同步，冲突少)
```

```bash
# 推荐的工作流程
git checkout feature-branch

# 每天或每周从 main 合并更新
git fetch origin
git merge origin/main

# 或者使用变基
git rebase origin/main

# 解决小冲突，保持分支与主分支同步
```

### 2. 小步提交（Small, Frequent Commitions）

保持提交小而专注，减少冲突范围：

```
不良做法（大提交）：

commit: "重构整个用户模块并添加新功能"
- 修改 50 个文件
- 涉及多个功能
- 冲突范围大，难以解决

推荐做法（小提交）：

commit 1: "重构用户模型结构"
commit 2: "添加用户验证方法"
commit 3: "更新用户控制器"
commit 4: "添加用户相关测试"
- 每个提交修改 3-5 个文件
- 职责单一，冲突范围小
```

### 3. 模块化设计

将代码组织成独立的模块，减少文件级冲突：

```
不良结构（所有代码在一个文件）：
src/
  app.js  ← 5000 行，多人同时修改容易冲突

推荐结构（模块化）：
src/
  user/
    userModel.js
    userController.js
    userService.js
  product/
    productModel.js
    productController.js
    productService.js
  shared/
    utils.js
    constants.js
```

### 4. 沟通协调

团队成员之间的沟通是预防冲突的关键：

```
沟通策略：

┌─────────────────────────────────────────────────┐
│  每日站会（Daily Standup）                       │
├─────────────────────────────────────────────────┤
│  - 今天要修改哪些文件？                          │
│  - 有没有其他人也在修改相同的文件？               │
│  - 如何协调避免冲突？                            │
└─────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────┐
│  代码所有权（Code Ownership）                    │
├─────────────────────────────────────────────────┤
│  - 指定文件/模块的主要负责人                     │
│  - 修改前通知负责人                              │
│  - 使用 CODEOWNERS 文件自动分配审查者            │
└─────────────────────────────────────────────────┘
```

**CODEOWNERS 文件示例：**

```plaintext
# .github/CODEOWNERS

# 用户模块
/src/user/     @zhangsan @lisi

# 支付模块
/src/payment/  @wangwu

# 配置文件
*.json         @team-lead
*.yml          @team-lead

# 文档
/docs/         @technical-writer
```

### 5. 使用功能开关（Feature Flags）

使用功能开关隔离未完成的代码：

```javascript
// 使用功能开关
function renderNewUI() {
  if (featureFlags.isEnabled('new-user-interface')) {
    return newUIComponent();
  }
  return oldUIComponent();
}
```

### 6. 文件锁定策略

对于二进制文件或容易冲突的配置文件，可以使用文件锁定：

```bash
# 使用 Git LFS 锁定文件
git lfs lock images/design.psd

# 解锁
git lfs unlock images/design.psd
```

---

## 放弃合并/变基操作

### 放弃合并（Merge Abort）

当合并冲突过于复杂，或者你决定放弃合并时：

```bash
# 放弃当前合并操作
git merge --abort

# 这会：
# 1. 恢复到合并前的状态
# 2. 清除所有冲突标记
# 3. 丢弃所有未提交的合并更改
```

**使用场景：**

```bash
# 场景1：冲突太多，需要重新规划
git merge feature-branch
# 发现 20+ 个文件冲突...
git merge --abort
# 重新思考合并策略

# 场景2：错误地合并了错误的分支
git merge wrong-branch
# 发现合并错了...
git merge --abort

# 场景3：需要先做一些准备工作
git merge feature-branch
# 需要先通知团队成员...
git merge --abort
```

### 放弃变基（Rebase Abort）

变基过程中如果遇到问题：

```bash
# 放弃当前变基操作
git rebase --abort

# 这会：
# 1. 恢复到变基前的状态
# 2. 丢弃所有已解决的冲突
# 3. 恢复原始的提交历史
```

**变基中断状态：**

```bash
# 当你在变基过程中，Git 会显示
git status
# 输出：
# interactive rebase in progress; onto abc1234
# Last commands done (3 commands done):
#    pick def5678 Add feature A
#    pick ghi9012 Add feature B
#    pick jkl3456 Add feature C (冲突)
# Next commands to do (2 remaining):
#    pick mno7890 Add feature D
#    pick pqr1234 Add feature E
# You are currently rebasing branch 'feature' on 'abc1234'.
#   (fix conflicts and then run "git rebase --continue")
#   (use "git rebase --abort" to cancel the rebase operation)
```

### 放弃 Cherry-pick

```bash
# 放弃 cherry-pick 操作
git cherry-pick --abort
```

### 放弃操作的注意事项

```
⚠️ 重要提醒：

┌─────────────────────────────────────────────────┐
│  放弃操作前的检查清单                            │
├─────────────────────────────────────────────────┤
│  □ 是否有未提交的更改？                         │
│    - 有：先 stash 或 commit                     │
│    - 没有：可以安全放弃                         │
│                                                 │
│  □ 是否已经解决了一部分冲突？                   │
│    - 是：放弃后需要重新解决                     │
│    - 否：放弃没有损失                           │
│                                                 │
│  □ 是否在公共分支上？                           │
│    - 是：谨慎操作，通知团队                     │
│    - 否：可以自由放弃                           │
└─────────────────────────────────────────────────┘
```

### 保存当前进度再放弃

如果你已经解决了一些冲突但想暂时放弃：

```bash
# 方法一：使用 stash 保存当前进度
git stash push -m "合并进度保存"
git merge --abort

# 稍后恢复
git stash pop

# 方法二：创建临时分支保存
git checkout -b temp-merge-progress
git add .
git commit -m "合并进度临时保存"
git checkout original-branch
git merge --abort

# 稍后从临时分支继续
git merge temp-merge-progress
```

---

## Git 冲突解决工具对比

### 工具概览

| 工具 | 平台 | 价格 | 特点 | 推荐指数 |
|------|------|------|------|---------|
| **Beyond Compare** | Windows/Mac/Linux | 付费 | 功能强大，三路合并 | ★★★★★ |
| **KDiff3** | Windows/Mac/Linux | 免费 | 开源，支持三路合并 | ★★★★☆ |
| **P4Merge** | Windows/Mac/Linux | 免费 | Perforce 出品，专业 | ★★★★☆ |
| **Meld** | Windows/Mac/Linux | 免费 | 开源，界面友好 | ★★★★☆ |
| **VS Code** | Windows/Mac/Linux | 免费 | 集成开发环境 | ★★★★☆ |
| **vimdiff** | 全平台 | 免费 | 命令行，轻量 | ★★★☆☆ |

### Beyond Compare

**优点：**
- 界面直观，功能强大
- 支持三路合并和文件夹对比
- 语法高亮丰富
- 可以比较二进制文件

**配置方法：**

```bash
# 配置 Beyond Compare 为合并工具
git config --global merge.tool bc
git config --global mergetool.bc.path "C:/Program Files/Beyond Compare 4/bcomp.exe"

# 配置为差异对比工具
git config --global diff.tool bc
git config --global difftool.bc.path "C:/Program Files/Beyond Compare 4/bcomp.exe"
```

**使用界面：**

```
┌────────────────────────────────────────────────────────────┐
│  Beyond Compare 4                                          │
├───────────────┬───────────────┬────────────────────────────┤
│   LEFT        │   CENTER      │   RIGHT                    │
│   (当前分支)   │   (合并结果)   │   (要合并分支)             │
├───────────────┼───────────────┼────────────────────────────┤
│               │               │                            │
│  host:        │  host:        │  host:                     │
│  'localhost'  │  (待决定)      │  'production.example.com'  │
│               │               │                            │
│  port:        │  port:        │  port:                     │
│  3306         │  (待决定)      │  5432                      │
│               │               │                            │
├───────────────┴───────────────┴────────────────────────────┤
│  [← Take Left]  [Take Center]  [Take Right →]  [Take Both] │
└────────────────────────────────────────────────────────────┘
```

### KDiff3

**优点：**
- 完全免费开源
- 支持三路合并
- 自动合并能力强
- 跨平台支持

**配置方法：**

```bash
# Linux
sudo apt-get install kdiff3
git config --global merge.tool kdiff3

# macOS
brew install kdiff3
git config --global merge.tool kdiff3

# Windows
# 下载安装后配置
git config --global merge.tool kdiff3
git config --global mergetool.kdiff3.path "C:/Program Files/KDiff3/kdiff3.exe"
```

**使用界面：**

```
┌─────────────────────────────────────────────────────────────┐
│  KDiff3 - Merge Tool                                        │
├────────────────┬────────────────┬───────────────────────────┤
│  A (Base)      │  B (Current)   │  C (Other)               │
│  共同祖先       │  当前分支       │  要合并分支              │
├────────────────┴────────────────┴───────────────────────────┤
│                                                             │
│  Merged Output (合并结果)                                    │
│                                                             │
│  <<<<<<< Merged                                             │
│  host: 'production.example.com'                             │
│  port: 3306                                                 │
│  >>>>>>>                                                    │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│  [Choose A]  [Choose B]  [Choose C]  [Auto-merge]          │
└─────────────────────────────────────────────────────────────┘
```

### P4Merge

**优点：**
- 免费使用
- Perforce 出品，专业级
- 界面简洁清晰
- 支持三路合并

**配置方法：**

```bash
# 下载安装 Perforce Visual Merge Tool
# https://www.perforce.com/downloads/visual-merge-tool

# 配置
git config --global merge.tool p4merge
git config --global mergetool.p4merge.path "C:/Program Files/Perforce/p4merge.exe"

# macOS
git config --global mergetool.p4merge.path "/Applications/p4merge.app/Contents/MacOS/p4merge"
```

### Meld

**优点：**
- 免费开源
- 界面美观现代
- 支持三路合并
- 集成文件浏览器

**配置方法：**

```bash
# Linux
sudo apt-get install meld
git config --global merge.tool meld

# macOS
brew install meld
git config --global merge.tool meld

# Windows
# 下载安装后配置
git config --global merge.tool meld
git config --global mergetool.meld.path "C:/Program Files/Meld/Meld.exe"
```

**使用界面：**

```
┌─────────────────────────────────────────────────────────────┐
│  Meld                                                       │
├────────────────┬────────────────┬───────────────────────────┤
│  Local         │  Base          │  Remote                   │
│  (本地)        │  (基础)         │  (远程)                   │
├────────────────┴────────────────┴───────────────────────────┤
│                                                             │
│  <<<<<<< HEAD                                               │
│  │ host: 'localhost',                                       │
│  │ port: 3306,                                              │
│  =======                                                    │
│  │ host: 'production.example.com',                          │
│  │ port: 5432,                                              │
│  >>>>>>> feature                                            │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│  [←]  [↑]  [↓]  [→]  [Auto Merge]                         │
└─────────────────────────────────────────────────────────────┘
```

### 工具选择建议

```
选择决策树：

你主要使用什么操作系统？
├── Windows → Beyond Compare (付费) 或 P4Merge (免费)
├── macOS → Beyond Compare 或 KDiff3
└── Linux → Meld 或 KDiff3

你的预算是多少？
├── 愿意付费 → Beyond Compare
└── 免费 → KDiff3, P4Merge, Meld

你更喜欢什么界面？
├── 图形界面 → Beyond Compare, KDiff3, Meld
└── 命令行 → vimdiff
```

---

## 团队冲突解决流程规范

### 建立团队规范

一个成熟的团队应该有明确的冲突解决流程：

```
团队冲突解决流程图：

  ┌──────────────────┐
  │ 发现冲突          │
  └────────┬─────────┘
           ▼
  ┌──────────────────┐
  │ 判断冲突类型      │
  └────────┬─────────┘
           ▼
  ┌──────────────────┐     ┌──────────────────┐
  │ 简单冲突？        │────→│ 自行解决          │
  │ (单文件、明确)    │  是 │ (5分钟内)         │
  └────────┬─────────┘     └──────────────────┘
           │ 否
           ▼
  ┌──────────────────┐     ┌──────────────────┐
  │ 复杂冲突？        │────→│ 寻求代码所有者    │
  │ (多文件、不确定)  │  是 │ 协助解决          │
  └────────┬─────────┘     └──────────────────┘
           │ 否
           ▼
  ┌──────────────────┐     ┌──────────────────┐
  │ 困难冲突？        │────→│ 团队会议讨论      │
  │ (涉及架构决策)    │  是 │ 共同决策          │
  └────────┬─────────┘     └──────────────────┘
           │
           ▼
  ┌──────────────────┐
  │ 记录解决方案      │
  │ 提交并通知团队    │
  └──────────────────┘
```

### 冲突解决规范文档

建议在团队 Wiki 或 README 中包含以下规范：

```markdown
# Git 冲突解决规范

## 1. 冲突解决责任
- 提交代码的开发者负责解决冲突
- 冲突解决时间不超过 30 分钟，否则请求协助

## 2. 冲突解决优先级
1. 保持功能正确性
2. 保持代码风格一致
3. 保留更优的实现方案

## 3. 冲突解决流程
1. `git status` 查看冲突文件
2. 评估冲突复杂度
3. 简单冲突自行解决，复杂冲突寻求协助
4. 解决后运行测试
5. 提交并通知相关开发者

## 4. 沟通要求
- 解决冲突前通知相关开发者
- 记录冲突原因和解决方案
- 如有疑问，在团队群组讨论
```

### 代码审查中的冲突处理

```
PR 合并时的冲突处理：

  ┌──────────────────┐
  │ PR 存在冲突      │
  └────────┬─────────┘
           ▼
  ┌──────────────────┐
  │ PR 作者负责解决   │
  └────────┬─────────┘
           ▼
  ┌──────────────────┐     ┌──────────────────┐
  │ 解决冲突后        │────→│ 重新触发 CI       │
  │ 更新 PR           │     │ 测试通过          │
  └────────┬─────────┘     └──────────────────┘
           │
           ▼
  ┌──────────────────┐
  │ 代码审查者确认    │
  │ 冲突解决正确      │
  └────────┬─────────┘
           ▼
  ┌──────────────────┐
  │ 合并 PR          │
  └──────────────────┘
```

### 自动化冲突检测

使用 CI/CD 工具自动检测冲突：

```yaml
# GitHub Actions 示例
name: Check Merge Conflicts

on:
  pull_request:
    types: [synchronize, opened]

jobs:
  check-conflicts:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0
      
      - name: Check for merge conflicts
        run: |
          git fetch origin main
          git merge --no-commit --no-ff origin/main || {
            echo "❌ PR 存在合并冲突，请解决后再提交"
            exit 1
          }
```

### 冲突解决记录模板

```markdown
## 冲突解决记录

**日期：** 2024-01-15
**PR：** #123
**分支：** feature/user-auth → main

### 冲突文件
1. src/auth/login.js
2. src/utils/validation.js
3. package.json

### 冲突原因
- login.js: 两个分支都修改了登录验证逻辑
- validation.js: 新增验证函数与重构冲突
- package.json: 依赖版本不一致

### 解决方案
1. login.js: 合并双方的验证逻辑，保留更严格的验证
2. validation.js: 使用重构后的结构，补充新验证函数
3. package.json: 使用较新的依赖版本

### 测试结果
- [x] 单元测试通过
- [x] 集成测试通过
- [x] 手动测试通过

### 备注
- 需要更新相关文档
- 建议后续拆分 validation.js 文件
```

---

## 常见冲突问题排查

### 问题一：冲突文件中出现乱码

**症状：** 冲突标记显示为乱码

**原因：** 文件编码不一致

```bash
# 检查文件编码
file --mime-encoding config.js
# 输出：config.js: utf-8

# 解决方案：统一文件编码
# 确保 .gitattributes 中设置正确
echo "* text=auto" > .gitattributes

# 或者针对特定文件类型
echo "*.js text eol=lf" >> .gitattributes
echo "*.css text eol=lf" >> .gitattributes
```

### 问题二：二进制文件显示为冲突

**症状：** 图片、PDF 等文件显示冲突

**解决方案：**

```bash
# 方案1：选择保留某个版本
git checkout --ours images/photo.png
git add images/photo.png

# 方案2：使用 Git LFS 管理二进制文件
git lfs install
git lfs track "*.png"
git lfs track "*.pdf"
git add .gitattributes
```

### 问题三：合并后文件消失

**症状：** 合并后某些文件不见了

**排查步骤：**

```bash
# 1. 检查文件是否被删除
git status --deleted

# 2. 查看文件是否在合并中被删除
git log --diff-filter=D --summary

# 3. 恢复文件
git checkout HEAD -- path/to/missing-file.js

# 4. 如果是删除冲突导致
git checkout --theirs -- path/to/file.js  # 保留对方的修改
```

### 问题四：冲突解决后无法提交

**症状：** `git commit` 提示还有未解决的冲突

```bash
# 检查是否有未解决的冲突
git status

# 查看未解决的文件
git diff --name-only --diff-filter=U

# 强制标记为已解决（谨慎使用）
git add <file>

# 然后提交
git commit
```

### 问题五：误解决了冲突想重新来

**症状：** 想重新解决已经解决的冲突

```bash
# 如果还没提交
git checkout --conflict=merge <file>

# 如果已经提交，重置合并
git reset --hard HEAD~1
# 重新开始合并
git merge feature-branch
```

### 问题六：变基过程中想修改之前的冲突解决

**症状：** 变基过程中，想修改之前某个提交的冲突解决

```bash
# 继续变基到有冲突的提交
git rebase --continue

# 如果想修改已经完成的冲突解决
# 使用交互式变基
git rebase -i HEAD~5  # 修改最近5个提交

# 在编辑器中，将对应提交标记为 edit
# Git 会暂停在该提交，让你修改
# 修改后
git add .
git commit --amend
git rebase --continue
```

### 问题七：合并时提示 "already exists"

**症状：** `error: The following untracked working tree files would be overwritten by merge`

```bash
# 查看冲突的未跟踪文件
git status

# 方案1：暂存未跟踪文件
git stash --include-untracked

# 执行合并
git merge feature-branch

# 恢复暂存的文件
git stash pop

# 方案2：删除未跟踪文件（确认不需要时）
git clean -fd
```

### 问题八：submodule 冲突无法解决

**症状：** 子模块冲突解决后仍然显示冲突

```bash
# 1. 进入子模块目录
cd lib/vendor-library

# 2. 确保子模块在正确的提交
git checkout <correct-commit-hash>

# 3. 返回主项目
cd ../..

# 4. 重新添加子模块
git add lib/vendor-library

# 5. 如果问题持续，尝试重新初始化子模块
git submodule update --init --recursive
```

### 问题九：合并后文件权限冲突

**症状：** 文件权限（executable bit）冲突

```bash
# 查看文件权限变化
git diff --stat

# 解决方案1：保留当前权限
git update-index --chmod=+x script.sh
git add script.sh

# 解决方案2：在 .gitattributes 中设置
echo "*.sh text eol=lf" >> .gitattributes
```

### 问题十：大量冲突不知道从何下手

**症状：** 冲突文件太多，感到无从下手

```bash
# 1. 先统计冲突数量
git diff --name-only --diff-filter=U | wc -l

# 2. 按目录分组查看
git diff --name-only --diff-filter=U | sort

# 3. 按文件类型分组
git diff --name-only --diff-filter=U | grep -E "\.js$" | wc -l
git diff --name-only --diff-filter=U | grep -E "\.css$" | wc -l

# 4. 优先处理配置文件
git diff --name-only --diff-filter=U | grep -E "(config|package\.json|\.env)"

# 5. 使用批量解决策略
# 对于简单的冲突，可以使用脚本自动解决
for file in $(git diff --name-only --diff-filter=U); do
  echo "处理文件: $file"
  # 这里可以添加自动解决逻辑
done
```

---

## 总结

### 关键要点回顾

```
合并冲突解决要点：

┌─────────────────────────────────────────────────────────┐
│  1. 理解冲突原因                                        │
│     - 两个分支修改了同一区域                            │
│     - Git 无法自动判断保留哪个                          │
│                                                         │
│  2. 掌握冲突标记                                        │
│     - <<<<<<< HEAD: 当前分支内容开始                    │
│     - =======: 分隔符                                  │
│     - >>>>>>> branch: 要合并分支内容结束                │
│                                                         │
│  3. 选择合适的工具                                      │
│     - 简单冲突：手动编辑                               │
│     - 复杂冲突：VS Code 或专用合并工具                  │
│                                                         │
│  4. 预防冲突发生                                        │
│     - 频繁合并主分支                                   │
│     - 小步提交                                         │
│     - 团队沟通协调                                     │
│                                                         │
│  5. 必要时放弃操作                                      │
│     - git merge --abort                                │
│     - git rebase --abort                               │
└─────────────────────────────────────────────────────────┘
```

### 推荐学习路径

```
初学者路径：
Week 1: 理解冲突标记，手动解决简单冲突
Week 2: 使用 VS Code 解决冲突
Week 3: 学习使用 git mergetool

进阶路径：
Week 4: 掌握三方合并原理
Week 5: 学习使用 git rerere
Week 6: 建立团队冲突解决规范
```

### 实践练习

```bash
# 练习1：创建简单冲突并解决
mkdir practice && cd practice
git init
echo "initial" > file.txt && git add . && git commit -m "init"
git checkout -b feature
echo "feature change" > file.txt && git add . && git commit -m "feature"
git checkout main
echo "main change" > file.txt && git add . && git commit -m "main"
git merge feature  # 产生冲突
# 手动解决冲突...
git add file.txt && git commit -m "merge"

# 练习2：使用 VS Code 解决多文件冲突
# 练习3：配置和使用 git mergetool
# 练习4：使用 git rerere 自动解决重复冲突
```

---

> **提示：** 解决冲突是 Git 使用中的常见场景，不要害怕冲突。随着经验的积累，你会越来越熟练地处理各种冲突情况。记住，良好的沟通和规范的工作流程是减少冲突的最佳方式。

---

[返回目录](#目录) | [上一章：分支管理与工作流](./11-branching-workflows.md) | [下一章：高级 Git 技巧](./13-advanced-git.md)
