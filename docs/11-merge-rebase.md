# 合并与变基

## 什么是合并和变基？

- **合并（Merge）**：将两个分支的更改合并到一起
- **变基（Rebase）**：将当前分支的提交"重放"到目标分支上

## 在 GitHub 网页合并

### 方法一：通过 PR 合并（推荐）

这是最安全的合并方式，适合团队协作。

**第 1 步：创建 PR**
1. 推送功能分支到远程
2. 在仓库页面点击 **Compare & pull request**

**第 2 步：审查代码**
1. 查看代码差异
2. 添加评论
3. 点击 **Approve** 批准

**第 3 步：合并 PR**
1. 在 PR 页面底部找到合并按钮
2. 选择合并方式
3. 点击 **Merge pull request**
4. 点击 **Confirm merge**

```
┌─────────────────────────────────────────────┐
│  Merge pull request                          │
│                                             │
│  All checks have passed                      │
│                                             │
│  ┌─────────────────────────────────────┐    │
│  │  ○ Create a merge commit            │    │
│  │    保留完整提交历史                   │    │
│  │                                     │    │
│  │  ○ Squash and merge                 │    │
│  │    压缩为一个提交                    │    │
│  │                                     │    │
│  │  ○ Rebase and merge                 │    │
│  │    线性历史                          │    │
│  └─────────────────────────────────────┘    │
│                                             │
│  Merge message:                             │
│  ┌─────────────────────────────────────┐    │
│  │ feat: 添加新按钮功能                 │    │
│  └─────────────────────────────────────┘    │
│                                             │
│        [Merge pull request]                 │
└─────────────────────────────────────────────┘
```

**合并方式说明：**

| 方式 | 特点 | 适用场景 |
|------|------|----------|
| **Create a merge commit** | 保留完整历史 | 默认方式，推荐 |
| **Squash and merge** | 压缩为一个提交 | 功能分支有很多小提交 |
| **Rebase and merge** | 线性历史 | 喜欢干净的历史 |

### 方法二：直接在仓库合并（不推荐）

⚠️ **注意：直接合并到主分支可能会破坏分支保护规则**

## 在终端合并

### 快进合并

当目标分支没有新提交时：

```bash
git checkout main
git merge feature
```

```
合并前：
main:    A --- B --- C
                      \
feature:               D --- E

合并后：
main:    A --- B --- C --- D --- E
```

### 三方合并

当两个分支都有新提交时：

```bash
git checkout main
git merge feature
```

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

### 禁用快进合并

```bash
git merge --no-ff feature
```

## 变基 (Rebase)

### 基本变基

```bash
git checkout feature
git rebase main
```

```
变基前：
main:    A --- B --- C
                      \
feature:               D --- E

变基后：
main:    A --- B --- C
                      \
feature:               D' --- E'
```

### 交互式变基

```bash
git rebase -i HEAD~3
```

打开编辑器，可以选择：
- `pick`：保留提交
- `reword`：修改提交信息
- `edit`：修改提交内容
- `squash`：压缩到前一个提交
- `drop`：删除提交

```
pick abc1234 feat: 添加按钮样式
pick def5678 feat: 实现按钮功能
pick ghi9012 fix: 修复按钮样式问题
```

修改为：
```
pick abc1234 feat: 添加按钮样式
squash def5678 feat: 实现按钮功能
squash ghi9012 fix: 修复按钮样式问题
```

这样会将三个提交压缩为一个。

## 合并 vs 变基

| 特性 | 合并 | 变基 |
|------|------|------|
| 历史 | 保留完整历史 | 线性历史 |
| 提交哈希 | 保留原哈希 | 生成新哈希 |
| 复杂度 | 简单 | 需要更多操作 |
| 冲突 | 一次性解决 | 可能需要多次解决 |
| 推荐场景 | 公共分支 | 个人/功能分支 |

## 最佳实践

```bash
# 功能分支合并到 main：使用 merge
git checkout main
git merge --no-ff feature

# 更新功能分支：使用 rebase
git checkout feature
git rebase main
```

**黄金规则：不要对已推送到远程的提交进行变基**

## 取消操作

```bash
# 取消合并
git merge --abort

# 取消变基
git rebase --abort
```

## 解决冲突

当合并或变基出现冲突时：

**第 1 步：查看冲突文件**

```bash
git status
```

**第 2 步：编辑冲突文件**

打开有冲突的文件，你会看到类似这样的标记：

```
<<<<<<< HEAD
这是当前分支的内容
=======
这是要合并的分支的内容
>>>>>>> feature
```

**第 3 步：解决冲突**

手动编辑文件，删除冲突标记，保留正确的内容：

```
这是合并后的内容
```

**第 4 步：标记冲突已解决**

```bash
git add 文件名
git commit  # 合并时
# 或
git rebase --continue  # 变基时
```

---

## 实践练习

### 练习：合并和变基

**任务 1：创建测试分支**
```bash
git checkout main
git checkout -b feature-test
echo "功能代码" > feature.txt
git add feature.txt
git commit -m "feat: 添加功能"
```

**任务 2：切换回 main 并修改**
```bash
git checkout main
echo "main 的修改" >> main.txt
git add main.txt
git commit -m "fix: 修改 main"
```

**任务 3：合并分支**
```bash
git merge feature-test
```

**任务 4：解决冲突（如果有）**
1. 查看冲突文件
2. 编辑文件解决冲突
3. 提交合并结果

**验证方法：**
- `git log --oneline` 显示合并后的提交历史
- 文件内容正确

## 下一步

[解决冲突 →](12-resolve-conflicts.md)
