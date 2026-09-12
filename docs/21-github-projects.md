# GitHub Projects 项目管理完全指南

> GitHub Projects 是 GitHub 官方提供的项目管理工具，帮助开发团队规划、追踪和管理软件开发工作。本文将深入介绍 GitHub Projects 的各项功能，从基础配置到高级用法，助你高效管理项目。

---

## 1. GitHub Projects 概述

### 新版 Projects vs 经典 Projects

GitHub 在 2022 年推出了全新的 Projects（也称为 Projects V2），相比经典版本有了质的飞跃。

| 特性 | 经典 Projects | 新版 Projects (V2) |
|------|--------------|-------------------|
| 视图类型 | 仅看板视图 | 看板、表格、路线图 |
| 自定义字段 | 有限支持 | 完整支持多种字段类型 |
| 自动化 | 基础规则 | 强大的内置自动化 |
| 数据分析 | 无 | 内置图表和报告 |
| 跨仓库支持 | 不支持 | 完整支持 |
| 迭代管理 | 不支持 | 内置迭代周期 |
| API 支持 | REST API | GraphQL API |

**新版 Projects 的核心优势：**

- **灵活的视图系统**：同一项目数据可以以看板、表格、路线图等多种方式呈现
- **丰富的自定义字段**：支持单选、多选、日期、数字、迭代等字段类型
- **强大的自动化**：内置多种自动化规则，减少手动操作
- **深度集成**：与 Issues、Pull Requests、GitHub Actions 无缝集成
- **实时协作**：多人同时编辑，实时同步更新

### 适用场景

- **敏捷开发团队**：管理 Sprint、用户故事和任务
- **开源项目**：追踪 Issues、PR 和发布计划
- **个人项目**：管理待办事项和学习计划
- **产品管理**：规划产品路线图和版本发布

### GitHub Projects 的核心概念

在深入学习之前，需要理解以下几个核心概念：

**项目（Project）**：一个项目是一个容器，用于组织和管理一组相关的工作项。项目可以属于个人账户、组织或仓库。每个项目都有独立的字段定义、视图配置和自动化规则。

**工作项（Item）**：工作项是项目中的基本单元，可以是 Issue、Pull Request 或草稿条目。每个工作项可以拥有自定义字段的值，用于记录状态、优先级、负责人等信息。

**视图（View）**：视图是项目数据的展示方式。同一个项目可以创建多个视图，每个视图可以有不同的筛选条件、分组方式、排序规则和展示格式。

**字段（Field）**：字段定义了工作项的属性。GitHub Projects 提供预设字段（如 Status、Assignees、Labels），也支持创建自定义字段来满足特定需求。

**工作流（Workflow）**：工作流是自动化规则的集合，用于在特定事件发生时自动执行操作，例如当 Issue 被添加到项目时自动设置状态为"待办"。

---

## 2. 创建和配置项目看板

### 通过网页界面创建

**步骤一：进入项目创建页面**

1. 打开 GitHub 仓库页面
2. 点击顶部的 **Projects** 标签
3. 点击绿色的 **New project** 按钮

**步骤二：选择项目类型**

GitHub 提供多种项目模板：

- **Board**：看板视图，适合任务流程管理
- **Table**：表格视图，适合数据分析
- **Roadmap**：路线图视图，适合时间规划

**步骤三：配置项目基本信息**

```
项目名称：My Awesome Project
项目描述：管理项目开发进度和任务分配
README：可选，添加项目说明
```

**步骤四：设置项目可见性**

- **Private**：仅项目成员可见
- **Public**：所有人可见（适合开源项目）

### 通过命令行创建

使用 GitHub CLI 可以快速创建项目：

```bash
# 创建个人项目
gh project create --title "My Project" --owner @me

# 创建组织项目
gh project create --title "Team Project" --owner my-org

# 使用模板创建
gh project create --title "Sprint 1" --owner @me --template "feature-tracker"

# 查看项目列表
gh project list --owner @me
```

### 项目基本配置

创建项目后，建议进行以下配置：

**1. 设置项目字段**

进入项目后，点击右上角的 **+** 号添加自定义字段。建议创建以下基础字段：

| 字段名 | 类型 | 说明 |
|--------|------|------|
| Status | Single select | 任务状态（待办、进行中、已完成） |
| Priority | Single select | 优先级（紧急、高、中、低） |
| Assignees | People | 负责人 |
| Labels | Labels | 标签分类 |
| Start Date | Date | 开始日期 |
| Due Date | Date | 截止日期 |

**2. 配置工作流**

点击 **Workflows** 按钮，设置自动化规则。推荐启用以下基础规则：

- 当 Issue 被添加到项目时，自动设置 Status 为"待办"
- 当 Issue 被关闭时，自动设置 Status 为"已完成"
- 当 PR 被合并时，自动设置 Status 为"已完成"

**3. 添加项目成员**

在项目设置中邀请团队成员并设置权限。GitHub Projects 支持三种权限级别：

- **Admin**：完全控制项目设置和内容
- **Write**：可以编辑项目条目和视图
- **Read**：只能查看项目内容

**4. 设置项目描述和 README**

清晰的项目描述有助于团队成员理解项目目标。README 支持 Markdown 格式，可以包含：

- 项目目标和范围说明
- 团队成员和角色分工
- 工作流程和规范说明
- 相关文档和资源链接

**5. 创建初始视图**

根据团队需求创建常用视图：

- **看板视图**：用于日常任务管理和站会
- **表格视图**：用于数据分析和批量操作
- **路线图视图**：用于长期规划和里程碑追踪

---

## 3. 项目视图

GitHub Projects 提供三种核心视图，每种视图适用于不同的场景。

### 表格视图（Table View）

表格视图提供类似电子表格的数据展示方式，适合数据分析和批量操作。

**主要特性：**

- **列管理**：自定义显示哪些字段列
- **排序功能**：按任意字段排序
- **筛选功能**：按条件筛选项目条目
- **分组功能**：按字段分组显示

**使用场景：**

- 查看所有任务的优先级和状态
- 按负责人分配任务
- 分析项目进度数据

**配置示例：**

```
列显示：Title, Status, Priority, Assignee, Due Date
排序：Priority (降序)
分组：Status
筛选：Assignee = @me
```

**表格视图的高级功能：**

- **列宽调整**：拖拽列边界调整宽度，双击自动适应内容
- **行内编辑**：直接点击单元格编辑字段值，无需打开详情面板
- **批量选择**：按住 Shift 或 Ctrl 多选行，批量修改字段值
- **导出数据**：将表格数据导出为 CSV 格式，用于离线分析
- **列排序**：点击列标题进行升序或降序排序

**表格视图常用配置方案：**

| 配置项 | 推荐设置 | 适用场景 |
|--------|---------|---------|
| 列显示 | Title, Status, Priority, Assignee, Sprint, Story Points | Sprint 管理 |
| 分组 | Status | 任务状态总览 |
| 筛选 | -status:done | 查看未完成任务 |
| 排序 | Priority (降序) | 优先级优先 |

### 看板视图（Board View）

看板视图以卡片形式展示任务，通过拖拽卡片来更新任务状态。

**核心概念：**

- **列（Columns）**：代表不同的工作状态
- **卡片（Cards）**：代表具体的任务或工作项
- **WIP 限制**：限制每列的卡片数量（可选）

**默认列配置：**

```
To Do（待办）→ In Progress（进行中）→ In Review（审核中）→ Done（完成）
```

**自定义列：**

你可以根据团队工作流程自定义列：

```
Backlog → Ready → Development → Testing → Staging → Production
```

**看板最佳实践：**

- 限制在制品数量（WIP Limit）
- 定期清理完成的任务
- 使用标签区分任务类型

**看板视图的卡片信息：**

每张卡片可以显示以下信息，帮助团队成员快速了解任务概况：

- **标题**：任务的简要描述
- **状态标签**：当前所处的工作阶段
- **优先级标识**：用颜色区分优先级高低
- **负责人头像**：分配给哪位团队成员
- **截止日期**：任务的到期时间
- **标签**：分类标签，如 Bug、Feature 等
- **关联 PR**：关联的 Pull Request 数量

