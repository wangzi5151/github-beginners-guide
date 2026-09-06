# 创建和管理仓库

## 在 GitHub 上创建仓库（图文详解）

### 方法一：网页创建（推荐新手）

**第 1 步：登录 GitHub**
1. 打开浏览器访问 **github.com**
2. 输入用户名/邮箱和密码登录

**第 2 步：进入创建页面**
1. 登录后，在任意 GitHub 页面
2. 点击右上角的 **+** 号（在头像左边）
3. 在下拉菜单中点击 **New repository**

```
┌─────────────────────────────────────────────┐
│  +  [🔔]  [头像]                             │
│  │                                          │
│  └─→ New repository  ← 点击这个              │
│      Import repository                      │
│      New organization                        │
└─────────────────────────────────────────────┘
```

**第 3 步：填写仓库信息**

你会看到创建仓库的表单，逐项填写：

```
┌─────────────────────────────────────────────┐
│  Create a new repository                     │
│                                             │
│  Owner: [你的用户名 ▼]                       │
│                                             │
│  Repository name: [my-project          ]     │
│  (必填，只能包含字母、数字、连字符、下划线)      │
│                                             │
│  Description: [这是一个示例项目        ]      │
│  (可选，简短描述仓库用途)                      │
│                                             │
│  ○ Public  ← 任何人都可以看到                   │
│  ○ Private ← 只有你和协作者可以看到             │
│                                             │
│  ☑ Add a README file  ← 强烈推荐勾选          │
│                                             │
│  Add .gitignore: [None ▼] ← 选择模板          │
│                                             │
│  Choose a license: [None ▼] ← 选择许可证      │
│                                             │
│        [Create repository]  ← 点击创建        │
└─────────────────────────────────────────────┘
```

**各项说明：**

| 选项 | 说明 | 建议 |
|------|------|------|
| **Repository name** | 仓库名称 | 使用英文，如 `my-project` |
| **Description** | 仓库描述 | 简短说明项目用途 |
| **Public** | 公开仓库 | 开源项目选这个 |
| **Private** | 私有仓库 | 个人项目选这个 |
| **Add a README** | 添加说明文件 | 强烈推荐勾选 |
| **.gitignore** | 忽略文件模板 | 根据项目语言选择 |
| **License** | 开源许可证 | 开源项目需要选择 |

**第 4 步：创建成功**
点击 **Create repository** 后，页面会跳转到新创建的仓库页面。

### 方法二：使用命令行创建

如果你已经安装了 GitHub CLI，可以在终端中创建：

```bash
# 创建公开仓库
gh repo create my-repo --public --description "我的项目"

# 创建私有仓库
gh repo create my-repo --private
```

## 仓库页面详解

创建仓库后，你会看到这样的界面：

```
┌─────────────────────────────────────────────┐
│  your-username / my-project                  │
│                                             │
│  [Code]  [Issues]  [Pull requests]          │
│  [Actions]  [Projects]  [Wiki]              │
├─────────────────────────────────────────────┤
│                                             │
│  📁 README.md                               │
│                                             │
│  This is a sample project...                │
│                                             │
└─────────────────────────────────────────────┘
```

**顶部标签说明：**

| 标签 | 功能 |
|------|------|
| **Code** | 查看代码文件 |
| **Issues** | 问题追踪 |
| **Pull requests** | 代码合并请求 |
| **Actions** | 自动化工作流 |
| **Projects** | 项目看板 |
| **Wiki** | 项目文档 |
| **Security** | 安全设置 |
| **Insights** | 数据分析 |
| **Settings** | 仓库设置 |

## 仓库设置（图文）

### 基本设置

**操作步骤：**
1. 进入仓库页面
2. 点击顶部的 **Settings** 标签
3. 在 **General** 选项卡中可以修改：
   - 仓库名称
   - 描述
   - 网站地址
   - 主题标签

```
┌─────────────────────────────────────────────┐
│  Settings                                    │
│                                             │
│  General  ← 当前页面                          │
│  Access                                       │
│  Branches                                    │
│  Tags                                        │
│  ...                                         │
├─────────────────────────────────────────────┤
│                                             │
│  Repository name: [my-project          ]     │
│  Description:     [这是一个示例项目    ]      │
│  Website:         [https://example.com]      │
│  Topics:          [react] [javascript]       │
│                                             │
│           [Save changes]                    │
└─────────────────────────────────────────────┘
```

### 添加协作者

**操作步骤：**
1. 进入 **Settings**
2. 左侧菜单选择 **Collaborators**
3. 点击 **Add people**
4. 输入对方的 GitHub 用户名
5. 点击 **Add [username] to this repository**

```
┌─────────────────────────────────────────────┐
│  Collaborators                                │
│                                             │
│  Search by username, full name, or email:    │
│  ┌─────────────────────────────────────┐    │
│  │ username                            │    │
│  └─────────────────────────────────────┘    │
│                                             │
│           [Add [username] to repository]    │
└─────────────────────────────────────────────┘
```

### 删除仓库

**操作步骤：**
1. 进入 **Settings**
2. 滚动到页面最底部
3. 找到 **Danger Zone** 区域
4. 点击 **Delete this repository**
5. 输入仓库名称确认删除

⚠️ **警告：删除操作不可逆，请谨慎操作！**

## 使用 SSH 克隆仓库

### 获取克隆地址

1. 进入仓库页面
2. 点击绿色的 **Code** 按钮
3. 选择 **SSH** 选项卡
4. 复制地址

```
┌─────────────────────────────────────────────┐
│  Code                                        │
│                                             │
│  HTTPS  SSH  GitHub CLI                      │
│                                             │
│  ┌─────────────────────────────────────┐    │
│  │ git@github.com:user/repo.git       │    │
│  │                          📋 复制    │    │
│  └─────────────────────────────────────┘    │
└─────────────────────────────────────────────┘
```

### 克隆到本地

```bash
# 克隆仓库
git clone git@github.com:user/repo.git

# 进入仓库目录
cd repo
```

## 好的仓库应该包含

```
my-repo/
├── README.md          # 项目说明（必须）
├── LICENSE            # 许可证（开源项目必须）
├── .gitignore         # 忽略文件（推荐）
├── CONTRIBUTING.md    # 贡献指南（推荐）
├── CHANGELOG.md       # 更新日志（推荐）
└── src/              # 源代码目录
```

---

## 实践练习

### 练习：创建你的第一个仓库

**任务 1：网页创建仓库**
1. 登录 GitHub
2. 点击右上角 **+** → **New repository**
3. 填写仓库名称：`my-first-repo`
4. 填写描述：`我的第一个 GitHub 仓库`
5. 选择 **Public**
6. 勾选 **Add a README file**
7. 点击 **Create repository**

**任务 2：探索仓库页面**
1. 点击各个标签（Code、Issues、Pull requests 等）
2. 点击 **Settings** 查看设置选项
3. 点击 **Code** 获取克隆地址

**任务 3：克隆仓库到本地**
1. 复制 SSH 克隆地址
2. 在终端中执行 `git clone 地址`
3. 进入仓库目录查看文件

**验证方法：**
- 仓库页面显示你创建的文件
- 本地可以进入仓库目录并看到文件

## 下一步

[README 与文档 →](15-readme-docs.md)
