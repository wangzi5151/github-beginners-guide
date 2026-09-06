# 配置 SSH 密钥

SSH 密钥让你可以不用每次推送代码都输入密码，更加安全方便。

## 完整操作步骤

### 第 1 步：打开终端

**Windows 用户：**
1. 按 `Win + R` 打开运行窗口
2. 输入 `cmd` 并按回车
3. 打开命令提示符

**macOS 用户：**
1. 按 `Command + 空格` 打开 Spotlight
2. 输入 `Terminal` 并按回车
3. 打开终端

**Linux 用户：**
1. 按 `Ctrl + Alt + T` 打开终端

### 第 2 步：检查是否已有密钥

在终端中输入以下命令并按回车：

```bash
ls -al ~/.ssh
```

**可能的结果：**
- 如果显示文件列表（如 `id_rsa.pub`），说明已有密钥，可以跳到第 4 步
- 如果显示 `No such file or directory`，说明没有密钥，继续下一步

### 第 3 步：生成 SSH 密钥

在终端中输入以下命令并按回车：

```bash
ssh-keygen -t ed25519 -C "your-email@example.com"
```

**操作说明：**
1. 将 `your-email@example.com` 替换为你的 GitHub 邮箱
2. 按回车后会提示保存位置，直接按回车使用默认位置
3. 提示设置密码时，可以直接按回车（不设置密码）或输入一个密码
4. 再次按回车确认

**成功输出：**
```
Your identification has been saved in /c/Users/你的用户名/.ssh/id_ed25519
Your public key has been saved in /c/Users/你的用户名/.ssh/id_ed25519.pub
The key fingerprint is:
SHA256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx your-email@example.com
```

### 第 4 步：复制公钥内容

根据你的操作系统，输入对应的命令：

**Windows (Git Bash)：**
```bash
cat ~/.ssh/id_ed25519.pub | clip
```

**macOS：**
```bash
cat ~/.ssh/id_ed25519.pub | pbcopy
```

**Linux：**
```bash
cat ~/.ssh/id_ed25519.pub | xclip -selection clipboard
```

执行后，公钥内容已复制到剪贴板。

### 第 5 步：在 GitHub 添加公钥

**操作步骤（图文）：**

1. 打开浏览器，登录 **github.com**

2. 点击右上角的 **头像**

3. 在下拉菜单中点击 **Settings**（设置）

```
┌──────────────────────┐
│  ┌────┐              │
│  │ 头像 │  👤 个人资料 │
│  └────┘  ⚙️ Settings │
│         🚪 Sign out  │
└──────────────────────┘
```

4. 在左侧菜单中，向下滚动找到 **Security**

5. 点击 **SSH and GPG keys**

```
左侧菜单：
├── Access
│   ├── Password and authentication
│   ├── SSH and GPG keys  ← 点击这里
│   └── Session log
```

6. 点击绿色的 **New SSH key** 按钮

```
┌─────────────────────────────────────────────┐
│  SSH keys                                    │
│                                             │
│  [New SSH key]  ← 点击这个按钮              │
└─────────────────────────────────────────────┘
```

7. 填写信息：
   - **Title**：输入一个名称，例如 "我的电脑"
   - **Key type**：选择 **Authentication Key**
   - **Key**：在输入框中按 `Ctrl+V` 粘贴刚才复制的公钥

```
┌─────────────────────────────────────────────┐
│  Add new SSH key                             │
│                                             │
│  Title: [我的电脑                    ]       │
│                                             │
│  Key type: ○ Authentication Key  ← 选择这个  │
│            ○ Signing Key                     │
│                                             │
│  Key: ┌─────────────────────────────┐       │
│        │ ssh-ed25519 AAAA...        │       │
│        │ (粘贴公钥内容)              │       │
│        └─────────────────────────────┘       │
│                                             │
│           [Add SSH key]                     │
└─────────────────────────────────────────────┘
```

8. 点击 **Add SSH key** 按钮保存

### 第 6 步：测试连接

在终端中输入以下命令并按回车：

```bash
ssh -T git@github.com
```

**成功输出：**
```
Hi your-username! You've successfully authenticated, but GitHub does not provide shell access.
```

看到这个消息，说明 SSH 密钥配置成功！

## 使用 SSH 克隆仓库

### 在 GitHub 页面获取 SSH 地址

1. 打开你要克隆的仓库页面
2. 点击绿色的 **Code** 按钮
3. 选择 **SSH** 选项卡
4. 复制地址

```
┌─────────────────────────────────────────────┐
│  Code                                        │
│                                             │
│  HTTPS  SSH  GitHub CLI  ← 选择 SSH         │
│                                             │
│  ┌─────────────────────────────────────┐    │
│  │ git@github.com:user/repo.git       │    │
│  │                          📋 复制    │    │
│  └─────────────────────────────────────┘    │
└─────────────────────────────────────────────┘
```

### 克隆仓库

```bash
git clone git@github.com:user/repo.git
```

## SSH 配置多账号

如果你有多个 GitHub 账号（个人和工作），可以编辑配置文件：

**Windows：** `C:\Users\你的用户名\.ssh\config`
**macOS/Linux：** `~/.ssh/config`

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

---

## 实践练习

### 练习：配置 SSH 密钥

**任务清单：**
- [ ] 打开终端
- [ ] 检查是否已有密钥
- [ ] 生成新的 SSH 密钥
- [ ] 复制公钥内容
- [ ] 在 GitHub 网页添加公钥
- [ ] 测试 SSH 连接
- [ ] 使用 SSH 克隆一个仓库

**验证方法：**
执行 `ssh -T git@github.com` 显示 "Hi xxx! You've successfully authenticated" 表示成功。

## 下一步

[Git 基本配置 →](05-git-config.md)