**列配置详解：**

看板的每一列对应 Status 字段的一个选项。你可以自定义列的顺序、颜色和显示规则：

| 列名 | 颜色建议 | 说明 |
|------|---------|------|
| Backlog | 灰色 | 待规划的任务池 |
| To Do | 蓝色 | 本迭代待开始的任务 |
| In Progress | 黄色 | 正在进行的任务 |
| In Review | 紫色 | 等待代码审核的任务 |
| Done | 绿色 | 已完成的任务 |

**拖拽操作：**

看板视图支持直观的拖拽操作：

- **横向拖拽**：将卡片从一列拖到另一列，自动更新 Status 字段
- **纵向拖拽**：在同一列内调整卡片顺序
- **批量拖拽**：选中多张卡片后一起拖拽（部分版本支持）

### 路线图视图（Roadmap View）

路线图视图以时间线形式展示项目计划，适合长期规划和里程碑管理。

**主要特性：**

- **时间轴**：按周、月、季度显示
- **日期字段**：使用日期字段确定时间范围
- **依赖关系**：可视化任务之间的依赖
- **里程碑**：标记重要的项目节点

**使用场景：**

- 产品版本发布规划
- 季度目标和关键结果追踪
- 跨团队协调和依赖管理

**配置步骤：**

1. 创建日期字段（如 Start Date、End Date）
2. 切换到路线图视图
3. 选择日期字段作为时间范围
4. 设置时间粒度（周/月/季度）

**路线图视图的时间粒度选择：**

| 时间粒度 | 适用场景 | 典型跨度 |
|---------|---------|---------|
| 周（Week） | Sprint 规划、短期任务 | 2-8 周 |
| 月（Month） | 季度规划、版本发布 | 1-6 个月 |
| 季度（Quarter） | 年度规划、长期目标 | 1-4 个季度 |

**路线图视图的使用技巧：**

- **设置里程碑**：在路线图上标记重要的项目节点，如版本发布日期、评审会议等
- **颜色编码**：使用不同颜色区分任务类型或优先级，便于快速识别
- **缩放操作**：通过缩放控制时间范围的显示粒度，查看整体或细节
- **筛选显示**：按负责人、优先级等条件筛选，只显示相关任务
- **导出分享**：截图或导出路线图，用于会议演示和汇报

---

## 4. 自定义字段

自定义字段是 GitHub Projects 的核心功能之一，让你能够灵活地定义和追踪项目数据。

### 字段类型详解

#### 单选字段（Single Select）

单选字段允许从预定义选项中选择一个值。

**常见用法：**

```yaml
字段名: Priority（优先级）
选项:
  - Urgent（紧急）: 红色
  - High（高）: 橙色
  - Medium（中）: 黄色
  - Low（低）: 绿色
```

```yaml
字段名: Category（分类）
选项:
  - Feature（功能）: 蓝色
  - Bug（缺陷）: 红色
  - Enhancement（增强）: 紫色
  - Documentation（文档）: 灰色
```

**配置步骤：**

1. 点击项目中的 **+** 号
2. 选择 **Single select**
3. 输入字段名称
4. 添加选项并设置颜色
5. 设置默认值（可选）

#### 多选字段（Multi Select）

多选字段允许选择多个值，适合标记具有多个属性的工作项。

**常见用法：**

```yaml
字段名: Tags（标签）
选项:
  - Frontend（前端）
  - Backend（后端）
  - Database（数据库）
  - API
  - UI/UX
```

```yaml
字段名: Platforms（平台）
选项:
  - iOS
  - Android
  - Web
  - Desktop
```

#### 日期字段（Date）

日期字段用于设置截止日期或时间范围。

**常见用法：**

- **Due Date**：任务截止日期
- **Start Date**：任务开始日期
- **End Date**：任务结束日期

**在路线图视图中的应用：**

路线图视图需要至少一个日期字段来确定时间范围。建议创建两个日期字段：

- Start Date：开始日期
- End Date：结束日期

#### 数字字段（Number）

数字字段用于存储数值数据。

**常见用法：**

```yaml
字段名: Story Points（故事点数）
范围: 1, 2, 3, 5, 8, 13, 21
用途: 估算工作量

字段名: Estimated Hours（预估工时）
范围: 0.5, 1, 2, 4, 8, 16
用途: 时间估算

字段名: Business Value（商业价值）
范围: 1-10
用途: 优先级排序
```

#### 迭代字段（Iteration）

迭代字段用于管理 Sprint 或迭代周期。

**配置选项：**

```yaml
字段名: Sprint
迭代周期: 2 周
开始日期: 2024-01-01
迭代数量: 自动创建
选项:
  - Sprint 1 (2024-01-01 ~ 2024-01-14)
  - Sprint 2 (2024-01-15 ~ 2024-01-28)
  - Sprint 3 (2024-01-29 ~ 2024-02-11)
  - ...
```

**迭代字段的特殊行为：**

- 支持 **Current iteration**（当前迭代）自动筛选
- 支持 **Past iterations**（过去迭代）回顾
- 支持 **Future iterations**（未来迭代）规划

#### 标签字段（Labels）

GitHub Labels 可以直接在项目中使用，用于分类和筛选。

**标签设计建议：**

```yaml
类型标签:
  - type:bug（缺陷）
  - type:feature（功能）
  - type:enhancement（增强）
  - type:docs（文档）

优先级标签:
  - priority:critical（严重）
  - priority:high（高）
  - priority:medium（中）
  - priority:low（低）

状态标签:
  - status:blocked（阻塞）
  - status:needs-review（需要审核）
  - status:ready-to-deploy（准备部署）
```

### 字段管理最佳实践

1. **保持字段简洁**：只创建必要的字段，避免信息过载
2. **统一命名规范**：团队内使用一致的字段命名
3. **合理使用颜色**：为选项设置有意义的颜色
4. **设置默认值**：为常用字段设置默认值，减少手动输入
5. **定期清理**：删除不再使用的字段和选项

### 其他字段类型

#### 文本字段（Text）

文本字段用于存储自由格式的短文本信息。与 Issue 或 PR 的正文不同，文本字段适合记录简短的补充信息，例如备注、链接或参考编号。

**常见用法：**

- **备注**：记录特殊情况或额外说明
- **外部链接**：关联外部系统或文档的链接
- **跟踪编号**：外部系统的工单编号或需求编号

#### 人员字段（People）

人员字段用于指定与工作项相关的人员。与 Assignees 字段不同，自定义人员字段可以记录多种角色的负责人。

**常见用法：**

- **Reviewer**：代码审核人
- **QA Owner**：测试负责人
- **Stakeholder**：利益相关者或需求方

#### 字段使用场景对照表

| 场景 | 推荐字段类型 | 示例 |
|------|------------|------|
| 任务状态追踪 | Single select | Status: 待办、进行中、已完成 |
| 优先级管理 | Single select | Priority: 紧急、高、中、低 |
| 工作量估算 | Number | Story Points: 1, 2, 3, 5, 8 |
| 时间规划 | Date | Due Date, Start Date |
| 迭代管理 | Iteration | Sprint: 两周一个周期 |
| 多维度分类 | Multi select | Tags: 前端、后端、数据库 |
| 角色分配 | People | Reviewer, QA Owner |

---

## 5. 自动化工作流

GitHub Projects 提供内置的自动化功能，帮助减少手动操作，提高工作效率。

### 内置自动化规则

#### 状态自动更新

```yaml
规则 1: Issue 添加到项目
触发条件: Issue 被添加到项目
执行操作: Status 设置为 "To Do"

规则 2: Issue 关闭
触发条件: Issue 被关闭
执行操作: Status 设置为 "Done"

规则 3: PR 创建
触发条件: Pull Request 被添加到项目
执行操作: Status 设置为 "In Progress"

规则 4: PR 合并
触发条件: Pull Request 被合并
执行操作: Status 设置为 "Done"
```

