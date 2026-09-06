# 练习 10：Interactive Rebase 实战

## 学习目标

- 掌握 `git rebase -i` 的使用
- 学会压缩提交
- 理解重排提交顺序
- 学会修改提交信息

## 场景描述

你在开发过程中创建了多个小提交，现在需要在提交 PR 前整理提交历史，使其更清晰。

## 步骤

### 步骤 1：创建测试仓库

```bash
mkdir rebase-practice
cd rebase-practice
git init
```

### 步骤 2：创建初始提交

```bash
echo "# 项目" > README.md
git add .
git commit -m "feat: 初始化项目"
```

### 步骤 3：创建多个提交

```bash
# 提交 1
echo "功能 1" > feature1.js
git add .
git commit -m "feat: 添加功能 1"

# 提交 2（修复拼写错误）
echo "功能 1 修复" >> feature1.js
git add .
git commit -m "fix: 修复功能 1 拼写错误"

# 提交 3
echo "功能 2" > feature2.js
git add .
git commit -m "feat: 添加功能 2"

# 提交 4（空格修复）
echo "功能 2 " >> feature2.js
git add .
git commit -m "chore: 修复空格"

# 提交 5
echo "功能 3" > feature3.js
git add .
git commit -m "feat: 添加功能 3"
```

### 步骤 4：查看提交历史

```bash
git log --oneline
```

### 步骤 5：启动 Interactive Rebase

```bash
# 重新排列最近 5 个提交
git rebase -i HEAD~5
```

## Interactive Rebase 命令

### 常用命令

| 命令 | 说明 |
|------|------|
| `pick` | 保留提交 |
| `reword` | 保留提交，修改提交信息 |
| `edit` | 暂停提交，允许修改内容 |
| `squash` | 压缩提交到前一个提交 |
| `fixup` | 压缩提交到前一个提交，丢弃提交信息 |
| `drop` | 删除提交 |

### 操作示例

编辑器会打开并显示：

```
pick abc1234 feat: 添加功能 1
pick def5678 fix: 修复功能 1 拼写错误
pick ghi9012 feat: 添加功能 2
pick jkl3456 chore: 修复空格
pick mno7890 feat: 添加功能 3
```

修改为：

```
pick abc1234 feat: 添加功能 1
fixup def5678 fix: 修复功能 1 拼写错误
pick ghi9012 feat: 添加功能 2
fixup jkl3456 chore: 修复空格
pick mno7890 feat: 添加功能 3
```

## 实战任务

### 任务 1：压缩提交

1. 创建 3 个相关提交
2. 使用 `squash` 将它们压缩成一个提交
3. 编写新的提交信息

```bash
# 创建测试提交
echo "代码 1" > file1.js
git add . && git commit -m "feat: 添加文件 1"

echo "代码 2" > file2.js
git add . && git commit -m "feat: 添加文件 2"

echo "代码 3" > file3.js
git add . && git commit -m "feat: 添加文件 3"

# 压缩提交
git rebase -i HEAD~3
```

### 任务 2：重排提交顺序

1. 创建 3 个独立的提交
2. 使用 interactive rebase 重排顺序
3. 验证顺序变化

### 任务 3：修改提交信息

1. 创建一个提交
2. 使用 `reword` 修改提交信息
3. 验证信息已更新

### 任务 4：编辑提交内容

1. 创建一个提交
2. 使用 `edit` 暂停提交
3. 修改文件内容
4. 继续 rebase

```bash
git rebase -i HEAD~1
# 将 pick 改为 edit
# 保存退出

# 修改文件
echo "修改后的内容" > file.js
git add .
git commit --amend

# 继续 rebase
git rebase --continue
```

## 高级技巧

### 使用 rebase 合并分支

```bash
# 将 feature 分支变基到 main
git checkout feature
git rebase main

# 解决冲突后继续
git rebase --continue

# 如果想放弃
git rebase --abort
```

### 自动化 rebase

```bash
# 自动压缩最近 3 个提交
GIT_SEQUENCE_EDITOR="sed -i '2s/pick/fixup/'" git rebase -i HEAD~3
```

## 验证清单

- [ ] 能够启动 interactive rebase
- [ ] 能够使用 `pick`、`squash`、`fixup` 命令
- [ ] 能够重排提交顺序
- [ ] 能够修改提交信息
- [ ] 能够编辑提交内容
- [ ] 理解 `rebase --continue` 和 `rebase --abort`

## 常见问题

### Q: rebase 和 merge 的区别？
A: rebase 会重写提交历史，产生线性历史；merge 会保留原始提交历史。

### Q: 什么时候不应该 rebase？
A: 不应该对已经推送到远程的公共分支进行 rebase。

### Q: 如何处理 rebase 冲突？
A: 解决冲突后使用 `git add` 标记已解决，然后使用 `git rebase --continue`。

## 下一步

完成本练习后，请继续进行 [练习 11：GitHub CLI 深入使用](exercise-11-github-cli.md)
