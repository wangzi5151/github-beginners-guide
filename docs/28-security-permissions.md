# GitHub 安全与权限管理完全指南

## 第一章：GitHub 权限模型概述

### 1.1 权限层级结构

GitHub 的权限模型采用三层架构：组织级别、团队级别、仓库级别。理解这个层级结构对于正确配置权限至关重要。每一层都有其特定的权限管理方式和应用场景。

**组织级别**：组织是 GitHub 中最高级别的权限管理单元。组织所有者拥有最高权限，可以管理所有团队、仓库和成员。组织级别的安全策略会影响组织内的所有仓库。

**团队级别**：团队是组织内的权限管理单元，用于将成员分组并分配权限。团队可以拥有自己的仓库访问权限，团队成员会继承团队的权限。

**仓库级别**：仓库是最基本的代码管理单元，每个仓库可以独立配置协作者权限。仓库级别的权限直接决定用户对代码的访问和操作能力。

### 1.2 组织级别权限详解

组织所有者（Organization Owner）拥有最高权限，可以执行以下操作：

**管理组织设置**：修改组织名称、头像、描述、网站等基本信息。配置组织的安全策略，如强制启用双因素认证、设置 IP 白名单等。

**管理团队**：创建、删除团队，管理团队成员，配置团队对仓库的访问权限。团队管理员（Team Maintainer）可以在团队内部管理成员，但不能删除团队或修改组织级设置。

**管理仓库**：创建、删除、转移仓库，配置仓库的默认设置。组织所有者可以为组织内的所有仓库设置默认的分支保护规则。

**配置安全策略**：设置组织级别的安全策略，如强制启用双因素认证、配置 SAML 单点登录、设置 IP 白名单等。这些策略会影响组织内的所有成员和仓库。

**管理计费**：查看和修改组织的订阅计划，管理支付方式，查看使用量报告。

**审计日志**：查看组织内所有活动的审计日志，包括成员加入/离开、仓库访问、权限变更等。审计日志对于安全审计和合规性检查非常重要。

### 1.3 团队级别权限详解

团队（Team）是组织内的权限管理单元，用于将成员分组并分配权限。团队可以拥有以下角色：

**团队成员（Member）**：基本团队成员，继承团队的仓库权限。成员可以查看团队信息，但不能管理团队设置或成员。

**团队维护者（Maintainer）**：可以管理团队成员和团队设置。维护者可以添加或移除成员，修改团队描述，但不能删除团队。

**团队管理员（Team Admin）**：在某些配置下，团队管理员拥有更高级别的权限，可以管理团队的仓库访问。

团队权限的继承规则：
- 团队成员继承团队对仓库的权限
- 如果用户同时属于多个团队，取最高权限
- 直接授予的仓库权限优先于团队权限

### 1.4 仓库级别权限详解

仓库协作者权限分为五个级别，每个级别拥有不同的操作能力：

**Read（读取）**：最基本的权限级别。拥有此权限的用户可以克隆仓库、查看代码、查看 Issue 和 Pull Request，但不能修改任何内容。适合需要查看代码但不需要修改的用户。

**Triage（分类）**：在 Read 权限的基础上，增加了管理 Issue 和 Pull Request 的能力。拥有此权限的用户可以关闭、重新打开 Issue，管理标签，但仍然不能修改代码。适合需要参与项目管理但不需要修改代码的用户。

**Write（写入）**：在 Triage 权限的基础上，增加了推送代码的能力。拥有此权限的用户可以创建分支、推送代码、合并 Pull Request。这是开发者最常用的权限级别。

**Maintain（维护）**：在 Write 权限的基础上，增加了管理仓库设置的能力。拥有此权限的用户可以管理分支保护规则、配置 Webhook、管理部署密钥等。适合需要管理仓库配置的高级开发者。

**Admin（管理员）**：最高级别的仓库权限。拥有此权限的用户可以执行所有操作，包括删除仓库、管理协作者权限、转移仓库所有权等。应该谨慎授予此权限。

## 第二章：仓库可见性

### 2.1 三种可见性类型

GitHub 提供三种仓库可见性类型，每种类型适用于不同的场景：

**Public（公开）**：仓库对所有人可见，任何人都可以查看代码、克隆仓库。公开仓库适合开源项目、个人作品集、公共文档等。即使是公开仓库，也应该注意不要提交敏感信息。

**Private（私有）**：仓库仅对授权用户可见，只有被添加为协作者或属于授权团队的用户才能访问。私有仓库适合商业项目、内部工具、包含敏感配置的项目等。

**Internal（内部）**：这是 GitHub Enterprise 特有的可见性类型。内部仓库对组织内的所有成员可见，但对外部用户不可见。适合公司内部共享的项目、跨团队协作文档等。

### 2.2 设置仓库可见性

```bash
# 使用 CLI 设置为私有
gh repo edit {owner}/{repo} --visibility private

# 设置为公开
gh repo edit {owner}/{repo} --visibility public

# 使用 API
gh api repos/{owner}/{repo} -X PATCH -f private=true
```

### 2.3 可见性选择建议

**适合设为 Public 的场景**：
- 开源项目，希望社区参与贡献
- 个人作品集，展示技术能力
- 文档和教程，方便他人学习
- 公共 API 和 SDK，供开发者使用

**适合设为 Private 的场景**：
- 商业项目源码，包含核心业务逻辑
- 内部工具和脚本，仅供内部使用
- 包含敏感配置的项目
- 客户项目，需要保密

**适合设为 Internal 的场景**：
- 公司内部共享项目，如内部框架、工具库
- 跨团队协作文档，如技术规范、设计文档
- 内部标准和规范，如代码规范、流程文档