#### 日期自动设置

```yaml
规则: 设置开始日期
触发条件: Status 变更为 "In Progress"
执行操作: Start Date 设置为当前日期

规则: 设置完成日期
触发条件: Status 变更为 "Done"
执行操作: End Date 设置为当前日期
```

#### 清理规则

```yaml
规则: 归档完成项
触发条件: Status 为 "Done" 且超过 14 天
执行操作: 归档该项目条目
```

### 配置自动化规则

**步骤一：进入工作流设置**

1. 打开项目页面
2. 点击右上角的 **...** 菜单
3. 选择 **Workflows**

**步骤二：选择或创建规则**

GitHub 提供预设的自动化模板，你也可以创建自定义规则。

**步骤三：配置触发条件和操作**

```
触发条件: When issues are added to this project
操作: Set status to "To Do"
```

**步骤四：启用规则**

点击 **Save** 保存并启用规则。

### 自动化触发器和操作详解

**可用触发器（Triggers）：**

| 触发器 | 说明 | 典型用法 |
|--------|------|---------|
| Item added to project | 工作项被添加到项目 | 自动设置初始状态 |
| Item removed from project | 工作项从项目移除 | 清理关联数据 |
| Issue opened | Issue 被创建 | 添加到项目并分类 |
| Issue closed | Issue 被关闭 | 更新状态为已完成 |
| Pull request opened | PR 被创建 | 标记为进行中 |
| Pull request merged | PR 被合并 | 更新状态为已完成 |
| Label added | 标签被添加 | 按标签更新优先级 |
| Review submitted | 代码审核完成 | 更新审核状态 |

**可用操作（Actions）：**

| 操作 | 说明 | 典型用法 |
|------|------|---------|
| Set status | 设置 Status 字段值 | 更新任务状态 |
| Set priority | 设置 Priority 字段值 | 按标签设置优先级 |
| Set iteration | 设置 Sprint 字段值 | 分配到当前迭代 |
| Set date field | 设置日期字段值 | 记录开始或完成时间 |
| Archive item | 归档工作项 | 清理已完成的任务 |

### 自定义自动化（使用 GitHub Actions）

对于更复杂的自动化需求，可以使用 GitHub Actions：

```yaml
# .github/workflows/project-automation.yml
name: Project Automation

on:
  issues:
    types: [opened, closed, reopened]
  pull_request:
    types: [opened, closed, merged]

jobs:
  update-project:
    runs-on: ubuntu-latest
    steps:
      - name: Add issue to project
        if: github.event_name == 'issues' && github.event.action == 'opened'
        uses: actions/add-to-project@v0.5.0
        with:
          project-url: https://github.com/users/my-org/projects/1
          github-token: ${{ secrets.GITHUB_TOKEN }}

      - name: Update status to Done
        if: github.event_name == 'issues' && github.event.action == 'closed'
        uses: actions/update-project-item@v1
        with:
          project-url: https://github.com/users/my-org/projects/1
          item-id: ${{ steps.get-item.outputs.item-id }}
          status: "Done"
```

### 自动化工作流最佳实践

1. **从简单开始**：先使用内置规则，逐步扩展
2. **测试规则**：在正式项目前测试自动化行为
3. **文档化**：记录自动化规则的逻辑和目的
4. **定期审查**：检查自动化是否按预期工作
5. **避免过度自动化**：某些手动操作可能更灵活

---

## 6. 与 Issues 和 Pull Requests 的集成

GitHub Projects 与 Issues、Pull Requests 深度集成，实现代码开发与项目管理的无缝连接。

### 将 Issue 添加到项目

#### 手动添加

1. 打开 Issue 页面
2. 在右侧栏找到 **Projects** 部分
3. 点击 **Add to project**
4. 选择目标项目

#### 批量添加

在项目视图中批量添加：

1. 点击 **+ Add item**
2. 选择 **Add items from repository**
3. 选择仓库
4. 勾选要添加的 Issues

#### 使用标签自动添加

配置自动化规则，当 Issue 被打上特定标签时自动添加到项目：

```yaml
触发条件: Issue 被添加标签 "sprint-ready"
执行操作: 添加到项目并设置 Status 为 "To Do"
```

### 将 Pull Request 添加到项目

#### 关联方式

Pull Request 可以通过以下方式关联到项目：

1. **手动添加**：在 PR 页面的 Projects 部分添加
2. **自动添加**：当 PR 关联的 Issue 在项目中时自动添加
3. **标签触发**：使用标签自动添加规则

#### PR 与 Issue 的联动

当 Pull Request 关联了 Issue 时：

- PR 的状态变化会自动更新关联 Issue 的状态
- 合并 PR 可以自动关闭关联的 Issue
- 项目中的状态可以自动同步

**在 PR 中关联 Issue 的方式：**

在 PR 描述中使用特定关键词可以自动关联并关闭 Issue：

```markdown
Closes #123
Fixes #456
Resolves #789
```

也可以关联其他仓库的 Issue：

```markdown
Closes org/other-repo#100
```

当 PR 被合并时，关联的 Issue 会自动关闭，如果这些 Issue 已经在项目中，项目状态也会自动更新。

**跨引用追踪：**

GitHub 会自动追踪 Issue 和 PR 之间的引用关系：

- Issue 页面会显示关联的 PR
- PR 页面会显示关联的 Issue
- 项目中的工作项会显示链接的 PR 数量

### 草稿工作项（Draft Items）

GitHub Projects 支持创建草稿工作项，这些条目尚未关联到 Issue 或 PR。

**使用场景：**

- 快速记录想法，稍后创建 Issue
- 规划阶段的任务分解
- 非代码相关的任务管理

**创建草稿：**

1. 在项目中点击 **+ Add item**
2. 输入标题
3. 按 Enter 创建草稿

**转换为 Issue：**

1. 点击草稿条目
2. 点击 **Create issue**
3. 选择目标仓库
4. 填写 Issue 详情

### 字段同步

某些字段可以在项目和 Issue/PR 之间同步：

| 项目字段 | Issue/PR 字段 | 同步方向 |
|---------|--------------|---------|
| Status | Issue 状态 | 双向 |
| Labels | Issue 标签 | 双向 |
| Assignees | Issue 指派人 | 双向 |
| Milestone | Issue 里程碑 | 单向（从 Issue 到项目） |
| Linked PRs | 关联的 PR | 自动 |

---

## 7. 迭代（Sprint）管理

GitHub Projects 的迭代字段为敏捷开发团队提供了完整的 Sprint 管理能力。

### 配置迭代字段

**创建迭代字段：**

1. 在项目中添加新字段
2. 选择 **Iteration** 类型
3. 配置迭代参数：

```yaml
字段名称: Sprint
迭代周期: 2 周
开始日期: 2024-01-01（周一）
迭代数量: 12（自动创建 12 个迭代）
```

**迭代命名规则：**

- Sprint 1, Sprint 2, Sprint 3...
- 或自定义：Week 1-2, Week 3-4...

### Sprint 规划

**规划流程：**

1. **创建看板视图**：按 Sprint 分组
2. **评估工作量**：使用 Story Points 字段
3. **分配任务**：设置 Assignee 字段
4. **设置优先级**：使用 Priority 字段

**Sprint 看板配置：**

```
视图名称: Sprint Board
分组: Sprint
筛选: Sprint = Current iteration
排序: Priority (降序)
```

**Sprint 规划会议要点：**

Sprint 规划会议是敏捷开发中的关键环节，通常在每个 Sprint 开始时举行。使用 GitHub Projects 可以让规划会议更加高效：

1. **回顾产品待办列表**：使用表格视图查看 Backlog 中的任务，按优先级排序
2. **评估团队容量**：查看团队成员的当前任务分配情况
3. **选择本 Sprint 任务**：将选中的任务分配到当前 Sprint
4. **估算工作量**：为每个任务分配 Story Points
5. **确认 Sprint 目标**：明确本 Sprint 需要完成的核心目标

