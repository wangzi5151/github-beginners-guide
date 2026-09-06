# GitHub Advanced Security

## 功能概览

| 功能 | Free | Team | Enterprise |
|------|------|------|------------|
| Secret Scanning | ✅ | ✅ | ✅ |
| Secret Scanning Push Protection | ❌ | ✅ | ✅ |
| Code Scanning (SAST) | ✅ 有限 | ✅ | ✅ |
| CodeQL | ✅ 有限 | ✅ | ✅ |
| Dependency Review | ✅ | ✅ | ✅ |
| Dependabot | ✅ | ✅ | ✅ |
| Security Overview | ❌ | ✅ | ✅ |

## Secret Scanning

### 自动扫描

GitHub 自动扫描仓库中的敏感信息：

```yaml
# .github/workflows/secret-scan.yml
name: Secret Scan

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
      with:
        fetch-depth: 0
    
    - name: Run TruffleHog
      uses: trufflesecurity/trufflehog@main
      with:
        extra_args: --only-verified
```

### Push Protection

在推送时阻止包含敏感信息的提交：

```yaml
# 启用 Push Protection
# Settings → Code security and analysis → Push protection
```

### 自定义 Secret 模式

```yaml
# .github/secret-scanning.yml
custom-patterns:
  - name: Internal API Key
    pattern: 'internal-api-key-[a-zA-Z0-9]{32}'
    description: Internal API keys
```

## Code Scanning

### CodeQL 分析

```yaml
# .github/workflows/codeql.yml
name: CodeQL

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 6 * * 1'  # 每周一

jobs:
  analyze:
    runs-on: ubuntu-latest
    permissions:
      security-events: write
    
    strategy:
      fail-fast: false
      matrix:
        language: ['javascript', 'python']
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Initialize CodeQL
      uses: github/codeql-action/init@v3
      with:
        languages: ${{ matrix.language }}
    
    - name: Autobuild
      uses: github/codeql-action/autobuild@v3
    
    - name: Perform CodeQL Analysis
      uses: github/codeql-action/analyze@v3
      with:
        category: "/language:${{ matrix.language }}"
```

### Semgrep (SAST)

```yaml
# .github/workflows/semgrep.yml
name: Semgrep

on:
  push:
    branches: [main]
  pull_request:

jobs:
  semgrep:
    runs-on: ubuntu-latest
    container:
      image: semgrep/semgrep
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Run Semgrep
      run: semgrep scan --config=auto --sarif -o results.sarif
    
    - name: Upload SARIF
      uses: github/codeql-action/upload-sarif@v3
      with:
        sarif_file: results.sarif
```

## Dependency Review

### PR 依赖审查

```yaml
# .github/workflows/dependency-review.yml
name: Dependency Review

on:
  pull_request:

permissions:
  contents: read
  pull-requests: write

jobs:
  dependency-review:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    
    - name: Dependency Review
      uses: actions/dependency-review-action@v4
      with:
        fail-on-severity: high
        deny-licenses: GPL-3.0, AGPL-3.0
```

### 配置审查策略

```yaml
# .github/dependency-review.yml
comment-summary-in-pr: always
fail-on-severity: high
license-check: true
deny-licenses:
  - GPL-3.0
  - AGPL-3.0
allow-dependencies-licenses:
  - package: mit
  - package: apache-2.0
```

## Dependabot

### 自动更新配置

```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
      day: "monday"
    open-pull-requests-limit: 10
    labels:
      - "dependencies"
      - "automated"
    groups:
      minor-and-patch:
        update-types:
          - "minor"
          - "patch"
  
  - package-ecosystem: "docker"
    directory: "/"
    schedule:
      interval: "weekly"
  
  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "weekly"
  
  - package-ecosystem: "pip"
    directory: "/"
    schedule:
      interval: "monthly"
```

### Dependabot Alerts

```yaml
# 启用 Dependabot Alerts
# Settings → Code security and analysis → Dependabot alerts
```

## Security Overview

### 安全仪表板

```
Organization → Security → Security overview

显示：
- 仓库安全状态
- 漏洞数量
- Secret 泄露
- 依赖问题
```

### 安全策略

```markdown
# SECURITY.md
## 安全策略

### 报告漏洞
请通过 security@yourcompany.com 报告安全漏洞

### 响应时间
- 严重漏洞：24 小时内响应
- 高危漏洞：72 小时内响应
- 中危漏洞：1 周内响应
```

## 供应链安全

### SLSA 合规

```yaml
# .github/workflows/slsa.yml
name: SLSA Build

on:
  push:
    tags: ['v*']

jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      hashes: ${{ steps.hash.outputs.hashes }}
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Build
      run: npm run build
    
    - name: Generate SLSA provenance
      uses: slsa-framework/slsa-github-generator@v1.9.0
      with:
        base64-subjects: "${{ steps.hash.outputs.hashes }}"
```

### SBOM 生成

```yaml
# .github/workflows/sbom.yml
name: Generate SBOM

on:
  push:
    branches: [main]

jobs:
  sbom:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    
    - name: Generate SBOM
      uses: anchore/sbom-action@v0
      with:
        artifact-name: sbom.spdx.json
        output-file: sbom.spdx.json
```

## 最佳实践

1. **启用所有安全功能**：Secret Scanning, Code Scanning, Dependabot
2. **配置 Push Protection**：防止敏感信息泄露
3. **定期审查依赖**：及时更新有漏洞的依赖
4. **使用 SLSA**：确保构建完整性
5. **生成 SBOM**：跟踪软件组成

## 相关资源

- [GitHub Advanced Security 文档](https://docs.github.com/en/github-security)
- [CodeQL 文档](https://codeql.github.com/)
- [SLSA 文档](https://slsa.dev/)

---

**上一篇：[安全与权限管理](28-security-permissions.md) | 下一篇：[GitHub Models AI/ML](W11-github-models.md)**
