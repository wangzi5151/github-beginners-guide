# Pull Request 协作

## 什么是 Pull Request？

Pull Request（PR）是向项目贡献代码的标准方式。它允许：
- 代码审查
- 讨论变更
- 自动化检查
- 安全合并

## PR 工作流程

```
1. Fork 仓库（如果是外部贡献）
2. 创建功能分支
3. 修改代码并提交
4. 推送到远程
5. 创建 Pull Request
6. 代码审查
7. 合并
```

## 创建 Pull Request

### 推送分支

```bash
git push -u origin feature-new-button
```

### 网页创建 PR

1. 进入仓库，点击 **Pull requests**
2. 点击 **New pull request**
3. 选择基础分支和比较分支
4. 填写标题和描述
5. 点击 **Create pull request**

### 命令行创建 PR

```bash
# 使用 GitHub CLI
gh pr create --title "feat: 添加新按钮" --body "描述变更..."

# 创建 Draft PR（草稿）
gh pr create --draft
```

## PR 描述模板

```markdown
## 变更说明
简要描述这个 PR 做了什么

## 变更类型
- [ ] 新功能 (feat)
- [ ] Bug 修复 (fix)
- [ ] 文档更新 (docs)
- [ ] 代码重构 (refactor)
- [ ] 其他

## 测试
- [ ] 已添加测试
- [ ] 已通过所有测试

## 关联 Issue
Closes #42

## 截图（如适用）
```

## 代码审查

### 审查清单
- 代码逻辑是否正确
- 是否有潜在 bug
- 代码风格是否一致
- 是否需要文档更新
- 测试是否充分

### 审查评论
- **总体评论**：对 PR 的整体评价
- **行内评论**：对特定代码的建议
- **建议更改**：直接提出修改建议

### 回应审查
```bash
# 根据审查修改代码
git add .
git commit -m "fix: 根据审查修改..."
git push
```

## 合并 PR

### 合并选项
- **Create a merge commit**：保留完整历史
- **Squash and merge**：压缩为一个提交
- **Rebase and merge**：线性历史

### 合并后清理

```bash
# 删除已合并的远程分支
git push origin --delete feature-new-button

# 删除本地分支
git branch -d feature-new-button
```

## PR 最佳实践

1. **保持小而专注**：一个 PR 只做一件事
2. **清晰的标题和描述**：让审查者快速理解
3. **关联 Issue**：说明解决什么问题
4. **及时响应审查**：不要让 PR 放太久
5. **测试充分**：确保代码质量

## Draft PR

用于早期阶段，表示 PR 尚未准备好审查：

```bash
gh pr create --draft --title "WIP: 正在开发的功能"
```

## 下一步

[Code Review 代码审查 →](18-code-review.md)