**Sprint 容量规划建议：**

| 团队规模 | 建议 Sprint 容量 | 说明 |
|---------|----------------|------|
| 3-5 人 | 20-40 Story Points | 小型团队，快速迭代 |
| 5-8 人 | 40-80 Story Points | 中型团队，平衡节奏 |
| 8-12 人 | 80-120 Story Points | 大型团队，需要更多协调 |

### Sprint 执行

**每日站会：**

使用看板视图展示当前 Sprint 的进度：

1. **To Do**：待开始的任务
2. **In Progress**：正在进行的任务
3. **In Review**：等待审核的任务
4. **Done**：已完成的任务

**站会流程建议：**

每日站会是团队同步进展的重要会议，建议控制在 15 分钟以内。使用 GitHub Projects 的看板视图可以让站会更加高效：

1. **打开看板视图**：筛选当前 Sprint 的任务
2. **逐个更新**：每位成员分享昨天完成的任务、今天的计划和遇到的阻碍
3. **识别阻塞**：关注 In Review 列中长时间未处理的任务
4. **更新状态**：会议中直接在看板上拖拽更新任务状态

**进度追踪：**

使用表格视图分析 Sprint 进度：

```
筛选: Sprint = Current iteration
分组: Status
显示字段: Title, Assignee, Story Points, Priority
```

**Sprint 进度监控指标：**

在 Sprint 执行过程中，需要关注以下指标来判断项目健康度：

| 指标 | 计算方式 | 健康范围 |
|------|---------|---------|
| 完成故事点 | 已完成的 Story Points 总和 | 接近 Sprint 计划量 |
| 剩余故事点 | 未完成的 Story Points 总和 | 随时间递减 |
| 任务分布 | 各状态的任务数量 | 不应集中在某一列 |
| 阻塞任务数 | 标记为 Blocked 的任务 | 越少越好 |

### Sprint 回顾

**创建回顾视图：**

```
视图名称: Sprint Review
筛选: Sprint = Past iteration (最近完成的迭代)
分组: Status
显示字段: Title, Assignee, Story Points, Actual Hours
```

**回顾指标：**

- **完成率**：已完成的故事点数 / 计划的故事点数
- **速度**：每个 Sprint 完成的故事点数
- **缺陷率**：Bug 数量 / 总任务数

**Sprint 回顾会议流程：**

Sprint 回顾会议是团队反思和改进的重要环节，通常在每个 Sprint 结束时举行：

1. **数据回顾**：使用项目图表展示 Sprint 完成情况
2. **做得好的方面**：讨论本 Sprint 中团队表现优秀的方面
3. **需要改进的方面**：识别可以改进的工作流程或协作方式
4. **行动计划**：制定具体的改进措施，分配到下个 Sprint

**速度趋势分析：**

通过追踪多个 Sprint 的速度数据，可以预测团队的交付能力：

```
Sprint 1: 35 Story Points
Sprint 2: 42 Story Points
Sprint 3: 38 Story Points
Sprint 4: 40 Story Points
平均速度: 38.75 Story Points
```

基于平均速度，团队可以更准确地规划后续 Sprint 的任务量。

### 多团队 Sprint 管理

对于多个团队的情况，可以：

1. **使用标签区分团队**：Team-A, Team-B
2. **创建团队视图**：每个团队一个筛选视图
3. **共享迭代字段**：统一 Sprint 周期
4. **独立看板**：每个团队独立的看板视图

---

## 8. 项目筛选和分组

### 筛选功能

筛选是项目管理中快速定位特定工作项的重要功能。

#### 筛选语法

```yaml
# 基本筛选
status:todo
status:"in progress"
priority:high
assignee:@me

# 组合筛选（AND）
status:todo priority:high
assignee:@me status:"in progress"

# 组合筛选（OR）
status:todo OR status:"in progress"
priority:high OR priority:urgent

# 排除筛选
-status:done
-assignee:@me

# 日期筛选
due:<2024-01-31
due:>2024-01-01
due:this-week
due:this-month
start:<2024-01-15

# 迭代筛选
sprint:"current iteration"
sprint:"past iteration"
sprint:"Sprint 1"

# 标签筛选
label:bug
label:"type:feature"
-label:"priority:low"

# 文本搜索
搜索词（搜索标题和描述）
```

#### 常用筛选组合

**我的任务：**
```
assignee:@me -status:done
```

**本周到期：**
```
due:this-week assignee:@me
```

**高优先级 Bug：**
```
label:bug priority:high OR priority:urgent
```

**当前 Sprint 任务：**
```
sprint:"current iteration" -status:done
```

**需要审核：**
```
status:"in review"
```

### 分组功能

分组功能将项目条目按特定字段进行分类显示。

#### 分组选项

| 字段 | 说明 | 适用视图 |
|------|------|---------|
| Status | 按状态分组 | 所有视图 |
| Priority | 按优先级分组 | 表格、看板 |
| Assignees | 按负责人分组 | 表格、看板 |
| Labels | 按标签分组 | 表格、看板 |
| Sprint | 按迭代分组 | 表格、看板 |
| Repository | 按仓库分组 | 表格、看板 |
| Milestone | 按里程碑分组 | 表格、看板 |
| 自定义字段 | 按自定义字段分组 | 表格、看板 |

#### 分组最佳实践

1. **看板视图**：通常按 Status 分组
2. **任务分配**：按 Assignees 分组查看每人工作量
3. **优先级管理**：按 Priority 分组识别紧急任务
4. **迭代规划**：按 Sprint 分组进行迭代规划
5. **跨仓库项目**：按 Repository 分组查看各仓库进度

### 排序功能

排序功能帮助你按特定顺序展示项目条目。

#### 排序选项

```yaml
# 单字段排序
Priority (降序)  # 高优先级在前
Due Date (升序)  # 最早到期在前
Story Points (降序)  # 最大工作量在前

# 多字段排序
1. Priority (降序)
2. Due Date (升序)
# 先按优先级，优先级相同按到期日期
```

### 视图组合应用

通过组合筛选、分组和排序，创建功能强大的视图：

**视图 1：我的本周任务**
```
筛选: assignee:@me due:this-week -status:done
分组: Priority
排序: Due Date (升序)
```

**视图 2：Sprint 进度**
```
筛选: sprint:"current iteration"
分组: Status
排序: Priority (降序)
```

**视图 3：团队工作分配**
```
筛选: -status:done
分组: Assignees
排序: Priority (降序)
```

**视图 4：缺陷追踪**
```
筛选: label:bug -status:done
分组: Priority
排序: Due Date (升序)
```

---

## 9. 项目模板

GitHub Projects 提供多种预设模板，帮助快速启动项目管理。

### 内置模板

#### 功能追踪模板（Feature Tracker）

```yaml
字段:
  - Status: Backlog, Ready, In Progress, In Review, Done
  - Priority: Urgent, High, Medium, Low
  - Size: XS, S, M, L, XL

视图:
  - Feature Board (看板视图)
  - Feature Table (表格视图)
```

#### Bug 追踪模板（Bug Tracker）

```yaml
字段:
  - Status: New, Confirmed, In Progress, Verified, Closed
  - Severity: Critical, High, Medium, Low
  - Environment: Production, Staging, Development

视图:
  - Bug Board (看板视图)
  - Bug Table (表格视图)
```

#### 路线图模板（Roadmap）

```yaml
字段:
  - Status: Planning, In Progress, Complete
  - Priority: High, Medium, Low
  - Start Date (日期)
  - Target Date (日期)

视图:
  - Roadmap (路线图视图)
  - Timeline (表格视图)
```

#### OKR 追踪模板（OKR Tracker）

```yaml
字段:
  - Status: Not Started, On Track, At Risk, Completed
  - Objective (单选): O1, O2, O3
  - Key Result (文本): 关键结果描述
  - Progress (数字): 进度百分比
  - Owner (人员): 负责人
  - Quarter (单选): Q1, Q2, Q3, Q4

视图:
  - OKR Dashboard (表格视图，按 Objective 分组)
  - Progress Overview (表格视图，按 Status 分组)
```

