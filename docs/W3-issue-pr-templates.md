# Issue 和 PR 模板完全指南

## 1. Issue 模板的作用与价值

### 1.1 为什么需要 Issue 模板

在开源项目或团队协作中，Issue 是最常用的沟通工具之一。然而，没有模板约束的 Issue 往往存在以下问题：

- **信息不完整：** 报告者忘记提供关键信息，如操作系统版本、复现步骤等，导致维护者需要反复追问
- **格式混乱：** 每个人提交 Issue 的风格不同，难以快速定位关键信息
- **分类模糊：** 不清楚应该报告 Bug 还是请求新功能，导致 Issue 被错误分类
- **效率低下：** 维护者需要花费大量时间整理和补充信息，而不是专注于解决问题

Issue 模板通过预定义的结构和必填字段，从根本上解决了这些问题。一个好的 Issue 模板应该像一份精心设计的表单，引导报告者提供完整、准确、有用的信息。

### 1.2 Issue 模板的核心价值

| 价值维度 | 具体表现 |
|----------|----------|
| **标准化** | 统一所有 Issue 的格式，便于批量处理和筛选 |
| **效率提升** | 减少维护者与报告者之间的来回沟通，加速问题解决 |
| **信息完整** | 通过必填字段确保关键信息不遗漏 |
| **自动化** | 配合 GitHub Actions 实现自动标签分配、自动指派等 |
| **新人友好** | 为首次贡献者提供清晰的提交指引 |

### 1.3 Issue 模板的类型

GitHub 支持两种格式的 Issue 模板：

- **Markdown 模板（.md）：** 传统的 Markdown 格式，用户提交时会自动填充模板内容，用户可以在模板基础上自由编辑
- **YAML 表单模板（.yml）：** 新一代的表单式模板，提供下拉菜单、复选框、输入框等表单元素，用户体验更好，信息收集更规范

**推荐使用 YAML 表单模板**，因为它能提供更好的结构化数据收集体验。

---

## 2. 创建 Issue 模板（YAML 格式）

### 2.1 目录结构

Issue 模板存放在仓库根目录的 `.github/ISSUE_TEMPLATE/` 文件夹下：

```
.github/
└── ISSUE_TEMPLATE/
    ├── bug_report.yml          # Bug 报告模板
    ├── feature_request.yml     # 功能请求模板
    ├── documentation.yml       # 文档改进建议模板
    ├── security_issue.yml      # 安全问题报告模板
    ├── config.yml              # 模板选择器配置
    └── PULL_REQUEST_TEMPLATE.md # PR 模板（也可以放在这里）
```

### 2.2 YAML 表单模板字段详解

YAML 表单模板支持以下字段类型：

| 字段类型 | 说明 | 适用场景 |
|----------|------|----------|
| `markdown` | 纯文本说明，不作为输入 | 添加提示信息、分割线 |
| `input` | 单行文本输入框 | 收集简短信息，如版本号、URL |
| `textarea` | 多行文本输入框 | 收集详细描述、复现步骤 |
| `dropdown` | 下拉菜单 | 选择单一选项，如操作系统、优先级 |
| `checkboxes` | 复选框组 | 选择多个选项，如确认事项 |
| `number` | 数字输入框 | 收集数值信息 |

### 2.3 完整 Bug 报告模板

```yaml
# .github/ISSUE_TEMPLATE/bug_report.yml
name: Bug 报告
description: 报告一个 Bug，帮助我们改进项目
title: "[Bug]: "
labels: ["bug", "needs-triage"]
assignees: []
body:
  - type: markdown
    attributes:
      value: |
        ## 感谢你报告 Bug！
        请尽可能详细地填写以下信息，这将帮助我们更快地定位和修复问题。
        
        **在提交之前，请先：**
        - 搜索现有的 Issue，确认该问题尚未被报告
        - 确保你使用的是最新版本
        - 查看[故障排除文档](https://docs.example.com/troubleshooting)

  - type: textarea
    id: description
    attributes:
      label: Bug 描述
      description: 清晰、简洁地描述这个 Bug
      placeholder: 例如：点击登录按钮后，页面没有响应，控制台报错 "TypeError: Cannot read property..."
    validations:
      required: true

  - type: textarea
    id: steps
    attributes:
      label: 复现步骤
      description: 提供详细的复现步骤
      placeholder: |
        1. 打开应用首页
        2. 点击右上角的 '登录' 按钮
        3. 输入用户名和密码
        4. 点击 '提交' 按钮
        5. 观察到错误
      value: |
        1. 
        2. 
        3. 
        4. 
    validations:
      required: true

  - type: textarea
    id: expected
    attributes:
      label: 期望行为
      description: 清晰地描述你期望发生什么
      placeholder: 例如：点击提交按钮后，应该跳转到首页并显示用户头像
    validations:
      required: true

  - type: textarea
    id: actual
    attributes:
      label: 实际行为
      description: 清晰地描述实际发生了什么
      placeholder: 例如：页面没有跳转，控制台显示 TypeError 错误
    validations:
      required: true

  - type: dropdown
    id: version
    attributes:
      label: 版本
      description: 你使用的软件版本
      options:
        - 最新版本 (main 分支)
        - v2.0.0
        - v1.9.0
        - v1.8.0
        - 其他（请在下方说明）
    validations:
      required: true

  - type: dropdown
    id: os
    attributes:
      label: 操作系统
      multiple: true
      options:
        - Windows 11
        - Windows 10
        - macOS 14 (Sonoma)
        - macOS 13 (Ventura)
        - Ubuntu 22.04
        - Ubuntu 20.04
        - 其他 Linux 发行版
    validations:
      required: true

  - type: dropdown
    id: browser
    attributes:
      label: 浏览器
      multiple: true
      options:
        - Chrome 120+
        - Firefox 120+
        - Safari 17+
        - Edge 120+
        - 不适用
    validations:
      required: false

  - type: input
    id: node-version
    attributes:
      label: Node.js 版本
      description: 运行 `node --version` 获取
      placeholder: "v20.10.0"
    validations:
      required: false

  - type: textarea
    id: logs
    attributes:
      label: 日志/错误信息
      description: 粘贴相关的日志输出或错误堆栈信息
      render: shell
    validations:
      required: false

  - type: textarea
    id: screenshots
    attributes:
      label: 截图
      description: 如果适用，添加截图帮助解释问题（可以直接拖拽图片到此处）
    validations:
      required: false

  - type: textarea
    id: environment
    attributes:
      label: 运行环境详情
      description: 提供任何其他相关的环境信息
      placeholder: |
        - 数据库版本：PostgreSQL 15
        - 内存：16GB
        - 网络环境：公司内网
    validations:
      required: false

  - type: checkboxes
    id: terms
    attributes:
      label: 确认事项
      options:
        - label: 我已经搜索了现有的 Issues，确认该问题尚未被报告
          required: true
        - label: 我正在使用最新版本
          required: false
        - label: 我愿意提交 PR 来修复这个问题
          required: false
```

