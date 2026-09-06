# GitHub Issue/PR 模板指南

## 为什么需要模板？

模板可以帮助你：
- 收集标准化的信息
- 减少来回沟通的次数
- 提高问题处理效率
- 保持项目管理的一致性

## Issue 模板

### 创建模板

在仓库根目录创建 `.github/ISSUE_TEMPLATE/` 目录：

```
.github/
└── ISSUE_TEMPLATE/
    ├── bug_report.md
    ├── feature_request.md
    ├── documentation.md
    └── config.yml
```

### Bug 报告模板

```markdown
---
name: Bug Report
about: 报告一个问题
title: '[BUG] '
labels: bug
assignees: ''
---

## 描述

简要描述问题。

## 复现步骤

1. 打开 '...'
2. 点击 '...'
3. 滚动到 '...'
4. 看到错误

## 期望行为

描述期望的行为。

## 实际行为

描述实际的行为。

## 截图

如果适用，添加截图帮助解释问题。

## 环境信息

- OS: [例如 Windows 11]
- Browser: [例如 Chrome 120]
- Version: [例如 1.0.0]

## 其他信息

添加任何其他关于问题的信息。
```

### 功能请求模板

```markdown
---
name: Feature Request
about: 建议新功能
title: '[FEATURE] '
labels: enhancement
assignees: ''
---

## 问题描述

描述你遇到的问题或需求。

## 建议的解决方案

描述你希望如何解决这个问题。

## 替代方案

描述你考虑过的其他解决方案。

## 附加信息

添加任何其他关于功能请求的信息。
```

## YAML 表单模板（推荐）

YAML 表单模板提供了更好的用户体验：

```yaml
# .github/ISSUE_TEMPLATE/bug_report.yml
name: Bug Report
description: 报告一个问题
title: '[BUG] '
labels: ["bug"]
body:
  - type: textarea
    id: description
    attributes:
      label: 问题描述
      description: 简要描述问题
    validations:
      required: true
  - type: textarea
    id: steps
    attributes:
      label: 复现步骤
      description: 描述如何复现问题
      value: |
        1. 打开 '...'
        2. 点击 '...'
        3. 滚动到 '...'
        4. 看到错误
    validations:
      required: true
  - type: dropdown
    id: os
    attributes:
      label: 操作系统
      options:
        - Windows
        - macOS
        - Linux
    validations:
      required: true
  - type: input
    id: version
    attributes:
      label: 版本号
      placeholder: '例如 1.0.0'
    validations:
      required: true
  - type: textarea
    id: screenshots
    attributes:
      label: 截图
      description: 如果适用，添加截图
```

## Pull Request 模板

### 创建 PR 模板

在仓库根目录创建 `.github/PULL_REQUEST_TEMPLATE.md`：

```markdown
## 描述

简要描述这个 PR 的内容。

## 变更类型

- [ ] Bug 修复
- [ ] 新功能
- [ ] 文档更新
- [ ] 代码重构
- [ ] 测试更新
- [ ] 其他

## 测试

描述你如何测试这些更改。

- [ ] 已在本地测试
- [ ] 已添加/更新测试
- [ ] 所有现有测试通过

## 截图（如适用）

添加截图说明更改。

## 相关 Issue

关联相关 Issue：Closes #123

## 检查清单

- [ ] 代码遵循项目规范
- [ ] 已自我审查代码
- [ ] 已添加注释（如需要）
- [ ] 已更新文档（如需要）
- [ ] 已添加测试（如适用）
- [ ] 所有现有测试通过
```

## 配置模板选择器

### config.yml

```yaml
# .github/ISSUE_TEMPLATE/config.yml
blank_issues_enabled: false
contact_links:
  - name: 技术支持
    url: https://github.com/your-org/your-repo/discussions
    about: 请在 Discussions 中提问
  - name: 文档
    url: https://docs.example.com
    about: 查看文档获取帮助
```

## 使用 GitHub CLI 创建

```bash
# 创建 Issue
gh issue create --title "Bug: 问题描述" --body-file .github/ISSUE_TEMPLATE/bug_report.md

# 创建 PR
gh pr create --title "feat: 新功能" --body-file .github/PULL_REQUEST_TEMPLATE.md
```

## 最佳实践

1. **保持模板简洁**：只要求必要信息
2. **提供清晰的选项**：使用下拉菜单和复选框
3. **添加有用的默认值**：帮助用户快速填写
4. **定期更新模板**：根据项目需求调整
5. **使用 YAML 表单**：提供更好的用户体验

## 相关资源

- [Issue 模板文档](https://docs.github.com/en/communities/using-templates-to-encourage-useful-issues-and-pull-requests)
- [PR 模板文档](https://docs.github.com/en/repositories/releasing-projects-on-github/automatically-generated-release-notes)

---

**上一篇：[README 与文档](15-readme-docs.md) | 下一篇：[Issue 问题追踪](16-issues.md)**