#### 发布管理模板（Release Manager）

```yaml
字段:
  - Status: Planning, Development, Testing, Ready, Released
  - Version (单选): v1.0, v1.1, v2.0
  - Release Date (日期)
  - Release Type (单选): Major, Minor, Patch
  - Breaking Changes (单选): Yes, No

视图:
  - Release Board (看板视图)
  - Release Roadmap (路线图视图)
  - Version History (表格视图)
```

### 模板选择指南

| 项目类型 | 推荐模板 | 适用场景 |
|---------|---------|---------|
| 功能开发 | Feature Tracker | 追踪新功能的开发进度 |
| 缺陷管理 | Bug Tracker | 管理和追踪软件缺陷 |
| 长期规划 | Roadmap | 规划产品路线图和版本发布 |
| 目标管理 | OKR Tracker | 追踪团队目标和关键结果 |
| 版本发布 | Release Manager | 管理版本发布流程 |
| 敏捷开发 | 自定义敏捷模板 | 管理 Sprint 和用户故事 |

### 自定义模板

你可以创建自己的项目模板：

**步骤一：创建基础项目**

1. 创建新项目并配置所有需要的字段
2. 创建多个视图
3. 配置自动化规则

**步骤二：保存为模板**

1. 进入项目设置
2. 点击 **Save as template**
3. 输入模板名称和描述

**步骤三：使用模板**

创建新项目时选择自定义模板。

### 团队模板设计

**敏捷团队模板：**

```yaml
字段:
  - Status: Backlog, Sprint Backlog, In Progress, Review, Testing, Done
  - Sprint (迭代)
  - Story Points (数字)
  - Priority: Critical, High, Medium, Low
  - Type: User Story, Bug, Task, Spike
  - Epic (单选)

视图:
  - Sprint Board (看板视图，按 Status 分组)
  - Sprint Planning (表格视图，按 Sprint 分组)
  - Roadmap (路线图视图，按 Epic 分组)
  - My Tasks (表格视图，筛选 assignee:@me)
```

**开源项目模板：**

```yaml
字段:
  - Status: Triage, Accepted, In Progress, Review, Done
  - Priority: Critical, High, Medium, Low
  - Type: Bug, Feature, Enhancement, Documentation
  - Good First Issue (单选: Yes, No)
  - Help Wanted (单选: Yes, No)

视图:
  - Main Board (看板视图)
  - Contributor Tasks (表格视图，筛选 Good First Issue = Yes)
  - Bug Tracker (表格视图，筛选 Type = Bug)
```

---

## 10. GitHub Projects 与 Jira 对比

### 功能对比

| 功能 | GitHub Projects | Jira |
|------|----------------|------|
| 价格 | 免费（公开项目）/ Pro（私有项目） | 免费（10人以下）/ 付费 |
| 学习曲线 | 低 | 中到高 |
| 集成 | GitHub 原生集成 | 丰富的第三方集成 |
| 自定义程度 | 中等 | 高度可定制 |
| 工作流 | 简单直观 | 复杂强大 |
| 报告 | 基础图表 | 丰富的报表 |
| API | GraphQL | REST + GraphQL |
| 自动化 | 内置 + Actions | 内置 + 插件 |
| 适合团队规模 | 小到中型 | 中到大型 |
| 敏捷支持 | 基础 | 完整 |

### 各自的优势和劣势

**GitHub Projects 的优势：**

- **无缝集成**：与 GitHub 的 Issues、PR、Actions 深度集成，无需切换工具
- **零成本入门**：公开项目完全免费，私有项目在 Pro 计划中也免费
- **简洁易用**：界面直观，学习成本低，新成员可以快速上手
- **实时协作**：多人同时编辑，变更实时同步
- **开发者友好**：通过 GraphQL API 和 GitHub Actions 实现高度自动化

**GitHub Projects 的劣势：**

- **功能相对简单**：缺乏复杂的自定义工作流和高级报表
- **报表能力有限**：内置图表类型较少，复杂分析需要借助 API
- **不支持子任务**：无法创建任务层级结构
- **自定义字段有限**：字段类型相对较少，无法满足复杂需求

**Jira 的优势：**

- **功能全面**：支持复杂的自定义工作流、字段和界面
- **强大的报表**：内置丰富的报表和仪表盘
- **子任务支持**：支持多层级的任务结构
- **插件生态**：丰富的第三方插件扩展功能

**Jira 的劣势：**

- **学习成本高**：功能复杂，配置繁琐
- **价格较高**：大规模团队需要付费
- **与代码集成弱**：与代码托管平台的集成需要额外配置
- **性能问题**：大型项目可能出现性能下降

### 选择建议

**选择 GitHub Projects 如果：**

- 团队已经在使用 GitHub
- 项目规模较小或中等
- 希望简单易用的工具
- 预算有限
- 开源项目

**选择 Jira 如果：**

- 需要复杂的自定义工作流
- 大型团队和复杂项目
- 需要详细的报告和分析
- 需要与多种工具集成
- 企业级项目管理需求

### 迁移建议

如果从 Jira 迁移到 GitHub Projects：

1. **评估需求**：列出 Jira 中使用的核心功能
2. **字段映射**：将 Jira 字段映射到 GitHub Projects 字段
3. **工作流简化**：简化复杂的工作流
4. **数据迁移**：使用 API 批量导入数据
5. **培训团队**：提供 GitHub Projects 培训

---

## 11. 团队项目管理最佳实践

### 项目结构设计

**推荐的项目结构：**

```
组织级项目（跨团队）
├── 产品路线图
├── 季度目标
└── 跨团队协调

团队级项目（单团队）
├── Sprint 看板
├── Bug 追踪
└── 技术债务

个人级项目（个人）
├── 个人任务
└── 学习计划
```

### 工作流设计

**标准敏捷工作流：**

```
Backlog → Sprint Backlog → In Progress → Code Review → Testing → Done
```

**简化工作流：**

```
To Do → In Progress → Done
```

**详细工作流（适合大型项目）：**

```
Triage → Ready → In Development → Code Review → QA Testing → 
UAT → Ready to Deploy → Deployed → Done
```

**工作流设计原则：**

设计工作流时应遵循以下原则，确保流程清晰且高效：

1. **状态有限**：每个工作流的状态不宜过多，建议 4-7 个状态
2. **单向流动**：任务通常从左向右流动，避免频繁回退
3. **明确入口**：每个状态应有明确的进入条件
4. **明确出口**：每个状态应有明确的完成标准
5. **避免瓶颈**：识别可能造成任务积压的状态，设置 WIP 限制

**不同团队的工作流选择：**

| 团队类型 | 推荐工作流 | 状态数量 |
|---------|-----------|---------|
| 创业团队 | 简化工作流 | 3-4 个 |
| 成熟产品团队 | 标准敏捷工作流 | 5-6 个 |
| 企业级团队 | 详细工作流 | 7-9 个 |
| 开源项目 | 简化工作流 + 分类标签 | 4-5 个 |

### 命名规范

**项目命名：**
```
[团队]-[项目类型]-[描述]
示例：
- frontend-sprint-board
- backend-bug-tracker
- product-roadmap-2024
```

**字段命名：**
```
使用清晰、一致的命名
示例：
- Status（而不是 state 或 phase）
- Priority（而不是 priority-level）
- Story Points（而不是 points 或 sp）
```

**视图命名：**
```
[用途]-[筛选条件]
示例：
- My Tasks（我的任务）
- Sprint Board（Sprint 看板）
- High Priority（高优先级）
```

### 权限管理

**权限级别：**

| 角色 | 权限 | 适用人员 |
|------|------|---------|
| Admin | 完全控制 | 项目经理 |
| Write | 编辑项目条目 | 开发人员 |
| Read | 只读访问 | 利益相关者 |

**权限设置建议：**

