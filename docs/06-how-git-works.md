# Git 工作原理深入解析

> 理解 Git 的内部工作原理，能让你在使用 Git 时更加得心应手。本章将从底层机制出发，带你全面了解 Git 是如何管理代码版本的。

---

## 目录

- [Git 的分布式版本控制原理](#git-的分布式版本控制原理)
- [Git 的三个工作区域](#git-的三个工作区域)
- [Git 文件的四种状态](#git-文件的四种状态)
- [Git 对象模型](#git-对象模型)
- [Git 引用](#git-引用)
- [SHA-1 哈希与内容寻址](#sha-1-哈希与内容寻址)
- [Git 的快照机制 vs 差异机制](#git-的快照机制-vs-差异机制)
- [Git 目录结构详解](#git-目录结构详解)
- [Git 的垃圾回收机制](#git-的垃圾回收机制)
- [理解 Git 的乐观锁策略](#理解-git-的乐观锁策略)
- [Git 与其他版本控制系统的对比](#git-与其他版本控制系统的对比)
- [图解 Git 工作流](#图解-git-工作流)

---

## Git 的分布式版本控制原理

### 什么是分布式版本控制

在 Git 出现之前，主流的版本控制系统（如 SVN、CVS）采用的是**集中式**架构。而 Git 采用了完全不同的**分布式**架构，这是理解 Git 一切行为的基础。

```
┌─────────────────────────────────────────────────────────────────┐
│                    集中式 vs 分布式                              │
├────────────────────────────┬────────────────────────────────────┤
│       集中式 (SVN)          │         分布式 (Git)               │
│                            │                                    │
│    ┌──────────┐            │   ┌──────────┐  ┌──────────┐      │
│    │ 中央服务器 │            │   │ 开发者 A  │  │ 开发者 B  │      │
│    │ (唯一仓库) │            │   │ (完整仓库) │  │ (完整仓库) │      │
│    └─────┬────┘            │   └─────┬────┘  └─────┬────┘      │
│      ┌───┼───┐             │         │             │            │
│      │   │   │             │         └──────┬──────┘            │
│      ▼   ▼   ▼             │                ▼                   │
│    ┌─┐ ┌─┐ ┌─┐            │          ┌──────────┐             │
│    │A│ │B│ │C│            │          │ 远程仓库   │             │
│    └─┘ └─┘ └─┘            │          │ (可选备份) │             │
│   开发者                    │          └──────────┘             │
│                            │                                    │
│  • 必须联网才能提交          │  • 每人都有完整仓库               │
│  • 中央服务器宕机则所有人停摆 │  • 离线也能提交、查看历史         │
│  • 只保存差异                │  • 保存完整快照                   │
└────────────────────────────┴────────────────────────────────────┘
```

### 分布式的核心优势

**1. 离线工作能力**

Git 允许你在没有网络连接的情况下进行几乎所有操作：

```bash
# 以下操作全部可以离线完成
git add .                    # 暂存文件
git commit -m "新功能"        # 提交更改
git log                      # 查看提交历史
git branch feature           # 创建分支
git checkout feature         # 切换分支
git diff                     # 查看差异
git blame file.txt           # 查看文件修改记录
```

**2. 每个开发者都拥有完整的仓库副本**

当你执行 `git clone` 时，你获取的不仅仅是最新的文件，而是整个项目的历史记录：

```
开发者 A 的电脑          开发者 B 的电脑          远程服务器
┌─────────────┐      ┌─────────────┐      ┌─────────────┐
│ .git/       │      │ .git/       │      │ .git/       │
│  ├── 完整历史│      │  ├── 完整历史│      │  ├── 完整历史│
│  ├── 所有分支│      │  ├── 所有分支│      │  ├── 所有分支│
│  ├── 所有标签│      │  ├── 所有标签│      │  ├── 所有标签│
│  └── 所有文件│      │  └── 所有文件│      │  └── 所有文件│
│ 工作区文件   │      │ 工作区文件   │      │             │
└─────────────┘      └─────────────┘      └─────────────┘
```

**3. 更快的操作速度**

由于绝大多数操作都在本地进行，Git 的操作速度非常快：

| 操作 | SVN（需要网络） | Git（本地操作） |
|------|----------------|----------------|
| 查看历史 | 慢（需要请求服务器） | 极快（本地数据） |
| 提交更改 | 慢（需要网络） | 极快（本地操作） |
| 创建分支 | 慢（需要服务器操作） | 瞬间（创建一个指针） |
| 切换分支 | 慢（需要更新文件） | 快（本地文件切换） |
| 比较差异 | 中等 | 极快（本地比较） |

**4. 更灵活的工作流**

分布式架构支持多种工作流模式：

```
┌──────────────────────────────────────────────────────────────┐
│                    Git 常见工作流                              │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  集中式工作流：                                               │
│  开发者A ──push──▶ main ◀──push── 开发者B                    │
│                                                              │
│  功能分支工作流：                                              │
│  main ◀──merge── feature-x ◀──commit── 开发者A               │
│                                                              │
│  Git Flow 工作流：                                            │
│  main ◀──merge── release ◀──merge── develop ◀── feature     │
│                                                              │
│  Fork 工作流：                                                │
│  原仓库 ◀──PR── 你的Fork ◀──clone── 本地仓库                 │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## Git 的三个工作区域

Git 的工作流程围绕三个核心区域展开。理解这三个区域是掌握 Git 的关键。

### 区域总览

```
┌─────────────────────────────────────────────────────────────────┐
│                      Git 三个工作区域                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────────┐   git add    ┌──────────┐   git commit       │
│  │              │ ──────────▶ │          │ ──────────▶         │
│  │   工作区      │             │  暂存区    │         ┌────────┐│
│  │  (Working    │ ◀────────── │ (Staging  │         │ 版本库  ││
│  │  Directory)  │   git       │  Area)    │         │(Repo)  ││
│  │              │   checkout   │          │         │        ││
│  └──────────────┘             └──────────┘         └────────┘│
│                                                                 │
│   你编辑文件的地方     准备提交的变更        已提交的历史记录     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 工作区（Working Directory）

工作区就是你电脑上看到的项目文件夹，是你直接编辑和查看文件的地方。

```bash
# 查看当前工作区中的文件
ls -la

# 输出示例
# drwxr-xr-x  user staff  4096  ./
# drwxr-xr-x  user staff  4096  ../
# drwxr-xr-x  user staff  4096  .git/
# -rw-r--r--  user staff   120  README.md
# -rw-r--r--  user staff   450  app.py
# -rw-r--r--  user staff   200  config.json
```

工作区中的文件可以处于不同的状态：已跟踪或未跟踪，已修改或未修改。Git 会监控工作区中文件的变化。

### 暂存区（Staging Area）

暂存区也叫索引（Index），是 Git 特有的一个概念。它是一个文件，位于 `.git/index`，记录了下次提交时将要包含的内容。

```bash
# 将文件添加到暂存区
git add README.md

# 查看暂存区的内容
git status

# 输出示例：
# On branch main
# Changes to be committed:
#   (use "git restore --staged <file>..." to unstage)
#         modified:   README.md

# 查看暂存区的详细信息
git ls-files --stage

# 输出示例：
# 100644 a1b2c3d4e5f6... 0	README.md
```

暂存区的作用：

```
┌─────────────────────────────────────────────────────────────┐
│                    暂存区的作用                               │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1. 精确控制提交内容                                          │
│     • 你可以只暂存部分修改                                    │
│     • 可以将一个大修改拆分成多个小提交                          │
│                                                             │
│  2. 提交前的缓冲区                                           │
│     • 先把要提交的内容准备好                                   │
│     • 检查无误后再正式提交                                    │
│                                                             │
│  3. 支持部分暂存                                             │
│     git add -p    # 交互式选择要暂存的代码块                   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 版本库（Repository）

版本库是 Git 存储所有历史记录的地方，位于项目根目录下的 `.git` 文件夹中。

```bash
# 查看版本库中的提交历史
git log --oneline

# 输出示例：
# a1b2c3d (HEAD -> main) 添加登录功能
# e4f5g6h 修复首页样式
# i7j8k9l 初始提交
```

版本库中存储的内容：

| 内容 | 说明 | 存储位置 |
|------|------|---------|
| 提交对象 | 每次提交的快照 | `.git/objects/` |
| 树对象 | 目录结构 | `.git/objects/` |
| 数据对象 | 文件内容 | `.git/objects/` |
| 引用 | 分支和标签指针 | `.git/refs/` |
| HEAD | 当前分支指针 | `.git/HEAD` |
| 配置 | 仓库级配置 | `.git/config` |

### 三个区域的数据流

```bash
# 完整的数据流动过程

# 1. 在工作区创建或修改文件
echo "Hello World" > hello.txt

# 2. 将文件添加到暂存区
git add hello.txt
# 此时：工作区 ──▶ 暂存区

# 3. 将暂存区的内容提交到版本库
git commit -m "添加 hello.txt"
# 此时：暂存区 ──▶ 版本库

# 4. 如果想撤销暂存（从暂存区移回工作区）
git restore --staged hello.txt
# 此时：暂存区 ◀── 工作区

# 5. 如果想丢弃工作区的修改
git restore hello.txt
# 此时：工作区恢复到上次提交或暂存的状态
```

---

## Git 文件的四种状态

Git 中的文件有四种基本状态，理解这些状态有助于你掌握文件在 Git 中的生命周期。

### 状态流转图

```
┌─────────────────────────────────────────────────────────────────┐
│                    Git 文件四种状态                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ┌──────────┐   git add   ┌──────────┐   git commit           │
│   │          │ ──────────▶ │          │ ──────────▶             │
│   │ 未跟踪    │            │  已暂存    │         ┌──────────┐  │
│   │(Untracked)│            │ (Staged)  │         │  已提交    │  │
│   │          │            │          │         │(Committed)│  │
│   └──────────┘            └──────────┘         └─────┬────┘  │
│                                                       │        │
│                       ┌──────────┐                    │        │
│                       │  已修改    │◀──── 修改文件 ─────┘        │
│                       │(Modified) │                             │
│                       │          │───── git add ────▶ 已暂存    │
│                       └──────────┘                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 状态详解

#### 1. 未跟踪（Untracked）

新创建的文件，Git 尚未开始追踪它。

```bash
# 创建一个新文件
echo "新文件内容" > new-file.txt

# 查看状态
git status

# 输出：
# On branch main
# Untracked files:
#   (use "git add <file>..." to include in what will be committed)
#         new-file.txt
```

特点：
- 文件存在于工作区中
- Git 不会记录它的变化
- 不会被包含在提交中
- 需要通过 `git add` 显式添加到 Git 追踪

#### 2. 已暂存（Staged）

文件已被添加到暂存区，等待下次提交。

```bash
# 将文件添加到暂存区
git add new-file.txt

# 查看状态
git status

# 输出：
# On branch main
# Changes to be committed:
#   (use "git restore --staged <file>..." to unstage)
#         new file:   new-file.txt
```

特点：
- 文件的当前版本被记录在暂存区
- 下次 `git commit` 时会被包含在提交中
- 可以通过 `git restore --staged` 取消暂存

#### 3. 已提交（Committed）

文件的当前版本已安全存储在版本库中。

```bash
# 提交暂存区的内容
git commit -m "添加新文件"

# 查看状态
git status

# 输出：
# On branch main
# nothing to commit, working tree clean
```

特点：
- 文件的快照已保存在 Git 数据库中
- 工作区、暂存区、版本库中的文件内容一致
- 可以通过 `git log` 查看提交历史

#### 4. 已修改（Modified）

文件已被 Git 追踪，但在工作区中有了新的修改，且尚未暂存。

```bash
# 修改一个已追踪的文件
echo "修改后的内容" >> README.md

# 查看状态
git status

# 输出：
# On branch main
# Changes not staged for commit:
#   (use "git add <file>..." to update what will be committed)
#         modified:   README.md
```

特点：
- 工作区中的版本与暂存区/版本库中的版本不同
- 需要通过 `git add` 将修改添加到暂存区
- 可以通过 `git restore` 丢弃修改

### 状态转换命令汇总

| 当前状态 | 目标状态 | 命令 |
|---------|---------|------|
| 未跟踪 → 已暂存 | 添加到暂存区 | `git add <file>` |
| 已暂存 → 已提交 | 提交更改 | `git commit -m "msg"` |
| 已提交 → 已修改 | 编辑文件 | 直接编辑文件 |
| 已修改 → 已暂存 | 重新暂存 | `git add <file>` |
| 已暂存 → 已修改 | 取消暂存 | `git restore --staged <file>` |
| 已修改 → 已提交 | 丢弃修改 | `git restore <file>` |
| 已提交 → 未跟踪 | 移除追踪 | `git rm --cached <file>` |

---

## Git 对象模型

Git 是一个内容寻址的文件系统，所有数据都以对象的形式存储。Git 有四种核心对象类型。

### 对象关系图

```
┌─────────────────────────────────────────────────────────────────┐
│                      Git 对象模型                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Tag 对象 ────────────────────────────────────────────────────┐ │
│  (标签)                                                       │ │
│  • 指向特定提交                                                │ │
│  • 可包含标签信息                                              │ │
│  • 永远不移动                                                  │ │
│                                                              │ │
│  Commit 对象 ◀──────────────────────────────────────────────┘ │
│  (提交)          │                                             │
│  • 包含作者信息   │                                             │
│  • 包含提交信息   │                                             │
│  • 指向一个树对象 │                                             │
│  • 指向父提交(s)  ▼                                             │
│              Tree 对象                                         │
│              (树)                                              │
│              • 表示目录结构                                     │
│              • 包含文件名和权限                                  │
│              • 指向 blob 或子 tree                              │
│                    │                                           │
│          ┌─────────┼─────────┐                                 │
│          ▼         ▼         ▼                                 │
│       Blob       Blob      Tree                                │
│       (blob)     (blob)    (子目录)                             │
│       • 存储文件内容                                            │
│       • 不包含文件名                                            │
│       • 相同内容只存一份                                        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Blob 对象（数据对象）

Blob 存储文件的实际内容，不包含文件名、权限等元数据。

```bash
# 手动创建一个 blob 对象
echo "Hello, Git!" | git hash-object -w --stdin

# 输出类似：
# 8ab686eafeb1f44702738c8b0f24f2567c36da6d

# 查看 blob 对象的内容
git cat-file -p 8ab686ea

# 输出：
# Hello, Git!

# 查看对象类型
git cat-file -t 8ab686ea

# 输出：
# blob
```

**重要特性：**
- 同一内容只存储一次，无论有多少个文件包含相同内容
- 文件名存储在 Tree 对象中，不在 Blob 中
- 这使得 Git 可以高效地存储数据

### Tree 对象（树对象）

Tree 对象表示目录结构，它记录了文件名、权限以及对 blob 或其他 tree 对象的引用。

```bash
# 查看某个提交的树对象
git cat-file -p HEAD^{tree}

# 输出示例：
# 100644 blob a1b2c3d4...    .gitignore
# 100644 blob e5f6g7h8...    README.md
# 040000 tree i9j0k1l2...    src
# 100644 blob m3n4o5p6...    config.json

# 查看子目录的树对象
git cat-file -p i9j0k1l2

# 输出示例：
# 100644 blob q7r8s9t0...    main.py
# 100644 blob u1v2w3x4...    utils.py
```

Tree 对象的结构：

```
Tree (根目录)
├── .gitignore    → Blob (a1b2c3d4...)
├── README.md     → Blob (e5f6g7h8...)
├── config.json   → Blob (m3n4o5p6...)
└── src/          → Tree (i9j0k1l2...)
    ├── main.py   → Blob (q7r8s9t0...)
    └── utils.py  → Blob (u1v2w3x4...)
```

### Commit 对象（提交对象）

Commit 对象记录了一次提交的完整信息，包括作者、提交者、提交信息、指向的树对象以及父提交。

```bash
# 查看提交对象的详细信息
git cat-file -p HEAD

# 输出示例：
# tree 8d9e0f1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e
# parent 1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b
# author Zhang San <zhangsan@example.com> 1699000000 +0800
# committer Zhang San <zhangsan@example.com> 1699000000 +0800
#
# 添加用户登录功能
```

Commit 对象包含的信息：

| 字段 | 说明 |
|------|------|
| tree | 指向的树对象的 SHA-1 哈希 |
| parent | 父提交的 SHA-1 哈希（首次提交无父提交） |
| author | 代码修改者的姓名和邮箱 |
| committer | 实际执行提交的人 |
| 提交信息 | 对本次提交的描述 |

```
Commit 对象结构：

Commit: a1b2c3d4...
├── tree:      8d9e0f1a...  ──▶ Tree 对象（项目的完整快照）
├── parent:    1a2b3c4d...  ──▶ 父 Commit 对象
├── parent:    5e6f7a8b...  ──▶ 第二个父提交（合并提交时）
├── author:    Zhang San <zhangsan@example.com>
├── committer: Zhang San <zhangsan@example.com>
└── message:   "添加用户登录功能"
```

### Tag 对象（标签对象）

Tag 对象用于给特定的提交打标签，通常用于标记版本号。

```bash
# 创建附注标签
git tag -a v1.0 -m "发布版本 1.0"

# 查看标签对象
git cat-file -p v1.0

# 输出示例：
# object a1b2c3d4e5f6...
# type commit
# tag v1.0
# tagger Zhang San <zhangsan@example.com> 1699000000 +0800
#
# 发布版本 1.0
```

**轻量标签 vs 附注标签：**

```
轻量标签（Lightweight Tag）：
• 只是一个指向提交的指针
• 不创建 Tag 对象
• 适合临时标记

附注标签（Annotated Tag）：
• 创建完整的 Tag 对象
• 包含标签信息、签名等
• 适合正式发布版本
```

---

## Git 引用

Git 引用是指向提交对象的指针，它们存储在 `.git/refs/` 目录中。

### HEAD 引用

HEAD 是一个特殊的引用，它指向当前所在的分支或提交。

```bash
# 查看 HEAD 的内容
cat .git/HEAD

# 输出（指向分支）：
# ref: refs/heads/main

# 查看 HEAD 指向的实际提交
git rev-parse HEAD

# 输出：
# a1b2c3d4e5f6...

# 分离 HEAD 状态（直接指向提交）
git checkout a1b2c3d
cat .git/HEAD
# 输出：
# a1b2c3d4e5f6...（直接是提交哈希，不是分支引用）
```

HEAD 的两种状态：

```
正常状态（指向分支）：
HEAD ──▶ refs/heads/main ──▶ Commit (a1b2c3d...)

分离 HEAD 状态（直接指向提交）：
HEAD ──▶ Commit (a1b2c3d...)
```

### 分支引用

分支本质上是一个指向某个提交的可移动指针。

```bash
# 查看分支引用
cat .git/refs/heads/main

# 输出：
# a1b2c3d4e5f6...

# 创建新分支（只是创建一个新的引用文件）
git branch feature
cat .git/refs/heads/feature
# 输出与 main 相同的哈希（指向同一个提交）

# 查看所有分支
git branch -v

# 输出：
# * main    a1b2c3d 最新提交信息
#   feature a1b2c3d 最新提交信息
```

分支的工作原理：

```
提交历史：
A ──▶ B ──▶ C ◀── main, HEAD
              ▲
              │
            feature

执行 git checkout feature 后：
A ──▶ B ──▶ C ◀── main
              ▲
              │
            feature, HEAD

在 feature 上新提交 D 后：
A ──▶ B ──▶ C ──▶ D ◀── feature, HEAD
              ▲
              │
            main（仍然指向 C）
```

### 标签引用

标签是固定的引用，不会像分支一样移动。

```bash
# 查看轻量标签
cat .git/refs/tags/v1.0
# 输出：a1b2c3d4e5f6...（直接指向提交）

# 查看附注标签
git rev-parse v1.0
# 输出：x9y8z7w6...（指向 Tag 对象）
git cat-file -p x9y8z7w6
# 输出：object a1b2c3d4...（Tag 对象指向实际提交）

# 查看所有标签
git tag -l
```

### 远程追踪引用

远程追踪分支记录远程仓库中分支的位置。

```bash
# 查看远程追踪分支
cat .git/refs/remotes/origin/main

# 输出：
# a1b2c3d4e5f6...

# 查看所有远程追踪分支
git branch -r

# 输出：
#   origin/main
#   origin/develop
#   origin/feature-x

# 更新远程追踪分支
git fetch origin
```

引用的完整结构：

```
.git/refs/
├── heads/           # 本地分支
│   ├── main        ──▶ Commit (a1b2c3d...)
│   ├── develop     ──▶ Commit (e5f6g7h...)
│   └── feature     ──▶ Commit (i9j0k1l...)
├── tags/            # 标签
│   ├── v1.0        ──▶ Commit (m3n4o5p...)
│   └── v2.0        ──▶ Tag Object ──▶ Commit
└── remotes/         # 远程追踪
    └── origin/
        ├── main    ──▶ Commit (a1b2c3d...)
        └── develop ──▶ Commit (q7r8s9t...)
```

---

## SHA-1 哈希与内容寻址

Git 使用 SHA-1 哈希算法来标识所有对象，这使得 Git 成为一个内容寻址的文件系统。

### SHA-1 的工作原理

```
输入内容 ──▶ SHA-1 算法 ──▶ 40位十六进制哈希值

示例：
"Hello, Git!" ──▶ SHA-1 ──▶ 8ab686eafeb1f44702738c8b0f24f2567c36da6d
```

### 为什么使用内容寻址

```
┌─────────────────────────────────────────────────────────────────┐
│                  内容寻址的优势                                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. 内容完整性验证                                               │
│     • 如果文件内容改变，哈希值也会改变                              │
│     • 可以检测文件是否被篡改                                      │
│                                                                 │
│  2. 去重存储                                                     │
│     • 相同内容产生相同哈希                                        │
│     • Git 只存储一份，节省空间                                    │
│                                                                 │
│  3. 快速比较                                                     │
│     • 只需比较哈希值，无需比较整个文件                              │
│     • 判断两个对象是否相同只需 O(1) 时间                           │
│                                                                 │
│  4. 分布式一致性                                                 │
│     • 相同内容在任何仓库中哈希值都相同                              │
│     • 天然支持分布式同步                                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Git 如何计算哈希

Git 在计算对象哈希时，会在内容前添加头部信息：

```
对象格式："<类型> <内容长度>\0<实际内容>"

示例（blob 对象）：
输入："Hello, Git!\n"
头部："blob 12\0"
完整内容："blob 12\0Hello, Git!\n"
SHA-1 结果：8ab686eafeb1f44702738c8b0f24f2567c36da6d
```

```bash
# 手动计算 Git 对象的 SHA-1
echo -n "blob 12\0Hello, Git!\n" | shasum

# 使用 Git 命令验证
echo "Hello, Git!" | git hash-object --stdin

# 两个命令应该输出相同的哈希值
```

### 短哈希

Git 允许使用短哈希（通常 7 位以上）来引用对象：

```bash
# 完整哈希（40 个字符）
git show a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6q7r8s9t0

# 短哈希（前 7 位，Git 能自动识别）
git show a1b2c3d

# Git 会自动查找匹配的对象
# 如果短哈希不唯一，Git 会提示你使用更长的前缀
# 错误：ambiguous argument 'a1b2': unknown revision or path not found.
```

为什么 7 位通常就够了？让我们计算一下：

```
7 位十六进制数 = 16^7 = 268,435,456 种可能

对于一个拥有 100 万次提交的大型项目：
发生哈希前缀冲突的概率极低

Git 的建议：
• 小型项目：7 位足够
• 中型项目：8-10 位
• 大型项目（如 Linux 内核）：12 位以上
```

### SHA-1 的安全性与未来发展

值得注意的是，SHA-1 算法在密码学上已经被认为不够安全。Git 社区正在逐步迁移到更安全的 SHA-256 算法：

```bash
# Git 2.29+ 开始支持 SHA-256
# 创建一个使用 SHA-256 的仓库
git init --object-format=sha256 my-sha256-repo

# 查看仓库的对象格式
git -C my-sha256-repo rev-parse --show-object-format
# 输出：sha256
```

SHA-1 与 SHA-256 的对比：

| 特性 | SHA-1 | SHA-256 |
|------|-------|---------|
| 哈希长度 | 160 位（40 个十六进制字符） | 256 位（64 个十六进制字符） |
| 安全性 | 已被证明存在碰撞攻击 | 目前仍然安全 |
| 性能 | 稍快 | 稍慢 |
| 兼容性 | 所有 Git 版本支持 | Git 2.29+ 支持 |
| 未来发展 | 逐步淘汰 | 推荐使用 |

对于日常使用来说，SHA-1 在 Git 中的用途主要是内容标识而非密码学安全，因此目前仍然可以放心使用。Git 的设计已经考虑了未来迁移到更安全哈希算法的可能性。

---

## Git 的快照机制 vs 差异机制

这是 Git 与其他版本控制系统的核心区别之一。

### 差异存储（Delta Storage）

SVN 等系统存储的是文件的差异（delta）：

```
版本 1：Hello
版本 2：Hello World    （存储：在 "Hello" 后添加 " World"）
版本 3：Hello Git      （存储：将 "World" 替换为 "Git"）
版本 4：Hi Git         （存储：将 "Hello" 替换为 "Hi"）

要得到版本 4，需要：
版本 1 + 差异1 + 差异2 + 差异3 ──▶ 版本 4
```

### 快照存储（Snapshot Storage）

Git 存储的是每个版本的完整快照：

```
版本 1：完整文件内容 "Hello"
版本 2：完整文件内容 "Hello World"
版本 3：完整文件内容 "Hello Git"
版本 4：完整文件内容 "Hi Git"

要得到版本 4，直接读取版本 4 的快照即可
```

### 两种方式的对比

```
┌─────────────────────────────────────────────────────────────────┐
│              差异存储 vs 快照存储                                  │
├──────────────────────────┬──────────────────────────────────────┤
│       差异存储 (SVN)       │         快照存储 (Git)               │
│                          │                                      │
│  版本1 ──▶ 版本2 ──▶ 版本3│  版本1    版本2    版本3              │
│   └──Δ──▶└──Δ──▶        │    │        │        │               │
│                          │    ▼        ▼        ▼               │
│  • 存储空间小             │  快照1    快照2    快照3              │
│  • 查看历史版本慢         │                                      │
│  • 依赖完整链             │  • 存储空间较大（有压缩优化）          │
│  • 分支创建快             │  • 查看任何版本都极快                  │
│  • 分支合并复杂           │  • 每个版本独立完整                    │
│                          │  • 分支创建和合并高效                  │
└──────────────────────────┴──────────────────────────────────────┘
```

### Git 的压缩优化

虽然 Git 存储快照，但它使用了多种优化技术来减少存储空间：

```bash
# 查看仓库大小
git count-objects -vH

# 输出示例：
# count: 150
# size: 2.34 MiB
# in-pack: 1200
# packs: 3
# size-pack: 15.67 MiB
# garbage: 0
# size-garbage: 0 bytes
```

Git 的压缩策略：

```
1. 对象打包（Packfile）
   • 将多个对象打包成一个文件
   • 使用 zlib 压缩
   • 存储对象之间的差异

2. 松散对象
   • 新创建的对象单独存储
   • 使用 zlib 压缩
   • 定期打包整理

3. 智能差异算法
   • 检测相似的文件内容
   • 只存储差异部分
   • 大大减少存储空间
```

```bash
# 手动触发打包
git gc

# 查看打包内容
git verify-pack -v .git/objects/pack/pack-*.idx

# 输出示例（部分）：
# a1b2c3d4... commit 234 156 12
# e5f6g7h8... blob   1024 456 168
# ...（更多对象）
```

---

## Git 目录结构详解

`.git` 目录是 Git 仓库的核心，了解它的结构有助于理解 Git 的工作原理。

### .git 目录结构

```bash
# 查看 .git 目录结构
tree .git -L 1

# 输出：
# .git
# ├── HEAD
# ├── config
# ├── description
# ├── hooks/
# ├── index
# ├── info/
# ├── logs/
# ├── objects/
# ├── packed-refs
# └── refs/
```

### 各文件和目录详解

```
.git/
├── HEAD                 # 指向当前分支的引用
├── config               # 仓库级配置文件
├── description          # 仓库描述（GitWeb 使用）
├── hooks/               # 钩子脚本目录
│   ├── pre-commit.sample
│   ├── post-commit.sample
│   ├── pre-push.sample
│   └── ...
├── index                # 暂存区索引文件
├── info/                # 辅助信息
│   └── exclude          # 全局忽略规则
├── logs/                # 引用变更日志
│   ├── HEAD
│   └── refs/
│       └── heads/
│           └── main
├── objects/             # 所有 Git 对象的存储位置
│   ├── info/
│   ├── pack/            # 打包后的对象
│   │   ├── pack-*.idx
│   │   └── pack-*.pack
│   ├── a1/              # 松散对象（按哈希前两位分目录）
│   │   └── b2c3d4e5f6...
│   └── ...
├── packed-refs          # 打包后的引用
└── refs/                # 引用目录
    ├── heads/           # 本地分支
    │   ├── main
    │   └── feature
    ├── tags/            # 标签
    │   └── v1.0
    └── remotes/         # 远程追踪分支
        └── origin/
            └── main
```

### 关键文件详解

#### HEAD 文件

```bash
cat .git/HEAD
# 输出：ref: refs/heads/main

# 这意味着 HEAD 指向 refs/heads/main 分支
# refs/heads/main 文件中存储的是实际的提交哈希
```

#### index 文件

```bash
# 查看暂存区内容
git ls-files --stage

# 输出示例：
# 100644 a1b2c3d4e5f6... 0	README.md
# 100644 e5f6g7h8i9j0... 0	src/main.py
# 100644 m3n4o5p6q7r8... 0	config.json

# 各列含义：
# 文件权限 | blob 哈希 | 阶段号 | 文件路径
```

#### config 文件

```bash
cat .git/config

# 输出示例：
# [core]
#     repositoryformatversion = 0
#     filemode = true
#     bare = false
#     logallrefupdates = true
# [remote "origin"]
#     url = https://github.com/user/repo.git
#     fetch = +refs/heads/*:refs/remotes/origin/*
# [branch "main"]
#     remote = origin
#     merge = refs/heads/main
```

#### objects 目录

```bash
# 查看松散对象
ls .git/objects/

# 输出示例：
# 0a  1b  2c  3d  4e  5f  6a  7b  8c  9d
# info  pack

# 松散对象按哈希前两位字符分目录存储
# 例如：哈希 0a1b2c3d... 存储在 .git/objects/0a/1b2c3d...

# 查看对象内容
git cat-file -p 0a1b2c3d
```

#### refs 目录

```bash
# 查看本地分支
ls .git/refs/heads/

# 输出：
# main  feature  develop

# 查看分支指向的提交
cat .git/refs/heads/main

# 输出：
# a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6q7r8s9t0
```

#### hooks 目录

钩子是 Git 在特定操作发生时自动执行的脚本，它们位于 `.git/hooks/` 目录中。钩子分为客户端钩子和服务端钩子两大类，是实现自动化工作流的重要机制。

```bash
# 查看可用的钩子示例
ls .git/hooks/*.sample

# 输出：
# applypatch-msg.sample  pre-commit.sample
# commit-msg.sample      pre-merge-commit.sample
# fsmonitor-watchman.sample  pre-push.sample
# post-update.sample     pre-rebase.sample
# pre-applypatch.sample  prepare-commit-msg.sample
# pre-receive.sample     update.sample

# 启用钩子：移除 .sample 后缀并添加可执行权限
cp .git/hooks/pre-commit.sample .git/hooks/pre-commit
chmod +x .git/hooks/pre-commit
```

常见的钩子及其用途：

| 钩子名称 | 触发时机 | 典型用途 |
|---------|---------|---------|
| pre-commit | 执行 `git commit` 之前 | 运行代码检查、格式化、单元测试 |
| prepare-commit-msg | 打开编辑器之前 | 自动生成提交信息模板 |
| commit-msg | 输入提交信息之后 | 验证提交信息格式是否规范 |
| pre-push | 执行 `git push` 之前 | 运行完整测试套件，防止推送有缺陷的代码 |
| post-merge | 执行 `git merge` 之后 | 自动安装依赖、更新配置 |
| post-checkout | 执行 `git checkout` 之后 | 清理临时文件、切换环境配置 |

一个实用的 pre-commit 钩子示例：

```bash
#!/bin/bash
# .git/hooks/pre-commit
# 提交前自动检查 Python 代码风格

# 获取暂存的 Python 文件
STAGED_FILES=$(git diff --cached --name-only --diff-filter=ACM | grep '\.py$')

if [ -n "$STAGED_FILES" ]; then
    echo "正在检查 Python 代码风格..."
    flake8 $STAGED_FILES
    if [ $? -ne 0 ]; then
        echo "代码风格检查失败，请修复后再提交"
        exit 1
    fi
fi

echo "代码检查通过"
exit 0
```

---

## Git 的垃圾回收机制

Git 使用垃圾回收（Garbage Collection）来清理不再需要的对象，优化存储空间。

### 什么是垃圾对象

```
┌─────────────────────────────────────────────────────────────────┐
│                    Git 垃圾对象的来源                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. 重置提交（git reset）                                        │
│     • 重置后原来的提交变成"悬空"对象                               │
│                                                                 │
│  2. 丢弃暂存（git restore --staged）                             │
│     • 取消暂存后，暂存区中的对象可能变成垃圾                        │
│                                                                 │
│  3. 修改提交（git commit --amend）                               │
│     • 修改提交后，原来的提交对象变成垃圾                            │
│                                                                 │
│  4. 删除分支                                                     │
│     • 删除分支后，只有该分支可达的对象变成垃圾                       │
│                                                                 │
│  5. 松散对象                                                     │
│     • 长期未打包的松散对象                                        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 垃圾回收的工作原理

```bash
# 查看当前仓库状态
git count-objects -vH

# 输出示例：
# count: 150           # 松散对象数量
# size: 2.34 MiB       # 松散对象总大小
# in-pack: 1200        # 打包对象数量
# packs: 3             # 包文件数量
# size-pack: 15.67 MiB # 打包对象总大小
# garbage: 0           # 垃圾对象数量
# size-garbage: 0 bytes

# 手动执行垃圾回收
git gc

# 输出示例：
# Enumerating objects: 1350, done.
# Counting objects: 100% (1350/1350), done.
# Delta compression using up to 8 threads
# Compressing objects: 100% (890/890), done.
# Writing objects: 100% (1350/1350), done.
# Total 1350 (delta 200), reused 1300 (delta 180), pack-reused 0
```

### 垃圾回收的触发条件

```bash
# Git 会自动在以下情况触发垃圾回收：

# 1. 执行某些命令时（如 git pull、git merge）
# 2. 松散对象数量超过阈值
# 3. 手动执行 git gc

# 查看自动 gc 的配置
git config --get gc.auto

# 输出：6700（默认值，松散对象超过 6700 个时自动 gc）

# 查看 gc 的详细配置
git config --list | grep gc

# 示例输出：
# gc.auto=6700
# gc.autopacklimit=50
# gc.pruneexpire=2.weeks.ago
# gc.reflogexpire=90.days
# gc.reflogexpireunreachable=30.days
```

### 保护期（Grace Period）

Git 不会立即删除垃圾对象，而是有一个保护期：

```bash
# 查看保护期配置
git config --get gc.pruneexpire

# 输出：2.weeks.ago（默认 2 周）

# 执行"试运行"垃圾回收（不实际删除）
git gc --auto --dry-run

# 强制清理所有悬空对象
git gc --prune=now

# 查看悬空对象
git fsck --unreachable

# 输出示例：
# Unreachable objects:
# dangling commit a1b2c3d4e5f6...
# dangling blob e5f6g7h8i9j0...

# 查看悬空对象的内容
git show a1b2c3d4
```

### 维护仓库的最佳实践

```bash
# 定期维护命令

# 1. 垃圾回收和打包
git gc

# 2. 验证仓库完整性
git fsck --full

# 3. 清理不必要的文件
git clean -fd

# 4. 优化仓库（包括垃圾回收和重新打包）
git repack -a -d --depth=250 --window=250

# 5. 查看仓库状态
git count-objects -vH
```

---

## 理解 Git 的乐观锁策略

Git 使用基于哈希的乐观锁策略来处理并发操作，这与传统的悲观锁完全不同。

### 什么是乐观锁

```
┌─────────────────────────────────────────────────────────────────┐
│                  悲观锁 vs 乐观锁                                 │
├──────────────────────────┬──────────────────────────────────────┤
│       悲观锁 (SVN)         │         乐观锁 (Git)                │
│                          │                                      │
│  操作前先锁定文件          │  操作时不锁定文件                     │
│  其他人不能修改            │  其他人可以同时修改                    │
│  操作完成后解锁            │  提交时检查是否有冲突                  │
│  安全但并发性低            │  并发性高但可能需要解决冲突             │
│                          │                                      │
│  ┌─────┐ lock ┌─────┐   │  ┌─────┐       ┌─────┐              │
│  │用户A│─────▶│文件X│   │  │用户A│───┬──▶│文件X│              │
│  └─────┘      └─────┘   │  └─────┘   │   └─────┘              │
│  ┌─────┐ 等待           │  ┌─────┐   │                        │
│  │用户B│─────...         │  │用户B│───┘                        │
│  └─────┘                │  └─────┘                            │
└──────────────────────────┴──────────────────────────────────────┘
```

### Git 如何实现乐观锁

**1. 基于哈希的版本检测**

```bash
# Git 使用 SHA-1 哈希来标识文件的特定版本

# 当你执行 git push 时：
# 1. Git 检查远程分支的当前哈希值
# 2. 如果远程分支的哈希与你上次拉取时相同，推送成功
# 3. 如果不同（说明有其他人推送了新内容），推送失败

# 推送失败的示例：
git push origin main

# 输出：
# To https://github.com/user/repo.git
#  ! [rejected]        main -> main (fetch first)
# error: failed to push some refs to 'https://github.com/user/repo.git'
# hint: Updates were rejected because the remote contains work that you do
# hint: not have locally.
```

**2. 提交的原子性**

```bash
# Git 的提交是原子操作
# 要么整个提交成功，要么完全不提交

# 提交过程：
# 1. 创建所有需要的对象（blob、tree、commit）
# 2. 更新引用（HEAD、分支指针）
# 3. 如果任何步骤失败，所有更改都会回滚

# 这保证了仓库的一致性
```

**3. 合并时的冲突检测**

```bash
# 当两个人修改了同一个文件的同一部分时
git merge feature

# 输出：
# Auto-merging src/main.py
# CONFLICT (content): Merge conflict in src/main.py
# Automatic merge failed; fix conflicts and then commit the result.

# 查看冲突
cat src/main.py

# 输出：
# <<<<<<< HEAD
# 这是 main 分支的内容
# =======
# 这是 feature 分支的内容
# >>>>>>> feature
```

### 乐观锁的优势

```
┌─────────────────────────────────────────────────────────────────┐
│                  乐观锁在 Git 中的优势                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. 高并发性                                                     │
│     • 多人可以同时修改同一个文件                                    │
│     • 不需要等待其他人完成操作                                     │
│                                                                 │
│  2. 离线工作                                                     │
│     • 所有操作在本地进行                                          │
│     • 不需要与服务器协调锁                                        │
│                                                                 │
│  3. 分支友好                                                     │
│     • 创建分支不需要任何锁操作                                     │
│     • 分支合并有完善的冲突检测机制                                  │
│                                                                 │
│  4. 简单的模型                                                   │
│     • 不需要管理锁的生命周期                                      │
│     • 不会出现死锁问题                                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 处理冲突的最佳实践

```bash
# 1. 频繁拉取最新代码
git pull origin main

# 2. 在推送前先拉取并合并
git fetch origin
git rebase origin/main
git push origin main

# 3. 使用功能分支，避免直接在 main 上工作
git checkout -b feature-x
# ... 开发 ...
git push origin feature-x
# 在 GitHub 上创建 Pull Request

# 4. 保持提交小而频繁
# 小提交更容易合并，冲突也更容易解决
```

---

## Git 与其他版本控制系统的对比

### Git vs SVN

```
┌─────────────────────────────────────────────────────────────────┐
│                    Git vs SVN 详细对比                            │
├──────────────┬────────────────────┬─────────────────────────────┤
│     特性      │        Git          │          SVN                │
├──────────────┼────────────────────┼─────────────────────────────┤
│ 架构         │ 分布式              │ 集中式                       │
│ 存储方式     │ 快照               │ 差异                         │
│ 分支         │ 轻量级，瞬间创建     │ 目录复制，较慢               │
│ 合并         │ 高效，常用操作       │ 复杂，相对较少使用            │
│ 离线工作     │ 完全支持            │ 不支持                       │
│ 速度         │ 极快（本地操作）     │ 较慢（需要网络）              │
│ 学习曲线     │ 较陡峭              │ 较平缓                       │
│ 权限控制     │ 有限               │ 精细到目录级别                │
│ 大文件支持   │ 需要 Git LFS        │ 原生支持                     │
│ 历史查询     │ 极快               │ 较慢                         │
│ 原子提交     │ 支持               │ 支持                         │
│ 签名提交     │ 支持               │ 有限支持                     │
└──────────────┴────────────────────┴─────────────────────────────┘
```

### Git vs Mercurial

```
┌─────────────────────────────────────────────────────────────────┐
│                  Git vs Mercurial 对比                           │
├──────────────┬────────────────────┬─────────────────────────────┤
│     特性      │        Git          │        Mercurial            │
├──────────────┼────────────────────┼─────────────────────────────┤
│ 架构         │ 分布式              │ 分布式                       │
│ 存储方式     │ 快照               │ 变更集 + 快照混合             │
│ 命令数量     │ 丰富（150+）        │ 精简（50+）                  │
│ 学习难度     │ 较难               │ 较易                         │
│ 分支模型     │ 灵活               │ 相对固定                     │
│ 扩展性       │ 高（钩子、别名等）   │ 中等                        │
│ 性能         │ 大型仓库表现优秀     │ 中小型仓库表现良好            │
│ 社区规模     │ 极大               │ 较小                         │
│ GitHub 支持  │ 原生支持            │ 不支持                       │
│ Windows 支持 │ 良好               │ 良好                         │
└──────────────┴────────────────────┴─────────────────────────────┘
```

### 为什么 Git 成为主流

Git 能够在众多版本控制系统中脱颖而出，成为当今软件开发的事实标准，有以下几个关键原因：

```
┌─────────────────────────────────────────────────────────────────┐
│                Git 成为主流的原因                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. GitHub 的推动                                                │
│     • GitHub 基于 Git 构建，成为全球最大的代码托管平台             │
│     • 开源社区几乎全部使用 GitHub，形成了强大的网络效应            │
│     • Pull Request 工作流成为行业标准                             │
│                                                                 │
│  2. 技术优势                                                     │
│     • 分布式架构更适合现代分布式团队的开发模式                      │
│     • 分支和合并操作极其高效，鼓励频繁分支                         │
│     • 性能优秀，即使面对超大型项目（如 Linux 内核）也能胜任        │
│                                                                 │
│  3. 生态系统                                                     │
│     • 丰富的第三方工具和平台（GitLab、Bitbucket 等）              │
│     • 完善的 CI/CD 集成（GitHub Actions、Jenkins 等）             │
│     • 海量的学习资源和活跃的社区支持                              │
│                                                                 │
│  4. 企业级采用                                                   │
│     • Google、Microsoft、Meta、阿里巴巴等全球科技巨头全面采用     │
│     • 推动了 Git 工具链和基础设施的持续发展                       │
│     • 企业需求反过来促进了 Git 的功能完善                         │
│                                                                 │
│  5. 开源精神                                                     │
│     • Git 本身也是开源项目，由 Linux 之父 Linus Torvalds 创建    │
│     • 开源理念与 Git 的分布式特性天然契合                         │
│     • 任何人都可以参与 Git 的开发和改进                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 图解 Git 工作流

### 基本工作流

```
┌─────────────────────────────────────────────────────────────────┐
│                    Git 基本工作流                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   工作区                 暂存区              版本库               │
│   (Working Dir)         (Staging)           (Repository)        │
│                                                                 │
│   ┌─────────┐           ┌─────────┐         ┌─────────┐        │
│   │         │  git add  │         │  git    │         │        │
│   │ hello.py│ ────────▶ │ hello.py│ commit  │ Commit  │        │
│   │ (修改)   │           │ (已暂存) │ ──────▶ │  a1b2c3 │        │
│   │         │           │         │         │         │        │
│   └─────────┘           └─────────┘         └─────────┘        │
│                                                                 │
│        ▲                      │                                 │
│        │    git restore       │                                 │
│        │    --staged          │                                 │
│        └──────────────────────┘                                 │
│                                                                 │
│   步骤详解：                                                     │
│   1. 在工作区修改文件                                            │
│   2. 使用 git add 将修改添加到暂存区                              │
│   3. 使用 git commit 将暂存区的内容提交到版本库                    │
│   4. 使用 git push 将本地提交推送到远程仓库                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 功能分支工作流

```
┌─────────────────────────────────────────────────────────────────┐
│                  功能分支工作流                                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   main     A ──── B ──── C ──────────────────── F ──── G        │
│            │            │                      ▲      ▲         │
│            │            │                      │      │         │
│   feature  │            D ──── E ──────────────┘      │         │
│            │                                          │         │
│   hotfix   │                                    H ────┘         │
│                                                                 │
│   工作流程：                                                     │
│                                                                 │
│   1. 从 main 创建 feature 分支                                   │
│      git checkout -b feature main                               │
│                                                                 │
│   2. 在 feature 分支上开发                                       │
│      git add .                                                  │
│      git commit -m "完成功能"                                    │
│                                                                 │
│   3. 推送 feature 分支到远程                                     │
│      git push origin feature                                    │
│                                                                 │
│   4. 在 GitHub 上创建 Pull Request                               │
│                                                                 │
│   5. 代码审查通过后，合并到 main                                  │
│      git checkout main                                          │
│      git merge feature                                          │
│                                                                 │
│   6. 删除 feature 分支                                          │
│      git branch -d feature                                      │
│      git push origin --delete feature                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Git Flow 工作流

```
┌─────────────────────────────────────────────────────────────────┐
│                    Git Flow 工作流                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   main      ──●─────────────────●─────────────────●──           │
│               │                 ▲                 ▲             │
│               │                 │                 │             │
│   release     │           ●─────┘           ●─────┘             │
│               │           │                 │                   │
│   develop     ●─────●─────●─────●─────●─────●─────●──           │
│               │     ▲           ▲           ▲                   │
│               │     │           │           │                   │
│   feature     │  ●──┘     ●─────┘     ●─────┘                   │
│               │                                                 │
│   hotfix      │                               ●───●             │
│                                                                 │
│   分支说明：                                                     │
│                                                                 │
│   main：主分支，只存放正式发布的版本                               │
│   develop：开发分支，集成最新的开发功能                            │
│   feature：功能分支，开发新功能                                   │
│   release：发布分支，准备新版本发布                                │
│   hotfix：热修复分支，修复生产环境的紧急问题                       │
│                                                                 │
│   工作流程：                                                     │
│   1. 从 develop 创建 feature 分支                                │
│   2. 在 feature 分支上开发功能                                   │
│   3. 完成后合并回 develop                                        │
│   4. 从 develop 创建 release 分支                                │
│   5. 测试通过后合并到 main 和 develop                             │
│   6. 在 main 上打标签发布                                        │
│   7. 紧急修复从 main 创建 hotfix 分支                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Fork 工作流

```
┌─────────────────────────────────────────────────────────────────┐
│                    Fork 工作流                                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   原仓库 (upstream)    你的 Fork (origin)     本地仓库            │
│   ┌───────────┐       ┌───────────┐       ┌───────────┐        │
│   │           │       │           │       │           │        │
│   │   main    │◀──PR──│   main    │◀─push─│   main    │        │
│   │           │       │           │       │           │        │
│   └───────────┘       └───────────┘       └───────────┘        │
│        │                   ▲                   ▲                │
│        │                   │                   │                │
│        └───── fetch ───────┴──── pull ─────────┘                │
│                                                                 │
│   工作流程：                                                     │
│                                                                 │
│   1. Fork 原仓库                                                │
│      在 GitHub 上点击 Fork 按钮                                  │
│                                                                 │
│   2. 克隆你的 Fork                                               │
│      git clone https://github.com/你的用户名/repo.git            │
│                                                                 │
│   3. 添加原仓库为远程源                                          │
│      git remote add upstream https://github.com/原作者/repo.git  │
│                                                                 │
│   4. 创建功能分支                                                │
│      git checkout -b feature-x                                  │
│                                                                 │
│   5. 开发并提交                                                  │
│      git add .                                                  │
│      git commit -m "添加新功能"                                  │
│                                                                 │
│   6. 推送到你的 Fork                                              │
│      git push origin feature-x                                  │
│                                                                 │
│   7. 创建 Pull Request                                           │
│      在 GitHub 上向原仓库发起 PR                                 │
│                                                                 │
│   8. 同步原仓库的更新                                            │
│      git fetch upstream                                         │
│      git checkout main                                          │
│      git merge upstream/main                                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Rebase 工作流

```
┌─────────────────────────────────────────────────────────────────┐
│                    Rebase 工作流                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   合并前：                                                       │
│   main      A ──── B ──── C ──── F                               │
│                        │         ▲                               │
│                        │         │                               │
│   feature             D ──── E ─┘                               │
│                                                                 │
│   执行 git merge feature（在 main 上）：                          │
│   main      A ──── B ──── C ──── F ──── G  (合并提交)            │
│                        │         ▲        ▲                      │
│                        │         │       ╱                       │
│   feature             D ──── E ─┘      ╱                        │
│                                                                 │
│   执行 git rebase main（在 feature 上）：                         │
│   main      A ──── B ──── C ──── F                               │
│                                    │                             │
│                                    │                             │
│   feature                         D'─── E'  (新的提交)           │
│                                                                 │
│   然后执行 git merge feature（快进合并）：                         │
│   main      A ──── B ──── C ──── F ──── D'─── E'                │
│                                                                 │
│   Rebase 的优势：                                                │
│   • 创建线性的提交历史                                            │
│   • 避免不必要的合并提交                                          │
│   • 使历史更清晰易读                                              │
│                                                                 │
│   Rebase 的注意事项：                                            │
│   • 不要对已经推送到远程的提交进行 rebase                           │
│   • 只对本地未推送的提交使用 rebase                                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 常用工作流对比

| 工作流 | 适用场景 | 复杂度 | 优点 | 缺点 |
|--------|---------|--------|------|------|
| 集中式 | 小团队、简单项目 | 低 | 简单易懂 | 不适合并行开发 |
| 功能分支 | 中小团队 | 中 | 支持并行开发 | 需要管理多个分支 |
| Git Flow | 大型项目、正式发布 | 高 | 流程规范 | 复杂，分支多 |
| Fork | 开源项目 | 中 | 贡献者无需权限 | 需要同步上游更新 |
| Rebase | 追求线性历史 | 中 | 历史清晰 | 需要谨慎使用 |

---

## 实用技巧：查看 Git 内部对象

### 探索对象数据库

```bash
# 查看所有对象
find .git/objects -type f | head -20

# 查看对象类型和内容
git cat-file -t <hash>    # 查看类型
git cat-file -p <hash>    # 查看内容
git cat-file -s <hash>    # 查看大小

# 查看提交的树对象
git cat-file -p HEAD^{tree}

# 查看树对象的详细信息
git ls-tree HEAD

# 查看递归的树结构
git ls-tree -r HEAD

# 查看暂存区的详细信息
git ls-files --stage

# 查看对象的存储大小
git count-objects -vH
```

### 实验：追踪一次提交的完整对象链

理解 Git 对象模型的最好方式是亲手创建它们并通过实验来追踪。下面的实验将带你从零开始构建一个完整的 Git 提交，并观察所有对象之间的关联关系：

```bash
# 准备：创建一个简单的项目结构
mkdir git-lab && cd git-lab
git init

# 创建文件并提交
mkdir src
echo "print('Hello')" > src/main.py
echo "# Git 实验" > README.md
git add .
git commit -m "初始提交"

# 现在让我们追踪这次提交的对象链

# 第一步：查看最新提交的哈希
COMMIT_HASH=$(git rev-parse HEAD)
echo "提交哈希：$COMMIT_HASH"

# 第二步：查看提交对象
echo "=== 提交对象 ==="
git cat-file -p $COMMIT_HASH
# 输出类似：
# tree 8d9e0f1a2b3c...
# parent （首次提交没有 parent）
# author ...
# committer ...
# 初始提交

# 第三步：查看提交指向的树对象
TREE_HASH=$(git rev-parse HEAD^{tree})
echo "=== 根树对象 ==="
git cat-file -p $TREE_HASH
# 输出类似：
# 100644 blob a1b2c3d4...    README.md
# 040000 tree e5f6g7h8...    src

# 第四步：查看 src 子目录的树对象
SRC_TREE=$(git ls-tree HEAD src | awk '{print $3}')
echo "=== src 目录树对象 ==="
git cat-file -p $SRC_TREE
# 输出类似：
# 100644 blob i9j0k1l2...    main.py

# 第五步：查看文件内容（blob 对象）
BLOB_HASH=$(git ls-tree -r HEAD src/main.py | awk '{print $3}')
echo "=== main.py 的 blob 对象 ==="
git cat-file -p $BLOB_HASH
# 输出：print('Hello')

# 完整的对象关系链：
echo "
提交 $COMMIT_HASH
  └── 树 $TREE_HASH (根目录)
        ├── blob a1b2c3d4 (README.md) -> '# Git 实验'
        └── 树 $SRC_TREE (src/)
              └── blob i9j0k1l2 (main.py) -> 'print(\"Hello\")'
"
```

这个实验展示了一个关键事实：**一次提交就是一棵完整的对象树**。当你检出某个提交时，Git 只需要遍历这棵树，将所有 blob 对象的内容写回工作区，就能完整还原项目在那个时刻的状态。

### 对象去重机制实验

Git 的内容寻址特性天然支持去重。让我们通过实验来验证：

```bash
# 创建两个内容相同的文件
echo "相同的内容" > file1.txt
echo "相同的内容" > file2.txt

# 将两个文件添加到暂存区
git add file1.txt file2.txt

# 查看暂存区
git ls-files --stage

# 输出：
# 100644 x1y2z3w4... 0	file1.txt
# 100644 x1y2z3w4... 0	file2.txt

# 注意：两个文件指向同一个 blob 对象！
# Git 只存储了一份内容，这就是内容寻址的去重优势

# 再创建一个内容不同的文件
echo "不同的内容" > file3.txt
git add file3.txt

git ls-files --stage
# 输出：
# 100644 x1y2z3w4... 0	file1.txt
# 100644 x1y2z3w4... 0	file2.txt
# 100644 m5n6o7p8... 0	file3.txt
# file3.txt 指向不同的 blob 对象
```

这个实验证明了 Git 的两个重要特性：

| 特性 | 说明 |
|------|------|
| 内容决定地址 | 相同内容产生相同哈希，无论文件名是什么 |
| 自动去重 | 相同内容只存储一份，节省磁盘空间 |

### Git 对象的不可变性

Git 对象一旦创建就不可修改。如果你修改了文件内容，Git 会创建一个全新的对象：

```bash
# 创建一个文件并提交
echo "版本 1" > version.txt
git add version.txt
git commit -m "添加 version.txt"

# 记录当前的 blob 哈希
git ls-files --stage version.txt
# 输出：100644 aabbccdd... 0	version.txt

# 修改文件内容
echo "版本 2" > version.txt
git add version.txt

# 再次查看 blob 哈希
git ls-files --stage version.txt
# 输出：100644 eeff0011... 0	version.txt

# 哈希值变了！原来的 blob (aabbccdd) 仍然存在于对象数据库中
# 新的 blob (eeff0011) 是一个全新的对象

# 查看原始 blob（它并没有被删除）
git cat-file -p aabbccdd
# 输出：版本 1

# 查看新 blob
git cat-file -p eeff0011
# 输出：版本 2
```

不可变性的重要意义：

```
┌─────────────────────────────────────────────────────────────────┐
│                  Git 对象不可变性的意义                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. 数据安全性                                                   │
│     • 一旦提交，内容无法被篡改                                    │
│     • 任何修改都会产生新的哈希值                                  │
│     • 可以通过哈希值验证数据完整性                                │
│                                                                 │
│  2. 历史可追溯                                                   │
│     • 每个版本都被永久保存                                        │
│     • 可以随时查看任何历史版本                                    │
│     • 提交之间通过父指针形成完整的链                              │
│                                                                 │
│  3. 并发安全                                                     │
│     • 不同分支可以同时操作，互不影响                               │
│     • 合并时通过哈希比较检测冲突                                  │
│                                                                 │
│  4. 高效存储                                                     │
│     • 相同内容自动去重                                           │
│     • 打包时可以高效压缩相似对象                                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 引用日志（Reflog）

Git 还有一个常被忽略但非常有用的功能——引用日志。它记录了 HEAD 和分支引用的每一次变更：

```bash
# 查看 HEAD 的引用日志
git reflog

# 输出示例：
# a1b2c3d HEAD@{0}: commit: 添加新功能
# e4f5g6h HEAD@{1}: checkout: moving from feature to main
# i7j8k9l HEAD@{2}: commit: 修复 bug
# m3n4o5p HEAD@{3}: commit: 初始提交

# 查看特定分支的引用日志
git reflog main

# 引用日志的用途：
# 1. 恢复误删的提交
git checkout HEAD@{2}    # 回到引用日志中的某个位置

# 2. 恢复误删的分支
git branch recovered-branch HEAD@{3}

# 3. 查看某段时间的操作历史
git reflog --since="2 days ago"
```

引用日志只保存在本地，不会被推送到远程仓库。默认保留期限为 90 天（可达引用）和 30 天（不可达引用）。

```
引用日志的工作原理：

HEAD 变更历史（时间从右到左）：

m3n4o5p ──▶ i7j8k9l ──▶ e4f5g6h ──▶ a1b2c3d
初始提交     修复 bug     切换分支     添加新功能
HEAD@{3}    HEAD@{2}     HEAD@{1}    HEAD@{0}

每当你执行提交、合并、重置、切换分支等操作时，
Git 都会在引用日志中添加一条新记录。
```

引用日志是 Git 的一道安全网。即使你不小心执行了 `git reset --hard` 丢失了提交，只要引用日志还在，你就有机会通过 `git reflog` 找回那些"丢失"的提交哈希，然后用 `git cherry-pick` 或 `git reset` 恢复它们。

---

## 总结

### 关键要点回顾

```
┌─────────────────────────────────────────────────────────────────┐
│                    Git 核心概念总结                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. 分布式架构                                                   │
│     • 每个开发者都有完整的仓库副本                                │
│     • 大多数操作可以离线完成                                     │
│                                                                 │
│  2. 三个工作区域                                                 │
│     • 工作区：你编辑文件的地方                                    │
│     • 暂存区：准备提交的变更                                     │
│     • 版本库：已提交的历史记录                                    │
│                                                                 │
│  3. 四种文件状态                                                 │
│     • 未跟踪、已暂存、已提交、已修改                              │
│                                                                 │
│  4. 对象模型                                                     │
│     • Blob：存储文件内容                                         │
│     • Tree：表示目录结构                                         │
│     • Commit：记录提交信息                                       │
│     • Tag：标记特定版本                                          │
│                                                                 │
│  5. 快照机制                                                     │
│     • 每个提交保存完整的项目快照                                  │
│     • 通过打包和压缩优化存储空间                                  │
│                                                                 │
│  6. 内容寻址                                                     │
│     • 使用 SHA-1 哈希标识所有对象                                │
│     • 保证内容完整性和去重                                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 学习建议

1. **多动手实践**：理论结合实践，通过实际操作加深理解。不要只是阅读，尝试在本地仓库中运行本章介绍的所有命令
2. **阅读官方文档**：Git 官方文档（https://git-scm.com/doc）是最权威的学习资源，特别是《Pro Git》一书的 Git 内部原理章节
3. **使用图形化工具**：如 GitKraken、SourceTree、Git GUI 等，可以帮助你可视化理解 Git 的对象模型和提交历史
4. **参与开源项目**：通过实际项目练习 Git 工作流，在真实场景中体会分布式版本控制的优势
5. **遇到问题不要慌**：Git 几乎所有操作都是可逆的，善用 `git reflog` 可以找回大多数"丢失"的提交
6. **理解而非记忆**：与其死记硬背命令，不如理解背后的原理。一旦理解了对象模型和引用机制，命令自然就记住了

---

## 下一步

现在你已经深入理解了 Git 的工作原理，接下来让我们学习如何创建和克隆仓库：

[创建与克隆仓库 →](07-init-clone.md)
