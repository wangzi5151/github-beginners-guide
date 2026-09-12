# GitHub 团队协作完全指南

> 本指南面向中国开发团队，系统讲解 GitHub 团队协作的模型、流程、工具与最佳实践，帮助团队建立高效、规范、安全的协作体系。

---

## 目录

- [一、GitHub 团队协作模型](#一github-团队协作模型)
- [二、分支策略与命名规范](#二分支策略与命名规范)
- [三、Pull Request 工作流最佳实践](#三pull-request-工作流最佳实践)
- [四、Code Review 规范与流程](#四code-review-规范与流程)
- [五、CODEOWNERS 文件配置](#五codeowners-文件配置)
- [六、保护分支规则（Branch Protection Rules）](#六保护分支规则branch-protection-rules)
- [七、GitHub Rulesets 详解](#七github-rulesets-详解)
- [八、团队权限管理（角色与权限）](#八团队权限管理角色与权限)
- [九、Issue 与 Project 管理规范](#九issue-与-project-管理规范)
- [十、团队沟通最佳实践](#十团队沟通最佳实践)
- [十一、多仓库协作](#十一多仓库协作)
- [十二、团队 CI/CD 规范](#十二团队-cicd-规范)
- [十三、知识管理与文档协作](#十三知识管理与文档协作)
- [十四、远程团队协作经验](#十四远程团队协作经验)
- [十五、中国开发团队使用 GitHub 的最佳实践](#十五中国开发团队使用-github-的最佳实践)

---

## 一、GitHub 团队协作模型

选择合适的协作模型是团队高效开发的基础。以下是五种主流模型的详细对比。

### 1.1 集中式工作流（Centralized Workflow）

这是最简单的协作方式，所有人直接在 `main` 分支上工作，适合非常小的团队或个人学习项目。

```
开发者 A ──push──→ main
开发者 B ──push──→ main
开发者 C ──push──→ main
```

**优点**：流程简单，学习成本低。

**缺点**：容易产生冲突，无法并行开发，不适合正式项目。

### 1.2 功能分支工作流（Feature Branch Workflow）

每个新功能在独立分支上开发，完成后通过 Pull Request 合并回主分支。这是目前最广泛使用的工作流。

```
main ─────●─────────────●─────────────●──→
           \           /             /
feature-A   ●────●────●             /
                                   /
main ─────●──────────────●────────●──→
           \            /
feature-B   ●────●────●
```

**工作流程**：

```bash
# 1. 从 main 创建功能分支
git checkout main
git pull origin main
git checkout -b feature/user-login

# 2. 开发并提交
git add .
git commit -m "feat(auth): 实现用户登录功能"

# 3. 推送并创建 PR
git push origin feature/user-login
# 在 GitHub 上创建 Pull Request

# 4. Code Review 通过后合并
# 在 GitHub 上点击 "Merge pull request"
```

**适用场景**：中小型团队、日常功能开发。

### 1.3 Git Flow 工作流

Git Flow 是一种严格的分支管理模型，由 Vincent Driessen 提出，适合有明确版本发布周期的项目。

```
main ─────●─────────────────────●──────────────────→ (生产分支)
           \                   ↑
            \                 / merge
             \               /
hotfix/*       ●────●───────●
             ↑
release/*  ●────●────●──→ (发布准备分支)
             ↑        ↑
develop ────●────●────●────●────●────●──→ (开发分支)
              \      ↑  \      ↑
feature/*      ●──●──●   ●──●──●
```

**分支说明**：

| 分支 | 用途 | 生命周期 |
|------|------|----------|
| `main` | 生产环境代码，只接受 release 和 hotfix 合并 | 永久 |
| `develop` | 开发主分支，集成所有功能 | 永久 |
| `feature/*` | 新功能开发 | 临时 |
| `release/*` | 版本发布准备 | 临时 |
| `hotfix/*` | 生产环境紧急修复 | 临时 |

**典型流程**：

```bash
# 开始新功能
git checkout develop
git checkout -b feature/payment

# 完成功能后合并回 develop
git checkout develop
git merge --no-ff feature/payment

# 准备发布
git checkout develop
git checkout -b release/v2.0.0
# 修复发布相关问题...

# 发布完成后合并到 main 和 develop
git checkout main
git merge --no-ff release/v2.0.0
git tag -a v2.0.0 -m "Release v2.0.0"
git checkout develop
git merge --no-ff release/v2.0.0

# 紧急修复
git checkout main
git checkout -b hotfix/fix-security
# 修复后合并回 main 和 develop
```

**适用场景**：有固定发布周期的企业项目、需要同时维护多个版本的产品。

### 1.4 GitHub Flow 工作流

GitHub Flow 是 GitHub 官方推荐的轻量级工作流，比 Git Flow 简单得多，强调持续部署。

```
main ─────●────●────●────●────●────●──→ (始终可部署)
           \   ↑  \   ↑  \   ↑
            PR     PR     PR
           /      /      /
feature  ●──●  ●──●  ●──●
```

**核心原则**：

1. `main` 分支上的代码始终可部署
2. 所有改动通过功能分支 + PR
3. PR 必须通过 CI 检查和 Code Review
4. 合并后立即部署

**适用场景**：SaaS 产品、持续部署项目、中小团队。

### 1.5 Trunk-Based Development（主干开发）

主干开发强调所有开发者频繁地向主干分支（通常是 `main`）提交小批量的改动，是 Google、Facebook 等大厂采用的模式。

```
main ──●──●──●──●──●──●──●──●──●──●──→
        ↑  ↑  ↑  ↑  ↑  ↑  ↑  ↑  ↑
       短命分支（存活时间 < 1天）
```

**核心原则**：

- 分支存活时间极短（通常不超过一天）
- 频繁集成，减少合并冲突
- 依赖 Feature Flags 控制未完成功能
- 高度依赖自动化测试

```bash
# 典型流程
git checkout main
git pull
git checkout -b quick-fix-123
# 小改动...
git commit -m "fix: 修复订单计算精度问题"
git push origin quick-fix-123
# 创建 PR，快速 review，合并
```

**适用场景**：成熟的 DevOps 团队、有完善自动化测试的项目。

### 1.6 模型选择决策表

| 因素 | 集中式 | 功能分支 | Git Flow | GitHub Flow | Trunk-Based |
|------|--------|----------|----------|-------------|-------------|
| 团队规模 | 1-2人 | 3-10人 | 5-20人 | 3-15人 | 5-50+人 |
| 发布频率 | 不固定 | 按需 | 定期 | 持续 | 持续 |
| 学习成本 | 极低 | 低 | 中 | 低 | 中 |
| 并行开发 | 不支持 | 支持 | 支持 | 支持 | 支持 |
| 版本维护 | 无 | 单版本 | 多版本 | 单版本 | 单版本 |
| 自动化要求 | 低 | 中 | 中 | 中 | 高 |

---

## 二、分支策略与命名规范

### 2.1 分支命名规范

统一的分支命名规范能让团队成员快速理解分支用途。推荐使用以下格式：

```
<type>/<ticket-id>-<short-description>
```

**常用类型前缀**：

| 前缀 | 用途 | 示例 |
|------|------|------|
| `feature/` | 新功能 | `feature/PROJ-123-user-login` |
| `fix/` | Bug 修复 | `fix/PROJ-456-null-pointer` |
| `hotfix/` | 生产紧急修复 | `hotfix/PROJ-789-payment-error` |
| `release/` | 发布准备 | `release/v2.1.0` |
| `docs/` | 文档更新 | `docs/PROJ-100-api-docs` |
| `refactor/` | 代码重构 | `refactor/PROJ-200-auth-module` |
| `test/` | 测试相关 | `test/PROJ-300-e2e-tests` |
| `chore/` | 构建/工具 | `chore/PROJ-400-update-deps` |

**命名规则**：

- 使用小写字母和连字符（kebab-case）
- 避免使用特殊字符和空格
- 包含 Issue/Task 编号以便追踪
- 描述简洁明了

```bash
# 好的命名
feature/SHOP-123-add-cart
fix/SHOP-456-fix-login-error
hotfix/SHOP-789-payment-timeout

# 不好的命名
feature/NewFeature          # 没有 ticket 编号
fix/bug                     # 描述太模糊
FEATURE/SHOP-123            # 不要用大写
feature_add_cart            # 用下划线而非连字符
```

### 2.2 提交信息规范（Conventional Commits）

提交信息是代码历史的重要组成部分。采用 Conventional Commits 规范可以让提交历史清晰可读。

**格式模板**：

```
<type>(<scope>): <subject>

<body>

<footer>
```

**类型说明**：

| 类型 | 说明 | 示例 |
|------|------|------|
| `feat` | 新功能 | `feat(auth): 添加微信登录` |
| `fix` | Bug 修复 | `fix(api): 修复分页参数解析错误` |
| `docs` | 文档变更 | `docs(readme): 更新部署说明` |
| `style` | 代码格式（不影响逻辑） | `style: 统一缩进为 4 空格` |
| `refactor` | 重构（既不是新功能也不是修复） | `refactor(db): 优化查询性能` |
| `perf` | 性能优化 | `perf(render): 减少首屏加载时间` |
| `test` | 测试相关 | `test(auth): 添加登录单元测试` |
| `build` | 构建系统或外部依赖 | `build: 升级 webpack 到 v5` |
| `ci` | CI 配置变更 | `ci: 添加 GitHub Actions 工作流` |
| `chore` | 其他杂项 | `chore: 清理无用文件` |
| `revert` | 回滚 | `revert: 回滚 feat(auth) 的改动` |

**实际示例**：

```
feat(user): 实现用户注册功能

- 添加邮箱验证
- 实现密码强度校验
- 集成短信验证码服务

Closes #123
Refs #124
```

```
fix(payment): 修复微信支付回调签名验证失败

微信支付 V3 版本的签名验证逻辑存在大小写问题，
导致部分回调请求被错误拒绝。

Fixes #456
```

**工具推荐**：

```bash
# 使用 commitizen 辅助提交
npm install -g commitizen cz-conventional-changelog
echo '{ "path": "cz-conventional-changelog" }' > ~/.czrc

# 之后使用 git cz 代替 git commit
git cz
```

---

## 三、Pull Request 工作流最佳实践

### 3.1 创建高质量的 Pull Request

一个好的 PR 描述能显著提高 Code Review 的效率。

**PR 模板示例**（`.github/pull_request_template.md`）：

```markdown
## 变更描述

简要描述本次变更的内容和目的。

## 变更类型

- [ ] 新功能 (feat)
- [ ] Bug 修复 (fix)
- [ ] 文档更新 (docs)
- [ ] 代码重构 (refactor)
- [ ] 性能优化 (perf)
- [ ] 测试相关 (test)
- [ ] 其他 (chore)

## 关联 Issue

Closes #123
Refs #456

## 变更详情

- 变更点 1
- 变更点 2
- 变更点 3

## 测试说明

描述如何测试本次变更：

1. 步骤一
2. 步骤二
3. 预期结果

## 截图/录屏

（如有 UI 变更，请附截图或录屏）

## 自检清单

- [ ] 代码符合团队编码规范
- [ ] 已添加/更新单元测试
- [ ] 已更新相关文档
- [ ] 无新增 lint 警告或错误
- [ ] 本地测试通过
```

### 3.2 PR 大小控制

保持 PR 小而专注是提高 Review 效率的关键。

| PR 大小 | 代码行数 | Review 时间 | 建议 |
|---------|----------|-------------|------|
| 微型 | < 50 行 | < 15 分钟 | 理想大小 |
| 小型 | 50-200 行 | 15-30 分钟 | 推荐 |
| 中型 | 200-500 行 | 30-60 分钟 | 可接受 |
| 大型 | 500-1000 行 | 1-2 小时 | 考虑拆分 |
| 超大 | > 1000 行 | > 2 小时 | 必须拆分 |

**拆分策略**：

```
大型功能 PR → 拆分为多个小 PR：

PR 1: 数据库模型和迁移
PR 2: API 接口实现
PR 3: 前端组件开发
PR 4: 集成测试
PR 5: 文档更新
```

### 3.3 PR 工作流程

```
创建分支 → 开发 → 自测 → 推送 → 创建 PR
                                      ↓
                               CI 自动检查
                                      ↓
                            ┌── 通过 ←─┤
                            ↓          │
                        Code Review    失败
                            ↓          ↓
                     ┌── 批准 ←─┤   修复问题
                     ↓          │      ↓
                   合并         请求修改 ──→ 修改代码
                     ↓                          ↓
                  部署/发布                  重新提交
```

**具体操作**：

```bash
# 1. 确保分支是最新的
git checkout main
git pull origin main
git checkout feature/my-feature
git rebase main

# 2. 运行本地检查
npm run lint
npm run test
npm run build

# 3. 推送并创建 PR
git push origin feature/my-feature

# 4. 根据 Review 意见修改
git add .
git commit -m "fix: 根据 review 意见修改"
git push origin feature/my-feature

# 5. 合并后清理
git checkout main
git pull origin main
git branch -d feature/my-feature
git push origin --delete feature/my-feature
```

---

## 四、Code Review 规范与流程

### 4.1 Code Review 的价值

Code Review 不仅是找 Bug，更是知识共享、团队成长和代码质量保障的核心手段。

**Code Review 的核心目标**：

- 发现潜在的逻辑错误和安全漏洞
- 确保代码符合团队规范
- 促进知识在团队中传播
- 提高代码的可维护性
- 帮助初级开发者成长

### 4.2 Reviewer 指南

**Review 的重点检查项**：

```
□ 功能正确性
  - 代码是否实现了需求
  - 边界条件是否处理
  - 错误处理是否完善

□ 代码质量
  - 命名是否清晰
  - 函数是否职责单一
  - 是否有重复代码
  - 是否有魔法数字/字符串

□ 性能考虑
  - 是否有 N+1 查询
  - 是否有不必要的循环
  - 内存泄漏风险

□ 安全性
  - 输入是否做了校验
  - 是否存在注入风险
  - 敏感信息是否暴露

□ 测试覆盖
  - 是否有单元测试
  - 测试用例是否充分
  - 边界条件是否覆盖

□ 可维护性
  - 代码是否易于理解
  - 是否有充分的注释
  - 是否需要更新文档
```

**Review 评论规范**：

```markdown
# 评论前缀约定

[必须修改] 存在 Bug 或严重问题，必须修复
[建议修改] 可以改进的地方，强烈建议修复
[疑问] 需要作者解释或说明
[赞] 代码写得好，值得学习
[非阻塞] 小建议，不阻塞合并
```

**示例评论**：

```markdown
[必须修改] 这里存在 SQL 注入风险，用户输入直接拼接到 SQL 语句中。
建议使用参数化查询：
```python
cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))
```

[建议修改] 这个函数有 200 行，建议拆分为更小的函数以提高可读性。

[疑问] 为什么这里选择使用双重检查锁而不是 synchronized？

[赞] 这个错误处理逻辑写得很优雅！
```

### 4.3 作者指南

**提交 Review 前的自检**：

1. 自己先 Review 一遍代码
2. 确保 CI 检查全部通过
3. 提供清晰的 PR 描述
4. 标注需要特别关注的地方
5. 控制 PR 大小在合理范围内

**对待 Review 意见的态度**：

- 保持开放心态，对事不对人
- 不理解的意见先沟通，不要直接忽略
- 同意的修改及时处理
- 不同意的意见给出合理解释

### 4.4 Code Review 流程

```
1. 作者提交 PR
       ↓
2. CI 自动检查（lint、test、build）
       ↓
   ┌── 通过 ←──┐
   ↓           │
3. 分配 Reviewer  失败 → 作者修复
   ↓
4. Reviewer 审查代码
   ↓
   ┌── 批准 ──────────┐
   │                   │
   ├── 请求修改 ──→ 作者修复 → 回到步骤 3
   │                   │
   └── 评论讨论 ──→ 达成共识
   ↓
5. 获得足够批准（通常 1-2 人）
   ↓
6. 合并 PR
```

---

## 五、CODEOWNERS 文件配置

CODEOWNERS 文件用于自动指定代码的负责人，当这些代码被修改时，系统会自动请求对应负责人进行 Review。

### 5.1 基本配置

在仓库根目录或 `.github/` 目录下创建 `CODEOWNERS` 文件：

```bash
# CODEOWNERS 文件位置（按优先级）：
# 1. 根目录 /CODEOWNERS
# 2. docs/CODEOWNERS
# 3. .github/CODEOWNERS
```

### 5.2 配置语法

```bash
# 全局所有者 - 对所有文件生效
*                       @team-lead @senior-dev

# 前端代码由前端团队负责
/src/frontend/          @org/frontend-team
*.css                   @org/frontend-team
*.html                  @org/frontend-team
*.js                    @org/frontend-team
*.tsx                   @org/frontend-team
*.ts                    @org/frontend-team

# 后端代码由后端团队负责
/src/backend/           @org/backend-team
*.py                    @org/backend-team
*.java                  @org/backend-team

# 数据库相关由 DBA 团队负责
*.sql                   @org/dba-team
/db/migrations/         @org/dba-team

# API 文档由后端团队和文档团队共同负责
/docs/api/              @org/backend-team @org/docs-team

# CI/CD 配置由 DevOps 团队负责
/.github/               @org/devops-team
/Dockerfile             @org/devops-team
/docker-compose.yml     @org/devops-team
/Jenkinsfile            @org/devops-team

# 安全相关文件需要安全团队审批
/src/auth/              @org/security-team
/src/crypto/            @org/security-team
*.pem                   @org/security-team

# 依赖文件变更需要架构师审批
/package.json           @org/architect
/requirements.txt       @org/architect
/pom.xml                @org/architect

# 特定个人负责的文件
/README.md              @zhangsan
/CONTRIBUTING.md        @zhangsan
/LICENSE                @zhangsan
```

### 5.3 CODEOWNERS 最佳实践

1. **保持更新**：代码职责变更时及时更新 CODEOWNERS
2. **避免过多全局审批人**：每个文件只需 1-2 个审批人
3. **使用团队而非个人**：便于人员变动时的维护
4. **结合分支保护规则**：要求 CODEOWNERS 审批后才能合并

---

## 六、保护分支规则（Branch Protection Rules）

分支保护规则可以强制执行特定的工作流标准，确保代码质量。

### 6.1 配置路径

```
Settings → Branches → Add rule
```

### 6.2 核心保护选项

**`main` 分支推荐配置**：

```yaml
分支名称模式: main

# Pull Request 要求
☑ Require a pull request before merging
  ☑ Require approvals: 2
  ☑ Dismiss stale pull request approvals when new commits are pushed
  ☑ Require review from Code Owners

# 状态检查要求
☑ Require status checks to pass before merging
  ☑ Require branches to be up to date before merging
  必需检查项:
    - ci/build
    - ci/test
    - ci/lint
    - security/scan

# 合并要求
☑ Require conversation resolution before merging
☑ Require linear history (禁止 merge commit)

# 推送限制
☑ Restrict pushes that create files
☐ Allow force pushes (禁止)
☐ Allow deletions (禁止)

# 管理员
☐ Include administrators (建议勾选，让管理员也遵守规则)
```

### 6.3 不同分支的保护策略

```
main (生产分支)
├── 最严格的保护
├── 要求 2 个 Reviewer 批准
├── 要求所有 CI 检查通过
├── 要求 CODEOWNERS 审批
├── 禁止 force push
└── 禁止直接推送

develop (开发分支)
├── 中等保护
├── 要求 1 个 Reviewer 批准
├── 要求 CI 检查通过
└── 允许管理员绕过

release/* (发布分支)
├── 较严格保护
├── 要求 1 个 Reviewer 批准
├── 要求所有 CI 检查通过
└── 只允许特定人员推送
```

---

## 七、GitHub Rulesets 详解

GitHub Rulesets 是 GitHub 提供的新一代分支管理规则系统，比传统的 Branch Protection Rules 更灵活、更强大。

### 7.1 Rulesets 与传统分支保护的区别

| 特性 | 传统分支保护 | Rulesets |
|------|-------------|----------|
| 规则粒度 | 每个分支模式一套规则 | 可组合多套规则 |
| 目标范围 | 仅分支 | 分支 + Tag |
| 继承关系 | 无 | 支持规则继承和覆盖 |
| 执行模式 | 始终阻塞 | 支持旁路（bypass） |
| 灵活性 | 较低 | 高度灵活 |

### 7.2 创建 Ruleset

**路径**：`Settings → Rules → Rulesets → New ruleset`

**创建分支规则集**：

```json
{
  "name": "主分支保护规则",
  "target": "branch",
  "enforcement": "active",
  "conditions": {
    "ref_name": {
      "include": ["refs/heads/main", "refs/heads/release/*"]
    }
  },
  "rules": [
    {
      "type": "pull_request",
      "parameters": {
        "required_approving_review_count": 2,
        "dismiss_stale_reviews_on_push": true,
        "require_code_owner_review": true,
        "require_last_push_approval": true
      }
    },
    {
      "type": "required_status_checks",
      "parameters": {
        "required_status_checks": [
          { "context": "ci/build" },
          { "context": "ci/test" },
          { "context": "ci/lint" }
        ],
        "strict_required_status_checks_policy": true
      }
    },
    {
      "type": "required_signatures",
      "parameters": {}
    },
    {
      "type": "non_fast_forward",
      "parameters": {}
    },
    {
      "type": "deletion",
      "parameters": {}
    }
  ],
  "bypass_actors": [
    {
      "actor_id": 1,
      "actor_type": "OrganizationAdmin",
      "bypass_mode": "always"
    }
  ]
}
```

### 7.3 创建 Tag 规则集

```json
{
  "name": "发布标签保护",
  "target": "tag",
  "enforcement": "active",
  "conditions": {
    "ref_name": {
      "include": ["refs/tags/v*"]
    }
  },
  "rules": [
    {
      "type": "creation",
      "parameters": {}
    },
    {
      "type": "update",
      "parameters": {}
    },
    {
      "type": "required_signatures",
      "parameters": {}
    }
  ]
}
```

### 7.4 Rulesets 层级

```
Organization Rulesets (组织级)
    ↓ 继承和覆盖
Repository Rulesets (仓库级)
    ↓ 继承和覆盖
具体分支规则

优先级：仓库级 > 组织级
```

### 7.5 Rulesets 最佳实践

1. **组织级定义基础规则**：确保所有仓库的基本保护
2. **仓库级按需扩展**：针对特定仓库的特殊需求
3. **使用旁路角色**：允许管理员在紧急情况下绕过规则
4. **定期审查**：根据团队需求调整规则

---

## 八、团队权限管理（角色与权限）

### 8.1 GitHub 权限模型概述

GitHub 的权限分为两个层级：组织级和仓库级。

**组织级角色**：

| 角色 | 权限范围 |
|------|----------|
| Owner | 完全控制组织，包括删除组织 |
| Member | 可以查看组织信息，访问被授权的仓库 |

**仓库级权限**：

| 权限级别 | 说明 |
|----------|------|
| Read | 可以查看和克隆代码 |
| Triage | 可以管理 Issue 和 PR（不能推送代码） |
| Write | 可以推送代码、合并 PR |
| Maintain | 包含 Write 权限，外加管理仓库设置 |
| Admin | 完全控制仓库 |

### 8.2 团队结构设计

```
Organization (公司/组织)
├── @company/admins        → Admin 权限（基础设施团队）
├── @company/maintainers   → Maintain 权限（技术负责人）
│
├── @frontend-team         → Write 权限
│   ├── @frontend-team/senior
│   └── @frontend-team/junior
│
├── @backend-team          → Write 权限
│   ├── @backend-team/senior
│   └── @backend-team/junior
│
├── @qa-team               → Triage 权限
├── @design-team           → Read 权限
└── @contractors           → Read 权限（外包团队）
```

### 8.3 权限配置实践

**创建团队并分配权限**：

```bash
# 使用 GitHub CLI 管理团队
gh api -X POST orgs/{org}/teams \
  -f name='frontend-team' \
  -f description='前端开发团队' \
  -f privacy='closed'

# 为团队添加仓库访问权限
gh api -X PUT orgs/{org}/teams/{team_slug}/repos/{owner}/{repo} \
  -f permission='push'
```

**细粒度权限（Fine-grained Personal Access Tokens）**：

```yaml
# 为 CI/CD 创建专用 Token
Token 名称: ci-deploy-bot
仓库访问: 仅特定仓库
权限:
  - Contents: Read
  - Actions: Write
  - Deployments: Write
  - Packages: Read
过期时间: 90 天
```

### 8.4 权限管理最佳实践

1. **最小权限原则**：只授予完成工作所需的最小权限
2. **使用团队而非个人**：便于批量管理和人员变动
3. **定期审计**：每季度审查一次权限分配
4. **及时回收**：成员离职时立即移除权限
5. **启用 SSO**：企业版用户启用 SAML SSO
6. **使用 PAT 管理**：定期检查和轮换 Personal Access Tokens

---

## 九、Issue 与 Project 管理规范

### 9.1 Issue 管理规范

**Issue 模板配置**（`.github/ISSUE_TEMPLATE/bug_report.md`）：

```markdown
---
name: Bug 报告
about: 提交 Bug 报告
title: '[BUG] '
labels: bug
assignees: ''
---

## Bug 描述

简要描述 Bug 的表现。

## 复现步骤

1. 打开 '...'
2. 点击 '...'
3. 滚动到 '...'
4. 看到错误

## 预期行为

描述你期望的行为。

## 实际行为

描述实际发生的行为。

## 环境信息

- 操作系统: [如 Windows 11]
- 浏览器: [如 Chrome 120]
- Node.js 版本: [如 v18.17.0]

## 截图/日志

（附加相关截图或错误日志）

## 补充信息

（其他相关信息）
```

**Feature Request 模板**（`.github/ISSUE_TEMPLATE/feature_request.md`）：

```markdown
---
name: 功能请求
about: 提出新功能建议
title: '[FEATURE] '
labels: enhancement
assignees: ''
---

## 功能描述

简要描述你希望添加的功能。

## 使用场景

描述该功能的使用场景和价值。

## 期望方案

描述你期望的实现方式。

## 替代方案

描述你考虑过的其他替代方案。

## 补充信息

（其他相关信息）
```

### 9.2 Issue 标签体系

建立完善的标签体系有助于 Issue 的分类和优先级管理。

**推荐标签分类**：

```yaml
# 类型标签
type/bug: Bug 修复
type/feature: 新功能
type/docs: 文档
type/refactor: 重构
type/test: 测试
type/chore: 杂项

# 优先级标签
priority/critical: 紧急，立即处理
priority/high: 高优先级
priority/medium: 中优先级
priority/low: 低优先级

# 状态标签
status/needs-triage: 待分类
status/accepted: 已接受
status/in-progress: 进行中
status/blocked: 被阻塞
status/wontfix: 不修复
status/duplicate: 重复

# 模块标签
module/auth: 认证模块
module/payment: 支付模块
module/user: 用户模块
module/api: API 模块

# 难度标签
difficulty/easy: 简单
difficulty/medium: 中等
difficulty/hard: 困难

# 其他标签
good-first-issue: 适合新手
help-needed: 需要帮助
breaking-change: 破坏性变更
security: 安全相关
```

### 9.3 GitHub Projects 管理

GitHub Projects 提供了类似看板的项目管理功能。

**看板列设计**：

```
Backlog → To Do → In Progress → In Review → Done
(待办)    (计划)   (进行中)     (审查中)    (完成)
```

**Sprint 管理示例**：

```yaml
Sprint 周期: 2 周
Sprint 规划: 每个 Sprint 开始时从 Backlog 选取 Issue
每日站会: 回顾看板，更新状态
Sprint 回顾: 总结完成情况，改进流程

自定义字段:
  - Sprint: Sprint 1, Sprint 2, Sprint 3...
  - Story Points: 1, 2, 3, 5, 8, 13
  - Priority: P0, P1, P2, P3
  - Due Date: 日期
  - Owner: 负责人
```

### 9.4 Issue 与 PR 的关联

```markdown
# 在 PR 描述中关联 Issue
Closes #123      # 合并 PR 后自动关闭 Issue
Fixes #456       # 同上
Resolves #789    # 同上
Refs #100        # 仅引用，不自动关闭

# 在 Commit 信息中关联
git commit -m "fix(auth): 修复登录超时问题

Fixes #123"
```

---

## 十、团队沟通最佳实践

### 10.1 GitHub Discussions

GitHub Discussions 是仓库内的讨论区，适合非代码相关的技术讨论。

**启用 Discussions**：`Settings → General → Features → Discussions`

**推荐分类**：

| 分类 | 用途 |
|------|------|
| 💬 General | 一般讨论 |
| 🙏 Q&A | 问答（支持采纳答案）|
| 💡 Ideas | 想法和建议 |
| 📢 Announcements | 公告（仅维护者可发）|
| 📖 Show and Tell | 展示作品 |
| 🗳️ Polls | 投票 |

### 10.2 PR 和 Issue 评论规范

```markdown
# 好的评论示例

## 提供上下文
"关于这个实现方式，我在项目 X 中遇到过类似场景，
当时采用了方案 B，因为..."

## 提出具体建议
"建议将这个函数拆分为两个：
1. `validateInput()` 负责输入校验
2. `processData()` 负责数据处理
这样可以提高可测试性。"

## 表达感谢
"感谢修复这个问题！测试用例覆盖得很全面。"

# 不好的评论示例

"这段代码不对"              # 太模糊，没说哪里不对
"LGTM"                     # 对于复杂改动过于简单
"你为什么要这样做？"         # 带有质疑语气，建议改为
                            # "这个实现方式的考虑是什么？"
```

### 10.3 沟通工具集成

```yaml
# 通知渠道配置

Slack/Microsoft Teams:
  - 新 PR 创建通知
  - CI 状态变更通知
  - Issue 分配通知
  - Release 发布通知

邮件通知:
  - @mention 提及
  - Review 请求
  - Issue 分配
  - 重要公告

GitHub Mobile:
  - 实时推送通知
  - 快速回复评论
  - 审批 PR
```

### 10.4 异步沟通原则

1. **写清楚**：提供足够的上下文信息
2. **用文档代替口头**：重要决策记录在 Issue 或 Wiki 中
3. **设置合理的响应时间**：非紧急问题 24 小时内回复
4. **使用表情回应**：用 👀 表示已看到，👍 表示同意
5. **善用引用**：引用具体内容进行回复

---

## 十一、多仓库协作

### 11.1 Git Submodules

Git Submodules 允许在一个仓库中引用另一个仓库的特定版本。

```bash
# 添加子模块
git submodule add https://github.com/org/shared-lib.git libs/shared-lib

# 克隆包含子模块的仓库
git clone --recurse-submodules https://github.com/org/main-project.git

# 或者分步操作
git clone https://github.com/org/main-project.git
git submodule init
git submodule update

# 更新子模块到最新版本
cd libs/shared-lib
git pull origin main
cd ../..
git add libs/shared-lib
git commit -m "chore: 更新 shared-lib 到最新版本"

# 批量更新所有子模块
git submodule update --remote --merge
```

**`.gitmodules` 文件示例**：

```ini
[submodule "libs/shared-lib"]
    path = libs/shared-lib
    url = https://github.com/org/shared-lib.git
    branch = main
[submodule "libs/ui-components"]
    path = libs/ui-components
    url = https://github.com/org/ui-components.git
    branch = main
```

**Submodules 的优缺点**：

| 优点 | 缺点 |
|------|------|
| 版本锁定精确 | 操作复杂，容易出错 |
| 代码隔离清晰 | 新人学习成本高 |
| 支持不同权限控制 | CI/CD 配置复杂 |

### 11.2 Git Subtree

Git Subtree 是 Submodules 的替代方案，将子仓库代码直接合并到主仓库中。

```bash
# 添加 subtree
git subtree add --prefix=libs/shared-lib https://github.com/org/shared-lib.git main --squash

# 更新 subtree
git subtree pull --prefix=libs/shared-lib https://github.com/org/shared-lib.git main --squash

# 推送修改回子仓库
git subtree push --prefix=libs/shared-lib https://github.com/org/shared-lib.git main
```

**Subtree vs Submodules 对比**：

| 特性 | Submodule | Subtree |
|------|-----------|---------|
| 代码存储 | 仅存储引用 | 存储完整代码 |
| 克隆速度 | 较慢（需额外拉取） | 较快 |
| 操作复杂度 | 高 | 中等 |
| 历史记录 | 分离 | 合并 |
| 学习成本 | 高 | 中等 |

### 11.3 GitHub Package Registry

GitHub Packages 可以将共享库发布为包，通过包管理器引用。

```yaml
# 发布 npm 包到 GitHub Packages
# package.json
{
  "name": "@myorg/shared-lib",
  "version": "1.0.0",
  "publishConfig": {
    "registry": "https://npm.pkg.github.com"
  }
}

# .npmrc
@myorg:registry=https://npm.pkg.github.com

# 使用包
npm install @myorg/shared-lib
```

**发布工作流**（`.github/workflows/publish.yml`）：

```yaml
name: Publish Package

on:
  release:
    types: [published]

jobs:
  publish:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '18'
          registry-url: 'https://npm.pkg.github.com'
      - run: npm ci
      - run: npm publish
        env:
          NODE_AUTH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

---

## 十二、团队 CI/CD 规范

### 12.1 CI/CD 工作流设计

```
代码提交 → Lint 检查 → 单元测试 → 构建 → 集成测试 → 部署预发 → E2E 测试 → 部署生产
```

### 12.2 标准 CI 工作流

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  lint:
    name: 代码检查
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'
      - run: npm ci
      - run: npm run lint
      - run: npm run type-check

  test:
    name: 单元测试
    runs-on: ubuntu-latest
    needs: lint
    strategy:
      matrix:
        node-version: [16, 18, 20]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'
      - run: npm ci
      - run: npm run test -- --coverage
      - uses: codecov/codecov-action@v3
        with:
          token: ${{ secrets.CODECOV_TOKEN }}

  build:
    name: 构建验证
    runs-on: ubuntu-latest
    needs: test
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'
      - run: npm ci
      - run: npm run build
      - uses: actions/upload-artifact@v4
        with:
          name: build-output
          path: dist/

  e2e:
    name: E2E 测试
    runs-on: ubuntu-latest
    needs: build
    steps:
      - uses: actions/checkout@v4
      - uses: actions/download-artifact@v4
        with:
          name: build-output
          path: dist/
      - run: npm ci
      - run: npm run test:e2e
```

### 12.3 CD 工作流

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  deploy-staging:
    name: 部署预发环境
    runs-on: ubuntu-latest
    environment:
      name: staging
      url: https://staging.example.com
    steps:
      - uses: actions/checkout@v4
      - name: 部署到预发
        run: |
          # 部署脚本
          echo "Deploying to staging..."
      - name: 运行冒烟测试
        run: |
          # 冒烟测试
          echo "Running smoke tests..."

  deploy-production:
    name: 部署生产环境
    runs-on: ubuntu-latest
    needs: deploy-staging
    environment:
      name: production
      url: https://www.example.com
    steps:
      - uses: actions/checkout@v4
      - name: 部署到生产
        run: |
          # 部署脚本
          echo "Deploying to production..."
```

### 12.4 CI/CD 最佳实践

1. **缓存依赖**：使用 `actions/cache` 或包管理器内置缓存
2. **并行执行**：独立任务并行运行以节省时间
3. **矩阵测试**：在多个环境/版本上测试
4. **安全扫描**：集成 CodeQL、Dependabot 等安全工具
5. **环境分离**：使用 Environments 管理不同环境的部署
6. **密钥管理**：使用 GitHub Secrets 存储敏感信息
7. **状态检查**：将 CI 结果设为 PR 合并的必要条件

---

## 十三、知识管理与文档协作

### 13.1 必备项目文档

```markdown
# 项目文档清单

## 基础文档
├── README.md              # 项目介绍、快速开始
├── CONTRIBUTING.md        # 贡献指南
├── CODE_OF_CONDUCT.md     # 行为准则
├── LICENSE                # 开源许可证
├── CHANGELOG.md           # 更新日志
└── SECURITY.md            # 安全策略

## 开发文档
├── docs/
│   ├── architecture.md    # 架构设计
│   ├── api/               # API 文档
│   ├── guides/            # 使用指南
│   ├── development.md     # 开发环境搭建
│   ├── deployment.md      # 部署指南
│   └── troubleshooting.md # 常见问题
```

### 13.2 README.md 结构

```markdown
# 项目名称

简短的项目描述。

## 功能特性

- 特性 1
- 特性 2
- 特性 3

## 快速开始

### 环境要求

- Node.js >= 18
- npm >= 9

### 安装

```bash
git clone https://github.com/org/project.git
cd project
npm install
```

### 运行

```bash
npm run dev
```

## 文档

- [架构设计](docs/architecture.md)
- [API 文档](docs/api/README.md)
- [部署指南](docs/deployment.md)

## 贡献

请阅读 [CONTRIBUTING.md](CONTRIBUTING.md)。

## 许可证

[MIT](LICENSE)
```

### 13.3 CHANGELOG 规范

采用 [Keep a Changelog](https://keepachangelog.com/) 格式：

```markdown
# 更新日志

本项目遵循 [语义化版本](https://semver.org/)。

## [未发布]

### 新增
- 新功能描述

### 变更
- 变更描述

### 修复
- Bug 修复描述

## [1.2.0] - 2024-01-15

### 新增
- 添加用户导出功能
- 支持微信登录

### 修复
- 修复分页计算错误
- 修复文件上传超时问题

## [1.1.0] - 2024-01-01

### 新增
- 添加数据统计功能

### 变更
- 优化首页加载速度
```

### 13.4 Wiki 使用场景

GitHub Wiki 适合维护长期的、需要多人协作的知识文档：

```yaml
Wiki 适用场景:
  - 项目架构决策记录 (ADR)
  - 开发规范文档
  - 运维手册
  - 会议纪要
  - 团队知识库

Wiki 不适用场景:
  - 需要版本管理的文档（放在仓库中）
  - API 文档（使用自动生成工具）
  - 临时笔记（使用 Issue 或 Discussion）
```

---

## 十四、远程团队协作经验

### 14.1 异步协作原则

远程团队的核心是异步协作，减少实时沟通的依赖。

```yaml
异步协作要点:
  1. 写清楚:
     - Issue/PR 描述要详细
     - 评论要提供足够上下文
     - 决策要记录原因

  2. 用文档代替口头:
     - 重要讨论在 Issue/Discussion 中进行
     - 技术方案写成 RFC 文档
     - 会议结论记录在 Wiki

  3. 设置合理的响应时间:
     - 非紧急: 24 小时内回复
     - 普通: 4 小时内回复
     - 紧急: 使用即时通讯工具

  4. 善用 GitHub 功能:
     - @mention 提及相关人员
     - 用 👀 表示已看到
     - 用 👍 表示同意
     - Draft PR 用于早期反馈
```

### 14.2 时区差异管理

```
典型场景：中国团队与海外团队协作

中国团队 (UTC+8)          海外团队 (UTC-8)
09:00 - 18:00              17:00 - 02:00 (次日)
     ↑                          ↑
     └──── 重叠时间 ────────────┘
           (17:00 - 18:00)
```

**应对策略**：

1. **交接文档**：每天结束时更新工作进度和待办事项
2. **异步代码审查**：利用 PR 评论进行异步 Review
3. **录屏说明**：复杂问题录制视频说明
4. **重叠时间利用**：将需要实时讨论的问题集中在重叠时间段
5. **轮换会议时间**：会议时间在不同时区间轮换

### 14.3 团队协作工具链

```yaml
代码协作:
  - GitHub (代码托管、PR、Issue)
  - VS Code + Live Share (远程结对编程)

即时通讯:
  - Slack / Microsoft Teams / 飞书 / 钉钉
  - 用于日常沟通和快速问题

文档协作:
  - GitHub Wiki / Notion / 飞书文档
  - 用于知识管理和文档协作

项目管理:
  - GitHub Projects / Jira / Linear
  - 用于任务管理和进度跟踪

视频会议:
  - 腾讯会议 / Zoom / Google Meet
  - 用于周会和重要讨论
```

---

## 十五、中国开发团队使用 GitHub 的最佳实践

### 15.1 网络访问优化

中国访问 GitHub 存在网络不稳定的问题，以下是几种优化方案：

**方案一：SSH 协议**

```bash
# 配置 SSH（比 HTTPS 更稳定）
git config --global url."git@github.com:".insteadOf "https://github.com/"

# SSH 配置优化 (~/.ssh/config)
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519
    TCPKeepAlive yes
    ServerAliveInterval 60
    ServerAliveCountMax 3
```

**方案二：GitHub 镜像加速**

```bash
# 使用 ghproxy 等镜像加速
git config --global url."https://ghproxy.com/https://github.com/".insteadOf "https://github.com/"

# 或使用 Gitee 镜像
# 1. Fork 仓库到 Gitee
# 2. 从 Gitee 克隆
# 3. 添加 GitHub 为 upstream
git remote add upstream https://github.com/original/repo.git
```

**方案三：使用代理**

```bash
# 配置 Git 代理
git config --global http.proxy http://127.0.0.1:7890
git config --global https.proxy http://127.0.0.1:7890

# 仅对 GitHub 使用代理
git config --global http.https://github.com.proxy http://127.0.0.1:7890
```

### 15.2 双语协作规范

对于有国际协作者的项目，建议采用双语规范：

```markdown
# Issue 模板支持双语

## Bug 描述 / Bug Description

中文描述...

English description...

## 复现步骤 / Steps to Reproduce

1. 步骤一 / Step 1
2. 步骤二 / Step 2
```

**提交信息双语**：

```
feat(auth): 添加微信登录 / Add WeChat Login

实现基于 OAuth 2.0 的微信登录功能。
Implement WeChat login based on OAuth 2.0.

Closes #123
```

### 15.3 国内替代方案对比

| 功能 | GitHub | Gitee | Gitea | GitLab |
|------|--------|-------|-------|--------|
| 托管 | ✅ | ✅ | 自建 | 自建/SaaS |
| 访问速度 | 慢 | 快 | 快 | 快 |
| CI/CD | Actions | 流水线 | Actions | CI |
| 生态 | 丰富 | 中等 | 中等 | 丰富 |
| 私有仓库 | 免费 | 免费 | 免费 | 免费 |
| 国际化 | 好 | 一般 | 好 | 好 |

**混合使用策略**：

```
国际开源项目 → GitHub
国内私有项目 → Gitee / GitLab 自建
需要快速访问 → Gitee 镜像 + GitHub 源
企业项目 → GitLab 自建 (数据安全)
```

### 15.4 合规与安全注意事项

```yaml
数据合规:
  - 敏感数据不要放在 GitHub 公开仓库
  - 使用 .gitignore 排除配置文件
  - 定期扫描仓库中的敏感信息
  - 使用 GitHub Secret Scanning

知识产权:
  - 选择合适的开源许可证
  - 注意第三方库的许可证兼容性
  - 企业代码与个人代码分离

安全实践:
  - 启用两步验证 (2FA)
  - 使用 Fine-grained PAT
  - 定期轮换密钥
  - 启用 Dependabot 安全更新
```

### 15.5 团队协作工具推荐

```yaml
代码托管与协作:
  - GitHub (国际项目首选)
  - Gitee (国内访问快)
  - Coding (腾讯旗下)

即时通讯:
  - 飞书 (功能全面，集成文档)
  - 钉钉 (企业用户多)
  - 企业微信 (与微信互通)

文档协作:
  - 飞书文档 (协作体验好)
  - 语雀 (知识管理)
  - Notion (功能强大)

项目管理:
  - GitHub Projects (与代码集成)
  - 飞书项目 (与飞书集成)
  - TAPD (腾讯敏捷开发)

CI/CD:
  - GitHub Actions (与 GitHub 集成)
  - 云效 (阿里云)
  - 腾讯云 CODING
```

---

## 总结

高效的 GitHub 团队协作需要在以下几个层面建立规范：

```
                    ┌─────────────┐
                    │   文化层面   │
                    │  开放、信任  │
                    │  知识共享    │
                    └──────┬──────┘
                           │
                    ┌──────▼──────┐
                    │   流程层面   │
                    │  分支策略    │
                    │  PR 流程     │
                    │  Code Review │
                    └──────┬──────┘
                           │
                    ┌──────▼──────┐
                    │   工具层面   │
                    │  分支保护    │
                    │  CI/CD      │
                    │  自动化      │
                    └──────┬──────┘
                           │
                    ┌──────▼──────┐
                    │   规范层面   │
                    │  命名规范    │
                    │  提交规范    │
                    │  文档规范    │
                    └─────────────┘
```

**关键要点回顾**：

1. **选择适合团队的协作模型**，不要盲目追求复杂度
2. **建立统一的分支命名和提交规范**，保持代码历史清晰
3. **PR 要小而专注**，降低 Review 成本
4. **Code Review 是学习机会**，不是挑错比赛
5. **善用 CODEOWNERS 和分支保护**，自动化执行规范
6. **文档是团队的共同财富**，持续维护和更新
7. **异步优先**，减少不必要的会议
8. **关注网络和合规**，中国特色的实践经验

---

[← 上一章：Fork 与开源贡献](22-fork-contribute.md) | [下一章：Git 工作流详解 →](24-git-workflow.md)
