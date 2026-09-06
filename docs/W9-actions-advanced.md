# GitHub Actions 高级用法

## 复用工作流 (Reusable Workflows)

### 创建复用工作流

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
        description: "构建版本号"
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
    
    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: ${{ inputs.node-version }}
    
    - run: npm ci
    - run: npm run build
    
    - name: Get version
      id: version
      run: echo "value=$(node -p 'require(\"./package.json\").version')" >> $GITHUB_OUTPUT
    
    - name: Run tests
      if: ${{ inputs.run-tests }}
      run: npm test
```

### 调用复用工作流

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

## 矩阵策略 (Matrix Strategy)

### 基础矩阵

```yaml
jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        node-version: [18, 20, 22]
    
    steps:
    - uses: actions/checkout@v4
    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: ${{ matrix.node-version }}
    - run: npm ci
    - run: npm test
```

### 高级矩阵配置

```yaml
jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      fail-fast: false  # 一个失败不影响其他
      matrix:
        os: [ubuntu-latest]
        node-version: [18, 20, 22]
        include:
          - os: windows-latest
            node-version: 20
          - os: macos-latest
            node-version: 20
        exclude:
          - os: ubuntu-latest
            node-version: 18
    
    steps:
    - uses: actions/checkout@v4
    - run: npm ci
    - run: npm test
```

## 并发控制 (Concurrency)

### 基础并发

```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true  # 取消正在运行的旧工作流
```

### 环境级并发

```yaml
jobs:
  deploy-staging:
    runs-on: ubuntu-latest
    environment: staging
    concurrency:
      group: deploy-staging
      cancel-in-progress: false  # 部署不允许取消
    
    steps:
    - uses: actions/checkout@v4
    - run: npm run deploy:staging

  deploy-production:
    needs: deploy-staging
    runs-on: ubuntu-latest
    environment: production
    concurrency:
      group: deploy-production
      cancel-in-progress: false
    
    steps:
    - uses: actions/checkout@v4
    - run: npm run deploy:production
```

## 自托管 Runner

### 安装 Runner

```bash
# 下载 Runner
mkdir actions-runner && cd actions-runner
curl -o actions-runner-linux-x64-2.311.0.tar.gz -L \
  https://github.com/actions/runner/releases/download/v2.311.0/actions-runner-linux-x64-2.311.0.tar.gz
tar xzf actions-runner-linux-x64-2.311.0.tar.gz

# 配置
./config.sh --url https://github.com/your-org/your-repo --token YOUR_TOKEN

# 安装为服务
sudo ./svc.sh install
sudo ./svc.sh start
```

### 使用 Runner

```yaml
jobs:
  build:
    runs-on: self-hosted  # 使用自托管 Runner
    
    steps:
    - uses: actions/checkout@v4
    - run: npm ci
    - run: npm run build
```

### Runner 标签

```yaml
jobs:
  build:
    runs-on: [self-hosted, linux, x64]
    
  gpu:
    runs-on: [self-hosted, gpu]
```

## 环境变量和 Secrets

### 环境变量

```yaml
env:
  NODE_ENV: production
  API_URL: https://api.example.com

jobs:
  build:
    runs-on: ubuntu-latest
    env:
      BUILD_VERSION: ${{ github.sha }}
    
    steps:
    - run: echo "Building version $BUILD_VERSION"
```

### Secrets 管理

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    
    - name: Deploy
      run: |
        echo "Deploying..."
        curl -X POST ${{ secrets.DEPLOY_WEBHOOK }} \
          -H "Authorization: Bearer ${{ secrets.DEPLOY_TOKEN }}" \
          -d '{"version": "${{ github.sha }}"}'
```

### Environment Secrets

```yaml
jobs:
  deploy-production:
    runs-on: ubuntu-latest
    environment: production  # 使用环境级 Secrets
    
    steps:
    - run: echo "Deploying to production"
      env:
        API_KEY: ${{ secrets.API_KEY }}  # 从 production 环境获取
```

## 工作流优化

### 缓存依赖

```yaml
steps:
- uses: actions/checkout@v4

- name: Cache Node.js
  uses: actions/cache@v4
  with:
    path: ~/.npm
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
    restore-keys: |
      ${{ runner.os }}-node-

- run: npm ci
```

### 条件执行

```yaml
steps:
- uses: actions/checkout@v4

- name: Build
  run: npm run build
  if: github.event_name == 'push'

- name: Test
  run: npm test
  if: github.event_name == 'pull_request'
```

### 继续错误

```yaml
steps:
- name: Try something
  run: npm run risky-command
  continue-on-error: true  # 即使失败也继续

- name: Check result
  if: steps.try.outcome == 'failure'
  run: echo "Previous step failed but we're continuing"
```

## 最佳实践

1. **使用复用工作流**：减少重复配置
2. **矩阵测试**：确保兼容性
3. **并发控制**：避免资源浪费
4. **缓存依赖**：加速构建
5. **使用 Secrets**：不要硬编码敏感信息
6. **环境保护**：生产环境需要审批

## 相关资源

- [复用工作流文档](https://docs.github.com/en/actions/using-workflows/reusing-workflows)
- [矩阵策略文档](https://docs.github.com/en/actions/using-jobs/using-a-matrix-strategy-for-jobs)
- [自托管 Runner 文档](https://docs.github.com/en/actions/hosting-your-own-runners)

---

**上一篇：[GitHub Actions 自动化](19-github-actions.md) | 下一篇：[GitHub Advanced Security](W10-advanced-security.md)**
