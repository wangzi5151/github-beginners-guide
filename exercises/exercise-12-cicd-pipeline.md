# 练习 12：CI/CD 流水线实战

## 学习目标

- 构建完整的 CI/CD 流水线
- 集成测试、构建、部署
- 使用矩阵策略和缓存

## 步骤

### 步骤 1：创建工作流

创建 `.github/workflows/ci-cd.yml`：

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - uses: actions/setup-node@v4
      with:
        node-version: '20'
        cache: 'npm'
    - run: npm ci
    - run: npm run lint

  test:
    runs-on: ubuntu-latest
    needs: lint
    strategy:
      matrix:
        node-version: [18, 20, 22]
    steps:
    - uses: actions/checkout@v4
    - uses: actions/setup-node@v4
      with:
        node-version: ${{ matrix.node-version }}
        cache: 'npm'
    - run: npm ci
    - run: npm test

  build:
    runs-on: ubuntu-latest
    needs: test
    steps:
    - uses: actions/checkout@v4
    - uses: actions/setup-node@v4
      with:
        node-version: '20'
        cache: 'npm'
    - run: npm ci
    - run: npm run build
    - uses: actions/upload-artifact@v4
      with:
        name: build
        path: dist/
```

### 步骤 2：添加缓存

```yaml
- uses: actions/cache@v4
  with:
    path: ~/.npm
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
    restore-keys: |
      ${{ runner.os }}-node-
```

### 步骤 3：添加并发控制

```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```

## 实战任务

1. 创建包含 lint、test、build 的完整流水线
2. 为 test 任务添加矩阵策略
3. 添加依赖缓存
4. 配置并发控制

## 验证清单

- [ ] 能够创建 CI/CD 工作流
- [ ] 能够使用矩阵策略
- [ ] 能够配置缓存
- [ ] 能够使用 artifact

## 下一步

继续 [练习 13：Docker 部署实战](exercise-13-docker-deploy.md)
