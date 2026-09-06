# Git 基本配置

## 用户信息

```bash
# 必须配置，否则提交会报错
git config --global user.name "Your Name"
git config --global user.email "your@email.com"
```

## 默认分支名

```bash
# 新仓库默认使用 main 而非 master
git config --global init.defaultBranch main
```

## 编辑器配置

```bash
# VS Code
git config --global core.editor "code --wait"

# Vim
git config --global core.editor "vim"

# Nano
git config --global core.editor "nano"

# Sublime Text
git config --global core.editor "subl -n -w"
```

## 差异和合并工具

```bash
# 使用 VS Code 作为差异工具
git config --global diff.tool vscode
git config --global difftool.vscode.cmd 'code --wait --diff $LOCAL $REMOTE'

# 使用 VS Code 作为合并工具
git config --global merge.tool vscode
git config --global mergetool.vscode.cmd 'code --wait $MERGED'
```

## 行尾处理

```bash
# Windows 用户推荐
git config --global core.autocrlf true

# macOS/Linux 用户推荐
git config --global core.autocrlf input
```

## 推送行为

```bash
# 简化推送命令（避免每次指定分支）
git config --global push.autoSetupRemote true

# 默认使用 current 分支推送
git config --global push.default current
```

## 拉取行为

```bash
# 默认使用 rebase 拉取（保持线性历史）
git config --global pull.rebase true
```

## 别名配置

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

## 查看所有配置

```bash
# 列出所有配置
git config --list

# 查看特定配置
git config user.name
git config core.editor
```

## 配置文件位置

直接编辑配置文件（效果等同于命令行配置）：

```bash
# 用户级配置
~/.gitconfig

# 仓库级配置
.git/config
```

## 下一步

[Git 工作原理 →](06-how-git-works.md)
