# 练习 28：配置 GitHub 安全扫描

## 学习目标

完成本练习后，你将能够：

- 理解 GitHub 安全功能的整体架构
- 配置和使用 CodeQL 进行代码扫描
- 启用 Secret Scanning 防止密钥泄露
- 配置 Dependabot 自动修复依赖漏洞
- 查看和处理安全警报
- 建立完整的安全扫描工作流

## 前置条件

- 拥有 GitHub 仓库（公开或私有）
- 了解基本的安全概念
- 熟悉 GitHub Actions 基础知识
- 了解常见安全漏洞类型

## 背景知识

### GitHub 安全功能概述

GitHub 提供了多层次的安全防护：

1. **Code Scanning（代码扫描）**
   - 使用 CodeQL 分析代码漏洞
   - 支持多种编程语言
   - 可集成到 CI/CD 流程

2. **Secret Scanning（密钥扫描）**
   - 检测代码中的敏感信息
   - 支持 100+ 种密钥格式
   - 自动通知服务提供商

3. **Dependabot**
   - 自动检测依赖漏洞
   - 自动生成修复 PR
   - 支持多种包管理器

4. **Security Advisories（安全公告）**
   - 私密报告漏洞
   - 协调修复发布

---

## 练习步骤

### 第一部分：启用 CodeQL 代码扫描

#### 步骤 1：创建 CodeQL 工作流

创建文件 `.github/workflows/codeql-analysis.yml`：

```yaml
name: "CodeQL Analysis"

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]
  schedule:
    # 每周一 UTC 00:00 运行
    - cron: '0 0 * * 1'

permissions:
  actions: read
  contents: read
  security-events: write

jobs:
  analyze:
    name: Analyze
    runs-on: ubuntu-latest
    
    strategy:
      fail-fast: false
      matrix:
        language: ['javascript', 'python']
        # 如果有更多语言，可以添加：
        # language: ['c-cpp', 'csharp', 'go', 'java-kotlin', 'javascript', 'python', 'ruby', 'swift']
    
    steps:
      - name: 检出代码
        uses: actions/checkout@v4
      
      - name: 初始化 CodeQL
        uses: github/codeql-action/init@v3
        with:
          languages: ${{ matrix.language }}
          # 使用安全扩展查询
          queries: security-extended
          # 或使用全面查询
          # queries: security-and-quality
      
      - name: Autobuild
        uses: github/codeql-action/autobuild@v3
      
      - name: 执行 CodeQL 分析
        uses: github/codeql-action/analyze@v3
        with:
          category: "/language:${{ matrix.language }}"
```

#### 步骤 2：为特定语言配置构建

如果自动构建不适用，可以手动配置。创建文件 `.github/workflows/codeql-python.yml`：

```yaml
name: "CodeQL Python Analysis"

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 0 * * 1'

permissions:
  actions: read
  contents: read
  security-events: write

jobs:
  analyze:
    name: Analyze Python
    runs-on: ubuntu-latest
    
    steps:
      - name: 检出代码
        uses: actions/checkout@v4
      
      - name: 设置 Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'
      
      - name: 安装依赖
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt
          pip install -r requirements-dev.txt || true
      
      - name: 初始化 CodeQL
        uses: github/codeql-action/init@v3
        with:
          languages: python
          queries: security-extended
      
      - name: 手动构建
        run: |
          # 如果需要编译步骤，在这里添加
          echo "构建完成"
      
      - name: 执行 CodeQL 分析
        uses: github/codeql-action/analyze@v3
        with:
          category: "/language:python"
```

#### 步骤 3：为 JavaScript/TypeScript 配置 CodeQL

创建文件 `.github/workflows/codeql-javascript.yml`：

```yaml
name: "CodeQL JavaScript Analysis"

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 0 * * 1'

permissions:
  actions: read
  contents: read
  security-events: write

jobs:
  analyze:
    name: Analyze JavaScript
    runs-on: ubuntu-latest
    
    steps:
      - name: 检出代码
        uses: actions/checkout@v4
      
      - name: 初始化 CodeQL
        uses: github/codeql-action/init@v3
        with:
          languages: javascript-typescript
          queries: security-extended
          # 自定义查询配置
          config: |
            name: "Custom JavaScript Configuration"
            queries:
              - uses: security-extended
              - uses: security-and-quality
            paths:
              - src
              - lib
            paths-ignore:
              - '**/node_modules'
              - '**/test'
              - '**/tests'
              - '**/__tests__'
              - '**/dist'
      
      - name: Autobuild
        uses: github/codeql-action/autobuild@v3
      
      - name: 执行 CodeQL 分析
        uses: github/codeql-action/analyze@v3
        with:
          category: "/language:javascript-typescript"
```

