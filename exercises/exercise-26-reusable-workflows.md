# 练习 26：创建可复用 GitHub Actions 工作流

## 学习目标

完成本练习后，你将能够：

- 理解可复用工作流（Reusable Workflows）的概念和优势
- 创建使用 `workflow_call` 触发的可复用工作流
- 在多个工作流中调用可复用工作流
- 传递输入参数和密钥给可复用工作流
- 使用可复用工作流的输出值
- 在组织范围内共享标准化的 CI/CD 流程

## 前置条件

- 已完成练习 1-25 的相关内容
- 拥有 GitHub 仓库且已启用 GitHub Actions
- 了解 GitHub Actions 工作流的基本语法
- 熟悉 YAML 格式

## 背景知识

### 什么是可复用工作流

可复用工作流是 GitHub Actions 提供的一种机制，允许你将工作流逻辑封装为可被其他工作流调用的独立组件。这意味着你可以：

1. **避免重复代码**：将通用的 CI/CD 逻辑写一次，在多个仓库中使用
2. **标准化流程**：确保团队或组织使用统一的构建、测试和部署流程
3. **简化维护**：更新一处即可影响所有调用方
4. **提高效率**：新项目可以快速接入成熟的 CI/CD 流程

### 可复用工作流的限制

- 一个可复用工作流最多可以调用 4 层嵌套的可复用工作流
- 可复用工作流最多可以有 10 个输入参数和 10 个密钥
- 可复用工作流最多可以有 10 个输出参数
- 环境变量不能在可复用工作流和调用方之间共享

---

## 练习步骤

### 第一部分：创建基础可复用工作流

#### 步骤 1：创建可复用工作流目录结构

首先，在你的仓库中创建必要的目录结构：

```bash
mkdir -p .github/workflows
```

#### 步骤 2：创建 Node.js 构建的可复用工作流

创建文件 `.github/workflows/reusable-node-build.yml`：

```yaml
name: Reusable Node.js Build

on:
  workflow_call:
    inputs:
      node-version:
        description: 'Node.js 版本'
        required: false
        type: string
        default: '18'
      working-directory:
        description: '工作目录'
        required: false
        type: string
        default: '.'
      run-tests:
        description: '是否运行测试'
        required: false
        type: boolean
        default: true
      build-command:
        description: '构建命令'
        required: false
        type: string
        default: 'npm run build'
    secrets:
      NPM_TOKEN:
        description: 'NPM 发布令牌'
        required: false
    outputs:
      build-artifact:
        description: '构建产物路径'
        value: ${{ jobs.build.outputs.artifact-path }}
      test-result:
        description: '测试结果'
        value: ${{ jobs.build.outputs.test-result }}

jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      artifact-path: ${{ steps.build.outputs.artifact-path }}
      test-result: ${{ steps.test.outputs.result }}

    steps:
      - name: 检出代码
        uses: actions/checkout@v4

      - name: 设置 Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ inputs.node-version }}
          cache: 'npm'
          cache-dependency-path: '${{ inputs.working-directory }}/package-lock.json'

      - name: 安装依赖
        working-directory: ${{ inputs.working-directory }}
        run: npm ci

      - name: 运行代码检查
        working-directory: ${{ inputs.working-directory }}
        run: npm run lint --if-present

      - name: 运行测试
        if: ${{ inputs.run-tests }}
        id: test
        working-directory: ${{ inputs.working-directory }}
        run: |
          npm test
          echo "result=passed" >> $GITHUB_OUTPUT

      - name: 构建项目
        id: build
        working-directory: ${{ inputs.working-directory }}
        run: |
          ${{ inputs.build-command }}
          echo "artifact-path=${{ inputs.working-directory }}/dist" >> $GITHUB_OUTPUT

      - name: 上传构建产物
        uses: actions/upload-artifact@v4
        with:
          name: build-output
          path: ${{ inputs.working-directory }}/dist
          retention-days: 7
```

#### 步骤 3：创建 Python 项目的可复用工作流

创建文件 `.github/workflows/reusable-python-build.yml`：