### 2.4 完整功能请求模板

```yaml
# .github/ISSUE_TEMPLATE/feature_request.yml
name: 功能请求
description: 建议一个新功能或改进
title: "[Feature]: "
labels: ["enhancement", "needs-review"]
assignees: []
body:
  - type: markdown
    attributes:
      value: |
        ## 感谢你的功能建议！
        请详细描述你的需求，这将帮助我们评估和规划这个功能。

  - type: textarea
    id: problem
    attributes:
      label: 问题描述
      description: 清晰地描述你遇到的问题或痛点
      placeholder: 例如：当我在处理大量数据时，当前的搜索功能响应很慢，平均需要等待 5 秒以上
    validations:
      required: true

  - type: textarea
    id: solution
    attributes:
      label: 建议的解决方案
      description: 描述你希望如何解决这个问题
      placeholder: 例如：建议添加索引支持和分页查询功能，将大数据集拆分为小批次处理
    validations:
      required: true

  - type: textarea
    id: alternatives
    attributes:
      label: 替代方案
      description: 描述你考虑过的其他解决方案
      placeholder: 例如：我尝试过使用 Redis 缓存，但这增加了系统复杂度
    validations:
      required: false

  - type: dropdown
    id: priority
    attributes:
      label: 优先级
      description: 你认为这个功能的紧急程度
      options:
        - 高 - 严重影响工作流程
        - 中 - 有一定影响，但可以绕过
        - 低 - 改善体验的锦上添花功能
    validations:
      required: true

  - type: dropdown
    id: scope
    attributes:
      label: 影响范围
      description: 这个功能会影响哪些用户
      options:
        - 所有用户
        - 高级用户
        - 管理员
        - 开发者
        - 特定场景用户
    validations:
      required: false

  - type: textarea
    id: additional
    attributes:
      label: 附加信息
      description: 任何其他有助于理解需求的信息，如参考链接、设计稿、竞品截图等
    validations:
      required: false

  - type: checkboxes
    id: terms
    attributes:
      label: 确认事项
      options:
        - label: 我已经搜索了现有的 Issues，确认该功能尚未被请求
          required: true
        - label: 我愿意参与这个功能的讨论和设计
          required: false
        - label: 我愿意提交 PR 来实现这个功能
          required: false
```

---

## 3. Issue 表单（Issue Forms）

### 3.1 Issue Forms 的优势

Issue Forms（YAML 表单模板）相比传统的 Markdown 模板有以下优势：

| 特性 | Markdown 模板 | YAML 表单模板 |
|------|--------------|--------------|
| 用户体验 | 自由编辑，容易破坏格式 | 表单式填写，结构清晰 |
| 数据验证 | 无验证机制 | 支持必填字段验证 |
| 下拉菜单 | 不支持 | 支持单选和多选 |
| 复选框 | 使用 Markdown 语法 | 原生支持 |
| 提交后格式 | 格式可能不一致 | 格式统一规范 |
| 自动化处理 | 难以解析 | 易于自动化处理 |

### 3.2 表单元素详解

**markdown 元素 - 添加说明文字：**

```yaml
- type: markdown
  attributes:
    value: |
      ## 标题
      这里可以写 **Markdown** 格式的说明文字。
      支持链接、图片、代码块等。
```

**input 元素 - 单行输入：**

```yaml
- type: input
  id: email
  attributes:
    label: 邮箱地址
    description: 用于通知你问题的解决进展
    placeholder: "your@email.com"
    value: "默认值"
  validations:
    required: true
    regex: "^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$"
    regex_error: "请输入有效的邮箱地址"
```

**textarea 元素 - 多行输入：**

```yaml
- type: textarea
  id: description
  attributes:
    label: 描述
    description: 详细描述你的问题
    placeholder: "请输入..."
    render: shell  # 自动添加代码块格式
  validations:
    required: true
```

**dropdown 元素 - 下拉选择：**

```yaml
# 单选
- type: dropdown
  id: priority
  attributes:
    label: 优先级
    options:
      - 高
      - 中
      - 低
  validations:
    required: true

# 多选
- type: dropdown
  id: platforms
  attributes:
    label: 受影响的平台
    multiple: true
    options:
      - Windows
      - macOS
      - Linux
      - iOS
      - Android
  validations:
    required: true
```

**checkboxes 元素 - 复选框：**

```yaml
- type: checkboxes
  id: checklist
  attributes:
    label: 提交前检查
    options:
      - label: 我已经阅读了贡献指南
        required: true
      - label: 我已经搜索了现有 Issue
        required: true
      - label: 我已经尝试了最新版本
        required: false
```

**number 元素 - 数字输入：**

```yaml
- type: number
  id: users
  attributes:
    label: 受影响的用户数
    description: 大约有多少用户会受到这个问题影响
    placeholder: "100"
  validations:
    required: false
    min: 0
    max: 1000000
```

### 3.3 表单验证规则

YAML 表单模板支持以下验证规则：

```yaml
validations:
  required: true                    # 必填字段
  regex: "^[a-zA-Z0-9]+$"         # 正则表达式验证
  regex_error: "只能包含字母和数字"  # 验证失败提示
  min: 0                            # 最小值（数字字段）
  max: 100                          # 最大值（数字字段）
```

---

## 4. PR 模板创建与配置

### 4.1 PR 模板的作用

Pull Request 模板为代码审查提供标准化的信息框架，确保提交者提供足够的上下文信息，帮助审查者快速理解变更内容和目的。

### 4.2 PR 模板文件位置

PR 模板可以放在以下位置：

```
# 位置 1：根目录（最常用）
.github/PULL_REQUEST_TEMPLATE.md

# 位置 2：GitHub 目录
PULL_REQUEST_TEMPLATE.md

# 位置 3：多模板模式
.github/PULL_REQUEST_TEMPLATE/
├── bug_fix.md
├── feature.md
├── docs.md
└── refactor.md
```