#### 步骤 4：创建 CodeQL 配置文件

创建文件 `.github/codeql/codeql-config.yml`：

```yaml
name: "CodeQL Configuration"

# 查询套件
queries:
  - uses: security-extended
  - uses: security-and-quality

# 自定义查询
query-filters:
  - include:
      tags:
        - security
        - correctness
  - exclude:
      id: js/unused-local-variable

# 路径配置
paths:
  - src
  - lib
  - app

paths-ignore:
  - '**/node_modules'
  - '**/test/**'
  - '**/tests/**'
  - '**/__tests__/**'
  - '**/vendor/**'
  - '**/dist/**'
  - '**/build/**'
  - '**/*.test.js'
  - '**/*.test.ts'
  - '**/*.spec.js'
  - '**/*.spec.ts'

# 自定义查询路径
packs:
  javascript:
    - codeql/javascript-queries
  python:
    - codeql/python-queries
```

在工作流中使用配置文件：

```yaml
- name: 初始化 CodeQL
  uses: github/codeql-action/init@v3
  with:
    languages: javascript
    config-file: ./.github/codeql/codeql-config.yml
```

### 第二部分：配置 Secret Scanning

#### 步骤 5：启用 Secret Scanning

1. 访问仓库的 Settings → Security → Code security and analysis
2. 找到 "Secret scanning" 部分
3. 启用以下选项：
   - Secret scanning
   - Secret scanning push protection（推荐）

或者通过 GitHub CLI：

```bash
# 启用 Secret Scanning
gh api -X PATCH repos/{owner}/{repo} \
  -f security_and_analysis='{"secret_scanning":{"status":"enabled"},"secret_scanning_push_protection":{"status":"enabled"}}'
```

#### 步骤 6：创建自定义 Secret Scanning 模式

创建文件 `.github/secret_scanning.yml`：

```yaml
# 自定义密钥扫描模式
# 格式：正则表达式 + 密钥类型描述

patterns:
  # 自定义 API 密钥模式
  - name: "Custom API Key"
    regex: '(?i)api[_-]?key\s*[:=]\s*["\']?([a-zA-Z0-9]{32,})["\']?'
    confidence: high
    
  # 自定义数据库连接字符串
  - name: "Database Connection String"
    regex: '(?i)(mysql|postgresql|mongodb)://[^\s]+'
    confidence: high
    
  # AWS 密钥模式
  - name: "AWS Access Key"
    regex: 'AKIA[0-9A-Z]{16}'
    confidence: high
    
  # 私钥模式
  - name: "Private Key"
    regex: '-----BEGIN (RSA |EC |DSA )?PRIVATE KEY-----'
    confidence: high
```

#### 步骤 7：在 CI 中集成密钥检测

创建文件 `.github/workflows/secret-detection.yml`：

```yaml
name: Secret Detection

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  detect-secrets:
    name: Detect Secrets
    runs-on: ubuntu-latest
    
    steps:
      - name: 检出代码
        uses: actions/checkout@v4
        with:
          fetch-depth: 0
      
      - name: 安装 gitleaks
        run: |
          wget https://github.com/gitleaks/gitleaks/releases/download/v8.18.0/gitleaks_8.18.0_linux_x64.tar.gz
          tar -xzf gitleaks_8.18.0_linux_x64.tar.gz
          sudo mv gitleaks /usr/local/bin/
      
      - name: 运行 gitleaks
        run: |
          gitleaks detect --source . --verbose --report-format sarif --report-path gitleaks-report.sarif
        continue-on-error: true
      
      - name: 上传 SARIF 报告
        uses: github/codeql-action/upload-sarif@v3
        if: always()
        with:
          sarif_file: gitleaks-report.sarif
          category: gitleaks
      
      - name: 检查结果
        if: failure()
        run: |
          echo "检测到潜在的密钥泄露！"
          echo "请查看安全警报并修复问题。"
          exit 1
```