```yaml
name: Reusable Python Build

on:
  workflow_call:
    inputs:
      python-version:
        description: 'Python 版本'
        required: false
        type: string
        default: '3.11'
      working-directory:
        description: '工作目录'
        required: false
        type: string
        default: '.'
      test-command:
        description: '测试命令'
        required: false
        type: string
        default: 'pytest'
      lint-command:
        description: '代码检查命令'
        required: false
        type: string
        default: 'ruff check .'
    secrets:
      PYPI_TOKEN:
        description: 'PyPI 发布令牌'
        required: false

jobs:
  build-and-test:
    runs-on: ubuntu-latest

    steps:
      - name: 检出代码
        uses: actions/checkout@v4

      - name: 设置 Python
        uses: actions/setup-python@v5
        with:
          python-version: ${{ inputs.python-version }}
          cache: 'pip'
          cache-dependency-path: '${{ inputs.working-directory }}/requirements*.txt'

      - name: 安装依赖
        working-directory: ${{ inputs.working-directory }}
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt
          pip install -r requirements-dev.txt || true

      - name: 运行代码检查
        working-directory: ${{ inputs.working-directory }}
        run: ${{ inputs.lint-command }}

      - name: 运行测试
        working-directory: ${{ inputs.working-directory }}
        run: ${{ inputs.test-command }}

      - name: 构建包
        working-directory: ${{ inputs.working-directory }}
        run: |
          pip install build
          python -m build

      - name: 上传构建产物
        uses: actions/upload-artifact@v4
        with:
          name: python-dist
          path: ${{ inputs.working-directory }}/dist/
          retention-days: 7
```

### 第二部分：在工作流中调用可复用工作流

#### 步骤 4：创建调用 Node.js 可复用工作流的工作流

创建文件 `.github/workflows/ci.yml`：

```yaml
name: CI Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  # 调用可复用的 Node.js 构建工作流
  build-frontend:
    uses: ./.github/workflows/reusable-node-build.yml
    with:
      node-version: '20'
      working-directory: './frontend'
      run-tests: true
      build-command: 'npm run build:production'
    secrets:
      NPM_TOKEN: ${{ secrets.NPM_TOKEN }}

  # 调用可复用的 Node.js 构建工作流（后端）
  build-backend:
    uses: ./.github/workflows/reusable-node-build.yml
    with:
      node-version: '20'
      working-directory: './backend'
      run-tests: true
      build-command: 'npm run build'

  # 使用构建产物进行部署
  deploy:
    needs: [build-frontend, build-backend]
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'

    steps:
      - name: 显示构建信息
        run: |
          echo "前端构建路径: ${{ needs.build-frontend.outputs.build-artifact }}"
          echo "测试结果: ${{ needs.build-frontend.outputs.test-result }}"

      - name: 下载前端构建产物
        uses: actions/download-artifact@v4
        with:
          name: build-output
          path: ./deploy/frontend

      - name: 部署到服务器
        run: |
          echo "部署前端和后端..."
          # 实际部署命令
```

#### 步骤 5：创建调用 Python 可复用工作流的工作流

创建文件 `.github/workflows/python-ci.yml`：

```yaml
name: Python CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build-api:
    uses: ./.github/workflows/reusable-python-build.yml
    with:
      python-version: '3.11'
      working-directory: './api'
      test-command: 'pytest --cov=api --cov-report=xml'
      lint-command: 'ruff check . && mypy .'
    secrets:
      PYPI_TOKEN: ${{ secrets.PYPI_TOKEN }}

  build-cli:
    uses: ./.github/workflows/reusable-python-build.yml
    with:
      python-version: '3.12'
      working-directory: './cli'
      test-command: 'pytest tests/ -v'
```

### 第三部分：跨仓库使用可复用工作流

#### 步骤 6：创建组织级可复用工作流仓库

假设你有一个组织 `my-org`，创建一个专门的仓库 `shared-workflows`：

```
my-org/shared-workflows/
├── .github/
│   └── workflows/
│       ├── ci-node.yml
│       ├── ci-python.yml
│       ├── deploy-aws.yml
│       └── security-scan.yml
└── README.md
```

#### 步骤 7：在其他仓库中调用组织级可复用工作流

在其他仓库中创建 `.github/workflows/ci.yml`：

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:

jobs:
  # 调用组织级可复用工作流
  ci:
    uses: my-org/shared-workflows/.github/workflows/ci-node.yml@v1
    with:
      node-version: '20'
      run-tests: true
    secrets:
      NPM_TOKEN: ${{ secrets.NPM_TOKEN }}

  # 调用组织级部署工作流
  deploy:
    needs: ci
    if: github.ref == 'refs/heads/main'
    uses: my-org/shared-workflows/.github/workflows/deploy-aws.yml@v1
    with:
      environment: production
      region: us-east-1
    secrets:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

