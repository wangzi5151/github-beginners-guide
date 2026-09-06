# 常用命令速查表

## 配置

```bash
git config --global user.name "名字"
git config --global user.email "邮箱"
git config --global init.defaultBranch main
git config --global core.editor "code --wait"
```

## 创建与克隆

```bash
git init                          # 初始化仓库
git clone <url>                   # 克隆仓库
git clone <url> <dir>             # 克隆到指定目录
git clone --depth 1 <url>         # 浅克隆
```

## 基本操作

```bash
git status                        # 查看状态
git add <file>                    # 添加文件
git add .                         # 添加所有文件
git commit -m "message"           # 提交
git commit -am "message"          # 添加并提交（已跟踪文件）
```

## 分支

```bash
git branch                        # 查看分支
git branch <name>                 # 创建分支
git checkout <branch>             # 切换分支
git checkout -b <branch>          # 创建并切换
git switch <branch>               # 切换分支（新语法）
git switch -c <branch>            # 创建并切换（新语法）
git branch -d <branch>            # 删除分支
git branch -D <branch>            # 强制删除
```

## 合并与变基

```bash
git merge <branch>                # 合并分支
git rebase <branch>               # 变基
git merge --no-ff <branch>        # 禁用快进合并
git merge --abort                 # 取消合并
git rebase --abort                # 取消变基
```

## 远程操作

```bash
git remote -v                     # 查看远程
git remote add <name> <url>       # 添加远程
git remote remove <name>          # 删除远程
git push -u origin <branch>       # 推送并设置上游
git push                          # 推送
git pull                          # 拉取
git fetch                         # 获取
git push --force                  # 强制推送（危险！）
```

## 查看历史

```bash
git log                           # 查看日志
git log --oneline                 # 简洁日志
git log --oneline --graph --all   # 图形化日志
git log -5                        # 最近 5 次
git diff                          # 查看差异
git diff --staged                 # 暂存区差异
git show <commit>                 # 查看提交
```

## 撤销操作

```bash
git reset --soft HEAD~1           # 软回退（保留修改）
git reset HEAD~1                  # 混合回退（默认）
git reset --hard HEAD~1           # 硬回退（丢弃修改）
git revert <commit>               # 反转提交
git checkout -- <file>            # 恢复文件
git reset HEAD <file>             # 取消暂存
git commit --amend                # 修改上次提交
```

## 标签

```bash
git tag <name>                    # 创建标签
git tag -a <name> -m "message"    # 创建附注标签
git push origin <tag>             # 推送标签
git push origin --tags            # 推送所有标签
git tag -d <name>                 # 删除标签
```

## 暂存

```bash
git stash                         # 暂存修改
git stash list                    # 查看暂存
git stash pop                     # 恢复暂存
git stash apply                   # 恢复（不删除）
git stash drop                    # 删除暂存
```

## 清理

```bash
git clean -f                      # 删除未跟踪文件
git clean -fd                     # 删除未跟踪文件和目录
git gc                            # 垃圾回收
```

## GitHub CLI

```bash
gh repo create <name>             # 创建仓库
gh repo clone <user>/<repo>       # 克隆仓库
gh issue create                   # 创建 Issue
gh issue list                     # 列出 Issue
gh pr create                      # 创建 PR
gh pr list                        # 列出 PR
gh pr merge <number>              # 合并 PR
gh release create <tag>           # 创建 Release
```
