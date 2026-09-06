# 练习 16：Monorepo 管理实战

## 学习目标

- 使用 pnpm workspaces
- 配置 Turborepo
- 创建增量构建工作流

## 步骤

### 步骤 1：初始化 Monorepo

```bash
mkdir my-monorepo && cd my-monorepo
pnpm init
```

### 步骤 2：配置 workspace

```yaml
# pnpm-workspace.yaml
packages:
  - 'packages/*'
```

### 步骤 3：创建包结构

```
my-monorepo/
├── package.json
├── pnpm-workspace.yaml
├── turbo.json
├── packages/
│   ├── shared/
│   │   ├── package.json
│   │   └── src/
│   ├── web/
│   │   ├── package.json
│   │   └── src/
│   └── api/
│       ├── package.json
│       └── src/
```

### 步骤 4：配置 Turborepo

```json
{
  "$schema": "https://turbo.build/schema.json",
  "tasks": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": ["dist/**"]
    },
    "test": {
      "dependsOn": ["build"]
    },
    "lint": {}
  }
}
```

### 步骤 5：创建 CI 工作流

```yaml
# .github/workflows/monorepo.yml
name: Monorepo CI

on:
  push:
    branches: [main]
  pull_request:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - uses: pnpm/action-setup@v2
      with:
        version: 8
    - uses: actions/setup-node@v4
      with:
        node-version: '20'
        cache: 'pnpm'
    - run: pnpm install --frozen-lockfile
    - run: pnpm turbo build test lint
```

## 实战任务

1. 初始化 pnpm workspace
2. 创建多个包
3. 配置 Turborepo
4. 创建增量构建工作流

## 验证清单

- [ ] 能够初始化 pnpm workspace
- [ ] 能够配置 Turborepo
- [ ] 能够运行增量构建
- [ ] 能够创建 Monorepo CI

## 完成

恭喜完成所有练习！你已经掌握了：
- Git 基础操作
- GitHub 核心功能
- CI/CD 流水线
- Docker 容器化
- 安全扫描
- 发布管理
- Monorepo 管理