### 第四部分：高级用法

#### 步骤 8：创建带条件执行的可复用工作流

创建文件 `.github/workflows/reusable-deploy.yml`：

```yaml
name: Reusable Deploy

on:
  workflow_call:
    inputs:
      environment:
        description: '部署环境'
        required: true
        type: string
      region:
        description: '部署区域'
        required: false
        type: string
        default: 'us-east-1'
      dry-run:
        description: '是否为模拟运行'
        required: false
        type: boolean
        default: false
    secrets:
      DEPLOY_KEY:
        required: true
    outputs:
      deployment-url:
        description: '部署 URL'
        value: ${{ jobs.deploy.outputs.url }}

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: ${{ inputs.environment }}
    outputs:
      url: ${{ steps.deploy.outputs.url }}

    steps:
      - name: 检出代码
        uses: actions/checkout@v4

      - name: 配置部署环境
        run: |
          echo "部署到 ${{ inputs.environment }} 环境"
          echo "区域: ${{ inputs.region }}"

      - name: 执行部署
        id: deploy
        if: ${{ !inputs.dry-run }}
        run: |
          # 实际部署逻辑
          DEPLOY_URL="https://${{ inputs.environment }}.example.com"
          echo "url=$DEPLOY_URL" >> $GITHUB_OUTPUT
          echo "部署完成: $DEPLOY_URL"

      - name: 模拟部署
        if: ${{ inputs.dry-run }}
        run: echo "这是模拟运行，不执行实际部署"
```

#### 步骤 9：创建矩阵策略的可复用工作流

创建文件 `.github/workflows/reusable-multi-platform.yml`：

```yaml
name: Reusable Multi-Platform Build

on:
  workflow_call:
    inputs:
      platforms:
        description: '目标平台列表（JSON 数组）'
        required: false
        type: string
        default: '["ubuntu-latest", "windows-latest", "macos-latest"]'
      node-versions:
        description: 'Node.js 版本列表（JSON 数组）'
        required: false
        type: string
        default: '["18", "20"]'

jobs:
  build:
    runs-on: ${{ matrix.os }}
    strategy:
      fail-fast: false
      matrix:
        os: ${{ fromJson(inputs.platforms) }}
        node-version: ${{ fromJson(inputs.node-versions) }}

    steps:
      - name: 检出代码
        uses: actions/checkout@v4

      - name: 设置 Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}

      - name: 安装依赖
        run: npm ci

      - name: 运行测试
        run: npm test

      - name: 构建
        run: npm run build

      - name: 上传产物
        uses: actions/upload-artifact@v4
        with:
          name: build-${{ matrix.os }}-node${{ matrix.node-version }}
          path: dist/
```

#### 步骤 10：调用矩阵可复用工作流

创建文件 `.github/workflows/release.yml`：

```yaml
name: Release

on:
  push:
    tags:
      - 'v*'

jobs:
  # 多平台构建
  build:
    uses: ./.github/workflows/reusable-multi-platform.yml
    with:
      platforms: '["ubuntu-latest", "windows-latest", "macos-latest"]'
      node-versions: '["18", "20", "22"]'

  # 创建发布
  release:
    needs: build
    runs-on: ubuntu-latest
    permissions:
      contents: write

    steps:
      - name: 下载所有构建产物
        uses: actions/download-artifact@v4
        with:
          path: artifacts/

      - name: 创建 GitHub Release
        uses: softprops/action-gh-release@v2
        with:
          files: artifacts/**/*
          generate_release_notes: true
```

---

## 验证练习结果

### 检查清单

完成练习后，请验证以下内容：

- [ ] 可复用工作流文件已创建且语法正确
- [ ] 调用方工作流正确引用可复用工作流
- [ ] 输入参数和密钥正确传递
- [ ] 输出值可以被调用方访问
- [ ] 工作流能在 GitHub Actions 中成功运行

### 验证命令

```bash
# 验证 YAML 语法
yamllint .github/workflows/*.yml

# 使用 actionlint 检查工作流语法
actionlint .github/workflows/*.yml

# 在本地测试（需要安装 act）
act workflow_call -W .github/workflows/reusable-node-build.yml
```

---

## 进阶挑战

### 挑战 1：创建完整的 CI/CD 可复用工作流套件

创建以下可复用工作流套件：

