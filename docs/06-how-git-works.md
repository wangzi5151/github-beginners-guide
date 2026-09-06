# Git 工作原理

## 三个区域

Git 仓库包含三个主要区域：

```
工作区 (Working Directory)  →  暂存区 (Staging Area)  →  仓库 (Repository)
     修改文件                    git add                    git commit
```

| 区域 | 说明 |
|------|------|
| 工作区 | 你编辑文件的地方 |
| 暂存区 | 准备提交的变更 |
| 仓库 | 已提交的历史记录 |

## 文件状态

```
Untracked → Staged → Committed → Modified → Staged → ...
```

- **Untracked**：新文件，未被 Git 追踪
- **Staged**：已暂存，等待提交
- **Committed**：已提交到仓库
- **Modified**：已修改但未暂存

## 对象模型

Git 使用三种核心对象：

```
Commit 对象
    ↓
Tree 对象（目录结构）
    ↓
Blob 对象（文件内容）
```

每次提交都是一个快照，而非差异。

## 提交哈希

每次提交都有唯一的 SHA-1 哈希值：

```
a1b2c3d4e5f6...（40位十六进制数）
```

可以用简写：`a1b2c3d`

## 分支本质

分支只是一个指向提交的可移动指针：

```
main → a1b2c3d
feature → e5f6g7h
HEAD → main
```

## HEAD 指针

HEAD 指向当前所在的分支：

```
HEAD → main → a1b2c3d
```

## 远程追踪分支

```
origin/main → 远程 main 分支的本地追踪副本
origin/feature → 远程 feature 分支的本地追踪副本
```

## 工作流程图

```
1. 修改文件
       ↓
2. git add（暂存）
       ↓
3. git commit（提交）
       ↓
4. git push（推送到远程）
```

## 下一步

[创建与克隆仓库 →](07-init-clone.md)
