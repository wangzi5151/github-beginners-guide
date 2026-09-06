# 练习 19：GitHub Actions 复用工作流

## 学习目标

- 创建复用工作流
- 调用复用工作流
- 使用矩阵策略

## 步骤

### 步骤 1：创建复用工作流

```yaml
# .github/workflows/reusable-build.yml
name: Reusable Build

on:
  workflow_call:
    inputs:
      node-version:
        required: false
        type: string
        default: '20'
      run-tests:
        required: false
        type: boolean
        default: true
    outputs:
      build-version:
        description: "构建版本"
        value: ${{ jobs.build.outputs.version }}
    secrets:
      NPM_TOKEN:
        required: true

jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      version: ${{ steps.version.outputs.value }}
    steps:
    - uses: actions/checkout@v4
    
    - uses: actions/setup-node@v4
      with:
        node-version: ${{ inputs.node-version }}
        cache: 'npm'
    
    - run: npm ci
    - run: npm run build
    
    - name: Get version
      id: version
      run: echo "value=$(node -p 'require(\"./package.json\").version')" >> $GITHUB_OUTPUT
    
    - name: Run tests
      if: ${{ inputs.run-tests }}
      run: npm test
```

### 步骤 2：调用复用工作流

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    uses: ./.github/workflows/reusable-build.yml
    with:
      node-version: '20'
      run-tests: true
    secrets:
      NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```

### 步骤 3：使用矩阵策略

```yaml
# .github/workflows/matrix.yml
name: Matrix Build

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [18, 20, 22]
        os: [ubuntu-latest, windows-latest]
      fail-fast: false
    
    steps:
    - uses: actions/checkout@v4
    
    - uses: actions/setup-node@v4
      with:
        node-version: ${{ matrix.node-version }}
    
    - run: npm ci
    - run: npm test
```

## 实战任务

1. 创建复用工作流
2. 配置输入参数和输出
3. 调用复用工作流
4. 使用矩阵策略测试多个环境

## 验证清单

- [ ] 复用工作流已创建
- [ ] 输入参数已配置
- [ ] 输出已配置
- [ ] 调用工作流正常
- [ ] 矩阵策略正常

## 下一步

继续 [练习 20：GitHub Projects 看板管理](exercise-20-project-board.md)
