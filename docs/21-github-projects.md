# GitHub Projects 项目管理

## 什么是 GitHub Projects？

GitHub Projects 是内置的项目管理工具，提供看板、表格和路线图视图。

## 创建项目

### 网页创建
1. 进入仓库 → **Projects** 标签
2. 点 **New project**
3. 选择模板（看板、表格等）
4. 设置项目名称

### 命令行创建

```bash
gh project create --title "My Project" --owner @me
```

## 看板视图 (Board)

### 列 (Columns)
- **To Do**：待办事项
- **In Progress**：进行中
- **Done**：已完成

### 卡片 (Cards)
- Issue
- Pull Request
- 便签

### 自动化
```
Settings → Workflows → 添加自动化规则
- 当 Issue 被添加时，移到 To Do
- 当 PR 被合并时，移到 Done
```

## 表格视图 (Table)

提供类似 Excel 的视图：
- 自定义列
- 筛选和分组
- 排序

## 路线图 (Roadmap)

时间线视图，用于规划：
- 里程碑
- 发布计划
- 长期目标

## 字段管理

### 自定义字段
| 字段类型 | 说明 |
|---------|------|
| Text | 文本 |
| Number | 数字 |
| Single select | 单选 |
| Iteration | 迭代周期 |
| Date | 日期 |

### 预设字段
- **Status**：状态
- **Priority**：优先级
- **Size**：工作量

## 自动化规则

### 可用触发器
- Issue 被添加/移除
- PR 被添加/移除
- Issue/PR 状态变更
- Review 状态变更

### 可用操作
- 移动到列
- 添加/移除字段值
- 归档项目

## 项目视图

| 视图 | 用途 |
|------|------|
| Board | 看板式任务管理 |
| Table | 数据表格分析 |
| Roadmap | 时间线规划 |

## 最佳实践

1. **定期更新**：保持项目状态最新
2. **使用标签**：用字段和标签分类
3. **设置自动化**：减少手动操作
4. **关联 Issue/PR**：追踪工作进展

## 下一步

[Fork 与开源贡献 →](22-fork-contribute.md)
