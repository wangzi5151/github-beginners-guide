# 分支操作

## 基本概念

分支是平行开发的独立线路。主分支通常是 `main`（或 `master`）。

```
main:     A --- B --- C
                   \
feature:            D --- E
```

**简单理解：**
- **main 分支**：主分支，稳定版本
- **feature 分支**：功能分支，开发新功能
- **bugfix 分支**：修复分支，修复 bug
- **hotfix 分支**：紧急修复分支

## 在 GitHub 网页查看分支

### 查看所有分支

1. 打开仓库页面
2. 点击分支下拉框（通常显示 `main`）

```
┌─────────────────────────────────────────────┐
│  Code  Issues  Pull requests                │
├─────────────────────────────────────────────┤
│                                             │
│  [main ▼]  ← 点击这个下拉框                 │
│                                             │
│  ┌─────────────────────────────────────┐    │
│  │ Switch branches/tags                │    │
│  │                                     │    │
│  │ Branches                            │    │
│  │   main                              │    │
│  │   feature-login                     │    │
│  │   feature-new-button                │    │
│  │                                     │    │
│  │ Tags                                │    │
│  │   v1.0.0                            │    │
│  │   v1.0.1                            │    │
│  └─────────────────────────────────────┘    │
│                                             │
└─────────────────────────────────────────────┘
```

### 在 GitHub 创建分支

**方法一：在仓库页面创建**

1. 点击分支下拉框
2. 在搜索框输入新分支名称
3. 点击 **Create branch: xxx from main**

```
┌─────────────────────────────────────┐
│  Switch branches/tags                │
│                                     │
│  Find or create a branch...         │
│  ┌─────────────────────────────┐    │
│  │ feature-new-button          │    │
│  └─────────────────────────────┘    │
│                                     │
│  ✓ Create branch:                   │
│    feature-new-button from main     │
└─────────────────────────────────────┘
```

**方法二：在 Push 后自动创建**

当你推送一个新分支到远程时，GitHub 会显示创建 PR 的提示：

```
┌─────────────────────────────────────────────┐
│  feature-new-button had recent pushes       │
│                                             │
│  [Compare & pull request]  ← 点击创建 PR     │
└─────────────────────────────────────────────┘
```

## 在终端操作分支

### 查看分支

```bash
# 查看本地分支
git branch

# 查看所有分支（含远程）
git branch -a

# 查看分支及最后一次提交
git branch -v
```

**输出示例：**
```
* main          abc1234 最新提交信息
  feature-login def5678 添加登录功能
  feature-cart  ghi9012 购物车功能
```

带 `*` 号的是当前分支。

### 创建分支

```bash
# 创建新分支
git branch feature-login

# 创建并切换到新分支（推荐）
git checkout -b feature-login

# 或使用新语法
git switch -c feature-login
```

### 切换分支

```bash
# 切换到已有分支
git checkout feature-login

# 或使用新语法
git switch feature-login
```

### 合并分支

```bash
# 切换到目标分支
git checkout main

# 合并功能分支
git merge feature-login
```

### 删除分支

```bash
# 删除已合并的分支
git branch -d feature-login

# 强制删除分支
git branch -D feature-login

# 删除远程分支
git push origin --delete feature-login
```

## 分支工作流

### 完整工作流示例

```
1. 创建功能分支
   git checkout main
   git pull
   git checkout -b feature-new-button

2. 在功能分支上工作
   修改文件...
   git add .
   git commit -m "feat: 添加新按钮"

3. 推送到远程
   git push -u origin feature-new-button

4. 在 GitHub 上创建 PR
   打开仓库页面
   点击 "Compare & pull request"
   填写信息并创建

5. 代码审查通过后合并
   在 GitHub 网页点击 "Merge pull request"

6. 删除功能分支
   在 GitHub 网页点击 "Delete branch"
```

## 分支保护（网页操作）

在 GitHub 上可以设置分支保护规则，防止直接推送到主分支。

### 设置步骤

1. 打开仓库页面
2. 点击 **Settings** 标签
3. 左侧菜单找到 **Branches**
4. 点击 **Add rule**

```
┌─────────────────────────────────────────────┐
│  Branch protection rules                     │
│                                             │
│  [Add rule]  ← 点击这个按钮                  │
│                                             │
│  Rules:                                      │
│  (暂无规则)                                   │
└─────────────────────────────────────────────┘
```

### 配置规则

```
┌─────────────────────────────────────────────┐
│  Branch protection rules                     │
│                                             │
│  Branch name pattern: [main           ]      │
│                                             │
│  ☑ Require a pull request before merging    │
│    ☑ Required number of approvals: [1  ▼]   │
│    ☑ Dismiss stale pull request approvals   │
│    ☑ Require review from Code Owners        │
│                                             │
│  ☑ Require status checks to pass            │
│    ☑ Require branches to be up to date      │
│    Status checks: [ci/test] [ci/lint]       │
│                                             │
│  ☑ Require conversation resolution          │
│                                             │
│  ☑ Do not allow bypassing the above settings│
│                                             │
│             [Create]                         │
└─────────────────────────────────────────────┘
```

**配置说明：**

| 选项 | 说明 |
|------|------|
| **Branch name pattern** | 分支名称，如 `main` |
| **Require a pull request** | 必须通过 PR 才能合并 |
| **Required approvals** | 需要多少人批准 |
| **Require status checks** | CI 检查必须通过 |
| **Require conversation resolution** | 对话必须解决 |

## 分支策略

### 推荐策略

```
main (主分支)
├── feature/xxx (功能分支)
├── bugfix/xxx (修复分支)
└── hotfix/xxx (紧急修复)
```

### 命名规范

| 分支类型 | 命名格式 | 示例 |
|----------|----------|------|
| 功能 | feature/描述 | feature/user-login |
| 修复 | bugfix/描述 | bugfix/fix-crash |
| 紧急修复 | hotfix/描述 | hotfix/security-patch |
| 文档 | docs/描述 | docs/update-readme |

---

## 实践练习

### 练习：分支操作

**任务 1：创建分支**
1. 打开仓库页面
2. 点击分支下拉框
3. 输入新分支名称
4. 创建分支

**任务 2：在终端操作**
```bash
# 克隆仓库
git clone 仓库地址
cd 仓库目录

# 查看分支
git branch

# 创建并切换分支
git checkout -b feature-test

# 修改文件并提交
echo "test" > test.txt
git add test.txt
git commit -m "测试分支"

# 切换回 main
git checkout main
```

**任务 3：合并分支**
```bash
# 合并功能分支
git merge feature-test

# 查看合并结果
git log --oneline
```

**验证方法：**
- 分支列表中可以看到创建的分支
- 合并后 main 分支包含功能分支的内容

## 下一步

[合并与变基 →](11-merge-rebase.md)
