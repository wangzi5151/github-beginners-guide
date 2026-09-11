# 第二章：安装与配置 Git

## 2.1 Windows 安装

### 方法一：官网下载（推荐新手）

#### 第 1 步：打开下载页面

1. 打开浏览器访问 **https://git-scm.com/download/win**
2. 页面会自动检测你的系统并开始下载
3. 如果没有自动下载，点击 **Click here to download manually**

```
┌─────────────────────────────────────────────────────┐
│  Download Git for Windows                            │
│                                                     │
│  Click here to download manually                     │
│                                                     │
│  ✓ 64-bit Git for Windows Setup                     │
│  ✓ 32-bit Git for Windows Setup                     │
│  ✓ Portable Git for Windows                          │
└─────────────────────────────────────────────────────┘
```

#### 第 2 步：运行安装程序

1. 找到下载的文件（通常在"下载"文件夹）
2. 双击运行 `Git-xxx-xxx-bit.exe`
3. 如果弹出安全提示，点击 **运行**

#### 第 3 步：安装向导

按照以下步骤点击：

**欢迎页面：**
```
┌─────────────────────────────────────────────┐
│  Git Setup                                   │
│                                             │
│  Welcome to Git Setup                       │
│                                             │
│  This will install Git version x.x.x on     │
│  your computer.                             │
│                                             │
│               [Next >]                       │
└─────────────────────────────────────────────┘
```

**许可协议：**
```
┌─────────────────────────────────────────────┐
│  GNU General Public License                 │
│                                             │
│  GNU GENERAL PUBLIC LICENSE                 │
│  Version 2, June 1991                       │
│                                             │
│  [阅读完整协议]                              │
│                                             │
│               [Next >]                       │
└─────────────────────────────────────────────┘
```

**选择组件：**
```
┌─────────────────────────────────────────────┐
│  Select Components                           │
│                                             │
│  ☑ Git Bash                                 │
│  ☑ Git GUI                                  │
│  ☑ Git LFS                                  │
│  ☑ Associate .git* files with Git           │
│  ☑ Add Git Bash Profile to Windows Terminal│
│  ☑ Add a Git Bash Profile to Windows       │
│                                             │
│               [Next >]                       │
└─────────────────────────────────────────────┘
```

**重要设置 - PATH 环境：**
```
┌─────────────────────────────────────────────┐
│  Adjusting your PATH environment            │
│                                             │
│  ● Use Git from the Windows Command Prompt  │
│    ← 推荐选择这个                            │
│  ○ Use Git and optional Unix tools from the │
│    Windows Command Prompt                   │
│  ○ Use Git from the Windows Command Prompt  │
│                                             │
│               [Next >]                       │
└─────────────────────────────────────────────┘
```

**重要设置 - SSH 可执行文件：**
```
┌─────────────────────────────────────────────┐
│  Choosing the SSH executable                 │
│                                             │
│  ● Use bundled OpenSSH                      │
│    ← 推荐选择这个                            │
│  ○ Use external OpenSSH                     │
│                                             │
│               [Next >]                       │
└─────────────────────────────────────────────┘
```

**重要设置 - 行尾转换：**
```
┌─────────────────────────────────────────────┐
│  Configuring line ending conversions         │
│                                             │
│  ● Checkout Windows-style, commit           │
│    Unix-style line endings                  │
│    ← Windows 用户推荐                        │
│                                             │
│  ○ Checkout as-is, commit Unix-style        │
│  ○ Checkout as-is, commit as-is             │
│                                             │
│               [Next >]                       │
└─────────────────────────────────────────────┘
```

#### 第 4 步：完成安装

1. 点击 **Install** 开始安装
2. 等待安装完成
3. 点击 **Finish**

### 方法二：使用 Winget

```powershell
# 打开命令提示符或 PowerShell
winget install Git.Git
```

### 方法三：使用 Chocolatey

```powershell
choco install git
```

## 2.2 macOS 安装

### 方法一：Homebrew（推荐）

#### 第 1 步：安装 Homebrew

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

#### 第 2 步：安装 Git

```bash
brew install git
```

### 方法二：Xcode Command Line Tools

```bash
xcode-select --install
```

弹出对话框时点击 **安装**。

### 方法三：官网下载

1. 访问 **https://git-scm.com/download/mac**
2. 下载安装包
3. 双击运行安装

## 2.3 Linux 安装

### Ubuntu/Debian

```bash
sudo apt update
sudo apt install git
```

### CentOS/RHEL

```bash
sudo yum install git
```

### Fedora

```bash
sudo dnf install git
```

### Arch Linux

```bash
sudo pacman -S git
```

### 从源码编译

```bash
# 安装依赖
sudo apt install build-essential libssl-dev libcurl4-gnutls-dev libexpat1-dev gettext

# 下载源码
git clone https://github.com/git/git.git
cd git

# 编译安装
make prefix=/usr/local all
sudo make prefix=/usr/local install
```

## 2.4 验证安装

### 打开终端

**Windows：**
- 按 `Win + R`
- 输入 `cmd`
- 按回车

**macOS：**
- 按 `Command + 空格`
- 输入 `Terminal`
- 按回车

**Linux：**
- 按 `Ctrl + Alt + T`

### 输入命令

