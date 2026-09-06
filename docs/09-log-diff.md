# 查看历史与差异

## 查看提交历史

### 基本日志

```bash
git log
```

### 单行显示

```bash
git log --oneline
```

### 图形化显示

```bash
git log --oneline --graph --all
```

### 显示最近 N 次提交

```bash
git log -5
```

### 显示每次提交的差异

```bash
git log -p
```

### 按作者筛选

```bash
git log --author="张三"
```

### 按日期筛选

```bash
git log --after="2024-01-01"
git log --before="2024-12-31"
```

### 按提交信息筛选

```bash
git log --grep="fix"
```

### 查看特定文件的历史

```bash
git log -- path/to/file
```

## 查看差异

### 工作区与暂存区的差异

```bash
git diff
```

### 暂存区与最新提交的差异

```bash
git diff --staged
# 或
git diff --cached
```

### 两个提交之间的差异

```bash
git diff commit1 commit2
```

### 两个分支之间的差异

```bash
git diff branch1..branch2
```

### 查看特定文件的差异

```bash
git diff -- filename.txt
```

### 统计差异

```bash
git diff --stat
```

### 只看文件名

```bash
git diff --name-only
```

## 查看特定提交

```bash
# 查看提交详情
git show commit-hash

# 查看提交的文件列表
git show --stat commit-hash
```

## 搜索历史

```bash
# 搜索提交信息
git log --grep="搜索关键词"

# 搜索代码变更
git log -S "搜索关键词"

# 搜索代码内容
git grep "搜索关键词"
```

## 有用的组合命令

```bash
# 美化日志
git log --pretty=format:"%h %s" --graph

# 查看最近 10 次提交的文件变更
git log --stat -10
```

## 下一步

[分支操作 →](10-branching.md)