### 2.4 公开仓库的安全注意事项

即使仓库是公开的，也应该注意安全问题：

```bash
# 1. 永远不要提交敏感信息
# 密码、API Key、证书、私钥等

# 2. 使用 .gitignore 排除敏感文件
echo "*.env" >> .gitignore
echo "*.pem" >> .gitignore
echo "config/secrets.yml" >> .gitignore

# 3. 使用环境变量存储敏感信息
# 在代码中使用 process.env.API_KEY

# 4. 定期检查提交历史
# 使用 git-secrets 或 trufflehog 扫描历史提交
git log --all --diff-filter=D -- "*.env"
```

## 第三章：分支保护规则

### 3.1 为什么需要分支保护

分支保护是 GitHub 提供的重要安全功能，可以防止以下问题：

**防止直接推送到重要分支**：在没有保护规则的情况下，任何拥有 Write 权限的用户都可以直接推送到 main 分支。分支保护可以强制要求通过 Pull Request 合并代码。

**防止未经审查的代码合并**：通过配置必需的代码审查，确保所有代码变更都经过至少一位其他开发者的审查。这有助于发现潜在的错误和安全漏洞。

**防止破坏构建的代码被合并**：通过配置必需的状态检查，确保所有代码变更都通过自动化测试和构建。只有通过所有检查的代码才能被合并。

**防止强制推送覆盖历史**：强制推送会覆盖 Git 历史，可能导致其他开发者的工作丢失。分支保护可以禁止强制推送。

### 3.2 配置分支保护规则

```bash
# 使用 CLI 创建分支保护规则
gh api repos/{owner}/{repo}/branches/main/protection \
  -X PUT \
  -f required_status_checks='{"strict":true,"contexts":["build","test"]}' \
  -f enforce_admins=true \
  -f required_pull_request_reviews='{"required_approving_review_count":2}' \
  -f restrictions=null

# 查看分支保护规则
gh api repos/{owner}/{repo}/branches/main/protection

# 删除分支保护规则
gh api -X DELETE repos/{owner}/{repo}/branches/main/protection
```

### 3.3 分支保护选项详解

| 选项 | 说明 | 推荐配置 |
|------|------|---------|
| Require a pull request | 要求通过 PR 合并 | ✅ 启用 |
| Require approvals | 要求审查批准 | ✅ 启用，至少 1 人 |
| Dismiss stale PR approvals | 新提交后失效旧审查 | ✅ 启用 |
| Require review from Code Owners | 要求代码所有者审查 | ✅ 启用 |
| Require status checks | 要求状态检查通过 | ✅ 启用 |
| Require branches to be up to date | 要求分支是最新的 | ✅ 启用 |
| Require conversation resolution | 要求解决所有对话 | ✅ 启用 |
| Require signed commits | 要求签名提交 | ⚠️ 按需 |
| Include administrators | 对管理员也生效 | ✅ 启用 |
| Restrict who can push | 限制推送人员 | ⚠️ 按需 |
| Allow force pushes | 允许强制推送 | ❌ 禁用 |
| Allow deletions | 允许删除分支 | ❌ 禁用 |

### 3.4 使用 Rulesets

GitHub Rulesets 是更现代的分支保护方式，提供更灵活的配置：

```bash
# 创建规则集
gh api repos/{owner}/{repo}/rulesets -X POST -f '
{
  "name": "Main Branch Protection",
  "target": "branch",
  "enforcement": "active",
  "conditions": {
    "ref_name": {
      "include": ["refs/heads/main"]
    }
  },
  "rules": [
    {
      "type": "pull_request",
      "parameters": {
        "required_approving_review_count": 2,
        "dismiss_stale_reviews_on_push": true,
        "require_code_owner_review": true
      }
    },
    {
      "type": "required_status_checks",
      "parameters": {
        "required_status_checks": [
          {"context": "build"},
          {"context": "test"}
        ],
        "strict_required_status_checks_policy": true
      }
    },
    {
      "type": "non_fast_forward"
    }
  ]
}'
```

### 3.5 CODEOWNERS 文件

CODEOWNERS 文件用于自动分配代码审查者，确保特定代码的变更由对应的专家审查：

```
# 文件位置：.github/CODEOWNERS 或 docs/CODEOWNERS

# 默认所有者
* @org/default-reviewers

# 前端代码
/src/frontend/ @org/frontend-team
*.css @org/design-team
*.js @org/frontend-team

# 后端代码
/src/backend/ @org/backend-team
*.py @org/python-experts

# 文档
/docs/ @org/docs-team
*.md @org/docs-team

# CI/CD 配置
/.github/ @org/devops-team

# 数据库相关
/db/ @org/dba-team
*.sql @org/dba-team

# 安全相关
/security/ @org/security-team
```

## 第四章：Required Reviews 和 Status Checks

### 4.1 配置必需审查

代码审查是保证代码质量的重要环节。通过配置必需审查，可以确保所有代码变更都经过审查：

```bash
# 设置最少审查人数
gh api repos/{owner}/{repo}/branches/main/protection \
  -X PUT \
  -f required_pull_request_reviews='{
    "required_approving_review_count": 2,
    "dismiss_stale_reviews": true,
    "require_code_owner_reviews": true,
    "dismissal_restrictions": {
      "users": ["admin-user"],
      "teams": ["core-team"]
    }
  }'
```

### 4.2 审查配置选项

| 选项 | 说明 |
|------|------|
| required_approving_review_count | 最少需要几人批准（1-6） |
| dismiss_stale_reviews | 新提交后是否失效旧审查 |
| require_code_owner_reviews | 是否要求 CODEOWNERS 审查 |
| dismissal_restrictions | 谁可以撤销审查 |
| pull_request_bypassers | 谁可以绕过 PR 要求 |

