# 暂存与提交

## 暂存文件

### 添加单个文件

```bash
git add filename.txt
```

### 添加多个文件

```bash
git add file1.txt file2.txt file3.txt
```

### 添加所有修改

```bash
git add .
```

### 添加特定类型的文件

```bash
git add *.js
git add src/*.css
```

### 添加所有跟踪文件的修改（不含新文件）

```bash
git add -u
```

### 交互式添加

```bash
git add -i
```

## 提交更改

### 基本提交

```bash
git commit -m "提交信息"
```

### 提交信息规范

```
类型(范围): 简短描述

详细描述（可选）

关联 Issue（可选）
```

### 提交类型

| 类型 | 说明 |
|------|------|
| `feat` | 新功能 |
| `fix` | 修复 bug |
| `docs` | 文档更新 |
| `style` | 代码格式（不影响功能） |
| `refactor` | 重构 |
| `test` | 测试相关 |
| `chore` | 构建/工具相关 |

### 示例

```bash
git commit -m "feat: 添加用户登录功能"

git commit -m "fix: 修复首页加载失败的问题"

git commit -m "docs: 更新 README 安装说明"
```

### 提交所有已暂存的更改

```bash
git commit -m "描述"
```

### 跳过暂存直接提交

```bash
git commit -a -m "描述"
```

注意：`-a` 只对已跟踪文件有效，新文件仍需 `git add`

## 查看状态

```bash
# 查看工作区状态
git status

# 简洁模式
git status -s
```

## 修改最近提交

```bash
# 修改提交信息
git commit --amend -m "新的提交信息"

# 添加漏掉的文件
git add forgotten-file.txt
git commit --amend --no-edit
```

## 下一步

[查看历史与差异 →](09-log-diff.md)
