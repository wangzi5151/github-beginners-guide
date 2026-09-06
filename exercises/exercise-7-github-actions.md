# 练习 7：使用 GitHub Actions

## 目标

学习如何创建和使用 GitHub Actions 工作流。

## 步骤

### 1. 创建练习仓库

```bash
mkdir actions-practice
cd actions-practice
git init

echo "# Actions Practice" > README.md
git add README.md
git commit -m "Initial commit"

git remote add origin git@github.com:你的用户名/actions-practice.git
git push -u origin main
```

### 2. 创建基本工作流

创建 `.github/workflows/ci.yml`：

```yaml
name: CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v4
    
    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '20'
        
    - name: Install dependencies
      run: npm install
      
    - name: Run tests
      run: npm test
      
    - name: Build project
      run: npm run build
```

### 3. 创建 package.json

```json
{
  "name": "actions-practice",
  "version": "1.0.0",
  "scripts": {
    "test": "echo \"Tests passed!\" && exit 0",
    "build": "echo \"Build successful!\" && exit 0"
  }
}
```

### 4. 提交并推送

```bash
git add .
git commit -m "feat: 添加 CI 工作流"
git push
```

### 5. 观察工作流运行

1. 进入仓库 **Actions** 标签
2. 查看工作流运行状态
3. 点击查看详情

### 6. 添加更多步骤

更新 `.github/workflows/ci.yml`：

```yaml
name: CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v4
    
    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '20'
        
    - name: Cache dependencies
      uses: actions/cache@v4
      with:
        path: ~/.npm
        key: ${{ runner.os }}-npm-${{ hashFiles('**/package-lock.json') }}
        
    - name: Install dependencies
      run: npm install
      
    - name: Run linter
      run: echo "Linting..."
      
    - name: Run tests
      run: npm test
      
    - name: Build project
      run: npm run build
      
    - name: Upload build artifacts
      uses: actions/upload-artifact@v4
      with:
        name: build-output
        path: dist/
```

### 7. 使用 Secrets

1. 进入仓库 **Settings** → **Secrets and variables** → **Actions**
2. 点击 **New repository secret**
3. 添加 secret：`MY_SECRET`

在工作流中使用：

```yaml
- name: Use secret
  run: echo "Secret value is available"
  env:
    MY_SECRET: ${{ secrets.MY_SECRET }}
```

### 8. 创建矩阵构建

```yaml
name: Matrix Build

on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    
    strategy:
      matrix:
        node-version: [18, 20, 22]
        
    steps:
    - uses: actions/checkout@v4
    
    - name: Setup Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v4
      with:
        node-version: ${{ matrix.node-version }}
        
    - run: npm install
    - run: npm test
```

## 知识点

- 工作流文件结构
- 触发事件
- 作业和步骤
- 使用 Actions
- 缓存依赖
- 上传制品
- 矩阵构建
- Secrets 使用

## 下一步

[练习 8：设置 GitHub Discussions →](exercise-8-discussions.md)
