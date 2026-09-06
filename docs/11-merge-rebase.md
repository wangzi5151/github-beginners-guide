# 合并与变基

## 合并 (Merge)

### 快进合并

当目标分支没有新提交时：

```bash
git checkout main
git merge feature
```

```
合并前：
main:    A --- B --- C
                      \
feature:               D --- E

合并后：
main:    A --- B --- C --- D --- E
```

### 三方合并

当两个分支都有新提交时：

```bash
git checkout main
git merge feature
```

```
合并前：
main:    A --- B --- C --- F
                      \
feature:               D --- E

合并后：
main:    A --- B --- C --- F --- M (merge commit)
                      \         /
feature:               D --- E
```

### 禁用快进合并

```bash
git merge --no-ff feature
```

## 变基 (Rebase)

将当前分支的提交"重放"到目标分支上：

```bash
git checkout feature
git rebase main
```

```
变基前：
main:    A --- B --- C
                      \
feature:               D --- E

变基后：
main:    A --- B --- C
                      \
feature:               D' --- E'
```

### 交互式变基

```bash
git rebase -i HEAD~3
```

可以：
- 修改提交信息
- 合并提交
- 删除提交
- 调整提交顺序

## 合并 vs 变基

| 特性 | 合并 | 变基 |
|------|------|------|
| 历史 | 保留完整历史 | 线性历史 |
| 提交哈希 | 保留原哈希 | 生成新哈希 |
| 复杂度 | 简单 | 需要更多操作 |
| 冲突 | 一次性解决 | 可能需要多次解决 |
| 推荐场景 | 公共分支 | 个人/功能分支 |

## 最佳实践

```bash
# 功能分支合并到 main：使用 merge
git checkout main
git merge --no-ff feature

# 更新功能分支：使用 rebase
git checkout feature
git rebase main
```

**黄金规则：不要对已推送到远程的提交进行变基**

## 取消操作

```bash
# 取消合并
git merge --abort

# 取消变基
git rebase --abort
```

## 下一步

[解决冲突 →](12-resolve-conflicts.md)