### 4.3 完整 PR 模板

```markdown
<!-- .github/PULL_REQUEST_TEMPLATE.md -->

## 变更概述

<!-- 用一句话描述这个 PR 做了什么 -->

## 变更类型

<!-- 请勾选适用的类型 -->

- [ ] 🐛 Bug 修复（不破坏现有功能的变更）
- [ ] ✨ 新功能（不破坏现有功能的新功能）
- [ ] 💥 破坏性变更（会导致现有功能无法正常工作的修复或功能）
- [ ] 📝 文档更新（仅文档变更）
- [ ] ♻️ 代码重构（既不修复 Bug 也不添加功能的代码变更）
- [ ] ⚡ 性能优化（提升性能的代码变更）
- [ ] ✅ 测试（添加缺失的测试或修正现有测试）
- [ ] 🔧 构建/CI（影响构建系统或外部依赖的变更）
- [ ] 🔙 回滚（回滚之前的变更）

## 变更详情

<!-- 详细描述你的变更，包括实现方式、设计决策等 -->

### 主要变更

- 

### 技术实现

- 

## 关联 Issue

<!-- 使用关键词自动关闭 Issue -->
<!-- 参考：https://docs.github.com/en/issues/tracking-your-work-with-issues/linking-a-pull-request-to-an-issue -->

Closes #
Fixes #
Resolves #

## 测试说明

<!-- 描述你如何测试这些变更 -->

### 测试环境

- OS: 
- Node.js: 
- Browser: 

### 测试用例

- [ ] 单元测试通过
- [ ] 集成测试通过
- [ ] 手动测试通过

### 测试步骤

1. 
2. 
3. 

## 截图/录屏

<!-- 如果是 UI 变更，请提供截图或录屏 -->

| 变更前 | 变更后 |
|--------|--------|
| ![before](url) | ![after](url) |

## 自查清单

<!-- 请在提交前完成以下检查 -->

### 代码质量

- [ ] 代码遵循项目的编码规范
- [ ] 已进行自我代码审查
- [ ] 代码有充分的注释，特别是难以理解的地方
- [ ] 已添加相应的测试（如适用）
- [ ] 新增和已有的测试都通过了

### 文档更新

- [ ] 已更新相关文档（如 README、API 文档等）
- [ ] 已更新 CHANGELOG（如适用）
- [ ] 变更不会影响现有文档的准确性

### 安全性

- [ ] 没有硬编码的密钥或敏感信息
- [ ] 没有引入新的安全漏洞
- [ ] 依赖项已检查安全漏洞

### 兼容性

- [ ] 变更向后兼容
- [ ] 如有破坏性变更，已在描述中说明
- [ ] 已考虑不同浏览器/设备的兼容性

## 额外说明

<!-- 任何审查者需要知道的额外信息 -->
```

### 4.4 特定类型的 PR 模板

**Bug 修复模板：**

```markdown
<!-- .github/PULL_REQUEST_TEMPLATE/bug_fix.md -->
## Bug 修复

### 问题描述

<!-- 描述被修复的 Bug -->

### 根本原因

<!-- 解释 Bug 的根本原因 -->

### 修复方案

<!-- 描述你的修复方案 -->

### 修复前后的对比

```typescript
// 修复前
const result = data.filter(item => item.id == id);

// 修复后
const result = data.filter(item => item.id === id);
```

### 测试覆盖

- [ ] 已添加复现 Bug 的测试用例
- [ ] 测试用例在修复前会失败
- [ ] 测试用例在修复后通过

Closes #
```

---

## 5. 多模板配置

### 5.1 配置模板选择器

通过 `config.yml` 文件，你可以自定义 Issue 创建页面的模板选择器：

```yaml
# .github/ISSUE_TEMPLATE/config.yml
blank_issues_enabled: false  # 禁止创建空白 Issue
contact_links:
  - name: 💬 提问与讨论
    url: https://github.com/your-org/your-repo/discussions
    about: 如果你有使用问题或想讨论某个话题，请在 Discussions 中发帖
  - name: 📖 查看文档
    url: https://docs.example.com
    about: 在提交 Issue 之前，请先查阅文档
  - name: 🔒 安全漏洞报告
    url: https://github.com/your-org/your-repo/security/advisories/new
    about: 如果你发现了安全漏洞，请通过安全通告报告，不要公开提交 Issue
  - name: 💰 赞助
    url: https://github.com/sponsors/your-org
    about: 如果这个项目对你有帮助，欢迎赞助支持
```

### 5.2 多 PR 模板配置

```yaml
# .github/PULL_REQUEST_TEMPLATE/bug_fix.md
---
name: Bug 修复
about: 修复一个已知的 Bug
labels: ["bug", "fix"]
---

## Bug 修复 PR

### 修复的 Issue

Closes #

### 问题描述

### 修复方案

### 测试
```

```yaml
# .github/PULL_REQUEST_TEMPLATE/feature.md
---
name: 新功能
about: 添加一个新功能
labels: ["enhancement", "feature"]
---

## 新功能 PR

### 功能描述

### 实现方案

### 测试覆盖

### 文档更新
```

### 5.3 使用 URL 参数选择模板

在创建 Issue 或 PR 时，可以通过 URL 参数直接指定模板：

```
# 使用 Issue 模板
https://github.com/owner/repo/issues/new?template=bug_report.yml

# 使用 PR 模板
https://github.com/owner/repo/compare/main...feature-branch?template=feature.md

# 预填充字段
https://github.com/owner/repo/issues/new?title=[Bug]:&labels=bug&body=## 描述
```

---

## 6. 模板最佳实践

### 6.1 模板设计原则

**简洁性原则：**

- 只要求必要的信息，避免过度设计
- 使用清晰、简洁的语言
- 提供有用的默认值和示例

**引导性原则：**

- 使用 placeholder 属性提供填写示例
- 通过 markdown 元素添加说明和链接
- 使用必填字段确保关键信息不遗漏

**一致性原则：**

- 所有模板使用统一的命名规范
- 保持相似的结构和格式
- 使用一致的标签和分类

### 6.2 标签策略

```yaml
# Bug 报告标签
labels: ["bug", "needs-triage"]

# 功能请求标签
labels: ["enhancement", "needs-review"]

# 文档问题标签
labels: ["documentation", "good first issue"]

# 优先级标签
labels: ["priority: high", "priority: medium", "priority: low"]
```

