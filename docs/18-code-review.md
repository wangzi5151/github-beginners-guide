# Code Review 代码审查

## 代码审查的意义

- 提高代码质量
- 分享知识
- 发现潜在问题
- 统一代码风格
- 团队协作

## 审查流程

### 1. 审查前准备
- 阅读 PR 描述和关联 Issue
- 了解变更的上下文
- 在本地测试代码

### 2. 审查要点

| 类别 | 检查项 |
|------|--------|
| 功能 | 逻辑是否正确，边界情况 |
| 设计 | 架构是否合理 |
| 可读性 | 命名、注释、结构 |
| 性能 | 有无性能问题 |
| 安全 | 有无安全漏洞 |
| 测试 | 测试是否充分 |
| 文档 | 是否需要更新文档 |

### 3. 提供反馈

**好的反馈：**
- 具体指出问题位置
- 解释为什么有问题
- 提供改进建议
- 使用代码建议

**不好的反馈：**
- "代码不好"
- "这有问题"
- 没有建设性的批评

## 使用 GitHub 审查功能

### 请求审查
```
在 PR 中点击 Reviewers → 选择审查者
```

### 提交审查
- **Comment**：仅评论
- **Approve**：批准
- **Request changes**：要求修改

### 代码建议
在行内评论中使用 **Suggestion** 功能：

```suggestion
修改后的代码
```

## 自动化审查

### Linter 集成
```yaml
# .github/workflows/lint.yml
name: Lint
on: pull_request
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm run lint
```

### 测试检查
```yaml
name: Test
on: pull_request
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm test
```

## 代码审查最佳实践

### 作为审查者
1. 及时审查，不要拖延
2. 保持友好和建设性
3. 区分必须修改和建议修改
4. 指出优点

### 作为被审查者
1. 不要把审查当成个人攻击
2. 认真考虑每条反馈
3. 及时响应和修改
4. 可以解释和讨论

## 下一步

[GitHub Actions 自动化 →](19-github-actions.md)
