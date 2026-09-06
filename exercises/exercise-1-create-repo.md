# 练习 1：创建你的第一个仓库

## 目标

学习如何在 GitHub 上创建仓库并进行基本操作。

## 步骤

### 1. 在 GitHub 上创建仓库

1. 登录 GitHub
2. 点击右上角 **+** → **New repository**
3. 填写信息：
   - Repository name: `my-first-repo`
   - Description: "我的第一个 GitHub 仓库"
   - 选择 **Public**
   - ✅ Add a README file
4. 点击 **Create repository**

### 2. 克隆仓库到本地

```bash
git clone git@github.com:你的用户名/my-first-repo.git
cd my-first-repo
```

### 3. 创建文件并提交

```bash
# 创建新文件
echo "# My First Repo" > index.html

# 查看状态
git status

# 添加文件
git add index.html

# 提交
git commit -m "feat: 添加首页文件"

# 推送
git push
```

### 4. 创建新分支

```bash
# 创建并切换到新分支
git checkout -b feature/add-about

# 创建 About 页面
echo "<h1>About Me</h1>" > about.html

# 提交并推送
git add about.html
git commit -m "feat: 添加关于页面"
git push -u origin feature/add-about
```

### 5. 创建 Pull Request

1. 在 GitHub 上打开仓库
2. 点击 **Pull requests**
3. 点击 **New pull request**
4. 选择 `feature/add-about` 分支
5. 填写标题和描述
6. 点击 **Create pull request**

### 6. 合并 Pull Request

1. 等待检查通过
2. 点击 **Merge pull request**
3. 点击 **Confirm merge**

## 验证

完成上述步骤后，你应该看到：
- GitHub 上有一个名为 `my-first-repo` 的仓库
- 仓库中有 `index.html` 和 `about.html` 文件
- 有一个已合并的 Pull Request

## 下一步

[练习 2：分支与合并练习 →](exercise-2-branch-merge.md)
