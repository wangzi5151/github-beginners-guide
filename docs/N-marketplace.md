# GitHub 市场 (Marketplace)

## 什么是 GitHub Marketplace？

GitHub Marketplace 是一个平台，让你可以发现和使用各种工具来增强 GitHub 的功能。

## 分类

| 类别 | 说明 | 示例 |
|------|------|------|
| **Actions** | 自动化工作流 | CI/CD、部署、代码质量检查 |
| **Apps** | GitHub 应用 | 项目管理、代码审查、安全扫描 |
| **bots** | 自动化机器人 | PR 审查、Issue 管理、通知 |
| **Skills** | 交互式学习 | Git 教程、GitHub 使用指南 |

## 发现工具

### 按类别浏览

1. 访问 [github.com/marketplace](https://github.com/marketplace)
2. 选择感兴趣的类别
3. 浏览工具列表

### 按功能搜索

```
搜索关键词：
- CI/CD
- code review
- security
- project management
- documentation
```

## 安装和使用

### 安装 GitHub App

1. 在 Marketplace 找到应用
2. 点击 **Install**
3. 选择仓库
4. 授予权限

### 使用 GitHub Action

```yaml
# 在工作流中使用
- uses: actions/checkout@v4
- uses: 某个action@版本
  with:
    参数: 值
```

## 热门工具推荐

### CI/CD

| 工具 | 说明 |
|------|------|
| GitHub Actions | GitHub 官方 CI/CD |
| CircleCI | 持续集成服务 |
| Buildkite | CI/CD 平台 |

### 代码质量

| 工具 | 说明 |
|------|------|
| SonarCloud | 代码质量扫描 |
| CodeClimate | 代码质量分析 |
| Codacy | 自动代码审查 |

### 项目管理

| 工具 | 说明 |
|------|------|
| ZenHub | 项目管理 |
| Jira | 项目跟踪 |
| Linear | 现代项目管理 |

### 安全

| 工具 | 说明 |
|------|------|
| Snyk | 安全扫描 |
| Dependabot | 依赖更新 |
| SonarCloud | 安全漏洞扫描 |

### 文档

| 工具 | 说明 |
|------|------|
| Read the Docs | 文档托管 |
| Mintlify | 文档生成 |
| Docusaurus | 文档站点 |

## 创建自己的工具

### 创建 GitHub Action

1. 创建一个新仓库
2. 添加 `action.yml` 文件
3. 编写 Action 代码
4. 发布到 Marketplace

### 创建 GitHub App

1. 访问 **Settings** → **Developer settings** → **GitHub Apps**
2. 点击 **New GitHub App**
3. 配置权限和事件
4. 发布到 Marketplace

## 最佳实践

1. **选择知名工具**：优先选择星级高、维护活跃的工具
2. **检查权限**：安装前查看工具需要的权限
3. **阅读文档**：了解工具的使用方法
4. **定期更新**：保持工具为最新版本
5. **监控使用**：查看工具的使用情况

## 相关资源

- [GitHub Marketplace 官方文档](https://docs.github.com/en/marketplace)
- [创建 GitHub Actions](https://docs.github.com/en/actions/creating-actions)
- [创建 GitHub App](https://docs.github.com/en/apps/creating-github-apps)
