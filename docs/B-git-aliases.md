# Git 别名配置

## 常用别名

```bash
# 基本操作
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.sw switch

# 日志
git config --global alias.lg "log --oneline --graph --all"
git config --global alias.last "log -1 HEAD --stat"
git config --global alias.today "log --since='1 day ago' --oneline"

# 差异
git config --global alias.df "diff"
git config --global alias.dfs "diff --staged"

# 暂存
git config --global alias.unstage "reset HEAD --"

# 撤销
git config --global alias.undo "reset --soft HEAD~1"

# 清理
git config --global alias.cleanup "!git branch --merged | grep -v '\\*\\|main\\|master' | xargs -n 1 git branch -d"
```

## 有用的组合别名

```bash
# 显示所有分支的最后提交
git config --global alias.branches "branch -a --v"

# 显示简洁状态
git config --global alias.s "status -sb"

# 添加所有修改并显示状态
git config --global alias.aa "!git add -A && git status"

# 提交所有修改
git config --global alias.ac "!git add -A && git commit"

# 推送到当前分支
git config --global alias.ps "push origin HEAD"

# 拉取并变基
git config --global alias.pull "!git pull --rebase"

# 显示 Git 别名
git config --global alias.aliases "config --get-regexp alias"
```

## 查看别名

```bash
# 列出所有别名
git config --global --list | grep alias

# 或直接查看配置文件
cat ~/.gitconfig
```

## 取消别名

```bash
git config --global --unset alias.st
```

## 配置文件

直接编辑 `~/.gitconfig`：

```ini
[alias]
    st = status
    co = checkout
    br = branch
    ci = commit
    lg = log --oneline --graph --all
    last = log -1 HEAD --stat
    unstage = reset HEAD --
    undo = reset --soft HEAD~1
    s = status -sb
```

## 别名脚本

创建复杂别名时可以使用函数：

```bash
git config --global alias.cleanup "!f() { git branch --merged | grep -v '\\*\\|main\\|master' | xargs -n 1 git branch -d; }; f"
```

## 推荐配置

```bash
# 完整的推荐别名配置
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.sw switch
git config --global alias.lg "log --oneline --graph --all"
git config --global alias.last "log -1 HEAD --stat"
git config --global alias.df "diff"
git config --global alias.dfs "diff --staged"
git config --global alias.unstage "reset HEAD --"
git config --global alias.undo "reset --soft HEAD~1"
git config --global alias.s "status -sb"
git config --global alias.aa "!git add -A && git status"
git config --global alias.ac "!git add -A && git commit"
git config --global alias.ps "push origin HEAD"
git config --global alias.pull "!git pull --rebase"
```