### 4.3 配置状态检查

状态检查（Status Checks）是 CI/CD 流水线的通过条件，确保代码变更不会破坏构建：

```bash
# 配置必需的状态检查
gh api repos/{owner}/{repo}/branches/main/protection \
  -X PUT \
  -f required_status_checks='{
    "strict": true,
    "contexts": [
      "build",
      "test",
      "lint",
      "security-scan"
    ]
  }'
```

### 4.4 状态检查最佳实践

```yaml
# .github/workflows/status-checks.yml
name: Status Checks

on:
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build
        run: make build
      - name: Test
        run: make test

  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Lint
        run: make lint

  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Security Scan
        uses: github/codeql-action/analyze@v2
```

## 第五章：GitHub Secrets 管理

在现代软件开发中，应用程序通常需要访问各种外部服务和资源，如数据库、云服务、第三方 API 等。这些访问通常需要使用敏感信息（如密码、密钥、令牌等）进行认证。将这些敏感信息直接写入代码是非常危险的做法，因为代码可能会被意外泄露或公开。GitHub Secrets 提供了一种安全的方式来存储和使用这些敏感信息。

### 5.1 Secrets 的类型

GitHub 提供多种类型的 Secrets，用于存储敏感信息。每种类型的 Secrets 有不同的作用域和用途，开发者需要根据实际需求选择合适的类型：

| 类型 | 作用域 | 用途 |
|------|--------|------|
| Repository Secrets | 单个仓库 | 仓库级别的敏感配置 |
| Environment Secrets | 特定环境 | 环境级别的敏感配置 |
| Organization Secrets | 组织级 | 跨仓库共享的配置 |
| Dependabot Secrets | Dependabot | 依赖更新使用 |

### 5.2 管理仓库 Secrets

仓库 Secrets 是最常用的 Secrets 类型，用于存储特定仓库需要的敏感信息。每个仓库可以独立配置自己的 Secrets，这些 Secrets 只在该仓库的 GitHub Actions 工作流中可用。仓库 Secrets 的管理非常简单，可以通过 GitHub CLI 或网页界面进行操作。

```bash
# 使用 CLI 添加 Secret
gh secret set API_KEY --body "your-api-key-here"

# 从文件读取
gh secret set PRIVATE_KEY --body "$(cat private-key.pem)"

# 使用环境
gh secret set DB_PASSWORD --env production --body "secure-password"

# 列出所有 Secrets
gh secret list

# 删除 Secret
gh secret delete OLD_SECRET
```

### 5.3 管理组织 Secrets

组织 Secrets 允许在组织级别共享敏感信息，避免在每个仓库中重复配置。组织 Secrets 可以配置为对所有仓库可用，或只对指定的仓库可用。这种机制特别适合需要在多个仓库中使用相同凭证的场景，如共享的部署密钥、通用的 API 令牌等。

```bash
# 添加组织 Secret
gh secret set SHARED_TOKEN --org my-org --body "shared-token"

# 为组织 Secret 配置仓库访问
gh secret set SHARED_TOKEN --org my-org \
  --repos "repo1,repo2" \
  --visibility selected

# 可见性选项：
# - all：所有仓库可用
# - private：仅私有仓库可用
# - selected：指定仓库可用
```

### 5.4 在 Actions 中使用 Secrets

在 GitHub Actions 工作流中，可以通过特殊的语法引用 Secrets。Secrets 会被自动注入到工作流的环境变量中，供步骤使用。需要注意的是，Secrets 在日志中会被自动遮盖，以防止意外泄露。但是，开发者仍然应该避免将 Secrets 打印到日志中，因为某些情况下可能会绕过这个保护机制。

```yaml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production
    steps:
      - name: Deploy to server
        env:
          API_KEY: ${{ secrets.API_KEY }}
          DB_PASSWORD: ${{ secrets.DB_PASSWORD }}
        run: |
          echo "Deploying with API key..."
          # 使用环境变量进行部署
```

### 5.5 Secrets 安全最佳实践

Secrets 的安全管理是整体安全策略的重要组成部分。不当的 Secrets 管理可能导致严重的安全事件，如数据泄露、服务被滥用等。以下是经过实践验证的 Secrets 安全最佳实践，开发者应该在日常工作中严格遵守。

**最小权限原则**：只授予必要的 Secrets 访问权限。不同的环境应该使用不同的 Secrets，避免跨环境使用。例如，生产环境的数据库密码不应该在测试环境中使用。

**定期轮换**：定期更新 Secrets 值，降低泄露风险。建议每 3-6 个月轮换一次关键 Secrets。在员工离职或怀疑 Secrets 泄露时，应该立即轮换相关 Secrets。

**审计使用**：定期检查 Secrets 的使用情况，确保没有异常访问。可以通过审计日志查看 Secrets 的访问记录，及时发现可疑行为。

**避免日志泄露**：确保 Secrets 不会被打印到日志中。在 Actions 中，不要使用 `echo` 直接输出 Secrets 值。可以使用 `::add-mask::` 命令将自定义值标记为敏感信息。

## 第六章：Environment Protection Rules

环境（Environment）是 GitHub Actions 中用于区分不同部署目标的概念。通过使用环境，开发者可以为不同的部署目标（如开发、测试、预发布、生产）配置不同的保护规则和 Secrets。环境保护规则可以帮助团队控制部署流程，确保只有经过授权和验证的代码才能部署到重要环境。

### 6.1 创建环境

环境可以在仓库设置中创建，也可以通过 API 动态创建。每个环境可以独立配置保护规则、Secrets 和变量。环境的创建和管理通常由仓库管理员或 DevOps 工程师负责。

