# 练习 24：创建 Issue 表单和 PR 模板

## 学习目标

通过本次练习，你将掌握以下技能：

- 理解 Issue 表单和 PR 模板的价值
- 创建 YAML 格式的 Issue 表单
- 设计不同类型的 Issue 模板
- 创建 Pull Request 模板
- 配置自动化标签分配
- 设置 Issue 和 PR 的验证规则

## 前置要求

- 拥有 GitHub 账号和一个代码仓库
- 基本的 YAML 语法知识
- 了解 GitHub Issues 和 Pull Requests 的基本使用

## 第一部分：理解 Issue 表单

### 为什么需要 Issue 表单？

传统的 Issue 创建方式是自由格式的文本框，这会导致：

- 信息不完整，需要反复沟通
- 格式不统一，难以快速浏览和分类
- 难以自动化处理（如自动打标签）
- 维护者需要花费大量时间整理信息

Issue 表单通过结构化的表单字段解决了这些问题，确保提交者提供完整、规范的信息。

## 第二部分：创建基本的 Issue 表单

Issue 表单使用 YAML 格式定义，支持多种字段类型，包括文本输入框、多行文本区域、下拉选择框、复选框等。通过合理组合这些字段类型，你可以设计出既美观又实用的 Issue 表单。在设计表单时，需要考虑用户体验，避免填写过多的必填字段，同时确保收集到足够的信息来帮助维护者快速理解和处理问题。好的表单设计应该做到：字段名称清晰易懂、提供适当的占位符提示、合理设置必填和选填字段、以及使用合适的字段类型来引导用户输入正确格式的信息。

### 步骤 1：创建模板目录

```bash
# 进入项目目录
cd your-project

# 创建 Issue 模板目录
mkdir -p .github/ISSUE_TEMPLATE

# 查看目录结构
ls -la .github/ISSUE_TEMPLATE/
```

### 步骤 2：创建 Bug 报告模板

创建文件 `.github/ISSUE_TEMPLATE/bug_report.yml`：

```yaml
name: Bug 报告
description: 报告一个 Bug
title: "[Bug]: "
labels: ["bug", "triage"]
assignees: []
body:
  - type: markdown
    attributes:
      value: |
        感谢你花时间报告这个 Bug！请填写以下信息，帮助我们快速定位和修复问题。

  - type: textarea
    id: description
    attributes:
      label: Bug 描述
      description: 清晰简洁地描述这个 Bug
      placeholder: 请描述你遇到的问题...
    validations:
      required: true

  - type: textarea
    id: steps
    attributes:
      label: 复现步骤
      description: 提供复现问题的详细步骤
      placeholder: |
        1. 进入 '...'
        2. 点击 '...'
        3. 滚动到 '...'
        4. 看到错误
    validations:
      required: true

  - type: textarea
    id: expected
    attributes:
      label: 期望行为
      description: 描述你期望发生的行为
      placeholder: 请描述你期望的结果...
    validations:
      required: true

  - type: textarea
    id: actual
    attributes:
      label: 实际行为
      description: 描述实际发生的行为
      placeholder: 请描述实际发生的结果...
    validations:
      required: true

  - type: textarea
    id: screenshots
    attributes:
      label: 截图
      description: 如果适用，添加截图来帮助解释问题
      placeholder: 拖拽图片到此处上传...
    validations:
      required: false

  - type: dropdown
    id: os
    attributes:
      label: 操作系统
      description: 你使用的操作系统
      options:
        - Windows 11
        - Windows 10
        - macOS Sonoma
        - macOS Ventura
        - Ubuntu 22.04
        - Ubuntu 20.04
        - 其他（请在描述中说明）
    validations:
      required: true

  - type: dropdown
    id: browser
    attributes:
      label: 浏览器
      description: 你使用的浏览器（如果是前端问题）
      options:
        - Chrome
        - Firefox
        - Safari
        - Edge
        - 其他
        - 不适用
    validations:
      required: false

  - type: input
    id: version
    attributes:
      label: 项目版本
      description: 你使用的项目版本号
      placeholder: "例如: 1.2.3"
    validations:
      required: true

  - type: textarea
    id: environment
    attributes:
      label: 运行环境
      description: 提供其他相关的环境信息
      placeholder: |
        - Node.js 版本: 20.x
        - npm 版本: 10.x
        - 数据库: PostgreSQL 16
    validations:
      required: false

  - type: textarea
    id: logs
    attributes:
      label: 日志信息
      description: 如果有相关的错误日志，请粘贴在这里
      render: shell
    validations:
      required: false

  - type: checkboxes
    id: terms
    attributes:
      label: 确认事项
      description: 在提交前请确认以下事项
      options:
        - label: 我已经搜索了现有的 Issue，确认这不是重复问题
          required: true
        - label: 我已经阅读了项目文档和 FAQ
          required: true
```

