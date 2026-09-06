# 练习 3：Pull Request 工作流

## 目标

掌握使用 Pull Request 进行协作开发的完整流程。

## 准备工作

### 1. 创建练习仓库

```bash
# 在 GitHub 上创建仓库 pr-practice
gh repo create pr-practice --public

# 克隆到本地
git clone git@github.com:你的用户名/pr-practice.git
cd pr-practice

# 创建初始文件
echo "# PR Practice" > README.md
git add README.md
git commit -m "Initial commit"
git push -u origin main
```

## 步骤

### 1. 创建功能分支

```bash
git checkout -b feature/add-todo
```

### 2. 实现功能

创建 `todo.js`：

```javascript
// todo.js
const todos = [];

function addTodo(text) {
  todos.push({
    id: Date.now(),
    text,
    completed: false
  });
}

function toggleTodo(id) {
  const todo = todos.find(t => t.id === id);
  if (todo) {
    todo.completed = !todo.completed;
  }
}

function removeTodo(id) {
  const index = todos.findIndex(t => t.id === id);
  if (index > -1) {
    todos.splice(index, 1);
  }
}

module.exports = { addTodo, toggleTodo, removeTodo };
```

### 3. 提交并推送

```bash
git add todo.js
git commit -m "feat: 添加 todo 功能"

git push -u origin feature/add-todo
```

### 4. 创建 Pull Request

```bash
# 使用 GitHub CLI
gh pr create \
  --title "feat: 添加 todo 功能" \
  --body "## 变更说明
- 添加了 addTodo 函数
- 添加了 toggleTodo 函数
- 添加了 removeTodo 函数

## 测试
- [x] 本地测试通过

## 关联 Issue
无"
```

### 5. 代码审查

在 GitHub 上：
1. 查看 PR 的代码变更
2. 添加审查评论
3. 点击 **Review changes**
4. 选择 **Approve** 或 **Request changes**

### 6. 响应审查

如果需要修改：

```bash
# 根据审查意见修改代码
git add todo.js
git commit -m "fix: 根据审查添加参数验证"
git push
```

PR 会自动更新。

### 7. 合并 PR

```bash
# 使用 GitHub CLI
gh pr merge --merge
```

### 8. 清理

```bash
# 删除已合并的远程分支
git push origin --delete feature/add-todo

# 删除本地分支
git checkout main
git pull
git branch -d feature/add-todo
```

## PR 模板

创建 `.github/pull_request_template.md`：

```markdown
## 变更说明
<!-- 描述这个 PR 做了什么 -->

## 变更类型
- [ ] 新功能 (feat)
- [ ] Bug 修复 (fix)
- [ ] 文档更新 (docs)
- [ ] 其他

## 测试
- [ ] 已添加测试
- [ ] 已通过所有测试

## 截图（如适用）
```

## 最佳实践

1. **PR 标题清晰**：遵循 Conventional Commits
2. **描述完整**：说明变更内容和原因
3. **小步提交**：一个 PR 只做一件事
4. **及时响应**：不要让审查者等待

## 下一步

[练习 4：修复 Merge Conflict →](exercise-4-fix-conflict.md)