1. **项目经理**：Admin 权限
2. **开发人员**：Write 权限
3. **测试人员**：Write 权限
4. **产品经理**：Write 或 Admin 权限
5. **外部人员**：Read 权限

### 沟通协作

**每日站会：**

1. 使用看板视图展示当前 Sprint
2. 团队成员更新任务状态
3. 识别阻塞和风险

**Sprint 规划：**

1. 使用表格视图评估工作量
2. 分配任务到 Sprint
3. 设置优先级

**Sprint 回顾：**

1. 分析完成情况
2. 识别改进点
3. 调整下个 Sprint 计划

---

## 12. 项目数据分析和报告

### 内置图表

GitHub Projects 提供多种内置图表，帮助分析项目数据。

#### 燃尽图（Burndown Chart）

燃尽图显示 Sprint 期间剩余工作量的变化趋势。

**配置：**

```yaml
图表类型: Burndown
时间范围: Current iteration
工作量字段: Story Points
```

**解读：**

- **理想线**：假设均匀完成任务的趋势线
- **实际线**：实际剩余工作量
- **高于理想线**：进度落后
- **低于理想线**：进度超前

#### 累积流量图（Cumulative Flow）

累积流量图显示各状态任务数量随时间的变化。

**配置：**

```yaml
图表类型: Cumulative Flow
时间范围: Last 30 days
分组字段: Status
```

**解读：**

- **带宽变宽**：该状态的任务在增加，可能存在瓶颈
- **带宽稳定**：工作流平稳
- **带宽变窄**：该状态的任务在减少

#### 饼图（Pie Chart）

饼图显示任务的分布情况。

**配置：**

```yaml
图表类型: Pie
分组字段: Priority
筛选条件: -status:done
```

**常见分析：**

- 按优先级分布：了解当前未完成任务的优先级分布，识别是否需要调整资源
- 按负责人分布：查看每位成员的任务分配情况，平衡工作负载
- 按类型分布：分析 Bug、Feature、Enhancement 等类型的比例
- 按标签分布：识别任务的技术领域分布，如前端、后端、数据库等

#### 柱状图（Bar Chart）

柱状图显示数值字段的统计信息。

**配置：**

```yaml
图表类型: Bar
X 轴: Assignees
Y 轴: Story Points (Sum)
筛选条件: sprint:"current iteration"
```

**柱状图的常见用法：**

- **工作量分布**：按负责人统计 Story Points 总和，识别工作负载不均
- **完成情况对比**：对比计划 vs 实际完成的 Story Points
- **趋势分析**：按时间维度展示任务完成趋势

### 图表组合分析

通过组合多个图表，可以构建完整的项目仪表盘：

**仪表盘配置建议：**

| 图表 | 类型 | 用途 |
|------|------|------|
| 任务状态分布 | 饼图 | 查看各状态任务占比 |
| 优先级分布 | 饼图 | 识别高优先级任务数量 |
| 团队工作量 | 柱状图 | 对比各成员工作负载 |
| Sprint 进度 | 燃尽图 | 追踪 Sprint 完成趋势 |
| 状态变化趋势 | 累积流量图 | 识别工作流瓶颈 |

### 自定义报告

使用 GitHub API 创建自定义报告：

```graphql
# GraphQL 查询示例
query {
  user(login: "my-org") {
    projectV2(number: 1) {
      items(first: 100) {
        nodes {
          content {
            ... on Issue {
              title
              state
              labels(first: 10) {
                nodes {
                  name
                }
              }
            }
          }
          fieldValues(first: 10) {
            nodes {
              ... on ProjectV2ItemFieldSingleSelectValue {
                name
                field {
                  ... on ProjectV2SingleSelectField {
                    name
                  }
                }
              }
              ... on ProjectV2ItemFieldNumberValue {
                number
                field {
                  ... on ProjectV2Field {
                    name
                  }
                }
              }
            }
          }
        }
      }
    }
  }
}
```

### 关键指标

**敏捷指标：**

| 指标 | 计算方法 | 目标 |
|------|---------|------|
| 速度（Velocity） | 过去 3-5 个 Sprint 的平均完成故事点 | 稳定 |
| 完成率 | 完成故事点 / 计划故事点 | > 80% |
| 周期时间 | 从开始到完成的平均时间 | 尽量短 |
| 累积流量 | 各状态任务数量 | 保持稳定 |

**质量指标：**

| 指标 | 计算方法 | 目标 |
|------|---------|------|
| 缺陷密度 | Bug 数 / 功能数 | 尽量低 |
| 修复时间 | 从发现到修复的平均时间 | 尽量短 |
| 回归率 | 重新打开的 Bug / 总 Bug | < 5% |

---

## 13. 跨仓库项目管理

### 创建跨仓库项目

GitHub Projects 可以管理来自多个仓库的工作项。这在微服务架构、多仓库项目或组织级项目管理中非常有用。

**跨仓库项目的典型场景：**

- **微服务架构**：前端、后端、数据库等分别在不同仓库
- **多平台应用**：iOS、Android、Web 等平台分别开发
- **基础设施项目**：API、SDK、文档、示例等分别维护
- **组织级管理**：跨团队、跨项目的统一视图

**步骤一：创建项目**

在组织级别创建项目（而不是仓库级别）：

```bash
# 使用 CLI 创建组织项目
gh project create --title "Cross-Repo Project" --owner my-org
```

**步骤二：添加多个仓库的 Issue**

1. 打开项目
2. 点击 **+ Add item**
3. 选择 **Add items from repository**
4. 选择不同的仓库
5. 选择要添加的 Issues

**步骤三：使用 Repository 字段**

项目会自动添加 Repository 字段，显示每个条目所属的仓库。

### 跨仓库视图

**按仓库分组：**

```
视图名称: By Repository
分组: Repository
显示字段: Title, Status, Priority, Assignee
```

**按团队分组（使用标签）：**

```
视图名称: By Team
分组: Team（自定义单选字段）
筛选: -status:done
```

### 跨仓库自动化

使用 GitHub Actions 实现跨仓库自动化：

```yaml
# .github/workflows/sync-project.yml
name: Sync Cross-Repo Project

on:
  schedule:
    - cron: '0 9 * * 1'  # 每周一早上 9 点
  workflow_dispatch:

jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - name: Sync issues from multiple repos
        uses: actions/github-script@v7
        with:
          script: |
            const repos = ['repo-frontend', 'repo-backend', 'repo-mobile'];
            const projectId = 'PVT_xxxxx';
            
            for (const repo of repos) {
              const issues = await github.rest.issues.listForRepo({
                owner: context.repo.owner,
                repo: repo,
                state: 'open',
                labels: 'sprint-ready'
              });
              
              for (const issue of issues.data) {
                // 添加到项目
                await github.graphql(`
                  mutation($projectId: ID!, $contentId: ID!) {
                    addProjectV2ItemById(input: {projectId: $projectId, contentId: $contentId}) {
                      item {
                        id
                      }
                    }
                  }
                `, {
                  projectId: projectId,
                  contentId: issue.node_id
                });
              }
            }
```

### 跨仓库项目最佳实践

1. **统一标签**：跨仓库使用统一的标签体系，确保分类一致性
2. **标准化 Issue 模板**：各仓库使用相似的 Issue 模板，便于汇总分析
3. **清晰的命名**：项目和视图命名清晰易懂，体现跨仓库特性
4. **定期同步**：确保所有仓库的 Issues 都已同步到项目
5. **权限管理**：合理设置跨仓库的访问权限，确保团队成员可以访问所需仓库
6. **使用 Repository 字段**：利用自动添加的 Repository 字段进行筛选和分组
7. **创建仓库专属视图**：为每个仓库创建独立视图，方便各团队查看
8. **统一工作流**：尽量使用统一的状态字段和工作流，减少管理复杂度

**跨仓库项目的挑战和解决方案：**

| 挑战 | 解决方案 |
|------|---------|
| 标签不统一 | 创建组织级标签模板，统一命名规范 |
| 权限管理复杂 | 使用团队权限管理，批量设置访问权限 |
| 通知过多 | 按仓库或标签筛选通知，减少干扰 |
| 数据量大 | 使用筛选和分组功能，只显示相关数据 |

