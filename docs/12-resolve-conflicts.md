# 解决冲突

## 什么是冲突？

当两个分支修改了同一文件的同一位置时，Git 无法自动合并，就会产生冲突。

```
<<<<<<< HEAD (当前分支)
当前分支的内容
=======
要合并的分支的内容
>>>>>>> feature
```

## 冲突解决步骤

### 1. 发现冲突

```bash
git merge feature
# Auto-merging file.txt
# CONFLICT (content): Merge conflict in file.txt
# Automatic merge failed; fix conflicts and then commit the result.
```

### 2. 查看冲突文件

```bash
git status
# Unmerged paths:
#   both modified: file.txt
```

### 3. 编辑冲突文件

打开文件，找到冲突标记：

```text
这是一些公共内容
<<<<<<< HEAD
这是当前分支的修改
=======
这是 feature 分支的修改
>>>>>>> feature
这是更多公共内容
```

### 4. 选择保留内容

手动编辑，删除冲突标记，保留需要的内容：

```text
这是一些公共内容
这是最终保留的内容
这是更多公共内容
```

### 5. 标记为已解决

```bash
git add file.txt
```

### 6. 完成合并

```bash
git commit -m "解决 file.txt 的冲突"
```

## 使用工具解决冲突

### VS Code
- 冲突文件会显示在编辑器中
- 有 "Accept Current"、"Accept Incoming"、"Accept Both" 按钮

### 命令行工具

```bash
# 使用 git 自带的 merge tool
git mergetool
```

### 配置 VS Code 为合并工具

```bash
git config --global merge.tool vscode
git config --global mergetool.vscode.cmd 'code --wait $MERGED'
```

## 取消合并

```bash
git merge --abort
```

## 预防冲突

1. **频繁同步**：定期从 main 拉取更新
   ```bash
   git checkout feature
   git rebase main
   ```

2. **小步提交**：频繁提交减少冲突范围

3. **沟通协作**：团队成员避免同时修改同一文件

4. **代码 ownership**：明确文件负责人

## 冲突解决示例

```bash
# 假设冲突在 src/app.js

# 1. 编辑文件解决冲突
vim src/app.js

# 2. 暂存解决后的文件
git add src/app.js

# 3. 继续变基（如果是 rebase 过程中）
git rebase --continue

# 或完成合并（如果是 merge 过程中）
git commit
```

## 下一步

[撤销操作 →](13-undo.md)