```bash
git --version
```

### 预期输出

```
git version 2.43.0.windows.1
```

看到版本号说明安装成功！

## 2.5 首次配置

### 为什么需要配置？

Git 需要知道你是谁，这样在每次提交时才能记录你的信息。

### 设置用户名

```bash
git config --global user.name "你的名字"
```

**示例：**
```bash
git config --global user.name "Zhang San"
```

### 设置邮箱

```bash
git config --global user.email "your@email.com"
```

**示例：**
```bash
git config --global user.email "zhangsan@example.com"
```

⚠️ **重要：邮箱必须与你的 GitHub 账号邮箱一致！**

### 设置默认分支名

```bash
git config --global init.defaultBranch main
```

### 设置默认编辑器

```bash
# 如果使用 VS Code
git config --global core.editor "code --wait"

# 如果使用 Vim
git config --global core.editor "vim"

# 如果使用 Nano
git config --global core.editor "nano"

# 如果使用 Notepad++
git config --global core.editor "'C:/Program Files/Notepad++/notepad++.exe' -multiInst -notabbar -nosession -noPlugin"
```

### 启用颜色输出

```bash
git config --global color.ui auto
```

### 配置行尾处理

```bash
# Windows 用户
git config --global core.autocrlf true

# macOS/Linux 用户
git config --global core.autocrlf input
```

### 配置中文文件名

```bash
# 防止中文文件名显示为乱码
git config --global core.quotepath false
```

### 配置默认合并工具

```bash
# 使用 VS Code 作为合并工具
git config --global merge.tool vscode
git config --global mergetool.vscode.cmd 'code --wait $MERGED'
```

## 2.6 查看配置

### 查看所有配置

```bash
git config --list
```

### 查看特定配置

```bash
# 查看用户名
git config user.name

# 查看邮箱
git config user.email

# 查看编辑器
git config core.editor
```

### 配置文件位置

| 级别 | 文件位置 | 说明 |
|------|---------|------|
| `--system` | `/etc/gitconfig` | 系统级配置 |
| `--global` | `~/.gitconfig` | 用户级配置（推荐） |
| `--local` | `.git/config` | 仓库级配置 |

**优先级：** `local` > `global` > `system`

### 配置文件内容

```ini
# ~/.gitconfig 文件内容示例
[user]
    name = Zhang San
    email = zhangsan@example.com
[core]
    editor = code --wait
    autocrlf = true
    quotepath = false
[init]
    defaultBranch = main
[color]
    ui = auto
```

## 2.7 高级配置

### 设置 Git 别名

```bash
# 常用别名
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.lg "log --oneline --graph --all"
git config --global alias.last "log -1 HEAD"
git config --global alias.unstage "reset HEAD --"
```

### 配置代理

```bash
# HTTP 代理
git config --global http.proxy http://127.0.0.1:7890
git config --global https.proxy http://127.0.0.1:7890

# SOCKS5 代理
git config --global http.proxy socks5://127.0.0.1:7890
git config --global https.proxy socks5://127.0.0.1:7890

# 取消代理
git config --global --unset http.proxy
git config --global --unset https.proxy
```

### 配置 SSH

```bash
# 生成 SSH 密钥
ssh-keygen -t ed25519 -C "your@email.com"

# 启动 ssh-agent
eval "$(ssh-agent -s)"

# 添加密钥
ssh-add ~/.ssh/id_ed25519
```

### 配置大文件处理

```bash
# 安装 Git LFS
git lfs install

# 跟踪大文件
git lfs track "*.psd"
git lfs track "*.zip"
```

## 2.8 常见问题

### Q: Windows 安装后找不到 git 命令？

**解决方法：**
1. 重新打开命令提示符
2. 检查 PATH 环境变量
3. 重新安装 Git，确保勾选"Add Git to PATH"

### Q: macOS 提示"command not found: git"？

**解决方法：**
```bash
# 安装 Xcode Command Line Tools
xcode-select --install

# 或使用 Homebrew 安装
brew install git
```

### Q: 配置了邮箱但提交时显示其他邮箱？

**解决方法：**
```bash
# 检查当前配置
git config --list

# 重新设置邮箱
git config --global user.email "correct@email.com"

# 检查系统级别配置
git config --system user.email
```

### Q: 如何删除配置？

```bash
# 删除全局配置
git config --global --unset user.name
git config --global --unset user.email

# 编辑配置文件
git config --global --edit
```

## 2.9 最佳实践

1. **使用真实信息**：用户名和邮箱要与 GitHub 账号一致
2. **使用全局配置**：除非有特殊需求，使用 `--global` 配置
3. **配置编辑器**：选择你熟悉的编辑器
4. **配置代理**：如果访问 GitHub 有问题，配置代理
5. **配置别名**：设置常用命令的别名，提高效率

## 2.10 本章小结

本章详细介绍了 Git 的安装和配置方法，包括：

- Windows、macOS、Linux 的安装方法
- 首次配置的必要步骤
- 高级配置选项
- 常见问题解决方法

**关键要点：**
- 安装 Git 后必须进行配置
- 邮箱必须与 GitHub 账号一致
- 可以根据需要配置代理、别名等

**下一步：**
[注册 GitHub 账号 →](03-signup-github.md)