---

## 14. GitHub Projects + Actions 联动

### 自动化场景

GitHub Projects 与 GitHub Actions 的联动可以实现复杂的自动化工作流。以下是一些常见的自动化场景：

#### 自动添加 Issue 到项目

当新 Issue 被创建时，自动添加到指定项目。这对于管理大量 Issue 的项目特别有用，可以确保所有 Issue 都被纳入项目管理范围。

```yaml
# .github/workflows/auto-add-to-project.yml
name: Auto Add to Project

on:
  issues:
    types: [opened]

jobs:
  add-to-project:
    runs-on: ubuntu-latest
    steps:
      - name: Add issue to project
        uses: actions/add-to-project@v0.5.0
        with:
          project-url: https://github.com/users/my-org/projects/1
          github-token: ${{ secrets.GITHUB_TOKEN }}
          labeled: sprint-ready
```

#### 自动更新项目状态

```yaml
# .github/workflows/update-project-status.yml
name: Update Project Status

on:
  issues:
    types: [closed]
  pull_request:
    types: [closed]

jobs:
  update-status:
    runs-on: ubuntu-latest
    steps:
      - name: Get project item ID
        id: get-item
        uses: actions/github-script@v7
        with:
          script: |
            // 获取项目条目 ID
            const issue = context.payload.issue || context.payload.pull_request;
            const projectId = 'PVT_xxxxx';
            
            const result = await github.graphql(`
              query($projectId: ID!) {
                node(id: $projectId) {
                  ... on ProjectV2 {
                    items(first: 100) {
                      nodes {
                        id
                        content {
                          ... on Issue {
                            id
                          }
                          ... on PullRequest {
                            id
                          }
                        }
                      }
                    }
                  }
                }
              }
            `, { projectId });
            
            const itemId = result.node.items.nodes.find(
              item => item.content.id === issue.node_id
            )?.id;
            
            return itemId;

      - name: Update status to Done
        if: steps.get-item.outputs.result
        uses: actions/github-script@v7
        with:
          script: |
            const projectId = 'PVT_xxxxx';
            const itemId = '${{ steps.get-item.outputs.result }}';
            const statusFieldId = 'PVTF_xxxxx';
            
            await github.graphql(`
              mutation($projectId: ID!, $itemId: ID!, $fieldId: ID!, $value: ProjectV2FieldValue!) {
                updateProjectV2ItemFieldValue(input: {
                  projectId: $projectId
                  itemId: $itemId
                  fieldId: $fieldId
                  value: $value
                }) {
                  projectV2Item {
                    id
                  }
                }
              }
            `, {
              projectId,
              itemId,
              fieldId: statusFieldId,
              value: { singleSelectOptionId: 'done_option_id' }
            });
```

#### 自动分配迭代

```yaml
# .github/workflows/auto-assign-sprint.yml
name: Auto Assign Sprint

on:
  issues:
    types: [opened]

jobs:
  assign-sprint:
    runs-on: ubuntu-latest
    steps:
      - name: Assign to current sprint
        uses: actions/github-script@v7
        with:
          script: |
            // 获取当前迭代
            const projectId = 'PVT_xxxxx';
            const sprintFieldId = 'PVTF_xxxxx';
            
            // 获取当前迭代选项 ID
            const result = await github.graphql(`
              query($projectId: ID!) {
                node(id: $projectId) {
                  ... on ProjectV2 {
                    fields(first: 20) {
                      nodes {
                        ... on ProjectV2IterationField {
                          id
                          configuration {
                            iterations {
                              id
                              startDate
                              duration
                            }
                          }
                        }
                      }
                    }
                  }
                }
              }
            `, { projectId });
            
            // 计算当前迭代
            const iterations = result.node.fields.nodes.find(
              f => f.id === sprintFieldId
            ).configuration.iterations;
            
            const today = new Date();
            const currentIteration = iterations.find(iter => {
              const start = new Date(iter.startDate);
              const end = new Date(start);
              end.setDate(end.getDate() + iter.duration);
              return today >= start && today < end;
            });
            
            return currentIteration?.id;
```

### 高级联动场景

#### PR 合并时自动关闭 Issue 并更新项目

```yaml
# .github/workflows/pr-merged.yml
name: PR Merged

on:
  pull_request:
    types: [closed]

jobs:
  update-project:
    if: github.event.pull_request.merged == true
    runs-on: ubuntu-latest
    steps:
      - name: Get linked issues
        id: get-issues
        uses: actions/github-script@v7
        with:
          script: |
            const pr = context.payload.pull_request;
            const body = pr.body || '';
            
            // 从 PR 描述中提取关联的 Issue 编号
            const issueNumbers = [];
            const patterns = [
              /(?:close[sd]?|fix(?:e[sd])?|resolve[sd]?)\s+#(\d+)/gi,
              /(?:close[sd]?|fix(?:e[sd])?|resolve[sd]?)\s+(?:https:\/\/github\.com\/[^\/]+\/[^\/]+\/issues\/(\d+))/gi
            ];
            
            for (const pattern of patterns) {
              let match;
              while ((match = pattern.exec(body)) !== null) {
                issueNumbers.push(match[1]);
              }
            }
            
            return issueNumbers;

      - name: Update project items
        if: steps.get-issues.outputs.result != '[]'
        uses: actions/github-script@v7
        with:
          script: |
            const issueNumbers = JSON.parse('${{ steps.get-issues.outputs.result }}');
            const projectId = 'PVT_xxxxx';
            
            for (const issueNumber of issueNumbers) {
              // 获取 Issue 的项目条目
              // 更新状态为 Done
              // 设置完成日期
            }
```

#### 定时报告生成

```yaml
# .github/workflows/weekly-report.yml
name: Weekly Project Report

on:
  schedule:
    - cron: '0 17 * * 5'  # 每周五下午 5 点
  workflow_dispatch:

jobs:
  generate-report:
    runs-on: ubuntu-latest
    steps:
      - name: Generate report
        uses: actions/github-script@v7
        with:
          script: |
            const projectId = 'PVT_xxxxx';
            
            // 查询项目数据
            const result = await github.graphql(`
              query($projectId: ID!) {
                node(id: $projectId) {
                  ... on ProjectV2 {
                    items(first: 200) {
                      nodes {
                        content {
                          ... on Issue {
                            title
                            state
                          }
                        }
                        fieldValues(first: 20) {
                          nodes {
                            ... on ProjectV2ItemFieldSingleSelectValue {
                              name
                              field { ... on ProjectV2SingleSelectField { name } }
                            }
                            ... on ProjectV2ItemFieldIterationValue {
                              title
                              startDate
                              duration
                              field { ... on ProjectV2IterationField { name } }
                            }
                          }
                        }
                      }
                    }
                  }
                }
              }
            `, { projectId });
            
            // 生成报告
            const items = result.node.items.nodes;
            const completed = items.filter(i => 
              i.fieldValues.nodes.some(f => 
                f.field?.name === 'Status' && f.name === 'Done'
              )
            );
            
            const report = `
            # 周报 - ${new Date().toISOString().split('T')[0]}
            
            ## 本周完成
            - 完成任务数: ${completed.length}
            - 完成率: ${((completed.length / items.length) * 100).toFixed(1)}%
            
            ## 待办任务
            - 待办任务数: ${items.length - completed.length}
            `;
            
            // 创建 Issue 作为报告
            await github.rest.issues.create({
              owner: context.repo.owner,
              repo: context.repo.repo,
              title: `周报 - ${new Date().toISOString().split('T')[0]}`,
              body: report,
              labels: ['weekly-report']
            });
```

---

## 15. 实战：管理一个开源项目

### 场景设定

假设你正在管理一个名为 `awesome-app` 的开源项目，需要使用 GitHub Projects 进行项目管理。该项目有以下特点：