### 6.3 模板命名规范

```
.github/ISSUE_TEMPLATE/
├── 01_bug_report.yml        # 使用数字前缀控制排序
├── 02_feature_request.yml
├── 03_documentation.yml
├── 04_question.yml
└── config.yml
```

### 6.4 国际化考虑

为国际化的项目提供多语言模板：

```yaml
# 英文版
name: Bug Report
description: Report a bug

# 中文版
name: Bug 报告
description: 报告一个问题
```

### 6.5 模板维护

定期审查和更新模板：

1. **收集反馈：** 询问贡献者对模板的使用体验
2. **分析数据：** 统计哪些字段经常被留空或填写不当
3. **持续优化：** 根据反馈调整模板内容和验证规则
4. **版本控制：** 模板变更应通过 PR 进行，便于团队审查

---

## 7. 自动化标签分配

### 7.1 基于模板自动标签

YAML 模板支持自动分配标签：

```yaml
# Bug 报告自动分配 bug 和 needs-triage 标签
labels: ["bug", "needs-triage"]

# 功能请求自动分配 enhancement 标签
labels: ["enhancement"]
```

### 7.2 基于内容自动标签

使用 GitHub Actions 根据 Issue 内容自动分配标签：

```yaml
# .github/workflows/auto-label.yml
name: 自动标签

on:
  issues:
    types: [opened, edited]

jobs:
  label:
    runs-on: ubuntu-latest
    permissions:
      issues: write
    steps:
      - uses: actions/checkout@v4

      - name: 根据标题分配标签
        uses: actions/github-script@v7
        with:
          script: |
            const title = context.payload.issue.title.toLowerCase();
            const labels = [];
            
            if (title.includes('[bug]') || title.includes('错误')) {
              labels.push('bug');
            }
            if (title.includes('[feature]') || title.includes('功能')) {
              labels.push('enhancement');
            }
            if (title.includes('[docs]') || title.includes('文档')) {
              labels.push('documentation');
            }
            if (title.includes('[security]') || title.includes('安全')) {
              labels.push('security');
            }
            
            if (labels.length > 0) {
              await github.rest.issues.addLabels({
                owner: context.repo.owner,
                repo: context.repo.repo,
                issue_number: context.issue.number,
                labels: labels
              });
            }

      - name: 根据内容分配标签
        uses: actions/github-script@v7
        with:
          script: |
            const body = context.payload.issue.body || '';
            const labels = [];
            
            // 检测优先级
            if (body.includes('严重') || body.includes('critical')) {
              labels.push('priority: critical');
            } else if (body.includes('高') || body.includes('high')) {
              labels.push('priority: high');
            }
            
            // 检测操作系统
            if (body.includes('Windows')) labels.push('os: windows');
            if (body.includes('macOS')) labels.push('os: macos');
            if (body.includes('Linux')) labels.push('os: linux');
            
            if (labels.length > 0) {
              await github.rest.issues.addLabels({
                owner: context.repo.owner,
                repo: context.repo.repo,
                issue_number: context.issue.number,
                labels: labels
              });
            }
```

### 7.3 使用 Labeler Action

```yaml
# .github/workflows/labeler.yml
name: PR 标签

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  label:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write
    steps:
      - uses: actions/labeler@v5
        with:
          repo-token: "${{ secrets.GITHUB_TOKEN }}"
```

```yaml
# .github/labeler.yml
# 根据修改的文件路径分配标签
frontend:
  - changed-files:
    - any-glob-to-any-file: 'frontend/**'

backend:
  - changed-files:
    - any-glob-to-any-file: 'backend/**'

documentation:
  - changed-files:
    - any-glob-to-any-file:
      - 'docs/**'
      - '*.md'

ci:
  - changed-files:
    - any-glob-to-any-file: '.github/**'
```

---

## 8. Issue/PR 模板与 GitHub Actions 联动

### 8.1 Issue 质量检查

创建一个 Action 来检查 Issue 是否遵循模板格式：

```yaml
# .github/workflows/issue-quality.yml
name: Issue 质量检查

on:
  issues:
    types: [opened]

jobs:
  check:
    runs-on: ubuntu-latest
    permissions:
      issues: write
    steps:
      - name: 检查 Issue 质量
        uses: actions/github-script@v7
        with:
          script: |
            const issue = context.payload.issue;
            const body = issue.body || '';
            const checks = [];
            
            // 检查是否包含必要的部分
            const requiredSections = ['## 描述', '## 复现步骤'];
            for (const section of requiredSections) {
              if (!body.includes(section)) {
                checks.push(`❌ 缺少 ${section} 部分`);
              }
            }
            
            // 检查是否有截图（如果是 Bug 报告）
            if (issue.labels.some(l => l.name === 'bug')) {
              if (!body.includes('![') && !body.includes('截图')) {
                checks.push('⚠️ Bug 报告建议添加截图');
              }
            }
            
            // 如果有问题，添加评论
            if (checks.length > 0) {
              const comment = [
                '## Issue 质量检查',
                '',
                '感谢你的 Issue！以下是检查结果：',
                '',
                ...checks,
                '',
                '请根据提示补充相关信息，这将帮助我们更快地解决问题。'
              ].join('\n');
              
              await github.rest.issues.createComment({
                owner: context.repo.owner,
                repo: context.repo.repo,
                issue_number: issue.number,
                body: comment
              });
            }
```

### 8.2 PR 检查清单验证

```yaml
# .github/workflows/pr-checklist.yml
name: PR 检查清单

on:
  pull_request:
    types: [opened, edited]

jobs:
  checklist:
    runs-on: ubuntu-latest
    permissions:
      pull-requests: write
    steps:
      - name: 检查清单完成度
        uses: actions/github-script@v7
        with:
          script: |
            const pr = context.payload.pull_request;
            const body = pr.body || '';
            
            // 统计未完成的检查项
            const unchecked = (body.match(/- \[ \]/g) || []).length;
            const checked = (body.match(/- \[x\]/g) || []).length;
            const total = unchecked + checked;
            
            if (total > 0) {
              const completionRate = Math.round((checked / total) * 100);
              
              // 如果完成度低于 80%，添加提醒
              if (completionRate < 80) {
                await github.rest.issues.createComment({
                  owner: context.repo.owner,
                  repo: context.repo.repo,
                  issue_number: pr.number,
                  body: `## 检查清单提醒\n\n当前完成度：${completionRate}% (${checked}/${total})\n\n请确保完成所有必要的检查项后再请求审查。`
                });
              }
            }