#### 步骤 8：创建 .gitignore 防止密钥提交

更新 `.gitignore` 文件：

```gitignore
# 环境变量和密钥文件
.env
.env.local
.env.*.local
*.pem
*.key
*.cert
*.p12
*.pfx

# IDE 配置（可能包含密钥）
.idea/
.vscode/settings.json

# 依赖目录
node_modules/
vendor/
venv/

# 构建产物
dist/
build/
*.egg-info/

# 日志文件
*.log
logs/

# 操作系统文件
.DS_Store
Thumbs.db

# 密钥文件
secrets/
credentials/
*.secret
*.credentials
```

### 第三部分：配置 Dependabot

#### 步骤 9：创建 Dependabot 配置文件

创建文件 `.github/dependabot.yml`：

```yaml
version: 2
updates:
  # GitHub Actions 依赖
  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "weekly"
      day: "monday"
      time: "09:00"
      timezone: "Asia/Shanghai"
    open-pull-requests-limit: 10
    reviewers:
      - "your-team-name"
    labels:
      - "dependencies"
      - "github-actions"
    commit-message:
      prefix: "ci"
      include: "scope"
  
  # npm 依赖
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
      day: "monday"
      time: "09:00"
      timezone: "Asia/Shanghai"
    open-pull-requests-limit: 10
    reviewers:
      - "your-team-name"
    labels:
      - "dependencies"
      - "npm"
    commit-message:
      prefix: "deps"
      include: "scope"
    # 忽略特定依赖的更新
    ignore:
      - dependency-name: "lodash"
        update-types: ["version-update:semver-major"]
      - dependency-name: "express"
        versions: [">=5.0.0"]
    # 分组更新
    groups:
      development:
        dependency-type: "development"
        patterns:
          - "@types/*"
          - "eslint*"
          - "prettier"
          - "jest"
          - "typescript"
      production:
        dependency-type: "production"
        patterns:
          - "express"
          - "mongoose"
          - "jsonwebtoken"
  
  # Python 依赖
  - package-ecosystem: "pip"
    directory: "/"
    schedule:
      interval: "weekly"
      day: "tuesday"
      time: "09:00"
      timezone: "Asia/Shanghai"
    open-pull-requests-limit: 10
    reviewers:
      - "your-team-name"
    labels:
      - "dependencies"
      - "python"
    commit-message:
      prefix: "deps"
      include: "scope"
    groups:
      development:
        dependency-type: "development"
        patterns:
          - "pytest*"
          - "ruff"
          - "mypy"
          - "black"
      production:
        dependency-type: "production"
        patterns:
          - "flask*"
          - "requests"
          - "sqlalchemy"
  
  # Docker 依赖
  - package-ecosystem: "docker"
    directory: "/"
    schedule:
      interval: "weekly"
    open-pull-requests-limit: 5
    labels:
      - "dependencies"
      - "docker"
  
  # Terraform 依赖
  - package-ecosystem: "terraform"
    directory: "/"
    schedule:
      interval: "weekly"
    open-pull-requests-limit: 5
    labels:
      - "dependencies"
      - "terraform"
```

#### 步骤 10：配置 Dependabot 安全更新

在仓库设置中启用自动安全更新：

```bash
# 使用 GitHub CLI 启用自动安全更新
gh api -X PATCH repos/{owner}/{repo} \
  -f '{"security_and_analysis":{"dependabot_security_updates":{"status":"enabled"}}}'
```

#### 步骤 11：创建 Dependabot 自动合并工作流

创建文件 `.github/workflows/dependabot-auto-merge.yml`：