### 步骤 3：创建功能请求模板

创建文件 `.github/ISSUE_TEMPLATE/feature_request.yml`：

```yaml
name: 功能请求
description: 提出新功能建议
title: "[Feature]: "
labels: ["enhancement"]
assignees: []
body:
  - type: markdown
    attributes:
      value: |
        感谢你的功能建议！请填写以下信息，帮助我们更好地理解你的需求。

  - type: textarea
    id: problem
    attributes:
      label: 问题描述
      description: 描述你遇到的问题或痛点
      placeholder: 我在使用...时，总是觉得...
    validations:
      required: true

  - type: textarea
    id: solution
    attributes:
      label: 期望的解决方案
      description: 描述你希望的功能
      placeholder: 我希望...
    validations:
      required: true

  - type: textarea
    id: alternatives
    attributes:
      label: 替代方案
      description: 描述你考虑过的其他替代方案
      placeholder: 我尝试过...
    validations:
      required: false

  - type: dropdown
    id: priority
    attributes:
      label: 优先级
      description: 这个功能对你的重要程度
      options:
        - 最好有 - 有这个功能会更好
        - 重要 - 显著改善使用体验
        - 非常重要 - 严重影响工作流程
    validations:
      required: true

  - type: dropdown
    id: category
    attributes:
      label: 功能类别
      description: 这个功能属于哪个类别
      options:
        - 用户界面/用户体验
        - 性能优化
        - API/接口
        - 文档
        - 测试
        - 其他
    validations:
      required: true

  - type: textarea
    id: additional
    attributes:
      label: 附加信息
      description: 提供任何其他相关的信息、截图或参考资料
      placeholder: 在这里添加任何额外的上下文...
    validations:
      required: false

  - type: checkboxes
    id: terms
    attributes:
      label: 确认事项
      options:
        - label: 我愿意参与这个功能的开发
          required: false
        - label: 我已经搜索了现有的功能请求，确认这不是重复建议
          required: true
```

### 步骤 4：创建安全漏洞报告模板

创建文件 `.github/ISSUE_TEMPLATE/security_vulnerability.yml`：

```yaml
name: 安全漏洞报告
description: 报告安全漏洞（敏感信息请通过安全渠道提交）
title: "[Security]: "
labels: ["security", "critical"]
assignees: []
body:
  - type: markdown
    attributes:
      value: |
        ## ⚠️ 安全漏洞报告须知

        如果你发现的是**严重的安全漏洞**，请不要在公开 Issue 中报告。
        请通过 GitHub 的私有安全漏洞报告功能提交，或发送邮件到 security@example.com。

        对于低风险的安全问题，可以使用此表单。

  - type: dropdown
    id: severity
    attributes:
      label: 严重程度
      description: 评估这个安全问题的严重程度
      options:
        - 低 - 影响有限，难以利用
        - 中 - 有一定影响，需要特定条件
        - 高 - 可能导致数据泄露或系统受损
        - 严重 - 可能导致大规模安全事件
    validations:
      required: true

  - type: textarea
    id: description
    attributes:
      label: 漏洞描述
      description: 描述发现的安全问题
      placeholder: 请描述安全漏洞...
    validations:
      required: true

  - type: textarea
    id: impact
    attributes:
      label: 潜在影响
      description: 描述这个漏洞可能造成的影响
      placeholder: 这个漏洞可能导致...
    validations:
      required: true

  - type: textarea
    id: reproduction
    attributes:
      label: 复现步骤
      description: 提供复现问题的步骤
      placeholder: |
        1. 访问 '...'
        2. 输入 '...'
        3. 观察到安全问题
    validations:
      required: true

  - type: checkboxes
    id: disclosure
    attributes:
      label: 披露计划
      options:
        - label: 我同意在修复发布前不公开披露此漏洞
          required: true
        - label: 我愿意配合验证修复
          required: false
```

### 步骤 5：创建文档改进模板

创建文件 `.github/ISSUE_TEMPLATE/documentation.yml`：

