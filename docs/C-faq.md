# 常见问题解答

## 基础问题

### Q: Git 和 GitHub 有什么区别？
**A:** Git 是版本控制软件（本地使用），GitHub 是基于 Git 的代码托管平台（在线服务）。

### Q: 如何修改 Git 提交的用户名和邮箱？
**A:**
```bash
git config --global user.name "新名字"
git config --global user.email "新邮箱"
# 注意：这不会修改历史提交，只对新提交生效
```

### Q: 如何撤销最后一次提交？
**A:**
```bash
# 保留修改
git reset --soft HEAD~1

# 丢弃修改
git reset --hard HEAD~1
```

### Q: 如何查看某个文件的修改历史？
**A:**
```bash
git log --follow -p filename
```

## 分支问题

### Q: 如何删除远程分支？
**A:**
```bash
git push origin --delete branch-name
```

### Q: 如何将一个分支的修改合并到另一个分支？
**A:**
```bash
git checkout target-branch
git merge source-branch
```

### Q: 如何在不提交的情况下切换分支？
**A:**
```bash
# 暂存修改
git stash

# 切换分支
git checkout other-branch

# 恢复修改
git stash pop
```

## 远程问题

### Q: 如何修改远程仓库地址？
**A:**
```bash
git remote set-url origin new-url
```

### Q: 如何同步 Fork？
**A:**
```bash
git remote add upstream original-url
git fetch upstream
git merge upstream/main
git push origin main
```

### Q: 如何强制推送？
**A:**
```bash
git push --force origin branch
# 或更安全的方式
git push --force-with-lease origin branch
```

## 冲突问题

### Q: 如何解决合并冲突？
**A:**
1. 打开冲突文件
2. 找到冲突标记 `<<<<<<<` 和 `>>>>>>>`
3. 手动选择保留的内容
4. 删除冲突标记
5. `git add` 标记为已解决
6. `git commit` 完成合并

### Q: 如何取消正在进行的合并？
**A:**
```bash
git merge --abort
```

### Q: 如何取消正在进行的变基？
**A:**
```bash
git rebase --abort
```

## 恢复问题

### Q: 如何恢复已删除的分支？
**A:**
```bash
# 查看 reflog 找到分支的最后提交
git reflog
# 找到 commit hash
git checkout -b recovered-branch commit-hash
```

### Q: 如何恢复误删的文件？
**A:**
```bash
# 恢复到最后一次提交的状态
git checkout HEAD -- filename

# 或使用 restore（Git 2.23+）
git restore filename
```

### Q: 如何恢复已推送的删除操作？
**A:**
```bash
# 找到删除前的提交
git reflog
# 恢复
git checkout commit-hash -- filename
git commit -m "恢复文件"
```

## 认证问题

### Q: 如何避免每次输入密码？
**A:**
```bash
# 使用 SSH（推荐）
git clone git@github.com:user/repo.git

# 或使用 credential helper
git config --global credential.helper cache
```

### Q: 如何使用 Token 认证？
**A:**
```bash
git clone https://<token>@github.com/user/repo.git
```

## 性能问题

### Q: 如何加速大仓库克隆？
**A:**
```bash
# 浅克隆
git clone --depth 1 url

# 只克隆特定分支
git clone --single-branch --branch main url
```

### Q: 如何清理本地仓库？
**A:**
```bash
# 垃圾回收
git gc

# 清理未跟踪文件
git clean -fd
```

## 下一步

[推荐学习资源 →](D-resources.md)