```yaml
name: Dependabot Auto Merge

on:
  pull_request:

permissions:
  contents: write
  pull-requests: write

jobs:
  auto-merge:
    runs-on: ubuntu-latest
    if: github.actor == 'dependabot[bot]'
    
    steps:
      - name: 获取 Dependabot 元数据
        id: metadata
        uses: dependabot/fetch-metadata@v2
        with:
          github-token: "${{ secrets.GITHUB_TOKEN }}"
      
      - name: 自动合并补丁更新
        if: steps.metadata.outputs.update-type == 'version-update:semver-patch'
        run: |
          gh pr merge --auto --squash "$PR_URL"
        env:
          PR_URL: ${{ github.event.pull_request.html_url }}
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      
      - name: 自动合并次要更新（开发依赖）
        if: |
          steps.metadata.outputs.update-type == 'version-update:semver-minor' &&
          steps.metadata.outputs.dependency-type == 'development:direct'
        run: |
          gh pr merge --auto --squash "$PR_URL"
        env:
          PR_URL: ${{ github.event.pull_request.html_url }}
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      
      - name: 添加审查标签
        if: |
          steps.metadata.outputs.update-type == 'version-update:semver-major' ||
          (steps.metadata.outputs.update-type == 'version-update:semver-minor' &&
           steps.metadata.outputs.dependency-type == 'production:direct')
        run: |
          gh pr edit "$PR_URL" --add-label "needs-review"
        env:
          PR_URL: ${{ github.event.pull_request.html_url }}
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### 第四部分：综合安全扫描工作流

#### 步骤 12：创建完整的安全扫描工作流

创建文件 `.github/workflows/security-scan.yml`：

```yaml
name: Security Scan

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]
  schedule:
    # 每天 UTC 02:00 运行
    - cron: '0 2 * * *'

permissions:
  actions: read
  contents: read
  security-events: write
  pull-requests: write

jobs:
  # 代码扫描
  codeql:
    name: CodeQL Analysis
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        language: ['javascript', 'python']
    
    steps:
      - name: 检出代码
        uses: actions/checkout@v4
      
      - name: 初始化 CodeQL
        uses: github/codeql-action/init@v3
        with:
          languages: ${{ matrix.language }}
          queries: security-extended
      
      - name: Autobuild
        uses: github/codeql-action/autobuild@v3
      
      - name: 执行 CodeQL 分析
        uses: github/codeql-action/analyze@v3
        with:
          category: "/language:${{ matrix.language }}"
  
  # 依赖扫描
  dependency-review:
    name: Dependency Review
    runs-on: ubuntu-latest
    if: github.event_name == 'pull_request'
    
    steps:
      - name: 检出代码
        uses: actions/checkout@v4
      
      - name: 依赖审查
        uses: actions/dependency-review-action@v4
        with:
          fail-on-severity: high
          deny-licenses: GPL-3.0, AGPL-3.0
  
  # 密钥扫描
  secret-scan:
    name: Secret Scanning
    runs-on: ubuntu-latest
    
    steps:
      - name: 检出代码
        uses: actions/checkout@v4
        with:
          fetch-depth: 0
      
      - name: 运行 trufflehog
        uses: trufflesecurity/trufflehog@main
        with:
          extra_args: --only-verified
  
  # SAST 扫描
  sast:
    name: SAST Scan
    runs-on: ubuntu-latest
    
    steps:
      - name: 检出代码
        uses: actions/checkout@v4
      
      - name: 运行 Semgrep
        uses: returntocorp/semgrep-action@v1
        with:
          config: >-
            p/security-audit
            p/secrets
            p/owasp-top-ten
            p/ci
  
  # 容器扫描
  container-scan:
    name: Container Scan
    runs-on: ubuntu-latest
    if: hashFiles('Dockerfile') != ''
    
    steps:
      - name: 检出代码
        uses: actions/checkout@v4
      
      - name: 构建 Docker 镜像
        run: docker build -t test-image .
      
      - name: 运行 Trivy 扫描
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: 'test-image'
          format: 'sarif'
          output: 'trivy-results.sarif'
          severity: 'CRITICAL,HIGH'
      
      - name: 上传 Trivy SARIF 报告
        uses: github/codeql-action/upload-sarif@v3
        if: always()
        with:
          sarif_file: 'trivy-results.sarif'
  
  # 安全报告
  security-report:
    name: Security Report
    runs-on: ubuntu-latest
    needs: [codeql, dependency-review, secret-scan, sast, container-scan]
    if: always()
    
    steps:
      - name: 生成安全报告
        run: |
          echo "# 安全扫描报告" > security-report.md
          echo "" >> security-report.md
          echo "## 扫描时间: $(date)" >> security-report.md
          echo "" >> security-report.md
          echo "## 扫描结果" >> security-report.md
          echo "" >> security-report.md
          echo "| 检查项 | 状态 |" >> security-report.md
          echo "|--------|------|" >> security-report.md
          echo "| CodeQL | ${{ needs.codeql.result }} |" >> security-report.md
          echo "| 依赖审查 | ${{ needs.dependency-review.result }} |" >> security-report.md
          echo "| 密钥扫描 | ${{ needs.secret-scan.result }} |" >> security-report.md
          echo "| SAST | ${{ needs.sast.result }} |" >> security-report.md
          echo "| 容器扫描 | ${{ needs.container-scan.result }} |" >> security-report.md
      
      - name: 上传报告
        uses: actions/upload-artifact@v4
        with:
          name: security-report
          path: security-report.md