```yaml
name: 文档改进
description: 报告文档问题或提出改进建议
title: "[Docs]: "
labels: ["documentation"]
assignees: []
body:
  - type: dropdown
    id: type
    attributes:
      label: 文档问题类型
      options:
        - 错误/不准确的信息
        - 缺失的内容
        - 不清晰的说明
        - 过时的内容
        - 翻译问题
        - 其他
    validations:
      required: true

  - type: input
    id: page
    attributes:
      label: 相关页面
      description: 提供相关文档页面的链接
      placeholder: "https://docs.example.com/..."
    validations:
      required: false

  - type: textarea
    id: description
    attributes:
      label: 问题描述
      description: 描述文档中的问题
      placeholder: 在文档的...部分，存在以下问题...
    validations:
      required: true

  - type: textarea
    id: suggestion
    attributes:
      label: 改进建议
      description: 提出你的改进建议
      placeholder: 建议修改为...
    validations:
      required: false
```

## 第三部分：创建 Issue 模板配置文件

### 创建配置文件

创建文件 `.github/ISSUE_TEMPLATE/config.yml`：

```yaml
blank_issues_enabled: true
contact_links:
  - name: 使用帮助
    url: https://github.com/your-org/your-repo/discussions
    about: 如果你需要使用帮助，请在 Discussions 中提问
  - name: 安全漏洞报告
    url: https://github.com/your-org/your-repo/security/advisories/new
    about: 对于严重的安全漏洞，请通过私有渠道报告
  - name: 查看文档
    url: https://docs.example.com
    about: 在提交 Issue 前，请先查看项目文档
```

## 第四部分：创建 Pull Request 模板

Pull Request 模板是指导代码贡献者提供规范描述的另一种重要工具。与 Issue 表单类似，PR 模板确保了代码变更描述的完整性和一致性。一个好的 PR 模板应该包含变更描述、变更类型、测试说明、截图（如果涉及界面变更）、以及自查清单等内容。通过使用 PR 模板，代码审查者可以快速了解变更的背景和目的，从而更高效地进行代码审查。同时，自查清单可以帮助贡献者在提交代码前进行自我检查，确保代码质量。PR 模板还支持 Markdown 格式，你可以充分利用表格、列表、链接等元素来组织信息，让描述更加清晰和专业。

### 步骤 1：创建基本 PR 模板

创建文件 `.github/PULL_REQUEST_TEMPLATE.md`：

```markdown
## 变更描述

简要描述这个 PR 做了什么变更。

## 变更类型

请勾选适用的类型：

- [ ] Bug 修复 (修复了一个问题)
- [ ] 新功能 (添加了新功能)
- [ ] 破坏性变更 (会导致现有功能无法正常工作)
- [ ] 文档更新
- [ ] 代码重构 (没有功能变更)
- [ ] 性能优化
- [ ] 测试相关
- [ ] CI/CD 相关
- [ ] 其他

## 相关 Issue

请链接相关的 Issue：

- 修复 #(issue 编号)
- 关闭 #(issue 编号)
- 相关 #(issue 编号)

## 测试说明

描述你如何测试这些变更：

- [ ] 添加了新的单元测试
- [ ] 已有的所有测试都通过
- [ ] 手动测试了以下场景:
  - [ ] 场景 1: ...
  - [ ] 场景 2: ...

## 截图（如适用）

如果变更涉及 UI，请提供截图：

| 变更前 | 变更后 |
|--------|--------|
| 截图   | 截图   |

## 自查清单

请在提交前确认以下事项：

- [ ] 我的代码遵循项目的代码风格
- [ ] 我已经进行了自我代码审查
- [ ] 我已经添加了必要的注释，特别是在难以理解的地方
- [ ] 我已经更新了相关文档
- [ ] 我的变更不会产生新的警告
- [ ] 我已经添加了证明我的修复有效或我的功能能工作的测试
- [ ] 新的和现有的单元测试在我的变更下都能通过
- [ ] 任何依赖的变更都已合并并发布

## 额外信息

提供任何其他你认为审查者需要知道的信息。
```

### 步骤 2：为不同类型的 PR 创建模板

可以创建多个 PR 模板，放在 `.github/PULL_REQUEST_TEMPLATE/` 目录下：

```bash
mkdir -p .github/PULL_REQUEST_TEMPLATE
```

创建 Bug 修复专用模板 `.github/PULL_REQUEST_TEMPLATE/bugfix.md`：

```markdown
---
name: Bug 修复
about: 修复一个 Bug
labels: bug
---

## Bug 描述

简要描述修复的 Bug：

## 根本原因分析

解释导致这个 Bug 的根本原因：

## 修复方案

描述你的修复方案：

## 测试验证

- [ ] 添加了重现 Bug 的测试用例
- [ ] 验证测试用例现在通过
- [ ] 没有引入新的问题

## 相关 Issue

修复 #

## 回归风险评估

评估这个修复可能导致的回归风险：

- 低 / 中 / 高

风险说明：
```