```yaml
# reusable-lint.yml - 代码质量检查
# reusable-test.yml - 测试执行
# reusable-build.yml - 构建打包
# reusable-deploy.yml - 部署发布
# reusable-notify.yml - 通知发送
```

### 挑战 2：实现版本化的可复用工作流

使用 Git 标签管理工作流版本：

```yaml
# 在调用方使用特定版本
uses: my-org/shared-workflows/.github/workflows/ci.yml@v1.2.0

# 使用主版本标签（自动获取最新补丁）
uses: my-org/shared-workflows/.github/workflows/ci.yml@v1
```

### 挑战 3：创建动态可复用工作流

实现根据输入动态选择执行步骤的可复用工作流：

```yaml
on:
  workflow_call:
    inputs:
      build-type:
        type: string
        # 'frontend', 'backend', 'mobile', 'all'

jobs:
  build:
    steps:
      - name: 前端构建
        if: inputs.build-type == 'frontend' || inputs.build-type == 'all'
        run: npm run build:frontend

      - name: 后端构建
        if: inputs.build-type == 'backend' || inputs.build-type == 'all'
        run: npm run build:backend

      - name: 移动端构建
        if: inputs.build-type == 'mobile' || inputs.build-type == 'all'
        run: npm run build:mobile
```

### 挑战 4：实现可复用工作流的自动文档生成

创建脚本自动从可复用工作流的输入、输出和密钥生成文档：

```bash
#!/bin/bash
# generate-workflow-docs.sh

for workflow in .github/workflows/reusable-*.yml; do
  echo "## $(basename $workflow .yml)"
  echo ""
  echo "### 输入参数"
  yq '.on.workflow_call.inputs | to_entries | .[] | "- **\(.key)**: \(.value.description) (默认: \(.value.default))"' "$workflow"
  echo ""
  echo "### 密钥"
  yq '.on.workflow_call.secrets | to_entries | .[] | "- **\(.key)**: \(.value.description)"' "$workflow"
  echo ""
done
```

---

## 可复用工作流设计模式与最佳实践

### 设计模式一：分层架构模式

在大型组织中，建议采用分层架构来组织可复用工作流。底层是基础工作流，负责最基础的操作，例如代码检出、环境配置和缓存管理。中间层是领域工作流，针对特定技术栈进行封装，例如前端构建工作流、后端测试工作流或数据库迁移工作流。顶层是业务工作流，组合多个领域工作流完成完整的业务流程。这种分层方式使得每一层都可以独立演化和维护，同时保证了代码的复用性和可读性。

### 设计模式二：模板方法模式

可复用工作流可以定义一个算法的骨架，将某些步骤的实现延迟到调用方。例如，一个通用的部署工作流可以定义检出代码、安装依赖、运行测试、构建产物、部署上线的标准流程，而具体的部署步骤可以通过输入参数动态指定。这种方式既保证了流程的一致性，又允许各个项目根据自身需求进行定制化配置。

### 设计模式三：策略模式

通过输入参数选择不同的执行策略。例如，一个测试工作流可以根据输入参数选择单元测试策略、集成测试策略或端到端测试策略。每种策略对应不同的测试命令、环境配置和报告格式。调用方只需指定策略名称，工作流会自动选择对应的执行路径。

### 命名规范建议

可复用工作流的命名应当遵循清晰、一致的原则。建议使用以下命名格式：首先是用途前缀，例如 `ci` 表示持续集成、`cd` 表示持续部署、`test` 表示测试、`build` 表示构建、`deploy` 表示部署、`notify` 表示通知。其次是技术栈标识，例如 `node`、`python`、`java`、`docker` 等。最后是环境或变体标识，例如 `prod`、`staging`、`lite`、`full` 等。完整的命名示例包括 `ci-node-standard.yml`、`deploy-aws-production.yml`、`test-python-integration.yml` 等。

### 版本管理策略

对于组织级的可复用工作流，版本管理至关重要。建议使用语义化版本号，主版本号表示不兼容的重大变更，次版本号表示向后兼容的功能新增，修订号表示向后兼容的问题修正。同时，为每个主版本创建一个浮动标签，例如 `v1`、`v2`，这样调用方可以选择使用固定的精确版本或浮动的主版本。在共享工作流仓库的发布说明中详细记录每个版本的变更内容，方便团队成员了解升级的影响。

### 安全性考虑