```

### 第五部分：安全策略配置

#### 步骤 13：创建安全策略文件

创建文件 `SECURITY.md`：

```markdown
# 安全策略

## 支持的版本

| 版本 | 支持状态 |
|------|---------|
| 最新版本 | ✅ 支持 |
| 旧版本 | ❌ 不支持 |

## 报告漏洞

如果您发现安全漏洞，请通过以下方式报告：

1. **不要**在公开的 issue 中报告安全漏洞
2. 使用 GitHub 的私密漏洞报告功能
3. 或者发送邮件至 security@example.com

### 报告内容

请在报告中包含：

- 漏洞类型
- 受影响的版本
- 复现步骤
- 潜在影响
- 修复建议（如果有）

### 响应时间

- 我们会在 48 小时内确认收到报告
- 我们会在 7 天内提供初步评估
- 我们会在 30 天内发布修复（如果需要）

## 安全最佳实践

### 对于用户

1. 始终使用最新版本
2. 定期检查依赖更新
3. 不要在代码中硬编码密钥
4. 使用环境变量存储敏感信息

### 对于贡献者

1. 遵循安全编码规范
2. 不要提交包含密钥的代码
3. 使用 CodeQL 和其他工具检查代码
4. 及时修复安全警报

## 安全相关配置

### 环境变量

使用 `.env` 文件存储本地配置，不要提交到版本控制：

```
DATABASE_URL=...
API_KEY=...
SECRET_KEY=...
```

### 依赖管理

使用 Dependabot 自动更新依赖，定期检查安全公告。
```

#### 步骤 14：创建 CODEOWNERS 文件

创建文件 `.github/CODEOWNERS`：

```
# 安全相关文件需要安全团队审查
SECURITY.md @your-org/security-team
.github/workflows/ @your-org/security-team
.github/dependabot.yml @your-org/security-team

# 依赖文件需要审查
package.json @your-org/backend-team
package-lock.json @your-org/backend-team
requirements.txt @your-org/backend-team

# Docker 文件需要审查
Dockerfile @your-org/devops-team
docker-compose.yml @your-org/devops-team
```

---

## 验证练习结果

### 检查清单

完成练习后，请验证以下内容：

- [ ] CodeQL 工作流已创建并能正常运行
- [ ] Secret Scanning 已启用
- [ ] Dependabot 配置已创建
- [ ] 安全策略文件已创建
- [ ] 安全扫描工作流能检测到测试漏洞

### 验证命令

```bash
# 检查 CodeQL 工作流语法
actionlint .github/workflows/codeql-analysis.yml

# 检查 Dependabot 配置
gh api repos/{owner}/{repo}/vulnerability-alerts

# 手动触发安全扫描
gh workflow run security-scan.yml

# 查看安全警报
gh api repos/{owner}/{repo}/code-scanning/alerts
```

### 测试安全扫描

创建测试文件 `test-vulnerabilities.js` 来验证扫描是否工作：

```javascript
// 测试 SQL 注入漏洞（仅用于测试，不要在生产代码中使用）
function getUserData(userId) {
  const query = "SELECT * FROM users WHERE id = " + userId;  // SQL 注入漏洞
  return db.query(query);
}

// 测试 XSS 漏洞
function displayUserInput(input) {
  document.getElementById("output").innerHTML = input;  // XSS 漏洞
}

// 测试硬编码密钥
const API_KEY = "sk-1234567890abcdef";  // 硬编码密钥

// 测试不安全的正则表达式
function validateInput(input) {
  return /^(a+)+$/.test(input);  // ReDoS 漏洞
}
```

CodeQL 应该能够检测到这些漏洞并生成警报。

---

## 进阶挑战

### 挑战 1：自定义 CodeQL 查询

创建自定义 CodeQL 查询文件 `.github/codeql/custom-query.ql`：

```ql
/**
 * @name Hardcoded credentials
 * @description Detect hardcoded credentials in source code
 * @kind problem
 * @problem.severity error
 * @security-severity 9.0
 * @precision high
 * @id javascript/hardcoded-credentials
 * @tags security
 *       external/cwe/cwe-798
 */

