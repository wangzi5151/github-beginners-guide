# 安装与配置 Git

## Windows 安装（图文详解）

### 方法一：官网下载（推荐）

**第 1 步：打开下载页面**
1. 打开浏览器访问 **https://git-scm.com/download/win**
2. 页面会自动检测你的系统并开始下载
3. 如果没有自动下载，点击 **Click here to download manually**

**第 2 步：运行安装程序**
1. 找到下载的文件（通常在"下载"文件夹）
2. 双击运行 `Git-xxx-xxx-bit.exe`

**第 3 步：安装向导**

按照以下步骤点击：

```
┌─────────────────────────────────────────────┐
│  Git Setup                                   │
│                                             │
│  Welcome to Git Setup                       │
│                                             │
│               [Next >]                       │
└─────────────────────────────────────────────┘
```

```
┌─────────────────────────────────────────────┐
│  GNU General Public License                 │
│                                             │
│  [阅读许可协议]                              │
│                                             │
│               [Next >]                       │
└─────────────────────────────────────────────┘
```

```
┌─────────────────────────────────────────────┐
│  Select Components                           │
│                                             │
│  ☑ Git Bash        ← 命令行工具，必须选     │
│  ☑ Git GUI         ← 图形界面工具           │
│  ☑ Git LFS         ← 大文件支持             │
│  ☑ Associate .git  ← 关联 .git 文件         │
│  ☑ Add Git Bash to ← 添加到终端             │
│    Terminal Explorer                        │
│                                             │
│               [Next >]                       │
└─────────────────────────────────────────────┘
```

**关键设置页面：**

```
┌─────────────────────────────────────────────┐
│  Adjusting your PATH environment            │
│                                             │
│  ○ Use Git from the Windows Command Prompt  │
│  ● Use Git from the Windows Command Prompt  │
│    and optional Unix tools from the Prompt  │
│    ← 推荐选择这个                            │
│  ○ Use Git and optional Unix tools from the │
│    Windows Command Prompt                   │
│                                             │
│               [Next >]                       │
└─────────────────────────────────────────────┘
```

```
┌─────────────────────────────────────────────┐
│  Choosing the SSH executable                 │
│                                             │
│  ● Use bundled OpenSSH  ← 推荐              │
│  ○ Use external OpenSSH                     │
│                                             │
│               [Next >]                       │
└─────────────────────────────────────────────┘
```

```
┌─────────────────────────────────────────────┐
│  Configuring line ending conversions         │
│                                             │
│  ● Checkout Windows-style, commit           │
│    Unix-style line endings  ← 推荐 Windows  │
│  ● Checkout as-is, commit Unix-style        │
│  ○ Checkout as-is, commit as-is             │
│                                             │
│               [Next >]                       │
└─────────────────────────────────────────────┘
```

**第 4 步：完成安装**
1. 点击 **Install** 开始安装
2. 等待安装完成
3. 点击 **Finish**

### 方法二：使用 Winget（命令行）

1. 按 `Win + R` 打开运行窗口
2. 输入 `cmd` 并回车
3. 输入以下命令并回车：

```powershell
winget install Git.Git
```

## macOS 安装（图文详解）

### 方法一：Homebrew（推荐）

**第 1 步：打开终端**
1. 按 `Command + 空格` 打开 Spotlight
2. 输入 `Terminal` 并回车

**第 2 步：安装 Homebrew（如果没有）**

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

**第 3 步：安装 Git**

```bash
brew install git
```

### 方法二：Xcode Command Line Tools

**第 1 步：打开终端**

**第 2 步：输入命令**

```bash
xcode-select --install
```

**第 3 步：确认安装**
1. 弹出对话框询问是否安装
2. 点击 **安装**
3. 等待安装完成

## Linux 安装

### Ubuntu/Debian

```bash
sudo apt update
sudo apt install git
```

### CentOS/RHEL

```bash
sudo yum install git
```

### Arch Linux

```bash
sudo pacman -S git
```

## 验证安装

**第 1 步：打开终端/命令提示符**

**第 2 步：输入以下命令**

```bash
git --version
```

**预期输出：**
```
git version 2.x.x
```

看到版本号说明安装成功！

## 首次配置（必须！）

安装完成后，需要配置你的身份信息。这些信息会出现在你的每次提交中。

**第 1 步：打开终端/命令提示符**

**第 2 步：设置用户名**

```bash
git config --global user.name "你的名字"
```

**第 3 步：设置邮箱**

```bash
git config --global user.email "your@email.com"
```

⚠️ **重要：邮箱必须与你的 GitHub 账号邮箱一致！**

**第 4 步：设置默认分支名**

```bash
git config --global init.defaultBranch main
```

**第 5 步：设置默认编辑器**

```bash
# 如果使用 VS Code
git config --global core.editor "code --wait"

# 如果使用 Vim
git config --global core.editor "vim"

# 如果使用 Nano
git config --global core.editor "nano"
```

**第 6 步：启用颜色输出**

```bash
git config --global color.ui auto
```

## 查看配置

```bash
# 查看所有配置
git config --list

# 查看用户名
git config user.name

# 查看邮箱
git config user.email
```

## 配置文件位置

| 级别 | 文件位置 | 说明 |
|------|---------|------|
| `--system` | `/etc/gitconfig` | 系统级配置 |
| `--global` | `~/.gitconfig` | 用户级配置（推荐） |
| `--local` | `.git/config` | 仓库级配置 |

优先级：`local` > `global` > `system`

---

## 实践练习

### 练习：安装并配置 Git

**任务 1：安装 Git**
- Windows：下载并运行安装程序
- macOS：使用 Homebrew 安装
- Linux：使用包管理器安装

**任务 2：验证安装**
- 打开终端/命令提示符
- 输入 `git --version`
- 确认显示版本号

**任务 3：配置身份信息**
- 设置用户名：`git config --global user.name "你的名字"`
- 设置邮箱：`git config --global user.email "your@email.com"`
- 查看配置：`git config --list`

**验证方法：**
- `git --version` 显示版本号
- `git config user.name` 显示你设置的用户名
- `git config user.email` 显示你设置的邮箱

## 下一步

[注册 GitHub 账号 →](03-signup-github.md)