- **项目类型**：全栈 Web 应用
- **团队规模**：5 名核心维护者 + 社区贡献者
- **仓库结构**：前端（React）、后端（Node.js）、文档（Docs）
- **发布周期**：每两周一个版本
- **贡献者数量**：约 50 名活跃贡献者

### 第一步：创建项目

**1. 创建组织级项目**

```bash
# 使用 CLI 创建
gh project create --title "Awesome App Development" --owner my-org
```

**2. 配置字段**

添加以下自定义字段：

| 字段名 | 类型 | 选项 |
|--------|------|------|
| Priority | Single select | Critical, High, Medium, Low |
| Type | Single select | Bug, Feature, Enhancement, Documentation |
| Sprint | Iteration | 2 周周期 |
| Story Points | Number | 1, 2, 3, 5, 8, 13 |
| Good First Issue | Single select | Yes, No |
| Help Wanted | Single select | Yes, No |
| Release | Single select | v1.0, v1.1, v2.0 |

**3. 创建视图**

```
视图 1: Main Board (看板视图)
- 分组: Status
- 筛选: 无
- 排序: Priority (降序)

视图 2: Sprint Planning (表格视图)
- 分组: Sprint
- 筛选: -status:done
- 排序: Priority (降序)

视图 3: Bug Tracker (表格视图)
- 分组: Priority
- 筛选: Type = Bug -status:done
- 排序: Priority (降序)

视图 4: Good First Issues (表格视图)
- 分组: Type
- 筛选: Good First Issue = Yes -status:done

视图 5: Roadmap (路线图视图)
- 时间范围: Release
```

### 第二步：配置自动化

**1. 内置自动化规则**

```
规则 1: Issue 添加到项目 → Status = Triage
规则 2: Issue 打上 "accepted" 标签 → Status = Accepted
规则 3: PR 创建并关联 Issue → Status = In Progress
规则 4: PR 合并 → Status = Done
规则 5: Issue 关闭 → Status = Done
```

**2. GitHub Actions 自动化**

```yaml
# .github/workflows/project-setup.yml
name: Project Setup

on:
  issues:
    types: [opened, labeled]

jobs:
  auto-triage:
    if: github.event.action == 'opened'
    runs-on: ubuntu-latest
    steps:
      - name: Add to project
        uses: actions/add-to-project@v0.5.0
        with:
          project-url: https://github.com/orgs/my-org/projects/1
          github-token: ${{ secrets.GITHUB_TOKEN }}

  auto-accept:
    if: github.event.label.name == 'accepted'
    runs-on: ubuntu-latest
    steps:
      - name: Update status to Accepted
        uses: actions/github-script@v7
        with:
          script: |
            // 更新项目条目状态
```

### 第三步：Issue 和 PR 模板

**Issue 模板：**

```yaml
# .github/ISSUE_TEMPLATE/bug_report.yml
name: Bug Report
description: Report a bug
labels: ["type:bug", "triage"]
body:
  - type: textarea
    id: description
    attributes:
      label: Description
      description: Describe the bug
    validations:
      required: true
  
  - type: textarea
    id: steps
    attributes:
      label: Steps to Reproduce
      description: Steps to reproduce the behavior
    validations:
      required: true
  
  - type: dropdown
    id: severity
    attributes:
      label: Severity
      options:
        - Critical
        - High
        - Medium
        - Low
    validations:
      required: true
```

```yaml
# .github/ISSUE_TEMPLATE/feature_request.yml
name: Feature Request
description: Request a new feature
labels: ["type:feature"]
body:
  - type: textarea
    id: description
    attributes:
      label: Description
      description: Describe the feature
    validations:
      required: true
  
  - type: textarea
    id: motivation
    attributes:
      label: Motivation
      description: Why is this feature important?
    validations:
      required: true
```

### 第四步：Sprint 管理流程

**Sprint 规划（周一）：**

1. 使用 Sprint Planning 视图
2. 从 Backlog 中选择任务
3. 分配到当前 Sprint
4. 设置 Story Points
5. 分配负责人

**每日站会：**

1. 使用 Main Board 视图
2. 团队成员更新任务状态
3. 识别阻塞和风险

**Sprint 回顾（周五）：**

1. 分析完成情况
2. 计算速度和完成率
3. 识别改进点
4. 规划下个 Sprint

### 第五步：社区贡献管理

**为新贡献者优化：**

1. 使用 Good First Issues 视图
2. 为简单任务打上 "good first issue" 标签
3. 提供详细的 Issue 描述
4. 设置 "help wanted" 标签请求帮助

**贡献流程：**

```
1. 贡献者 Fork 仓库
2. 选择 Good First Issues
3. 创建 Pull Request
4. 代码审核
5. 合并并更新项目状态
```

**社区贡献管理最佳实践：**

管理开源项目的社区贡献需要特别的关注和策略。以下是一些经过验证的最佳实践：

1. **创建欢迎视图**：专门为新贡献者创建一个视图，只显示标记为 "good first issue" 的任务，并提供详细的入门指南
2. **及时响应**：对新贡献者的 Issue 和 PR 在 48 小时内给予初步回复
3. **详细指导**：在 Issue 描述中提供清晰的任务说明、验收标准和相关代码位置
4. **标签体系**：使用标签帮助贡献者找到适合自己的任务
   - `good first issue`：适合新贡献者的简单任务
   - `help wanted`：需要社区帮助的任务
   - `documentation`：文档相关任务
   - `bug`：缺陷修复任务
5. **贡献者指南**：在仓库中维护 CONTRIBUTING.md 文件，说明贡献流程和代码规范
6. **认可贡献者**：在发布说明中感谢贡献者，建立积极的社区氛围

**贡献者标签体系设计：**

| 标签 | 颜色 | 说明 |
|------|------|------|
| good first issue | 紫色 | 适合新贡献者的简单任务 |
| help wanted | 绿色 | 需要社区帮助的任务 |
| documentation | 蓝色 | 文档相关任务 |
| bug | 红色 | 缺陷修复任务 |
| enhancement | 黄色 | 功能增强任务 |
| question | 灰色 | 问题咨询 |
| wontfix | 黑色 | 不会修复的问题 |

### 第六步：版本发布管理

**使用 Release 字段：**

1. 为每个 Issue 分配目标版本
2. 使用 Roadmap 视图查看版本计划
3. 使用筛选功能查看特定版本的任务

**发布检查清单：**

```
筛选: Release = v1.0 -status:done
```

检查所有 v1.0 的任务是否完成。

### 第七步：项目报告

**周报生成：**

```bash
# 使用 CLI 获取项目数据
gh project item-list 1 --owner my-org --format json | \
  jq '[.items[] | select(.status == "Done")] | length'
```

**月度分析：**

- 完成任务数
- Bug 修复率
- 新贡献者数量
- 社区活跃度

---

## 总结

GitHub Projects 是一个功能强大且易用的项目管理工具，特别适合 GitHub 用户。通过本文的学习，你应该能够：

1. **理解 GitHub Projects 的核心概念**：视图、字段、自动化
2. **创建和配置项目**：选择合适的模板和配置
3. **使用不同视图**：表格、看板、路线图
4. **自定义字段**：根据需求创建各种类型的字段
5. **配置自动化**：减少手动操作，提高效率
6. **集成 Issues 和 PR**：实现代码开发与项目管理的无缝连接
7. **管理迭代**：支持敏捷开发流程
8. **分析项目数据**：使用图表和报告追踪进度
9. **跨仓库管理**：管理多个仓库的项目
10. **与 Actions 联动**：实现高级自动化

### 进一步学习

- [GitHub Projects 官方文档](https://docs.github.com/en/issues/planning-and-tracking-with-projects)
- [GitHub Projects API 文档](https://docs.github.com/en/graphql/reference/objects#projectv2)
- [GitHub Actions 文档](https://docs.github.com/en/actions)
- [敏捷开发实践指南](https://www.atlassian.com/agile)

---

[← 上一章：GitHub Actions](19-github-actions.md) | [下一章：Fork 与开源贡献 →](22-fork-contribute.md)