import javascript

from DataFlow::Node source, string name
where
  source = any(ConstantExpr c).flow() and
  (
    name = source.(DataFlow::PropRead).getPropertyName() and
    name.toLowerCase().matches(["%password%", "%secret%", "%key%", "%token%"])
  )
select source, "Hardcoded credential found: " + name
```

### 挑战 2：集成第三方安全工具

在工作流中集成更多安全工具：

```yaml
- name: 运行 OWASP ZAP 扫描
  uses: zaproxy/action-full-scan@v0.7.0
  with:
    target: 'https://your-app.example.com'

- name: 运行 SonarQube 扫描
  uses: sonarsource/sonarcloud-github-action@master
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
```

### 挑战 3：创建安全仪表板

使用 GitHub Pages 创建安全仪表板：

```yaml
name: Security Dashboard

on:
  schedule:
    - cron: '0 0 * * *'

jobs:
  dashboard:
    runs-on: ubuntu-latest
    
    steps:
      - name: 收集安全数据
        run: |
          # 获取 CodeQL 警报
          # 获取 Dependabot 警报
          # 生成报告
          # 部署到 GitHub Pages
```

### 挑战 4：实现安全门禁

在 PR 合并前强制通过安全检查：

```yaml
name: Security Gate

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  security-gate:
    runs-on: ubuntu-latest
    
    steps:
      - name: 检查安全警报
        run: |
          ALERTS=$(gh api repos/{owner}/{repo}/code-scanning/alerts --jq '[.[] | select(.state == "open")] | length')
          
          if [ "$ALERTS" -gt 0 ]; then
            echo "存在 $ALERTS 个未解决的安全警报"
            exit 1
          fi
