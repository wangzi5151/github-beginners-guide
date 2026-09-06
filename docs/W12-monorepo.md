# Monorepo 管理

## 什么是 Monorepo？

Monorepo 是一种将多个项目代码放在同一个仓库中的管理方式。

## 工具对比

| 工具 | 语言 | 特点 |
|------|------|------|
| pnpm workspaces | Node.js | 轻量、高效 |
| Turborepo | 任意 | 增量构建、缓存 |
| Nx | 任意 | 智能构建、依赖图 |
| Lerna | Node.js | 传统工具 |

## pnpm Workspaces

### 配置

```json
// package.json
{
  "name": "monorepo",
  "private": true,
  "scripts": {
    "build": "pnpm -r build",
    "test": "pnpm -r test",
    "dev": "pnpm -r --parallel dev"
  },
  "pnpm": {
    "workspace": "packages/*"
  }
}
```

### 项目结构

```
monorepo/
├── package.json
├── pnpm-workspace.yaml
├── packages/
│   ├── shared/          # 共享库
│   │   ├── package.json
│   │   └── src/
│   ├── web/             # 前端应用
│   │   ├── package.json
│   │   └── src/
│   └── api/             # 后端 API
│       ├── package.json
│       └── src/
└── pnpm-lock.yaml
```

### pnpm-workspace.yaml

```yaml
packages:
  - 'packages/*'
  - 'apps/*'
```

## Turborepo

### 配置

```json
// turbo.json
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
    "lint": {},
    "dev": {
      "cache": false,
      "persistent": true
    }
  }
}
```

### 使用

```bash
# 构建所有包
turbo run build

# 只构建特定包
turbo run build --filter=web

# 运行测试
turbo run test

# 开发模式
turbo run dev
```

### 缓存

```bash
# 查看缓存
turbo run build --dry

# 清除缓存
turbo prune

# 远程缓存
turbo login
turbo link
```

## Nx

### 配置

```json
// nx.json
{
  "targetDefaults": {
    "build": {
      "dependsOn": ["^build"],
      "inputs": ["production", "^production"]
    }
  },
  "defaultBase": "main"
}
```

### 使用

```bash
# 构建所有
nx run-many -t build

# 只构建受影响的
nx affected -t build

# 运行测试
nx run-many -t test

# 查看依赖图
nx graph
```

### 依赖图

```bash
# 生成依赖图
nx graph

# 可视化特定项目
nx graph --affected
```

## GitHub Actions 集成

```yaml
# .github/workflows/monorepo.yml
name: Monorepo CI

on:
  push:
    branches: [main]
  pull_request:

jobs:
  detect-changes:
    runs-on: ubuntu-latest
    outputs:
      packages: ${{ steps.changes.outputs.packages }}
    steps:
    - uses: actions/checkout@v4
    
    - name: Detect changes
      id: changes
      run: |
        PACKAGES=$(git diff --name-only HEAD~1 | grep -oP 'packages/\K[^/]+' | sort -u | jq -R -s -c 'split("\n")[:-1]')
        echo "packages=$PACKAGES" >> $GITHUB_OUTPUT

  build:
    needs: detect-changes
    runs-on: ubuntu-latest
    strategy:
      matrix:
        package: ${{ fromJson(needs.detect-changes.outputs.packages) }}
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '20'
    
    - run: pnpm install --frozen-lockfile
    - run: pnpm --filter ${{ matrix.package }} build
    - run: pnpm --filter ${{ matrix.package }} test
```

## 最佳实践

1. **合理拆分包**：按功能边界拆分
2. **使用共享库**：避免代码重复
3. **增量构建**：只构建受影响的包
4. **远程缓存**：加速 CI/CD
5. **依赖管理**：使用 workspace 协议

## 相关资源

- [pnpm Workspaces 文档](https://pnpm.io/workspaces)
- [Turborepo 文档](https://turbo.build/repo/docs)
- [Nx 文档](https://nx.dev/docs)

---

**上一篇：[GitHub Models AI/ML](W11-github-models.md) | 下一篇：[供应链安全](W13-supply-chain-security.md)**