```bash
# 使用 CLI 创建环境
gh api repos/{owner}/{repo}/environments/production \
  -X PUT \
  -f wait_timer=30 \
  -f reviewers='[{"type":"Team","id":123}]' \
  -f deployment_branch_policy='{
    "protected_branches": true,
    "custom_branch_policies": false
  }'
```

### 6.2 环境保护规则

环境保护规则定义了部署到该环境前必须满足的条件。这些规则可以帮助团队实现部署审批流程、延迟部署、分支限制等功能。通过合理配置保护规则，可以大大降低误部署和未授权部署的风险。

| 规则 | 说明 |
|------|------|
| Required reviewers | 部署需要指定人员批准 |
| Wait timer | 部署前等待指定时间（分钟） |
| Branch restrictions | 限制可以部署的分支 |
| Environment Secrets | 环境专用的 Secrets |

### 6.3 多环境配置示例

在实际项目中，通常需要配置多个环境来支持完整的软件交付流程。典型的环境包括开发环境（development）、测试环境（testing）、预发布环境（staging）和生产环境（production）。每个环境都有不同的保护级别，生产环境通常有最严格的保护规则。

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  deploy-staging:
    runs-on: ubuntu-latest
    environment:
      name: staging
      url: https://staging.example.com
    steps:
      - uses: actions/checkout@v4
      - name: Deploy to Staging
        env:
          STAGING_API_KEY: ${{ secrets.STAGING_API_KEY }}
        run: ./deploy.sh staging

  deploy-production:
    runs-on: ubuntu-latest
    needs: deploy-staging
    environment:
      name: production
      url: https://example.com
    steps:
      - uses: actions/checkout@v4
      - name: Deploy to Production
        env:
          PRODUCTION_API_KEY: ${{ secrets.PRODUCTION_API_KEY }}
        run: ./deploy.sh production
```

## 第七章：Dependabot 安全更新

在现代软件开发中，项目通常依赖大量的第三方库和框架。这些依赖可能存在安全漏洞，如果不及时修复，可能会被攻击者利用。手动检查和更新依赖是一项繁琐且容易遗漏的工作。Dependabot 是 GitHub 提供的自动化依赖更新工具，可以帮助团队及时发现和修复依赖漏洞，保持项目的安全性。

### 7.1 启用 Dependabot

Dependabot 的配置通过在仓库中创建 `.github/dependabot.yml` 文件来完成。配置文件定义了要监控的包管理器、检查频率、审查者等信息。启用 Dependabot 后，它会定期检查项目的依赖，当发现有安全更新或新版本时，会自动创建 Pull Request。

```yaml
# .github/dependabot.yml
version: 2
updates:
  # npm 依赖
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
    open-pull-requests-limit: 10
    reviewers:
      - "team-lead"
    assignees:
      - "developer"
    labels:
      - "dependencies"
      - "security"

  # Python 依赖
  - package-ecosystem: "pip"
    directory: "/"
    schedule:
      interval: "weekly"

  # GitHub Actions
  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "weekly"

  # Docker
  - package-ecosystem: "docker"
    directory: "/"
    schedule:
      interval: "weekly"
```

### 7.2 Dependabot 配置选项

Dependabot 提供了丰富的配置选项，允许开发者自定义依赖更新的行为。通过合理配置这些选项，可以让 Dependabot 更好地适应项目的具体需求。例如，可以设置检查频率、限制同时打开的 PR 数量、指定审查者和负责人等。

| 选项 | 说明 |
|------|------|
| package-ecosystem | 包管理器类型 |
| directory | 配置文件所在目录 |
| schedule.interval | 检查频率（daily/weekly/monthly） |
| open-pull-requests-limit | 最大 PR 数量 |
| reviewers | PR 审查者 |
| assignees | PR 负责人 |
| labels | PR 标签 |
| target-branch | 目标分支 |
| versioning-strategy | 版本策略 |

### 7.3 Dependabot Alerts

Dependabot Alerts 是 Dependabot 的核心功能之一。当 GitHub 的安全公告数据库中发布了新的漏洞信息时，Dependabot 会自动扫描所有使用该依赖的仓库，并为受影响的仓库创建安全警报。开发者可以在仓库的 Security 选项卡中查看所有 Dependabot Alerts，并根据建议进行修复。

```bash
# 查看 Dependabot Alerts
gh api repos/{owner}/{repo}/dependabot/alerts

# 查看特定 Alert
gh api repos/{owner}/{repo}/dependabot/alerts/{alert_number}

# 更新 Alert 状态
gh api repos/{owner}/{repo}/dependabot/alerts/{alert_number} \
  -X PATCH \
  -f state=dismissed \
  -f dismissed_reason=no_bandwidth
```

### 7.4 Dependabot 自动合并

对于低风险的依赖更新（如补丁版本更新），可以配置自动合并功能。自动合并可以减少开发者的手动操作，加快依赖更新的速度。但是，自动合并应该谨慎使用，建议只对经过充分测试的更新类型启用自动合并。

```yaml
# .github/workflows/dependabot-auto-merge.yml
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
      - name: Fetch Dependabot metadata
        id: metadata
        uses: dependabot/fetch-metadata@v1
        with:
          github-token: "${{ secrets.GITHUB_TOKEN }}"

      - name: Auto merge minor updates
        if: steps.metadata.outputs.update-type == 'version-update:semver-minor'
        run: gh pr merge --auto --squash "$PR_URL"
        env:
          PR_URL: ${{ github.event.pull_request.html_url }}
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

## 第八章：Code Scanning（代码扫描）