```

### 8.3 自动审查者分配

```yaml
# .github/workflows/auto-reviewer.yml
name: 自动分配审查者

on:
  pull_request:
    types: [opened]

jobs:
  assign:
    runs-on: ubuntu-latest
    permissions:
      pull-requests: write
    steps:
      - name: 根据变更文件分配审查者
        uses: actions/github-script@v7
        with:
          script: |
            const pr = context.payload.pull_request;
            const files = await github.rest.pulls.listFiles({
              owner: context.repo.owner,
              repo: context.repo.repo,
              pull_number: pr.number
            });
            
            const reviewers = new Set();
            const filePatterns = {
              'frontend/': ['frontend-team'],
              'backend/': ['backend-team'],
              '.github/': ['devops-team'],
              'docs/': ['docs-team']
            };
            
            for (const file of files.data) {
              for (const [pattern, teams] of Object.entries(filePatterns)) {
                if (file.filename.startsWith(pattern)) {
                  teams.forEach(team => reviewers.add(team));
                }
              }
            }
            
            if (reviewers.size > 0) {
              await github.rest.pulls.requestReviewers({
                owner: context.repo.owner,
                repo: context.repo.repo,
                pull_number: pr.number,
                reviewers: [...reviewers]
              });
            }
```

### 8.4 自动关闭不完整的 Issue

```yaml
# .github/workflows/auto-close.yml
name: 自动关闭不完整 Issue

on:
  issues:
    types: [opened]

jobs:
  close:
    runs-on: ubuntu-latest
    permissions:
      issues: write
    steps:
      - name: 检查 Issue 完整性
        uses: actions/github-script@v7
        with:
          script: |
            const issue = context.payload.issue;
            const body = issue.body || '';
            
            // 检查是否是空白 Issue（没有使用模板）
            if (body.length < 50) {
              await github.rest.issues.createComment({
                owner: context.repo.owner,
                repo: context.repo.repo,
                issue_number: issue.number,
                body: '## Issue 已关闭\n\n你的 Issue 似乎没有使用模板或信息不完整。请使用提供的模板重新提交，并确保填写所有必要信息。\n\n[创建新 Issue](https://github.com/' + context.repo.owner + '/' + context.repo.repo + '/issues/new/choose)'
              });
              
              await github.rest.issues.update({
                owner: context.repo.owner,
                repo: context.repo.repo,
                issue_number: issue.number,
                state: 'closed',
                state_reason: 'not_planned'
              });
            }
```

---

## 9. 模板管理策略

### 9.1 模板版本管理

将模板纳入版本控制，通过 PR 进行变更：

```markdown
# .github/ISSUE_TEMPLATE/CHANGELOG.md

## 模板变更记录

### 2024-01-15
- 新增安全漏洞报告模板
- 更新 Bug 报告模板，添加浏览器字段

### 2024-01-01
- 初始版本发布
- 包含 Bug 报告、功能请求、文档改进三个模板
```

### 9.2 模板审查流程

1. **提案阶段：** 创建 Issue 说明模板变更的原因和内容
2. **实施阶段：** 创建 PR 修改模板文件
3. **审查阶段：** 团队成员审查模板变更
4. **合并阶段：** 合并 PR，模板立即生效
5. **反馈阶段：** 收集使用反馈，持续优化

### 9.3 模板指标监控

通过 GitHub API 监控模板使用情况：

```javascript
// 统计各模板的使用比例
const issues = await github.rest.issues.listForRepo({
  owner: 'your-org',
  repo: 'your-repo',
  state: 'all',
  per_page: 100
});

const templates = {};
for (const issue of issues.data) {
  const template = issue.body?.includes('## 复现步骤') ? 'bug_report' :
                   issue.body?.includes('## 问题描述') ? 'feature_request' :
                   'other';
  templates[template] = (templates[template] || 0) + 1;
}

console.log('模板使用统计:', templates);
```

### 9.4 常见问题与解决方案

**问题 1：用户不使用模板**

解决方案：
- 在 `config.yml` 中设置 `blank_issues_enabled: false`
- 使用 Action 自动关闭不使用模板的 Issue
- 在贡献指南中明确要求使用模板

**问题 2：模板过于复杂**

解决方案：
- 简化模板，只保留必要字段
- 使用条件性字段（通过 markdown 元素分组）
- 提供多个模板供选择

**问题 3：模板更新后旧 Issue 不兼容**

解决方案：
- 保持模板的向后兼容性
- 新增字段设为非必填
- 通过评论引导用户补充新字段

### 9.5 高级模板技巧

**条件性内容：**

```yaml
- type: markdown
  attributes:
    value: |
      <!-- 以下内容仅适用于 Bug 报告 -->
      如果你报告的是 Bug，请继续填写以下信息。
      
      <!-- 如果是功能请求，可以跳到下方的功能请求部分 -->
```

**模板继承：**

通过 `includes` 关键字复用模板片段：

```yaml
# .github/ISSUE_TEMPLATE/_common.yml（公共片段，不会作为模板显示）
- type: checkboxes
  id: common-terms
  attributes:
    label: 确认事项
    options:
      - label: 我已搜索现有 Issue
        required: true
      - label: 我正在使用最新版本
        required: false
