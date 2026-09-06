# 配置 SSH 密钥

SSH 密钥让你可以不用每次推送代码都输入密码。

## 生成 SSH 密钥

### 检查是否已有密钥

```bash
ls -al ~/.ssh
# 看是否有 id_rsa.pub 或 id_ed25519.pub
```

### 生成新密钥（推荐 Ed25519）

```bash
ssh-keygen -t ed25519 -C "your@email.com"
```

按 Enter 使用默认文件位置，然后设置密码（可选）。

### 生成密钥（RSA，兼容性更好）

```bash
ssh-keygen -t rsa -b 4096 -C "your@email.com"
```

## 添加到 SSH Agent

### 启动 ssh-agent

```bash
eval "$(ssh-agent -s)"
# 输出：Agent pid xxxx
```

### 添加私钥

```bash
ssh-add ~/.ssh/id_ed25519
```

## 添加公钥到 GitHub

### 1. 复制公钥内容

```bash
# macOS
cat ~/.ssh/id_ed25519.pub | pbcopy

# Linux
cat ~/.ssh/id_ed25519.pub | xclip -selection clipboard

# Windows (Git Bash)
cat ~/.ssh/id_ed25519.pub | clip
```

### 2. 在 GitHub 添加
1. 登录 GitHub
2. 点击右上角头像 → **Settings**
3. 左侧菜单选择 **SSH and GPG keys**
4. 点击 **New SSH key**
5. 粘贴公钥内容
6. 点击 **Add SSH key**

## 测试连接

```bash
ssh -T git@github.com
```

成功输出：
```
Hi username! You've successfully authenticated, but GitHub does not provide shell access.
```

## 使用 SSH 克隆仓库

```bash
# SSH 格式（推荐）
git clone git@github.com:user/repo.git

# HTTPS 格式（需要每次输入密码，除非配置 credential helper）
git clone https://github.com/user/repo.git
```

## SSH 配置多账号

编辑 `~/.ssh/config`：

```ssh
# 个人账号
Host github.com-personal
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_personal

# 工作账号
Host github.com-work
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_work
```

克隆时使用自定义 Host：
```bash
git clone git@github.com-personal:user/repo.git
```

## 下一步

[Git 基本配置 →](05-git-config.md)