代码扫描是 GitHub 提供的静态代码分析功能，可以自动发现代码中的安全漏洞和代码质量问题。通过在开发流程中集成代码扫描，团队可以在代码合并到主分支之前发现并修复潜在的安全问题，大大降低安全风险。

### 8.1 什么是 Code Scanning

Code Scanning 使用 CodeQL 引擎对代码进行深度分析。CodeQL 是 GitHub 开发的语义代码分析引擎，它将代码转换为可查询的数据库，然后使用预定义的查询规则来检测各种安全漏洞和代码缺陷。CodeQL 能够检测多种类型的安全问题，包括 SQL 注入、跨站脚本（XSS）、路径遍历、硬编码凭证等。

### 8.2 启用 CodeQL

CodeQL 可以通过 GitHub Actions 工作流来启用。配置 CodeQL 扫描后，它会在每次代码推送或 Pull Request 时自动运行，分析代码中的安全问题。CodeQL 支持多种编程语言，包括 JavaScript、Python、Java、C/C++、C#、Go、Ruby 等。对于大型项目，可以配置定期扫描（如每周一次），以持续监控代码安全状况。

```yaml
# .github/workflows/codeql.yml
name: "CodeQL"

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 6 * * 1'  # 每周一 UTC 6:00

jobs:
  analyze:
    name: Analyze
    runs-on: ubuntu-latest
    permissions:
      security-events: write
      actions: read
      contents: read

    strategy:
      fail-fast: false
      matrix:
        language: ['javascript', 'python']

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Initialize CodeQL
        uses: github/codeql-action/init@v2
        with:
          languages: ${{ matrix.language }}

      - name: Autobuild
        uses: github/codeql-action/autobuild@v2

      - name: Perform CodeQL Analysis
        uses: github/codeql-action/analyze@v2
        with:
          category: "/language:${{ matrix.language }}"
```

### 8.3 CodeQL 支持的语言

CodeQL 持续扩展对不同编程语言的支持。目前，CodeQL 对主流编程语言都有良好的支持，包括前端、后端和移动端开发常用的语言。不同语言的支持程度可能有所不同，建议查阅官方文档了解最新的支持状态。

CodeQL 支持多种编程语言，包括：

| 语言 | 支持状态 |
|------|---------|
| JavaScript/TypeScript | 稳定 |
| Python | 稳定 |
| Java/Kotlin | 稳定 |
| C/C++ | 稳定 |
| C# | 稳定 |
| Go | 稳定 |
| Ruby | 稳定 |
| Swift | beta |
| Rust | beta |

### 8.4 查看扫描结果

CodeQL 扫描完成后，会在仓库的 Security 选项卡中显示扫描结果。每个安全警报都会包含详细的漏洞描述、受影响的代码位置、漏洞的严重程度，以及修复建议。开发者可以根据这些信息快速定位和修复安全问题。对于误报或已知风险，开发者可以选择关闭警报，并说明原因。

```bash
# 查看代码扫描警报
gh api repos/{owner}/{repo}/code-scanning/alerts

# 查看特定警报
gh api repos/{owner}/{repo}/code-scanning/alerts/{alert_number}

# 更新警报状态
gh api repos/{owner}/{repo}/code-scanning/alerts/{alert_number} \
  -X PATCH \
  -f state=dismissed \
  -f dismissed_reason=false_positive
```

## 第九章：Secret Scanning（密钥扫描）

密钥泄露是软件开发中常见的安全问题。开发者可能会不小心将 API 密钥、数据库密码、访问令牌等敏感信息提交到代码仓库中。一旦这些信息被泄露，攻击者可能会利用它们访问敏感数据或执行未授权操作。Secret Scanning 是 GitHub 提供的密钥泄露检测功能，可以自动扫描仓库中的敏感信息，帮助开发者及时发现和修复密钥泄露问题。

### 9.1 什么是 Secret Scanning

Secret Scanning 会自动扫描仓库中的代码和提交历史，检测已知的密钥模式。它支持检测数百种不同服务的密钥格式，包括 AWS、Azure、Google Cloud、GitHub、Slack 等。当检测到可能的密钥泄露时，GitHub 会向仓库管理员发送安全警报，并提供修复建议。

### 9.2 启用 Secret Scanning

Secret Scanning 可以在仓库设置中启用。启用后，GitHub 会自动扫描仓库中的所有代码和提交历史。对于公开仓库，Secret Scanning 默认启用。对于私有仓库，需要在仓库设置中手动启用。此外，还可以启用 Push Protection 功能，在代码推送时实时检测密钥泄露，防止敏感信息被推送到仓库中。

```bash
# 在仓库 Settings → Security → Code security and analysis
# 启用 "Secret scanning"
# 启用 "Push protection"

# 或使用 API
gh api repos/{owner}/{repo} \
  -X PATCH \
  -f security_and_analysis='{
    "secret_scanning": {
      "status": "enabled"
    },
    "secret_scanning_push_protection": {
      "status": "enabled"
    }
  }'
```

### 9.3 Secret Scanning 功能

Secret Scanning 提供了多个功能模块，帮助开发者全面保护密钥安全。这些功能相互配合，形成了完整的密钥泄露检测和防护体系。开发者应该充分利用这些功能，构建多层安全防护。

| 功能 | 说明 |
|------|------|
| Secret scanning | 扫描仓库中的已知密钥模式 |
| Push protection | 阻止包含密钥的推送 |
| Partner alerts | 与合作伙伴集成的密钥检测 |
| Custom patterns | 自定义密钥模式 |

### 9.4 自定义密钥模式

除了内置的密钥检测规则，Secret Scanning 还支持自定义密钥模式。这对于检测组织内部使用的特定密钥格式非常有用。例如，如果组织内部使用特定格式的 API 密钥，可以创建自定义模式来检测这些密钥。自定义模式使用正则表达式定义，非常灵活。

