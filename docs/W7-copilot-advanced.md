# GitHub Copilot 进阶功能

## Copilot 功能矩阵

| 功能 | Free | Pro | Business | Enterprise |
|------|------|-----|----------|------------|
| 代码补全 | ✅ 有限 | ✅ | ✅ | ✅ |
| Copilot Chat | ✅ 有限 | ✅ | ✅ | ✅ |
| Agent 模式 | ❌ | ✅ | ✅ | ✅ |
| Extensions | ❌ | ✅ | ✅ | ✅ |
| Workspace | ❌ | ❌ | ✅ | ✅ |
| Audit Log | ❌ | ❌ | ❌ | ✅ |

## Agent 模式

Agent 模式让 Copilot 不仅能写代码，还能执行多步骤任务。

### 使用方法

在 VS Code 中启用 Agent 模式：

```
Ctrl+Shift+P → "Copilot: Enable Agent Mode"
```

### Agent 能力

```bash
# 让 Agent 创建完整的项目结构
"创建一个 Express API 项目，包含用户认证、数据库连接和 Docker 配置"

# 让 Agent 修复 CI/CD
"查看 GitHub Actions 失败日志并修复工作流"

# 让 Agent 重构代码
"将这个函数重构为更清晰的结构，添加错误处理"
```

### Agent 常用命令

| 命令 | 说明 |
|------|------|
| `#file` | 引用文件 |
| `#selection` | 引用选中代码 |
| `#terminalLastCommand` | 引用上次终端命令 |
| `#changes` | 引用当前更改 |
| `#codebase` | 搜索整个代码库 |
| `#problems` | 引用代码问题 |
| `#exec` | 执行终端命令 |

### 示例工作流

```bash
# 1. 分析项目
"分析这个项目的架构和依赖关系"

# 2. 添加功能
"为用户管理模块添加 CRUD API，使用 Prisma 连接 PostgreSQL"

# 3. 编写测试
"为用户 API 编写单元测试和集成测试"

# 4. 更新 CI
"更新 GitHub Actions 工作流以运行这些测试"

# 5. 更新文档
"更新 README 中的 API 文档"
```

## Copilot Extensions

### 什么是 Extensions？

Extensions 让 Copilot 可以调用外部服务和 API，扩展其能力。

### 内置 Extensions

| Extension | 功能 |
|-----------|------|
| `@terminal` | 终端命令辅助 |
| `@workspace` | 项目工作区搜索 |
| `@vscode` | VS Code 功能 |
| `@github` | GitHub 平台操作 |

### 安装第三方 Extensions

```bash
# 在 VS Code 中
1. 打开 Copilot Chat
2. 点击 Extensions 图标
3. 搜索并安装需要的 Extension
```

### 常用 Extensions

| Extension | 用途 |
|-----------|------|
| Docker | Docker 配置辅助 |
| Kubernetes | K8s YAML 生成 |
| Sentry | 错误追踪集成 |
| Vercel | 部署配置 |
| PlanetScale | 数据库操作 |

## Copilot Workspace

### 功能概览

Copilot Workspace 是一个基于 AI 的开发环境，可以直接从 Issue 生成代码变更。

### 工作流程

1. **从 Issue 开始**：在 Issue 页面点击 "Open in Workspace"
2. **AI 分析**：Copilot 理解 Issue 需求
3. **生成计划**：自动创建实现计划
4. **编写代码**：AI 生成代码变更
5. **验证测试**：自动运行测试
6. **创建 PR**：一键创建 Pull Request

### 使用场景

```bash
# 从 Issue 到 PR 的完整流程
Issue: "添加用户导出 CSV 功能"
    ↓
Copilot Workspace:
    1. 分析现有代码结构
    2. 生成导出逻辑
    3. 添加 API 端点
    4. 编写测试
    5. 更新文档
    6. 创建 PR
```

## 最佳实践

### 提示词优化

```bash
# 差的提示
"修复这个 bug"

# 好的提示
"在 src/api/users.ts 中，getUser 函数在用户不存在时返回 500 错误，应该返回 404"

# 差的提示
"添加认证"

# 好的提示
"使用 JWT 实现用户认证，包含登录、注册、刷新 token 三个端点，密码使用 bcrypt 加密"
```

### 利用上下文

```bash
# 引用相关文件
"参考 @src/models/user.ts 的结构，创建 @src/models/product.ts"

# 引用终端输出
"@terminalLastCommand 显示测试失败，请修复"

# 引用代码问题
"@problems 中的 TypeScript 错误，请修复"
```

### 团队协作

```yaml
# .github/copilot-instructions.md
# 项目级 Copilot 配置

## 代码风格
- 使用 TypeScript strict 模式
- 函数命名使用 camelCase
- 组件命名使用 PascalCase
- 测试文件使用 .test.ts 后缀

## 技术栈
- 前端：React + TypeScript
- 后端：Node.js + Express
- 数据库：PostgreSQL + Prisma
- 测试：Vitest + Playwright
```

## 常见问题

### Q: Copilot 生成的代码安全吗？
A: Copilot 不会直接发送你的代码到外部服务，但建议：
- 审查生成的代码
- 不要在提示中包含敏感信息
- 使用 Business/Enterprise 版本获得更好的隐私保护

### Q: 如何提高 Copilot 准确率？
A:
- 提供清晰的上下文
- 使用类型注解
- 编写清晰的注释
- 保持代码结构清晰

### Q: Copilot 支持哪些语言？
A: 支持几乎所有主流编程语言，对 Python、JavaScript、TypeScript、Java、Go、Rust 支持最好。

## 相关资源

- [Copilot 官方文档](https://docs.github.com/en/copilot)
- [Copilot Extensions](https://github.com/marketplace?type=apps&query=copilot)
- [Copilot 政策](https://docs.github.com/en/copilot/overview-of-github-copilot/about-github-copilot-business)

---

**上一篇：[GitHub Marketplace](N-marketplace.md) | 下一篇：[GitHub Copilot Extensions 开发](W8-copilot-extensions-dev.md)**
