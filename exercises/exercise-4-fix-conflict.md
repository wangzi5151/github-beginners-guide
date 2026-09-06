# 练习 4：修复 Merge Conflict

## 目标

学会识别和解决 Git 合并冲突。

## 步骤

### 1. 创建练习仓库

```bash
mkdir conflict-practice
cd conflict-practice
git init

# 创建初始文件
echo "Hello World" > greeting.txt
git add greeting.txt
git commit -m "Initial commit"

git remote add origin git@github.com:你的用户名/conflict-practice.git
git push -u origin main
```

### 2. 模拟冲突场景

```bash
# 创建两个分支
git checkout -b alice-branch
git checkout main
git checkout -b bob-branch

# Alice 的修改
git checkout alice-branch
echo "Hello Alice" > greeting.txt
git add greeting.txt
git commit -m "Alice: 修改问候语"

# Bob 的修改
git checkout bob-branch
echo "Hello Bob" > greeting.txt
git add greeting.txt
git commit -m "Bob: 修改问候语"

# 推送分支
git push -u origin alice-branch
git push -u origin bob-branch
```

### 3. 创建冲突的 PR

在 GitHub 上：
1. 从 `alice-branch` 创建 PR 到 `main`
2. 合并 PR
3. 从 `bob-branch` 创建 PR 到 `main`

这时会发现 PR 有冲突。

### 4. 解决冲突

#### 方法一：本地解决

```bash
# 更新 main 分支
git checkout main
git pull

# 尝试合并 bob-branch
git merge bob-branch
# 这里会产生冲突

# 查看冲突文件
git status

# 打开 greeting.txt，你会看到：
# <<<<<<< HEAD
# Hello Alice
# =======
# Hello Bob
# >>>>>>> bob-branch
```

#### 方法二：编辑文件解决

打开 `greeting.txt`，选择保留的内容：

```text
Hello Alice and Bob
```

#### 方法三：使用工具解决

```bash
# 使用 VS Code
code greeting.txt

# 在 VS Code 中，冲突会高亮显示
# 可以使用 "Accept Current"、"Accept Incoming"、"Accept Both" 按钮
```

### 5. 完成合并

```bash
# 标记为已解决
git add greeting.txt

# 完成提交
git commit -m "解决 Alice 和 Bob 的合并冲突"

# 推送到远程
git push
```

### 6. 在 PR 中解决

如果在 PR 中发现冲突：

```bash
# 在本地解决
git checkout bob-branch
git merge main
# 解决冲突
git add greeting.txt
git commit -m "解决与 main 分支的冲突"
git push
```

PR 会自动更新。

## 冲突标记详解

```text
<<<<<<< HEAD (当前分支的内容)
这是当前分支的修改
=======
这是要合并的分支的内容
>>>>>>> branch-name
```

## 高级技巧

### 使用 rebase 避免冲突

```bash
# 在功能分支上
git checkout alice-branch
git rebase main
# 解决冲突后
git rebase --continue
```

### 放弃合并

```bash
# 放弃 merge
git merge --abort

# 放弃 rebase
git rebase --abort
```

### 预防冲突

1. **频繁同步**：定期从 main 拉取
2. **小步提交**：减少冲突范围
3. **沟通协作**：避免同时修改同一文件

## 知识点

- 冲突标记的含义
- 手动解决冲突的方法
- 使用工具解决冲突
- `git merge --abort` 放弃合并
- `git rebase` 避免冲突

## 下一步

[练习 5：使用 GitHub Pages 部署网站 →](exercise-5-github-pages.md)