```bash
# 创建自定义密钥模式
gh api repos/{owner}/{repo}/secret-scanning/push-protection/custom-patterns \
  -X POST \
  -f '{
    "name": "Internal API Key",
    "pattern": "internal_api_key_[a-zA-Z0-9]{32}",
    "is_enabled": true
  }'
```

### 9.5 处理 Secret Scanning 警报

当 Secret Scanning 检测到密钥泄露时，会创建安全警报。开发者应该及时处理这些警报，撤销泄露的密钥，并生成新的密钥。处理密钥泄露的标准流程包括：确认泄露、撤销泄露的密钥、生成新密钥、更新使用该密钥的服务、关闭安全警报。及时处理密钥泄露可以最大限度地降低安全风险。

```bash
# 查看警报
gh api repos/{owner}/{repo}/secret-scanning/alerts

# 更新警报状态
gh api repos/{owner}/{repo}/secret-scanning/alerts/{alert_number} \
  -X PATCH \
  -f state=resolved \
  -f resolution=revoked

# 解决方案选项：
# - revoked：密钥已撤销
# - false_positive：误报
# - wont_fix：不会修复
# - used_in_tests：测试中使用
```

## 第十章：Security Advisories

安全公告（Security Advisory）是 GitHub 提供的安全漏洞披露机制。当项目中发现安全漏洞时，维护者可以创建安全公告来协调漏洞的修复和披露。安全公告提供了一个安全的空间，让维护者和安全研究人员可以私下讨论漏洞细节，直到修复方案准备好后才公开披露。

### 10.1 创建安全公告

安全公告的创建通常遵循负责任的披露流程。首先，发现漏洞的人员（可以是维护者、安全研究人员或用户）会私下报告漏洞。然后，维护者创建安全公告草案，与报告者协作开发修复方案。修复版本发布后，安全公告会被公开，向社区披露漏洞信息和修复方案。

```bash
# 创建安全公告草案
gh api repos/{owner}/{repo}/security-advisories \
  -X POST \
  -f '{
    "summary": "SQL Injection vulnerability",
    "description": "A SQL injection vulnerability was found in the login endpoint.",
    "severity": "high",
    "vulnerabilities": [
      {
        "package": {
          "ecosystem": "npm",
          "name": "example-package"
        },
        "vulnerable_version_range": "< 1.2.3",
        "patched_versions": ">= 1.2.3"
      }
    ]
  }'
```

### 10.2 安全公告流程

安全公告的发布是一个严谨的过程，需要协调多方利益相关者。以下是典型的安全公告发布流程，每个步骤都需要仔细执行，以确保漏洞得到妥善处理，同时最大限度地保护用户安全。

1. **发现漏洞**：通过代码审查、安全扫描或外部报告发现漏洞
2. **创建安全公告草案**：在 GitHub 上创建私有的安全公告草案
3. **与维护者协作修复**：与项目维护者协作开发修复方案
4. **发布修复版本**：发布包含修复的新版本
5. **发布安全公告**：公开披露漏洞信息和修复方案
6. **CVE 分配**：如果需要，申请 CVE 编号

## 第十一章：2FA 双因素认证

双因素认证（Two-Factor Authentication，简称 2FA）是保护账户安全的重要措施。传统的密码认证只依赖一个因素（用户知道的信息），而双因素认证要求用户提供两个不同类型的认证因素。这大大增加了攻击者获取账户访问权限的难度，即使密码被泄露，攻击者仍然无法访问账户。

### 11.1 启用 2FA

启用 2FA 是保护 GitHub 账户安全的第一步。GitHub 支持多种 2FA 方式，包括 TOTP 应用、SMS 短信和安全密钥。推荐使用 TOTP 应用，因为它不依赖网络连接，且安全性高于 SMS 短信。启用 2FA 后，每次登录都需要输入动态验证码，确保只有账户持有者能够访问账户。

**启用步骤**：
1. 访问 Settings → Password and authentication
2. 点击 "Enable two-factor authentication"
3. 选择认证方式（推荐使用 TOTP 应用）
4. 扫描二维码或输入密钥
5. 保存恢复代码

### 11.2 2FA 认证方式

GitHub 支持多种 2FA 认证方式，每种方式有不同的安全性和便捷性。开发者应该根据自己的安全需求和使用习惯选择合适的认证方式。对于高安全需求的账户，建议使用安全密钥（如 YubiKey）作为主要认证方式。

| 方式 | 安全性 | 便捷性 | 推荐 |
|------|--------|--------|------|
| TOTP 应用 | 高 | 高 | ✅ 推荐 |
| SMS 短信 | 中 | 高 | ⚠️ 次选 |
| 安全密钥 | 最高 | 中 | ✅ 推荐 |
| 恢复代码 | - | - | 必须保存 |

### 11.3 推荐的 TOTP 应用

TOTP（Time-based One-Time Password）是基于时间的一次性密码算法。TOTP 应用会根据当前时间和共享密钥生成动态验证码，验证码每 30 秒更新一次。以下是几款常用的 TOTP 应用，它们都支持多平台和多账户管理。

- **1Password**：跨平台密码管理器，支持 TOTP
- **Authy**：支持多设备同步的 TOTP 应用
- **Microsoft Authenticator**：微软官方 TOTP 应用
- **Google Authenticator**：谷歌官方 TOTP 应用

### 11.4 恢复代码管理

恢复代码是 2FA 丢失时的唯一恢复方式。当用户无法使用 2FA 设备（如手机丢失、更换设备等）时，可以使用恢复代码重新获得账户访问权限。恢复代码在启用 2FA 时生成，只会显示一次，必须立即保存到安全的位置。