创建功能开发专用模板 `.github/PULL_REQUEST_TEMPLATE/feature.md`：

```markdown
---
name: 新功能
about: 添加一个新功能
labels: enhancement
---

## 功能描述

描述这个新功能：

## 设计文档

如果适用，链接相关的设计文档或讨论：

## 实现方案

描述你的实现方案和架构设计：

## 使用示例

提供使用这个新功能的示例代码：

## 测试覆盖

- [ ] 添加了单元测试
- [ ] 添加了集成测试
- [ ] 添加了文档示例

## 文档更新

- [ ] 更新了 API 文档
- [ ] 更新了使用指南
- [ ] 更新了 CHANGELOG

## 相关 Issue

实现 #
```

## 第五部分：配置自动化标签分配

手动为 Issue 和 Pull Request 添加标签不仅耗时，而且容易出现遗漏或不一致的情况。通过配置自动化标签分配，你可以根据预设的规则自动为 Issue 和 PR 添加合适的标签。GitHub 提供了多种自动化标签的方式，包括基于文件路径的标签分配（适用于 PR）、基于内容关键词的标签分配（适用于 Issue）、以及基于提交信息的标签分配。自动化标签分配不仅节省了维护者的时间，还能帮助团队更好地分类和跟踪问题。例如，你可以自动为涉及前端代码的 PR 添加"前端"标签，为包含"性能"关键词的 Issue 添加"性能优化"标签。结合 GitHub 的项目看板和过滤功能，自动化标签可以大大提升项目管理的效率。

### 步骤 1：创建标签配置文件

创建文件 `.github/labeler.yml`：

```yaml
# 基于文件路径的标签配置
documentation:
  - changed-files:
    - any-glob-to-any-file:
      - docs/**
      - "**/*.md"
      - "**/*.rst"

frontend:
  - changed-files:
    - any-glob-to-any-file:
      - src/components/**
      - src/pages/**
      - src/styles/**
      - "**/*.tsx"
      - "**/*.jsx"
      - "**/*.css"
      - "**/*.scss"

backend:
  - changed-files:
    - any-glob-to-any-file:
      - src/api/**
      - src/services/**
      - src/models/**
      - "**/*.py"
      - "**/*.java"

tests:
  - changed-files:
    - any-glob-to-any-file:
      - tests/**
      - **/*.test.*
      - **/*.spec.*

ci-cd:
  - changed-files:
    - any-glob-to-any-file:
      - .github/**
      - Dockerfile
      - docker-compose.yml

dependencies:
  - changed-files:
    - any-glob-to-any-file:
      - package.json
      - package-lock.json
      - requirements.txt
      - Pipfile
      - pom.xml

database:
  - changed-files:
    - any-glob-to-any-file:
      - migrations/**
      - src/models/**
      - **/*.sql
```

### 步骤 2：创建标签分配工作流

创建文件 `.github/workflows/labeler.yml`：

```yaml
name: Label Pull Requests

on:
  pull_request_target:
    types: [opened, synchronize, reopened]

permissions:
  contents: read
  pull-requests: write

jobs:
  label:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/labeler@v5
        with:
          repo-token: "${{ secrets.GITHUB_TOKEN }}"
          configuration-path: .github/labeler.yml
          sync-labels: true
```

### 步骤 3：基于 Issue 内容自动分配标签

创建文件 `.github/workflows/issue-labeler.yml`：

```yaml
name: Label Issues

on:
  issues:
    types: [opened, edited]

permissions:
  issues: write

jobs:
  label:
    runs-on: ubuntu-latest
    steps:
      - uses: github/issue-labeler@v3
        with:
          repo-token: "${{ secrets.GITHUB_TOKEN }}"
          configuration-path: .github/issue-labeler.yml
```

创建 Issue 标签配置 `.github/issue-labeler.yml`：

```yaml
# 基于 Issue 标题和内容的标签配置
bug:
  - "(bug|错误|异常|崩溃|crash|error|exception)"

feature:
  - "(功能|特性|feature|enhancement|建议|suggestion)"

performance:
  - "(性能|慢|卡顿|performance|slow|timeout)"

security:
  - "(安全|漏洞|security|vulnerability|CVE)"

documentation:
  - "(文档|说明|documentation|docs|readme)"

question:
  - "(问题|疑问|如何|怎么|question|how|why)"
```

## 第六部分：进阶挑战