```

**动态链接：**

```yaml
- type: markdown
  attributes:
    value: |
      📋 在提交之前，请查看：
      - [贡献指南](https://github.com/${{ github.repository }}/blob/main/CONTRIBUTING.md)
      - [常见问题](https://github.com/${{ github.repository }}/wiki/FAQ)
```

---

## 总结

Issue 和 PR 模板是项目管理的基础设施之一。通过精心设计的模板，你可以：

1. **标准化沟通：** 确保所有 Issue 和 PR 包含必要的信息
2. **提升效率：** 减少维护者与贡献者之间的来回沟通
3. **自动化工作流：** 配合 GitHub Actions 实现自动标签、自动分配、质量检查
4. **改善体验：** 为贡献者提供清晰的指引，降低参与门槛
5. **数据驱动：** 通过模板收集结构化数据，分析项目健康度

记住，好的模板应该像一份好的表单——简洁、清晰、有引导性。定期收集反馈并持续优化模板，是保持项目健康发展的关键实践。

## 10. 高级模板技巧与实战案例

### 10.1 多语言模板支持

对于国际化项目，可以为 Issue 模板提供多语言版本。GitHub 会根据用户的语言偏好自动显示对应语言的模板：

```yaml
# .github/ISSUE_TEMPLATE/bug_report.yml
name: Bug Report / Bug 报告
description: Report a bug / 报告一个问题
title: "[Bug]: "
labels: ["bug", "needs-triage"]
body:
  - type: markdown
    attributes:
      value: |
        ## English
        Please fill out the form below to report a bug.

        ## 中文
        请填写以下表单来报告一个问题。

  - type: textarea
    id: description
    attributes:
      label: Description / 问题描述
      description: Describe the bug clearly / 清晰地描述问题
      placeholder: |
        English: What happened? What did you expect?
        中文：发生了什么？你期望什么？
    validations:
      required: true
```

### 10.2 根据标签动态显示不同模板内容

使用条件逻辑来显示不同的提示信息：

```yaml
- type: markdown
  attributes:
    value: |
      ## 填写指引

      **如果是 Bug 报告：** 请提供复现步骤、期望行为和实际行为。
      **如果是功能请求：** 请描述问题场景和期望的解决方案。
      **如果是文档问题：** 请指出文档中不准确或缺失的部分。

      ---
      > 请删除不适用的部分，只保留与你 Issue 类型相关的内容。
```

### 10.3 收集系统信息的自动化脚本

在 Issue 模板中嵌入收集系统信息的脚本，帮助维护者快速定位环境问题：

```yaml
- type: textarea
    id: system-info
    attributes:
      label: 系统信息
      description: |
        请运行以下命令并粘贴输出：
        ```bash
        echo "OS: $(uname -a)" && echo "Node: $(node -v)" && echo "npm: $(npm -v)" && echo "Git: $(git --version)"
        ```
      render: shell
    validations:
      required: true
```

### 10.4 安全漏洞报告模板

安全漏洞报告需要特殊的处理流程，不应公开提交：

```yaml
# .github/ISSUE_TEMPLATE/security_vulnerability.yml
name: 安全漏洞报告
description: 报告安全漏洞（将通过安全通告处理）
title: "[Security]: "
labels: ["security", "critical"]
assignees: []
body:
  - type: markdown
    attributes:
      value: |
        ## 安全漏洞报告

        **重要提示：** 如果你发现了严重的安全漏洞，请不要公开提交此 Issue。
        请通过 [GitHub Security Advisories](https://github.com/owner/repo/security/advisories/new) 私密报告。

        此模板仅用于低风险的安全问题。

  - type: dropdown
    id: severity
    attributes:
      label: 漏洞严重程度
      options:
        - 低 - 信息泄露、低风险配置问题
        - 中 - 权限绕过、数据泄露风险
        - 高 - 远程代码执行、SQL 注入等
    validations:
      required: true

  - type: textarea
    id: description
    attributes:
      label: 漏洞描述
      description: 详细描述安全漏洞
    validations:
      required: true

  - type: textarea
    id: impact
    attributes:
      label: 影响范围
      description: 描述该漏洞可能影响的用户和系统
    validations:
      required: true

  - type: textarea
    id: reproduction
    attributes:
      label: 复现步骤
      description: 提供复现漏洞的步骤（仅限低风险漏洞）
    validations:
      required: false

  - type: checkboxes
    id: terms
    attributes:
      label: 确认事项
      options:
        - label: 我理解此漏洞可能被恶意利用
          required: true
        - label: 我同意在修复完成前不公开披露此漏洞
          required: true
```

### 10.5 性能问题报告模板

```yaml
# .github/ISSUE_TEMPLATE/performance_issue.yml
name: 性能问题
description: 报告性能相关的问题
title: "[Perf]: "
labels: ["performance"]
body:
  - type: textarea
    id: description
    attributes:
      label: 性能问题描述
      description: 描述你观察到的性能问题
    validations:
      required: true

  - type: textarea
    id: benchmark
    attributes:
      label: 性能数据
      description: 提供性能测试数据或截图
      placeholder: |
        请求响应时间：XXms
        内存使用：XX MB
        CPU 使用率：XX%
    validations:
      required: true

  - type: input
    id: environment
    attributes:
      label: 测试环境
      description: 描述性能测试的环境配置
      placeholder: "4核 CPU, 16GB RAM, SSD 存储"
    validations:
      required: true

  - type: textarea
    id: profile
    attributes:
      label: 性能分析结果
      description: 如果有的话，提供性能分析工具的输出结果
      render: shell
    validations:
      required: false
```

### 10.6 PR 模板中的代码审查检查清单

为不同类型的 PR 创建专门的审查检查清单：

```markdown
## 审查重点

### 对于 Bug 修复 PR
- [ ] 是否有对应的测试用例覆盖该 Bug
- [ ] 修复是否针对根本原因而非症状
- [ ] 是否引入了新的回归问题
- [ ] 是否更新了相关文档

### 对于新功能 PR
- [ ] 功能设计是否经过讨论
- [ ] 是否有完整的测试覆盖
- [ ] API 设计是否向后兼容
- [ ] 是否添加了使用示例
- [ ] 是否更新了 CHANGELOG

### 对于性能优化 PR
- [ ] 是否有性能基准测试数据
- [ ] 优化是否影响代码可读性
- [ ] 是否在不同场景下测试
- [ ] 是否有回退方案

### 对于依赖更新 PR
- [ ] 是否检查了依赖的许可证变更
- [ ] 是否运行了完整的测试套件
- [ ] 是否检查了安全漏洞
- [ ] 是否评估了对包大小的影响
```

### 10.7 使用模板引导社区贡献

创建专门的社区贡献模板：

```yaml
# .github/ISSUE_TEMPLATE/community_contribution.yml
name: 社区贡献
description: 提出社区贡献计划
title: "[Community]: "
labels: ["community", "contribution"]
body:
  - type: textarea
    id: proposal
    attributes:
      label: 贡献计划
      description: 描述你计划为社区做什么贡献
    validations:
      required: true

  - type: dropdown
    id: type
    attributes:
      label: 贡献类型
      options:
        - 文档翻译
        - 教程编写
        - 示例代码
        - Bug 修复
        - 新功能开发
        - 性能优化
        - 其他
    validations:
      required: true

  - type: input
    id: timeline
    attributes:
      label: 预计完成时间
      description: 你预计什么时候完成这个贡献
      placeholder: "2 周内"
    validations:
      required: false

  - type: textarea
    id: help-needed
    attributes:
      label: 需要的帮助
      description: 描述你需要项目维护者提供什么帮助
    validations:
      required: false
```

### 10.8 模板的国际化最佳实践

为全球化的开源项目提供完善的多语言模板支持：

1. **模板名称双语：** 使用 `name: Bug Report / Bug 报告` 格式
2. **字段标签双语：** 使用 `label: Description / 问题描述` 格式
3. **提示文字双语：** 在 placeholder 中提供两种语言的示例
4. **说明文字双语：** 在 markdown 元素中提供两种语言的说明
5. **下拉选项双语：** 使用 `Bug 修复 / Bug Fix` 格式

### 10.9 模板与项目管理工具集成

将 Issue 模板与项目管理工具（如 GitHub Projects）集成：

```yaml
- type: dropdown
    id: sprint
    attributes:
      label: 所属迭代
      options:
        - Sprint 1
        - Sprint 2
        - Sprint 3
        - Backlog
    validations:
      required: false

  - type: dropdown
    id: story-points
    attributes:
      label: 故事点数
      options:
        - 1 - 简单
        - 2 - 中等
        - 3 - 复杂
        - 5 - 非常复杂
        - 8 - 需要拆分
    validations:
      required: false
```

### 10.10 模板的持续优化策略

定期收集和分析模板使用数据，持续优化模板设计：

1. **收集反馈：** 在模板中添加反馈链接，邀请用户评价模板质量
2. **分析数据：** 统计哪些字段经常被留空，考虑是否需要这些字段
3. **A/B 测试：** 尝试不同的模板设计，比较哪种设计收集的信息更完整
4. **版本迭代：** 每次模板变更都通过 PR 进行，便于团队审查和回溯
5. **文档更新：** 模板变更后及时更新贡献指南中的相关说明

---

## 11. 模板的常见错误与解决方案

### 11.1 常见错误一：模板过于复杂

**问题表现：** 模板包含大量字段，用户填写时感到困惑或厌烦，导致很多字段被留空或随意填写。

**解决方案：**

- 精简模板，只保留最核心的必填字段
- 将可选字段放在模板末尾，使用"附加信息"分组
- 提供清晰的 placeholder 示例，帮助用户理解每个字段的用途
- 考虑为不同场景创建多个简单模板，而非一个复杂模板

### 11.2 常见错误二：没有提供足够的指引

**问题表现：** 用户不知道该如何填写某些字段，提交的 Issue 信息质量参差不齐。

**解决方案：**

- 在 markdown 元素中添加详细的填写指引
- 提供示例内容，展示期望的填写格式
- 使用 dropdown 和 checkboxes 替代自由文本输入
- 在贡献指南中添加模板使用说明

### 11.3 常见错误三：模板没有及时更新

**问题表现：** 模板中的选项过时，不再反映项目的实际状态，导致用户无法选择正确的选项。

**解决方案：**

- 建立模板审查机制，定期检查模板内容
- 在项目里程碑中包含模板更新任务
- 使用模板变更日志记录每次修改
- 鼓励社区反馈模板问题

### 11.4 常见错误四：忽略模板的可访问性

**问题表现：** 模板只考虑了英语用户，没有为国际用户提供多语言支持。

**解决方案：**

- 使用双语标签和说明文字
- 提供多语言模板版本
- 在模板中使用简单、清晰的语言，避免俚语和成语
- 确保颜色对比度符合无障碍标准

### 11.5 常见错误五：没有充分利用自动化

**问题表现：** 手动处理模板提交的 Issue，效率低下，容易出错。

**解决方案：**

- 使用标签自动分配规则
- 配置 GitHub Actions 自动处理特定类型的 Issue
- 使用 Issue Forms 的验证功能减少无效提交
- 建立自动化的工作流处理标准问题

### 11.6 常见错误六：PR 模板缺乏针对性

**问题表现：** 所有类型的 PR 使用同一个模板，无法针对不同类型的变更提供专门的检查清单。

**解决方案：**

- 为不同类型的 PR 创建专门的模板（Bug 修复、新功能、文档更新等）
- 使用 GitHub 的多 PR 模板功能
- 在模板中根据变更类型动态调整检查清单
- 提供模板选择指引，帮助用户选择正确的模板

### 11.7 常见错误七：模板没有版本管理

**问题表现：** 模板变更没有记录，无法追溯历史修改，难以评估模板改进效果。

**解决方案：**

- 将模板变更纳入版本控制
- 使用 PR 进行模板修改，便于团队审查
- 建立模板变更日志
- 定期评估模板使用效果

---

## 12. 模板最佳实践总结

### 12.1 设计原则

1. **简洁性：** 只要求必要的信息，避免过度设计
2. **引导性：** 提供清晰的指引和示例，帮助用户正确填写
3. **一致性：** 所有模板使用统一的风格和格式
4. **灵活性：** 允许用户根据实际情况调整模板内容
5. **可维护性：** 建立模板更新和维护机制

### 12.2 技术实现

1. **使用 YAML 表单：** 提供更好的用户体验和数据验证
2. **合理使用标签：** 自动分配标签，便于分类和筛选
3. **配置模板选择器：** 引导用户选择正确的模板
4. **集成自动化：** 使用 GitHub Actions 自动处理模板提交
5. **版本控制：** 将模板纳入版本控制，记录变更历史

### 12.3 团队协作

1. **建立规范：** 制定模板使用规范，确保团队成员遵循
2. **定期审查：** 定期审查模板效果，收集团队反馈
3. **持续改进：** 根据使用情况不断优化模板设计
4. **文档化：** 在贡献指南中详细说明模板使用方法
5. **培训新人：** 为新成员提供模板使用培训

### 12.4 社区管理

1. **多语言支持：** 为国际社区提供多语言模板
2. **新人友好：** 设计易于理解的模板，降低参与门槛
3. **及时响应：** 对模板提交的 Issue 及时响应和处理
4. **认可贡献：** 对高质量的 Issue 和 PR 给予认可和奖励
5. **透明沟通：** 在模板变更时与社区充分沟通

---

## 13. 附录：模板速查参考

### 13.1 YAML 表单字段类型速查

| 字段类型 | 用途 | 关键属性 |
|----------|------|----------|
| `markdown` | 显示说明文字 | `value` |
| `input` | 单行文本输入 | `label`, `placeholder`, `required` |
| `textarea` | 多行文本输入 | `label`, `placeholder`, `render`, `required` |
| `dropdown` | 下拉选择 | `label`, `options`, `multiple`, `required` |
| `checkboxes` | 复选框组 | `label`, `options` |
| `number` | 数字输入 | `label`, `min`, `max`, `required` |

### 13.2 验证规则速查

```yaml
validations:
  required: true                    # 必填字段
  regex: "^[a-zA-Z0-9]+$"         # 正则表达式验证
  regex_error: "只能包含字母和数字"  # 验证失败提示
  min: 0                            # 最小值（数字字段）
  max: 100                          # 最大值（数字字段）
```

### 13.3 常用标签颜色配置

```yaml
# .github/labels.yml
- name: "bug"
  color: "d73a4a"
  description: "Something isn't working"

- name: "enhancement"
  color: "a2eeef"
  description: "New feature or request"

- name: "documentation"
  color: "0075ca"
  description: "Improvements or additions to documentation"

- name: "good first issue"
  color: "7057ff"
  description: "Good for newcomers"

- name: "help wanted"
  color: "008672"
  description: "Extra attention is needed"

- name: "priority: critical"
  color: "b60205"
  description: "Critical priority"

- name: "priority: high"
  color: "d93f0b"
  description: "High priority"

- name: "priority: medium"
  color: "fbca04"
  description: "Medium priority"

- name: "priority: low"
  color: "0e8a16"
  description: "Low priority"

- name: "status: needs-triage"
  color: "ededed"
  description: "Needs to be triaged"

- name: "status: in-progress"
  color: "1d76db"
  description: "Currently being worked on"

- name: "status: blocked"
  color: "e4e669"
  description: "Blocked by another issue"
```

### 13.4 模板文件清单

完整的模板目录结构参考：

```
.github/
├── ISSUE_TEMPLATE/
│   ├── 01_bug_report.yml           # Bug 报告
│   ├── 02_feature_request.yml      # 功能请求
│   ├── 03_documentation.yml        # 文档问题
│   ├── 04_performance.yml          # 性能问题
│   ├── 05_security.yml             # 安全问题
│   ├── 06_question.yml             # 提问
│   ├── 07_community.yml            # 社区贡献
│   └── config.yml                  # 模板选择器配置
├── PULL_REQUEST_TEMPLATE/
│   ├── bug_fix.md                  # Bug 修复 PR
│   ├── feature.md                  # 新功能 PR
│   ├── docs.md                     # 文档更新 PR
│   ├── refactor.md                 # 重构 PR
│   └── dependencies.md             # 依赖更新 PR
├── PULL_REQUEST_TEMPLATE.md        # 默认 PR 模板
├── CONTRIBUTING.md                 # 贡献指南
├── CODE_OF_CONDUCT.md              # 行为准则
└── SECURITY.md                     # 安全政策
```

### 13.5 GitHub Actions 自动处理模板的工作流模板

```yaml
# .github/workflows/issue-triage.yml
name: Issue 自动分类

on:
  issues:
    types: [opened, edited]

jobs:
  triage:
    runs-on: ubuntu-latest
    permissions:
      issues: write
    steps:
      - name: 自动分配标签
        uses: actions/github-script@v7
        with:
          script: |
            const title = context.payload.issue.title.toLowerCase();
            const body = (context.payload.issue.body || '').toLowerCase();
            const labels = [];

            // 根据标题关键词分配标签
            if (title.includes('[bug]')) labels.push('bug');
            if (title.includes('[feature]')) labels.push('enhancement');
            if (title.includes('[docs]')) labels.push('documentation');
            if (title.includes('[perf]')) labels.push('performance');
            if (title.includes('[security]')) labels.push('security');

            // 根据内容关键词分配优先级
            if (body.includes('紧急') || body.includes('critical')) {
              labels.push('priority: critical');
            } else if (body.includes('高') || body.includes('high')) {
              labels.push('priority: high');
            }

            // 添加需要分类的标签
            labels.push('status: needs-triage');

            // 应用标签
            if (labels.length > 0) {
              await github.rest.issues.addLabels({
                owner: context.repo.owner,
                repo: context.repo.repo,
                issue_number: context.issue.number,
                labels: labels
              });
            }
```

### 13.6 模板常见字段的中英文对照

| 英文 | 中文 | 使用场景 |
|------|------|----------|
| Bug Report | Bug 报告 | Bug 报告模板 |
| Feature Request | 功能请求 | 功能请求模板 |
| Description | 描述 | 通用描述字段 |
| Steps to Reproduce | 复现步骤 | Bug 报告字段 |
| Expected Behavior | 期望行为 | Bug 报告字段 |
| Actual Behavior | 实际行为 | Bug 报告字段 |
| Environment | 环境信息 | Bug 报告字段 |
| Version | 版本号 | 版本信息字段 |
| Screenshots | 截图 | 可选的视觉辅助 |
| Additional Information | 附加信息 | 补充说明字段 |
| Checklist | 检查清单 | 自查确认事项 |
| Related Issues | 相关 Issue | 关联问题字段 |
| Change Type | 变更类型 | PR 模板字段 |
| Testing | 测试说明 | PR 模板字段 |
| Breaking Changes | 破坏性变更 | PR 模板字段 |
| Documentation | 文档更新 | PR 模板字段 |
| Code Review | 代码审查 | PR 模板字段 |
| Merge Request | 合并请求 | PR 模板字段 |
| Summary | 摘要 | 通用摘要字段 |
| Motivation | 动机 | 功能请求字段 |
| Alternatives | 替代方案 | 功能请求字段 |
| Use Case | 使用场景 | 功能请求字段 |
| Impact | 影响范围 | 评估字段 |
| Priority | 优先级 | 分类字段 |
| Severity | 严重程度 | Bug 报告字段 |
| Affected Components | 受影响组件 | 问题范围字段 |
| Proposed Solution | 建议方案 | 功能请求字段 |
| Acceptance Criteria | 验收标准 | 需求确认字段 |
| Definition of Done | 完成定义 | 任务完成标准 |
| Release Notes | 发布说明 | 版本发布字段 |
```

---

**上一篇：[README 与文档](15-readme-docs.md) | 下一篇：[Issue 问题追踪](16-issues.md)**
