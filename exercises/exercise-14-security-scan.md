# 练习 14：安全扫描实战

## 学习目标

- 配置 CodeQL 代码扫描
- 使用 Trivy 扫描容器漏洞
- 配置 Dependabot 自动更新

## 步骤

### 步骤 1：CodeQL 分析

```yaml
# .github/workflows/codeql.yml
name: CodeQL

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 6 * * 1'

jobs:
  analyze:
    runs-on: ubuntu-latest
    permissions:
      security-events: write
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Initialize CodeQL
      uses: github/codeql-action/init@v3
      with:
        languages: javascript
    
    - name: Autobuild
      uses: github/codeql-action/autobuild@v3
    
    - name: Perform CodeQL Analysis
      uses: github/codeql-action/analyze@v3
```

### 步骤 2：Dependabot 配置

```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "weekly"
```

### 步骤 3：容器扫描

```yaml
# .github/workflows/container-scan.yml
name: Container Scan

on:
  push:
    branches: [main]

jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    
    - name: Build image
      run: docker build -t my-app:scan .
    
    - name: Run Trivy
      uses: aquasecurity/trivy-action@master
      with:
        image-ref: 'my-app:scan'
        format: 'sarif'
        output: 'trivy-results.sarif'
        severity: 'CRITICAL,HIGH'
    
    - name: Upload results
      uses: github/codeql-action/upload-sarif@v3
      with:
        sarif_file: 'trivy-results.sarif'
```

## 实战任务

1. 配置 CodeQL 代码扫描
2. 配置 Dependabot 自动更新
3. 创建容器安全扫描工作流
4. 查看扫描结果

## 验证清单

- [ ] 能够配置 CodeQL 分析
- [ ] 能够配置 Dependabot
- [ ] 能够运行容器扫描
- [ ] 能够解读扫描结果

## 下一步

继续 [练习 15：发布管理实战](exercise-15-release-management.md)
