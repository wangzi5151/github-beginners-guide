# README 与文档

## README 的重要性

README 是项目的门面，好的 README 应该让读者快速了解：
- 这个项目是什么
- 如何安装
- 如何使用
- 如何贡献

## README 模板

```markdown
# 项目名称

> 简短的一句话描述项目

## 功能特性

- 功能1
- 功能2
- 功能3

## 快速开始

### 环境要求

- Node.js >= 16
- npm >= 8

### 安装

```bash
git clone https://github.com/user/repo.git
cd repo
npm install
```

### 运行

```bash
npm start
```

## 使用示例

```javascript
const something = require('repo');
something.doWork();
```

## API 文档

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `doWork()` | 无 | void | 执行工作 |

## 贡献指南

请阅读 [CONTRIBUTING.md](CONTRIBUTING.md) 了解如何贡献。

## 许可证

[MIT](LICENSE)
```

## Markdown 语法速查

### 标题
```markdown
# 一级标题
## 二级标题
### 三级标题
```

### 文本样式
```markdown
**粗体**
*斜体*
~~删除线~~
`代码`
```

### 列表
```markdown
- 无序列表项1
- 无序列表项2

1. 有序列表项1
2. 有序列表项2
```

### 链接和图片
```markdown
[链接文本](https://example.com)
![图片描述](image.png)
```

### 代码块
````markdown
```javascript
const code = '语法高亮';
```
````

### 表格
```markdown
| 列1 | 列2 | 列3 |
|-----|-----|-----|
| 数据 | 数据 | 数据 |
```

### 引用
```markdown
> 引用内容
```

### 任务列表
```markdown
- [x] 已完成任务
- [ ] 未完成任务
```

## 其他文档

### CONTRIBUTING.md
说明如何参与项目贡献：
- 如何报告 bug
- 如何提交代码
- 代码规范
- 提交信息规范

### CHANGELOG.md
记录版本更新历史：

```markdown
# Changelog

## [1.0.0] - 2024-01-01

### Added
- 新功能

### Changed
- 变更内容

### Fixed
- 修复内容
```

## 文档最佳实践

1. **保持更新**：代码变更时同步更新文档
2. **简洁明了**：避免冗长，突出重点
3. **有示例**：代码示例比文字描述更直观
4. **有结构**：使用标题和目录组织内容

## 下一步

[Issue 问题追踪 →](16-issues.md)