在掌握了基本的 Issue 表单和 PR 模板创建后，你可以尝试更高级的配置来进一步优化项目的贡献流程。以下进阶挑战将帮助你探索 GitHub 提供的更多自动化功能，包括条件表单、项目看板集成、以及自动关闭不活跃 Issue 等功能。这些高级技巧可以显著提升项目管理的自动化程度，让维护者能够将更多精力投入到核心功能的开发中。

### 挑战 1：创建带有条件逻辑的表单

利用 GitHub Issue 表单的高级功能：

```yaml
name: 高级 Bug 报告
description: 带有条件字段的 Bug 报告
body:
  - type: dropdown
    id: bug-type
    attributes:
      label: Bug 类型
      options:
        - 前端 Bug
        - 后端 Bug
        - 数据库 Bug
    validations:
      required: true

  - type: textarea
    id: frontend-details
    attributes:
      label: 前端详细信息
      description: 请提供前端相关信息
      placeholder: |
        - 浏览器版本:
        - 屏幕分辨率:
        - 操作系统版本:
    validations:
      required: false
    # 注意：GitHub 目前不支持条件显示，但可以通过注释说明

  - type: textarea
    id: backend-details
    attributes:
      label: 后端详细信息
      description: 请提供后端相关信息
      placeholder: |
        - 服务器操作系统:
        - 运行时版本:
        - 数据库版本:
    validations:
      required: false
```

### 挑战 2：创建项目看板自动化

配置 GitHub Actions 自动将 Issue 添加到项目看板：

```yaml
# .github/workflows/project-board.yml
name: Add to Project Board

on:
  issues:
    types: [opened]

jobs:
  add-to-project:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/add-to-project@v1.0.2
        with:
          project-url: https://github.com/orgs/your-org/projects/1
          github-token: ${{ secrets.PROJECT_TOKEN }}
          labeled: bug, enhancement
```

### 挑战 3：创建 Issue 和 PR 的自动关闭规则

配置在特定条件下自动关闭 Issue：

```yaml
# .github/workflows/stale.yml
name: Close Stale Issues

on:
  schedule:
    - cron: '0 0 * * *'

permissions:
  issues: write
  pull-requests: write

jobs:
  stale:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/stale@v9
        with:
          repo-token: ${{ secrets.GITHUB_TOKEN }}
          days-before-stale: 30
          days-before-close: 7
          stale-issue-label: stale
          stale-issue-message: |
            这个 Issue 已经30天没有活动了，已被标记为 stale。
            如果这个 Issue 仍然相关，请留下评论。
            否则，它将在7天后自动关闭。
          stale-pr-label: stale
          stale-pr-message: |
            这个 PR 已经30天没有活动了，已被标记为 stale。
            请更新状态或留下评论。
```

## 验证清单

完成练习后，请确认以下事项：

- [ ] 创建了至少一个 Issue 表单模板
- [ ] 表单包含多种字段类型（输入框、下拉框、复选框等）
- [ ] 配置了 `config.yml` 文件
- [ ] 创建了 Pull Request 模板
- [ ] 了解如何配置自动化标签分配
- [ ] 测试了创建 Issue 和 PR 的流程

## 常见问题

### Q1：如何测试 Issue 表单？

在仓库的 Issues 页面点击 "New Issue"，你会看到配置的模板。选择模板后即可测试表单的各个字段。

### Q2：如何限制只能通过模板创建 Issue？

在 `config.yml` 中设置 `blank_issues_enabled: false` 即可禁用空白 Issue。

### Q3：PR 模板没有生效怎么办？

确保 PR 模板文件位于正确的位置：
- 单个模板：`.github/PULL_REQUEST_TEMPLATE.md`
- 多个模板：`.github/PULL_REQUEST_TEMPLATE/*.md`

## 总结

通过本次练习，你学会了如何创建结构化的 Issue 表单和 PR 模板。这些工具可以帮助你规范项目的贡献流程，提高团队协作效率，减少沟通成本。自动化标签分配进一步提升了 Issue 和 PR 的管理效率。在实际项目中，建议根据团队的实际需求和项目特点来定制这些模板。例如，面向用户的项目可能需要更详细的 Bug 报告表单，而面向开发者的工具项目可能更需要功能请求模板。随着项目的演进，你可能需要不断调整和优化这些模板，以适应新的需求和流程。建立完善的贡献指南，配合 Issue 表单和 PR 模板，可以大大降低开源项目的维护成本，吸引更多贡献者参与项目开发。持续优化你的项目管理流程，将帮助你打造一个高效、友好的开源社区。
