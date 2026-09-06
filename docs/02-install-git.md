# 安装与配置 Git

## Windows 安装

### 方法一：官网下载
1. 访问 [Git 官网](https://git-scm.com/download/win)
2. 下载安装包
3. 运行安装程序，使用默认设置即可

### 方法二：使用 Chocolatey
```powershell
choco install git
```

### 方法三：使用 Winget
```powershell
winget install Git.Git
```

## macOS 安装

### 方法一：Homebrew（推荐）
```bash
brew install git
```

### 方法二：Xcode Command Line Tools
```bash
xcode-select --install
```

## Linux 安装

### Ubuntu/Debian
```bash
sudo apt update
sudo apt install git
```

### CentOS/RHEL
```bash
sudo yum install git
# 或
sudo dnf install git
```

### Arch Linux
```bash
sudo pacman -S git
```

## 验证安装

```bash
# 查看 Git 版本
git --version

# 预期输出
# git version 2.x.x
```

## 首次配置

安装完成后，必须进行基本配置：

```bash
# 设置用户名（会显示在你的提交中）
git config --global user.name "你的名字"

# 设置邮箱（必须与 GitHub 账号邮箱一致）
git config --global user.email "your@email.com"

# 设置默认分支名为 main（而非 master）
git config --global init.defaultBranch main

# 设置默认编辑器
git config --global core.editor "code --wait"  # VS Code
# git config --global core.editor "vim"        # Vim
# git config --global core.editor "nano"       # Nano

# 启用颜色输出
git config --global color.ui auto
```

## 查看配置

```bash
# 查看所有配置
git config --list

# 查看某个特定配置
git config user.name
git config user.email
```

## 配置文件位置

| 级别 | 文件位置 | 说明 |
|------|---------|------|
| `--system` | `/etc/gitconfig` | 系统级配置 |
| `--global` | `~/.gitconfig` | 用户级配置（推荐） |
| `--local` | `.git/config` | 仓库级配置 |

优先级：`local` > `global` > `system`

## 下一步

[注册 GitHub 账号 →](03-signup-github.md)