**恢复代码的安全存储建议**：
- 将恢复代码保存到密码管理器中
- 打印恢复代码并保存在安全的物理位置
- 不要将恢复代码存储在可能被他人访问的设备上
- 不要将恢复代码存储在 GitHub 仓库中
- 定期检查恢复代码是否仍然有效

### 11.5 组织强制 2FA

组织所有者可以要求所有成员启用 2FA，这是保护组织安全的重要措施。当组织启用强制 2FA 策略后，未启用 2FA 的成员将无法访问组织资源。组织管理员应该定期检查成员的 2FA 状态，确保所有成员都已启用 2FA。对于未启用 2FA 的成员，应该及时提醒并提供帮助。

```bash
# 查看未启用 2FA 的成员
gh api orgs/{org}/members?filter=2fa_disabled

# 移除未启用 2FA 的成员
gh api -X DELETE orgs/{org}/members/{username}
```

## 第十二章：SSH Key 和 PAT 安全管理

SSH Key 和 Personal Access Token（PAT）是访问 GitHub 的两种主要认证方式。SSH Key 用于 Git 操作（如 clone、push、pull），PAT 用于 API 访问和 HTTPS 认证。正确管理这些凭证对于保护账户安全至关重要。本章将详细介绍如何安全地创建、使用和管理这些凭证。

### 12.1 SSH Key 管理

SSH Key 是一种基于公钥加密的认证方式，比密码认证更安全。SSH Key 由公钥和私钥组成，公钥上传到 GitHub，私钥保存在本地。当进行 Git 操作时，SSH 客户端会使用私钥进行认证，无需输入密码。推荐使用 Ed25519 算法生成 SSH Key，因为它比 RSA 更安全、更快。

```bash
# 生成 SSH 密钥（推荐 Ed25519）
ssh-keygen -t ed25519 -C "your_email@example.com"

# 或使用 RSA
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

# 启动 ssh-agent
eval "$(ssh-agent -s)"

# 添加密钥到 agent
ssh-add ~/.ssh/id_ed25519

# 复制公钥
cat ~/.ssh/id_ed25519.pub | pbcopy  # macOS
cat ~/.ssh/id_ed25519.pub | xclip -selection clipboard  # Linux

# 测试连接
ssh -T git@github.com
```

### 12.2 SSH Key 安全最佳实践

SSH Key 的安全管理是保护账户安全的关键环节。以下是经过实践验证的 SSH Key 安全最佳实践，开发者应该在日常工作中严格遵守。这些最佳实践可以帮助开发者避免常见的安全风险，确保 SSH Key 的安全使用。

**使用 Ed25519 算法**：Ed25519 是目前最安全的 SSH Key 算法，比 RSA 更安全、更快。建议所有新生成的 SSH Key 都使用 Ed25519 算法。

**为私钥设置密码短语**：密码短语（passphrase）是对私钥的额外保护。即使私钥文件被泄露，没有密码短语也无法使用。建议为所有私钥设置强密码短语。

**定期轮换 SSH Key**：定期更换 SSH Key 可以降低密钥泄露的风险。建议每 6-12 个月更换一次 SSH Key，或者在怀疑密钥泄露时立即更换。

**不同设备使用不同密钥**：为不同的设备生成不同的 SSH Key，便于管理和撤销。当某个设备丢失或被盗时，只需要撤销该设备的密钥，不会影响其他设备。

**使用 SSH 配置文件简化管理**：SSH 配置文件（~/.ssh/config）可以简化多密钥的管理。通过配置文件，可以为不同的 GitHub 账户指定不同的密钥。

```bash
# ~/.ssh/config
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_personal
    IdentitiesOnly yes

Host github-work
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_work
    IdentitiesOnly yes
```

### 12.3 Personal Access Token (PAT) 管理

Personal Access Token（PAT）是访问 GitHub API 的认证方式。PAT 可以替代密码用于 HTTPS 认证，也可以用于 API 访问。GitHub 提供两种类型的 PAT：经典 PAT（Classic）和细粒度 PAT（Fine-grained）。细粒度 PAT 提供更精细的权限控制，推荐使用。

**经典 PAT**：经典 PAT 提供广泛的权限范围，如 repo、workflow、admin:org 等。每个权限范围包含多个子权限，无法单独控制。经典 PAT 适用于需要广泛访问权限的场景。

**细粒度 PAT**：细粒度 PAT 允许精确控制对特定仓库的访问权限。可以为每个仓库单独配置读取或写入权限，大大提高了安全性。细粒度 PAT 适用于只需要访问特定仓库的场景。

```bash
# 创建经典 PAT
# Settings → Developer settings → Personal access tokens

# 创建细粒度 PAT（推荐）
# Settings → Developer settings → Personal access tokens → Fine-grained tokens

# 使用 PAT
export GITHUB_TOKEN="ghp_xxxxxxxxxxxx"
gh auth login --with-token <<< "$GITHUB_TOKEN"
```

### 12.4 PAT 权限范围

经典 PAT 的权限范围定义了令牌可以访问的资源类型。开发者应该根据实际需求选择最小必要的权限范围，避免授予过多的权限。以下是常用的权限范围及其说明。

| 范围 | 说明 |
|------|------|
| repo | 完整仓库访问 |
| workflow | GitHub Actions |
| admin:repo_hook | Webhook 管理 |
| read:org | 组织信息 |
| user | 用户信息 |
| gist | Gist 访问 |

### 12.5 PAT 安全最佳实践

PAT 的安全管理与 SSH Key 同样重要。不当的 PAT 管理可能导致账户被未授权访问。以下是 PAT 安全最佳实践，开发者应该在日常工作中严格遵守。

