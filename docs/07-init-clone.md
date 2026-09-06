# 创建与克隆仓库

## 创建新仓库

### 方法一：命令行创建

```bash
# 创建新目录
mkdir my-project
cd my-project

# 初始化 Git 仓库
git init

# 创建第一个文件
echo "# My Project" > README.md

# 添加到暂存区
git add README.md

# 提交
git commit -m "Initial commit"
```

### 方法二：GitHub 上创建

1. 登录 GitHub
2. 点击右上角 **+** → **New repository**
3. 填写仓库名称和描述
4. 选择公开或私有
5. 勾选 **Add a README file**
6. 点击 **Create repository**

### 方法三：使用 GitHub CLI

```bash
# 创建远程仓库
gh repo create my-project --public

# 创建并克隆到本地
gh repo create my-project --public --clone
```

## 克隆现有仓库

### SSH 方式（推荐）

```bash
git clone git@github.com:user/repo.git
```

### HTTPS 方式

```bash
git clone https://github.com/user/repo.git
```

### 指定目录

```bash
git clone git@github.com:user/repo.git my-local-name
```

### 克隆特定分支

```bash
git clone -b feature-branch git@github.com:user/repo.git
```

### 浅克隆（只获取最新提交）

```bash
git clone --depth 1 git@github.com:user/repo.git
```

## 远程仓库操作

```bash
# 查看远程仓库
git remote -v

# 添加远程仓库
git remote add origin git@github.com:user/repo.git

# 修改远程仓库 URL
git remote set-url origin git@github.com:user/new-repo.git

# 删除远程仓库
git remote remove origin
```

## 推送本地仓库到 GitHub

```bash
# 首次推送
git push -u origin main

# 后续推送
git push
```

## 下一步

[暂存与提交 →](08-add-commit.md)