可复用工作流的安全性不容忽视。首先，应当限制可复用工作流的访问权限，确保只有授权的人员可以修改共享工作流。其次，在传递密钥时使用 `secrets: inherit` 要格外谨慎，只传递必要的密钥。第三，对可复用工作流的输入参数进行验证，防止注入攻击。第四，定期审查和更新共享工作流，确保使用最新的 Actions 版本和安全补丁。最后，在组织层面建立工作流审查机制，所有共享工作流的变更都需要经过代码审查和安全审查。

### 性能优化技巧

优化可复用工作流的执行效率可以从多个方面入手。合理使用缓存机制可以显著减少依赖安装时间，建议缓存 npm、pip、Maven 等包管理器的缓存目录。使用矩阵策略并行执行任务，充分利用 GitHub Actions 提供的并发执行能力。合理设置超时时间，避免任务长时间挂起。使用条件执行跳过不必要的步骤，例如仅在特定文件变更时才运行相关测试。优化构建产物的大小，只上传必要的文件，减少上传和下载时间。

### 团队协作建议

在团队中推广可复用工作流需要制定明确的协作规范。建立工作流贡献指南，说明如何创建、测试和提交新的可复用工作流。设立工作流维护者角色，负责审查和合并工作流变更。定期组织工作流分享会，让团队成员了解可用的工作流及其使用方法。建立工作流使用反馈机制，收集使用过程中遇到的问题和改进建议。维护一份工作流目录文档，列出所有可用的可复用工作流及其用途、参数和使用示例。

### 监控与告警

可复用工作流的运行状况需要持续监控。建议配置工作流运行失败的邮件或即时通讯通知。定期检查工作流的运行统计，包括成功率、平均运行时间和资源消耗。设置运行时间告警，当工作流运行时间异常增长时及时通知相关人员。分析工作流的使用情况，识别使用频率最高的工作流，优先进行优化和维护。建立工作流健康度指标体系，从可靠性、性能、安全性和可维护性等维度进行评估。

### 迁移策略

将现有的重复工作流迁移到可复用工作流需要循序渐进。首先，识别重复度最高的工作流片段，优先进行抽取。其次，创建可复用工作流并在一个试点项目中验证。第三，逐步将其他项目切换到使用可复用工作流，同时保留旧的工作流作为备份。第四，验证所有项目都正常运行后，删除旧的工作流。在整个迁移过程中，保持与团队的沟通，及时解决遇到的问题。

---

## 常见问题

### Q1：可复用工作流和复合动作有什么区别？

| 特性 | 可复用工作流 | 复合动作 |
|------|------------|---------|
| 触发方式 | `workflow_call` | 在步骤中使用 `uses` |
| 运行环境 | 独立的作业 | 当前作业的步骤 |
| 可用功能 | 完整的工作流功能 | 有限的步骤功能 |
| 适用场景 | 完整的 CI/CD 流程 | 封装多个步骤 |

### Q2：如何调试可复用工作流？

1. 使用 `workflow_dispatch` 手动触发测试
2. 在工作流中添加调试输出
3. 使用 `act` 工具本地测试
4. 检查 GitHub Actions 的运行日志

### Q3：可复用工作流支持哪些触发器？

可复用工作流仅支持 `workflow_call` 作为触发器，但调用方可以使用任何触发器（push、pull_request、schedule 等）。

### Q4：如何在组织中共享可复用工作流？

1. 创建专门的共享工作流仓库
2. 将仓库设为公开或允许组织访问
3. 使用 `org/repo/.github/workflows/name.yml@ref` 格式调用
4. 使用版本标签管理工作流版本

---

## 延伸阅读

- [GitHub Actions: Reusing workflows 官方文档](https://docs.github.com/en/actions/using-workflows/reusing-workflows)
- [Sharing workflows with your organization](https://docs.github.com/en/actions/using-workflows/sharing-workflows-secrets-and-runners-with-your-organization)
- [Security hardening for GitHub Actions](https://docs.github.com/en/actions/security-guides/security-hardening-for-github-actions)

---

## 练习总结

通过本练习，你已经学会了：

1. ✅ 创建使用 `workflow_call` 触发的可复用工作流
2. ✅ 定义输入参数、密钥和输出值
3. ✅ 在多个工作流中调用可复用工作流
4. ✅ 跨仓库共享组织级可复用工作流
5. ✅ 使用高级特性如矩阵策略和条件执行

可复用工作流是实现 CI/CD 标准化和自动化的强大工具，建议在团队中推广使用以提高开发效率和代码质量。