**使用细粒度 PAT**：细粒度 PAT 提供更精细的权限控制，比经典 PAT 更安全。建议优先使用细粒度 PAT，只在必要时使用经典 PAT。

**设置过期时间**：为 PAT 设置合理的过期时间，避免长期有效的令牌。过期后需要重新创建令牌，这可以降低令牌泄露的风险。

**定期轮换**：定期更换 PAT，建议每 3-6 个月轮换一次。在员工离职或怀疑令牌泄露时，应该立即撤销相关令牌。

**安全存储**：将 PAT 存储在安全的位置，如密码管理器。不要将 PAT 存储在代码、配置文件或文档中。

**监控使用**：定期检查 PAT 的使用情况，确保没有异常访问。可以通过 GitHub 的安全日志查看 PAT 的使用记录。

## 第十三章：安全最佳实践清单

安全是一个持续的过程，需要在多个层面进行防护。本章提供了一个全面的安全最佳实践清单，涵盖个人账户安全、仓库安全、组织安全、CI/CD 安全等方面。开发者和团队应该定期审查这些清单，确保安全措施得到落实。

### 13.1 个人账户安全

个人账户安全是整体安全的基础。每个开发者都应该保护好自己的账户，避免账户被未授权访问。以下是个人账户安全的最佳实践清单。

- [ ] 启用 2FA 双因素认证
- [ ] 使用强密码（12+ 字符，包含大小写字母、数字、特殊字符）
- [ ] 使用密码管理器（1Password、Bitwarden 等）
- [ ] 定期检查账户活动
- [ ] 审查已授权的应用
- [ ] 使用 SSH Key 而不是密码
- [ ] 为不同用途创建不同的 PAT
- [ ] 设置 PAT 过期时间

### 13.2 仓库安全

仓库安全是保护代码和项目的关键。每个仓库都应该配置适当的安全措施，防止未授权访问和代码泄露。以下是仓库安全的最佳实践清单。

- [ ] 启用分支保护规则
- [ ] 配置必需的代码审查
- [ ] 启用状态检查
- [ ] 使用 CODEOWNERS 文件
- [ ] 启用 Dependabot 安全更新
- [ ] 启用 Secret Scanning
- [ ] 启用 Code Scanning
- [ ] 配置 .gitignore 排除敏感文件
- [ ] 定期审查协作者权限
- [ ] 使用 GitHub Secrets 存储敏感信息

### 13.3 组织安全

组织安全是保护整个团队和项目的关键。组织管理员应该配置适当的安全策略，确保所有成员和仓库都符合安全要求。以下是组织安全的最佳实践清单。

- [ ] 强制成员启用 2FA
- [ ] 配置 SAML SSO（Enterprise）
- [ ] 使用团队管理权限
- [ ] 定期审查组织成员
- [ ] 配置 IP 白名单（Enterprise）
- [ ] 启用审计日志
- [ ] 配置安全策略（SECURITY.md）
- [ ] 使用 Environment Protection Rules
- [ ] 配置 Dependabot 自动合并策略

### 13.4 CI/CD 安全

CI/CD 安全是保护自动化流程的关键。不当的 CI/CD 配置可能导致敏感信息泄露或未授权的部署。以下是 CI/CD 安全的最佳实践清单。

- [ ] 使用最小权限的 GITHUB_TOKEN
- [ ] 配置 Environment Protection Rules
- [ ] 审查第三方 Actions
- [ ] 使用 Actions 锁定版本
- [ ] 不要在日志中打印 Secrets
- [ ] 使用 OIDC 连接云服务
- [ ] 定期轮换 Secrets
- [ ] 配置 Actions 运行超时

### 13.5 SECURITY.md 模板

SECURITY.md 文件是项目的安全政策文档，用于告知用户和贡献者如何报告安全漏洞。每个开源项目都应该创建 SECURITY.md 文件，定义漏洞报告流程和响应时间。以下是一个 SECURITY.md 模板，开发者可以根据项目需求进行修改。

```markdown
# Security Policy

## Supported Versions

| Version | Supported          |
| ------- | ------------------ |
| 5.1.x   | ✅ Yes |
| 5.0.x   | ❌ No |
| 4.0.x   | ✅ Yes |
| < 4.0   | ❌ No |

## Reporting a Vulnerability

Please report security vulnerabilities to security@example.com.

**Do not** open a public GitHub issue for security vulnerabilities.

### What to include:
- Description of the vulnerability
- Steps to reproduce
- Potential impact
- Suggested fix (if any)

### Response timeline:
- Initial response: 48 hours
- Status update: 7 days
- Fix release: 30 days
```

### 13.6 安全工具推荐

除了 GitHub 内置的安全功能，还有许多第三方安全工具可以帮助团队提升代码安全性。这些工具可以集成到 CI/CD 流程中，实现自动化的安全检测。以下是常用的安全工具推荐，开发者可以根据项目需求选择合适的工具。

| 工具 | 用途 | 集成方式 |
|------|------|---------|
| CodeQL | 静态代码分析 | GitHub Actions |
| Dependabot | 依赖安全更新 | 原生支持 |
| Trivy | 容器安全扫描 | GitHub Actions |
| Snyk | 依赖和容器扫描 | GitHub App |
| SonarQube | 代码质量分析 | GitHub Actions |
| GitLeaks | 密钥泄露检测 | GitHub Actions |
| Semgrep | 自定义规则扫描 | GitHub Actions |

通过本指南，你应该已经掌握了 GitHub 安全与权限管理的完整知识。从个人账户安全到组织级安全策略，从代码扫描到密钥管理，GitHub 提供了全面的安全工具链。记住，安全是一个持续的过程，需要定期审查和更新安全配置。