```

---

## 安全扫描深度解析

### 常见安全漏洞类型

理解安全漏洞类型有助于更好地利用安全扫描工具。注入漏洞是最常见的安全问题之一，包括 SQL 注入、命令注入和 LDAP 注入等。攻击者通过构造恶意输入，使应用程序执行非预期的命令或查询。跨站脚本攻击允许攻击者在受害者的浏览器中执行恶意脚本，窃取会话凭证或进行钓鱼攻击。跨站请求伪造攻击利用已认证用户的身份，在用户不知情的情况下执行恶意操作。不安全的反序列化可能导致远程代码执行，攻击者通过构造恶意的序列化数据来触发漏洞。敏感数据泄露包括未加密存储密码、在日志中记录敏感信息或在错误消息中暴露系统细节。

### 代码扫描的工作原理

代码扫描工具通过静态分析技术来检测代码中的安全问题。模式匹配是最基本的检测方式，通过预定义的规则匹配已知的不安全代码模式。数据流分析追踪数据在程序中的流动路径，识别从不可信来源到敏感操作的数据流。控制流分析检查程序的执行路径，识别可能导致安全问题的代码分支。污点分析是一种高级技术，标记来自外部输入的数据为"污点"数据，追踪这些数据是否未经适当处理就到达了敏感操作点。语义分析理解代码的含义，能够检测逻辑层面的安全问题。

### 密钥泄露的应急响应

当检测到密钥泄露时，需要立即采取应急响应措施。第一步是立即撤销泄露的密钥，防止攻击者利用。第二步是检查泄露密钥的使用日志，确认是否已被恶意使用。第三步是生成新的密钥并更新所有使用该密钥的服务。第四步是审查代码提交历史，确认泄露的范围和时间。第五步是分析泄露原因，是人为失误还是流程缺陷。第六步是改进防护措施，例如启用推送保护、加强代码审查或使用密钥管理服务。第七步是编写事件报告，记录事件经过和改进措施，供团队学习参考。

### 依赖漏洞的风险评估

依赖漏洞的风险评估需要综合考虑多个因素。漏洞的严重程度可以通过 CVSS 评分来衡量，评分越高风险越大。漏洞的可利用性决定了攻击者利用该漏洞的难度，远程可利用的漏洞比本地漏洞更危险。漏洞的影响范围包括机密性影响、完整性影响和可用性影响。依赖的使用方式也很关键，开发依赖中的漏洞通常比生产依赖的风险低。漏洞是否已有公开的利用代码会显著增加风险。补丁的可用性决定了修复的紧迫性，已有修复版本的漏洞应当优先处理。

### 安全左移的理念与实践

安全左移是指将安全检查从开发流程的后期移到早期。传统的安全检查通常在部署前或上线后进行，发现问题时修复成本已经很高。安全左移的理念是在编码阶段就开始进行安全检查。在开发者提交代码时，集成开发环境中的安全插件可以实时检测安全问题。在代码提交时，预提交钩子可以阻止包含密钥或已知漏洞的代码被提交。在代码审查阶段，安全扫描结果可以作为审查的参考。在持续集成阶段，自动化安全扫描确保每次构建都符合安全标准。通过这种方式，安全问题能够在最早期被发现和修复，大幅降低修复成本。

### 安全合规与审计

对于企业级应用，安全合规和审计是必不可少的环节。常见的安全合规标准包括 SOC 2、ISO 27001、PCI DSS 和 GDPR 等。GitHub 的安全功能可以帮助满足这些合规要求。CodeQL 扫描结果可以作为安全审计的证据，证明代码经过了自动化安全检查。Secret Scanning 的启用记录可以证明组织采取了防止密钥泄露的措施。Dependabot 的配置和执行记录可以证明依赖管理符合安全要求。建议定期导出安全扫描报告，归档保存以备审计使用。同时建立安全事件响应流程，确保在发现安全问题时能够快速响应和处理。

### 团队安全意识培训

工具只是安全防护的一部分，团队的安全意识同样重要。建议定期组织安全培训，内容包括常见的安全漏洞类型和防范措施、安全编码的最佳实践、安全工具的使用方法和安全事件的应急响应流程。建立安全冠军机制，在每个团队中培养一名安全冠军，负责推广安全实践和协助解决安全问题。创建安全知识库，收集和整理安全相关的文档、案例和最佳实践。定期进行安全演练，模拟安全事件的发现、报告和处理过程，提升团队的应急响应能力。

### 安全度量指标

衡量安全扫描的效果需要建立一套度量指标体系。漏洞发现率衡量安全扫描工具发现的漏洞数量和类型。漏洞修复时间衡量从发现漏洞到修复完成的平均时间。误报率衡量安全扫描工具的准确性，误报率过高会影响开发效率。覆盖率衡量安全扫描覆盖的代码范围和语言类型。依赖健康度衡量项目依赖中已知漏洞的数量和严重程度。密钥泄露防护率衡量成功阻止的密钥泄露次数。建议定期统计和分析这些指标，持续改进安全扫描的配置和流程。

---

## 常见问题

### Q1：CodeQL 扫描需要多长时间？

扫描时间取决于：
- 代码库大小
- 选择的语言数量
- 查询复杂度

通常小型项目 5-10 分钟，大型项目可能需要 30 分钟以上。

### Q2：如何减少误报？

1. 使用 `paths-ignore` 排除测试文件
2. 调整查询配置
3. 在代码中使用注释标记误报
4. 使用自定义查询过滤

### Q3：Secret Scanning 检测到误报怎么办？

1. 在仓库设置中标记为 "False positive"
2. 在 `.github/secret_scanning.yml` 中排除特定模式
3. 使用环境变量替代硬编码

### Q4：Dependabot PR 如何自动测试？

1. 配置 CI 工作流在 PR 上运行
2. 使用 `pull_request` 触发器
3. 配置自动合并条件

---

## 延伸阅读

- [GitHub Advanced Security 官方文档](https://docs.github.com/en/code-security)
- [CodeQL 文档](https://codeql.github.com/docs/)
- [Dependabot 文档](https://docs.github.com/en/code-security/dependabot)
- [Secret Scanning 文档](https://docs.github.com/en/code-security/secret-scanning)

---

## 练习总结

通过本练习，你已经学会了：

1. ✅ 配置 CodeQL 进行代码扫描
2. ✅ 启用 Secret Scanning 防止密钥泄露
3. ✅ 配置 Dependabot 自动修复依赖漏洞
4. ✅ 创建综合安全扫描工作流
5. ✅ 建立安全策略和报告机制

安全扫描是现代软件开发的重要组成部分，建议将安全检查集成到 CI/CD 流程中，实现安全左移。
