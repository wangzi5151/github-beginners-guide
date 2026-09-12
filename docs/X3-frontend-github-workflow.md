# 前端开发者的 GitHub 工作流

> 本文档为前端开发者提供完整的 GitHub 工作流指南，涵盖仓库管理、CI/CD、自动化测试、性能监控、代码质量保障等核心环节，帮助前端团队建立高效规范的开发流程。

---

## 目录

1. [前端项目 GitHub 仓库结构](#1-前端项目-github-仓库结构)
2. [前端项目 CI/CD](#2-前端项目-cicd)
3. [静态网站部署方案](#3-静态网站部署方案)
4. [前端自动化测试](#4-前端自动化测试)
5. [Lighthouse CI 自动化性能测试](#5-lighthouse-ci-自动化性能测试)
6. [前端代码质量工具](#6-前端代码质量工具)
7. [依赖管理与安全](#7-依赖管理与安全)
8. [Storybook 与组件库管理](#8-storybook-与组件库管理)
9. [前端 Monorepo 管理](#9-前端-monorepo-管理)
10. [npm/pnpm 包发布流程](#10-npmpnpm-包发布流程)
11. [前端国际化（i18n）协作](#11-前端国际化i18n协作)
12. [设计稿与代码协作](#12-设计稿与代码协作)
13. [国内前端团队 GitHub 使用经验](#13-国内前端团队-github-使用经验)

---

## 1. 前端项目 GitHub 仓库结构

### 1.1 标准前端项目目录结构

一个规范的前端项目仓库应该具备清晰的目录结构，便于团队协作和自动化工具集成。以下是一个典型的 React 项目结构：

```
my-frontend-app/
├── .github/
│   ├── workflows/
│   │   ├── ci.yml                  # 持续集成流水线
│   │   ├── deploy.yml              # 部署流水线
│   │   ├── lighthouse.yml          # 性能检测流水线
│   │   └── release.yml             # 发布流水线
│   ├── ISSUE_TEMPLATE/
│   │   ├── bug_report.md           # Bug 报告模板
│   │   └── feature_request.md      # 功能需求模板
│   ├── PULL_REQUEST_TEMPLATE.md    # PR 模板
│   ├── CODEOWNERS                  # 代码所有者配置
│   └── dependabot.yml              # 依赖更新配置
├── src/
│   ├── components/                 # 通用组件
│   │   ├── Button/
│   │   │   ├── Button.tsx
│   │   │   ├── Button.test.tsx
│   │   │   ├── Button.stories.tsx
│   │   │   └── index.ts
│   │   └── index.ts
│   ├── pages/                      # 页面组件
│   ├── hooks/                      # 自定义 Hook
│   ├── utils/                      # 工具函数
│   ├── services/                   # API 服务层
│   ├── stores/                     # 状态管理
│   ├── styles/                     # 全局样式
│   ├── types/                      # TypeScript 类型定义
│   ├── locales/                    # 国际化资源
│   └── App.tsx
├── public/                         # 静态资源
├── tests/                          # 测试配置和工具
│   ├── setup.ts
│   └── mocks/
├── scripts/                        # 构建和部署脚本
├── docs/                           # 项目文档
├── .eslintrc.js                    # ESLint 配置
├── .prettierrc                     # Prettier 配置
├── .stylelintrc.js                 # Stylelint 配置
├── .lintstagedrc.js                # lint-staged 配置
├── tsconfig.json                   # TypeScript 配置
├── vite.config.ts                  # Vite 构建配置
├── package.json
├── pnpm-lock.yaml
└── README.md
```

### 1.2 `.github` 目录详解

`.github` 目录是 GitHub 项目的核心配置目录，包含工作流、模板、代码审查规则等关键配置。

**CODEOWNERS 文件示例：**

```
# 全局代码所有者
*                       @frontend-team

# 组件库由特定成员维护
/src/components/        @zhangsan @lisi

# 构建配置变更需要架构师审批
/vite.config.ts         @architect-wang
/tsconfig.json          @architect-wang
/.github/               @devops-team

# 国际化文件由翻译团队维护
/src/locales/           @i18n-team
```

**Issue 模板配置（`.github/ISSUE_TEMPLATE/config.yml`）：**

```yaml
blank_issues_enabled: false
contact_links:
  - name: 功能讨论
    url: https://github.com/orgs/your-org/discussions
    about: 请在 Discussions 中讨论新功能想法
  - name: 查看文档
    url: https://your-docs-site.com
    about: 查看项目文档获取帮助
```

### 1.3 分支管理策略

前端项目推荐采用 Git Flow 或 Trunk-Based Development：

| 策略 | 适用场景 | 分支模型 | 优点 | 缺点 |
|------|---------|---------|------|------|
| Git Flow | 大型项目、版本发布 | main/develop/feature/release/hotfix | 规范清晰、适合多版本并行 | 流程复杂、合并冲突多 |
| Trunk-Based | 持续部署、小团队 | main/feature-short-lived | 速度快、冲突少 | 需要完善的自动化测试 |
| GitHub Flow | 简单项目、快速迭代 | main/feature | 简单易学 | 不适合复杂发布流程 |

**Git Flow 示例：**

```bash
# 创建功能分支
git checkout develop
git pull origin develop
git checkout -b feature/user-login

# 开发完成后推送
git push origin feature/user-login

# 在 GitHub 上创建 PR，目标分支为 develop
# Code Review 通过后合并

# 发布流程
git checkout -b release/v1.2.0 develop
# 修复发布相关问题后
git checkout main
git merge release/v1.2.0
git tag -a v1.2.0 -m "Release v1.2.0"
git push origin main --tags
```

### 1.4 Commit 规范

前端项目应严格遵循 Conventional Commits 规范，便于生成 CHANGELOG 和自动化版本管理：

```
<type>(<scope>): <subject>

[body]

[footer]
```

**常用 type 说明：**

| type | 说明 | 示例 |
|------|------|------|
| feat | 新功能 | `feat(login): 添加微信扫码登录` |
| fix | 修复 Bug | `fix(cart): 修复数量选择器溢出问题` |
| docs | 文档更新 | `docs: 更新 API 接口文档` |
| style | 代码格式调整 | `style: 统一缩进为 2 空格` |
| refactor | 代码重构 | `refactor(hooks): 抽取 useRequest 通用逻辑` |
| perf | 性能优化 | `perf(list): 虚拟列表优化大数据渲染` |
| test | 测试相关 | `test(login): 补充登录模块单元测试` |
| chore | 构建/工具变更 | `chore: 升级 vite 至 5.0` |
| ci | CI 配置变更 | `ci: 添加 Lighthouse CI 检测` |
| revert | 回滚 | `revert: 回滚 feat(login) 微信登录` |

**使用 commitlint 自动校验：**

```bash
pnpm add -D @commitlint/cli @commitlint/config-conventional husky
```

```javascript
// commitlint.config.js
module.exports = {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'type-enum': [2, 'always', [
      'feat', 'fix', 'docs', 'style', 'refactor',
      'perf', 'test', 'chore', 'ci', 'revert'
    ]],
    'scope-enum': [2, 'always', [
      'login', 'cart', 'product', 'user', 'order',
      'payment', 'common', 'config', 'deps'
    ]],
    'subject-max-length': [2, 'always', 100],
    'body-max-line-length': [1, 'always', 200],
  }
};
```

---

## 2. 前端项目 CI/CD

### 2.1 React 项目 GitHub Actions

**完整的 React + TypeScript 项目 CI 流水线：**

```yaml
# .github/workflows/ci.yml
name: React App CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

env:
  NODE_VERSION: '20'
  PNPM_VERSION: '9'

jobs:
  lint:
    name: 代码检查
    runs-on: ubuntu-latest
    steps:
      - name: 检出代码
        uses: actions/checkout@v4

      - name: 安装 pnpm
        uses: pnpm/action-setup@v4
        with:
          version: ${{ env.PNPM_VERSION }}

      - name: 设置 Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'pnpm'

      - name: 安装依赖
        run: pnpm install --frozen-lockfile

      - name: ESLint 检查
        run: pnpm eslint . --ext .ts,.tsx --format json --output-file eslint-report.json

      - name: Prettier 检查
        run: pnpm prettier --check "src/**/*.{ts,tsx,css,scss}"

      - name: Stylelint 检查
        run: pnpm stylelint "src/**/*.{css,scss}"

      - name: TypeScript 类型检查
        run: pnpm tsc --noEmit

      - name: 上传检查报告
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: lint-reports
          path: eslint-report.json

  test:
    name: 自动化测试
    runs-on: ubuntu-latest
    needs: lint
    steps:
      - name: 检出代码
        uses: actions/checkout@v4

      - name: 安装 pnpm
        uses: pnpm/action-setup@v4
        with:
          version: ${{ env.PNPM_VERSION }}

      - name: 设置 Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'pnpm'

      - name: 安装依赖
        run: pnpm install --frozen-lockfile

      - name: 运行单元测试
        run: pnpm test -- --coverage --reporters=default --reporters=jest-junit
        env:
          JEST_JUNIT_OUTPUT_DIR: ./test-results

      - name: 上传测试结果
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: test-results
          path: test-results/

      - name: 上传覆盖率报告
        uses: actions/upload-artifact@v4
        with:
          name: coverage
          path: coverage/

      - name: 检查覆盖率阈值
        run: |
          COVERAGE=$(cat coverage/coverage-summary.json | jq '.total.lines.pct')
          echo "当前代码覆盖率: ${COVERAGE}%"
          if (( $(echo "$COVERAGE < 80" | bc -l) )); then
            echo "::warning::代码覆盖率 ${COVERAGE}% 低于阈值 80%"
          fi

  build:
    name: 构建验证
    runs-on: ubuntu-latest
    needs: test
    strategy:
      matrix:
        node-version: [18, 20, 22]
    steps:
      - name: 检出代码
        uses: actions/checkout@v4

      - name: 安装 pnpm
        uses: pnpm/action-setup@v4
        with:
          version: ${{ env.PNPM_VERSION }}

      - name: 设置 Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'pnpm'

      - name: 安装依赖
        run: pnpm install --frozen-lockfile

      - name: 构建项目
        run: pnpm build
        env:
          NODE_ENV: production

      - name: 分析构建产物大小
        run: |
          echo "## 构建产物分析" >> $GITHUB_STEP_SUMMARY
          echo "| 文件类型 | 大小 | 文件数量 |" >> $GITHUB_STEP_SUMMARY
          echo "|---------|------|---------|" >> $GITHUB_STEP_SUMMARY
          for ext in js css html json png jpg svg woff2; do
            SIZE=$(find dist -name "*.${ext}" -exec du -ch {} + 2>/dev/null | tail -1 | cut -f1)
            COUNT=$(find dist -name "*.${ext}" | wc -l)
            if [ "$COUNT" -gt 0 ]; then
              echo "| .${ext} | ${SIZE:-0} | ${COUNT} |" >> $GITHUB_STEP_SUMMARY
            fi
          done

      - name: 上传构建产物
        uses: actions/upload-artifact@v4
        with:
          name: build-${{ matrix.node-version }}
          path: dist/
          retention-days: 7
```

### 2.2 Vue 3 项目 GitHub Actions

```yaml
# .github/workflows/vue-ci.yml
name: Vue 3 App CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  ci:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: pnpm/action-setup@v4
        with:
          version: 9

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'pnpm'

      - run: pnpm install --frozen-lockfile

      - name: Vue 类型检查 (vue-tsc)
        run: pnpm vue-tsc --noEmit

      - name: ESLint
        run: pnpm eslint . --ext .vue,.ts,.tsx

      - name: 单元测试 (Vitest)
        run: pnpm vitest run --coverage
        env:
          VITE_API_BASE_URL: https://test-api.example.com

      - name: 构建
        run: pnpm build

      - name: 部署预览（PR 环境）
        if: github.event_name == 'pull_request'
        uses: amondnet/vercel-action@v25
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
          working-directory: ./
```

### 2.3 Angular 项目 GitHub Actions

```yaml
# .github/workflows/angular-ci.yml
name: Angular App CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'

      - run: npm ci

      - name: Lint
        run: npx ng lint

      - name: 单元测试 (Karma)
        run: npx ng test --watch=false --code-coverage --browsers=ChromeHeadless

      - name: E2E 测试 (Playwright)
        run: npx playwright install --with-deps chromium && npx ng e2e

      - name: 构建
        run: npx ng build --configuration=production

      - name: 上传覆盖率到 Codecov
        uses: codecov/codecov-action@v4
        with:
          directory: ./coverage
          token: ${{ secrets.CODECOV_TOKEN }}
```

### 2.4 Svelte/SvelteKit 项目 GitHub Actions

```yaml
# .github/workflows/sveltekit-ci.yml
name: SvelteKit CI

on:
  push:
    branches: [main]
  pull_request:

jobs:
  ci:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: pnpm/action-setup@v4
        with:
          version: 9

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'pnpm'

      - run: pnpm install --frozen-lockfile

      - name: Svelte Check
        run: pnpm svelte-kit sync && pnpm svelte-check

      - name: Lint
        run: pnpm lint

      - name: Test (Vitest)
        run: pnpm test -- --coverage

      - name: Build
        run: pnpm build

      - name: Adapter 验证
        run: ls -la build/
```

### 2.5 前端项目的完整 CI/CD 流水线设计

一个成熟的前端项目 CI/CD 流水线应包含以下阶段：

```
代码提交 → 代码检查 → 单元测试 → 构建验证 → E2E测试 → 预览部署 → 生产部署 → 监控
```

**各阶段职责：**

| 阶段 | 工具 | 耗时 | 失败处理 |
|------|------|------|---------|
| 代码检查 | ESLint/Prettier/Stylelint | 1-2 分钟 | 阻止合并 |
| 单元测试 | Jest/Vitest | 2-5 分钟 | 阻止合并 |
| 构建验证 | Vite/Webpack | 2-4 分钟 | 阻止合并 |
| E2E 测试 | Playwright/Cypress | 5-15 分钟 | 阻止合并（可选） |
| 预览部署 | Vercel/Netlify | 1-3 分钟 | 告警通知 |
| 生产部署 | GitHub Actions | 3-10 分钟 | 自动回滚 |
| 监控 | Sentry/Lighthouse | 持续 | 告警通知 |

---

## 3. 静态网站部署方案

### 3.1 GitHub Pages 部署

GitHub Pages 是最简单的免费静态网站托管方案，适合个人项目、文档站点和小型应用。

**基础部署配置：**

```yaml
# .github/workflows/deploy-pages.yml
name: Deploy to GitHub Pages

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: pages
  cancel-in-progress: true

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: pnpm/action-setup@v4
        with:
          version: 9

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'pnpm'

      - run: pnpm install --frozen-lockfile
      - run: pnpm build

      - name: 配置 GitHub Pages
        uses: actions/configure-pages@v4

      - name: 上传构建产物
        uses: actions/upload-pages-artifact@v3
        with:
          path: dist

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - name: 部署到 GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

**SPA 路由处理（Vue Router / React Router）：**

由于 GitHub Pages 不支持服务端路由，SPA 应用需要特殊处理：

```yaml
# 在构建后创建 404.html 处理 SPA 路由
- name: SPA 路由处理
  run: |
    cp dist/index.html dist/404.html
```

或者使用 `spa-github-pages` 方案，在 `index.html` 中添加重定向脚本。

**自定义域名配置：**

```yaml
# 在仓库 Settings > Pages 中配置自定义域名
# 或在构建步骤中自动配置
- name: 配置自定义域名
  if: github.ref == 'refs/heads/main'
  run: |
    echo "www.example.com" > dist/CNAME
```

### 3.2 Vercel 部署

Vercel 是前端项目最流行的部署平台之一，提供极佳的开发者体验。

**自动部署配置：**

```json
// vercel.json
{
  "buildCommand": "pnpm build",
  "outputDirectory": "dist",
  "framework": "vite",
  "rewrites": [
    { "source": "/api/(.*)", "destination": "https://api.example.com/$1" }
  ],
  "headers": [
    {
      "source": "/assets/(.*)",
      "headers": [
        { "key": "Cache-Control", "value": "public, max-age=31536000, immutable" }
      ]
    }
  ]
}
```

**GitHub Actions 与 Vercel 集成：**

```yaml
# .github/workflows/vercel-deploy.yml
name: Vercel Deploy

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: 安装 Vercel CLI
        run: npm install -g vercel

      - name: 拉取 Vercel 环境配置
        run: vercel pull --yes --environment=${{ github.ref == 'refs/heads/main' && 'production' || 'preview' }} --token=${{ secrets.VERCEL_TOKEN }}

      - name: 构建
        run: vercel build --token=${{ secrets.VERCEL_TOKEN }}

      - name: 部署
        id: deploy
        run: |
          if [ "${{ github.ref }}" = "refs/heads/main" ]; then
            url=$(vercel deploy --prebuilt --prod --token=${{ secrets.VERCEL_TOKEN }})
          else
            url=$(vercel deploy --prebuilt --token=${{ secrets.VERCEL_TOKEN }})
          fi
          echo "url=$url" >> $GITHUB_OUTPUT

      - name: 评论 PR 部署链接
        if: github.event_name == 'pull_request'
        uses: actions/github-script@v7
        with:
          script: |
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: `🚀 预览部署: ${{ steps.deploy.outputs.url }}`
            })
```

### 3.3 Netlify 部署

```yaml
# .github/workflows/netlify-deploy.yml
name: Netlify Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: pnpm/action-setup@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'pnpm'

      - run: pnpm install --frozen-lockfile && pnpm build

      - name: 部署到 Netlify
        uses: nwtgck/actions-netlify@v3
        with:
          publish-dir: ./dist
          production-deploy: ${{ github.ref == 'refs/heads/main' }}
          github-token: ${{ secrets.GITHUB_TOKEN }}
          deploy-message: "Deploy from GitHub Actions - ${{ github.sha }}"
          netlify-config-path: ./netlify.toml
        env:
          NETLIFY_AUTH_TOKEN: ${{ secrets.NETLIFY_AUTH_TOKEN }}
          NETLIFY_SITE_ID: ${{ secrets.NETLIFY_SITE_ID }}
```

**Netlify 配置文件：**

```toml
# netlify.toml
[build]
  command = "pnpm build"
  publish = "dist"

[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200

[[headers]]
  for = "/assets/*"
  [headers.values]
    Cache-Control = "public, max-age=31536000, immutable"

[[plugins]]
  package = "@netlify/plugin-lighthouse"
```

### 3.4 Cloudflare Pages 部署

Cloudflare Pages 提供全球 CDN 和 Workers 支持，适合对性能要求高的项目。

```yaml
# .github/workflows/cloudflare-pages.yml
name: Cloudflare Pages Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: pnpm/action-setup@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'pnpm'

      - run: pnpm install --frozen-lockfile && pnpm build

      - name: 部署到 Cloudflare Pages
        uses: cloudflare/wrangler-action@v3
        with:
          apiToken: ${{ secrets.CLOUDFLARE_API_TOKEN }}
          accountId: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
          command: pages deploy dist --project-name=my-app
```

### 3.5 部署方案对比

| 特性 | GitHub Pages | Vercel | Netlify | Cloudflare Pages |
|------|-------------|--------|---------|-----------------|
| 免费额度 | 100GB/月 | 100GB/月 | 100GB/月 | 无限带宽 |
| 构建时间 | 需自建 CI | 6000分钟/月 | 300分钟/月 | 500次/月 |
| 自定义域名 | 支持 | 支持 | 支持 | 支持 |
| SSL 证书 | 自动 | 自动 | 自动 | 自动 |
| Serverless | 不支持 | 支持 | 支持 | Workers |
| 预览部署 | 不支持 | 支持 | 支持 | 支持 |
| 国内访问 | 较慢 | 较慢 | 较慢 | 较快 |
| 适合场景 | 文档/博客 | 全栈应用 | 静态站点 | 全球化应用 |

---

## 4. 前端自动化测试

### 4.1 Jest 单元测试

Jest 是 React 生态中最流行的测试框架，也广泛用于 Vue 和 Angular 项目。

**Jest 配置示例：**

```javascript
// jest.config.js
module.exports = {
  testEnvironment: 'jsdom',
  setupFilesAfterSetup: ['<rootDir>/tests/setup.ts'],
  moduleNameMapper: {
    '^@/(.*)$': '<rootDir>/src/$1',
    '\\.(css|less|scss)$': 'identity-obj-proxy',
    '\\.(jpg|jpeg|png|gif|svg)$': '<rootDir>/tests/__mocks__/fileMock.js',
  },
  collectCoverageFrom: [
    'src/**/*.{ts,tsx}',
    '!src/**/*.d.ts',
    '!src/**/index.ts',
    '!src/main.tsx',
  ],
  coverageThresholds: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80,
    },
  },
};
```

**React 组件测试示例：**

```typescript
// src/components/UserCard/UserCard.test.tsx
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import { vi } from 'vitest';
import { UserCard } from './UserCard';
import { fetchUser } from '@/services/user';

vi.mock('@/services/user');

const mockUser = {
  id: '1',
  name: '张三',
  email: 'zhangsan@example.com',
  avatar: 'https://example.com/avatar.jpg',
};

describe('UserCard', () => {
  beforeEach(() => {
    vi.mocked(fetchUser).mockResolvedValue(mockUser);
  });

  afterEach(() => {
    vi.clearAllMocks();
  });

  it('应该正确渲染用户信息', async () => {
    render(<UserCard userId="1" />);

    await waitFor(() => {
      expect(screen.getByText('张三')).toBeInTheDocument();
      expect(screen.getByText('zhangsan@example.com')).toBeInTheDocument();
    });
  });

  it('应该显示加载状态', () => {
    render(<UserCard userId="1" />);
    expect(screen.getByTestId('loading-spinner')).toBeInTheDocument();
  });

  it('应该处理加载失败', async () => {
    vi.mocked(fetchUser).mockRejectedValue(new Error('网络错误'));

    render(<UserCard userId="1" />);

    await waitFor(() => {
      expect(screen.getByText('加载失败')).toBeInTheDocument();
    });
  });

  it('点击按钮应该触发回调', async () => {
    const onEdit = vi.fn();
    render(<UserCard userId="1" onEdit={onEdit} />);

    await waitFor(() => {
      expect(screen.getByText('张三')).toBeInTheDocument();
    });

    fireEvent.click(screen.getByRole('button', { name: '编辑' }));
    expect(onEdit).toHaveBeenCalledWith(mockUser);
  });
});
```

### 4.2 Cypress E2E 测试

```javascript
// cypress/e2e/login.cy.ts
describe('登录功能', () => {
  beforeEach(() => {
    cy.visit('/login');
  });

  it('应该显示登录表单', () => {
    cy.get('[data-testid="login-form"]').should('be.visible');
    cy.get('input[name="username"]').should('exist');
    cy.get('input[name="password"]').should('exist');
    cy.get('button[type="submit"]').should('contain', '登录');
  });

  it('空表单提交应该显示验证错误', () => {
    cy.get('button[type="submit"]').click();
    cy.get('.error-message').should('have.length', 2);
    cy.get('.error-message').first().should('contain', '请输入用户名');
  });

  it('应该成功登录并跳转', () => {
    cy.intercept('POST', '/api/auth/login', {
      statusCode: 200,
      body: { token: 'fake-jwt-token', user: { name: '测试用户' } },
    }).as('loginRequest');

    cy.get('input[name="username"]').type('testuser');
    cy.get('input[name="password"]').type('password123');
    cy.get('button[type="submit"]').click();

    cy.wait('@loginRequest');
    cy.url().should('include', '/dashboard');
    cy.get('.user-name').should('contain', '测试用户');
  });

  it('登录失败应该显示错误提示', () => {
    cy.intercept('POST', '/api/auth/login', {
      statusCode: 401,
      body: { message: '用户名或密码错误' },
    });

    cy.get('input[name="username"]').type('wronguser');
    cy.get('input[name="password"]').type('wrongpass');
    cy.get('button[type="submit"]').click();

    cy.get('.alert-error').should('contain', '用户名或密码错误');
  });
});
```

### 4.3 Playwright E2E 测试

Playwright 是新一代 E2E 测试工具，支持多浏览器、自动等待和更好的调试体验。

```typescript
// tests/e2e/shopping.spec.ts
import { test, expect } from '@playwright/test';

test.describe('购物流程', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/');
  });

  test('浏览商品并添加到购物车', async ({ page }) => {
    // 浏览商品列表
    await page.getByRole('link', { name: '商品列表' }).click();
    await expect(page.getByTestId('product-list')).toBeVisible();

    // 点击第一个商品
    const firstProduct = page.getByTestId('product-card').first();
    const productName = await firstProduct.getByTestId('product-name').textContent();
    await firstProduct.click();

    // 添加到购物车
    await page.getByRole('button', { name: '加入购物车' }).click();

    // 验证购物车数量
    await expect(page.getByTestId('cart-count')).toHaveText('1');

    // 验证购物车内容
    await page.getByTestId('cart-icon').click();
    await expect(page.getByTestId('cart-item')).toContainText(productName!);
  });

  test('完整的下单流程', async ({ page }) => {
    // 登录
    await page.goto('/login');
    await page.getByLabel('用户名').fill('testuser');
    await page.getByLabel('密码').fill('password123');
    await page.getByRole('button', { name: '登录' }).click();
    await expect(page.getByText('欢迎回来')).toBeVisible();

    // 添加商品
    await page.goto('/products/1');
    await page.getByRole('button', { name: '加入购物车' }).click();

    // 结算
    await page.getByRole('button', { name: '去结算' }).click();
    await expect(page).toHaveURL('/checkout');

    // 填写收货地址
    await page.getByLabel('收货人').fill('张三');
    await page.getByLabel('手机号').fill('13800138000');
    await page.getByLabel('详细地址').fill('北京市朝阳区xxx路xxx号');

    // 提交订单
    await page.getByRole('button', { name: '提交订单' }).click();
    await expect(page.getByText('订单提交成功')).toBeVisible();
  });

  test('页面性能验证', async ({ page }) => {
    const startTime = Date.now();
    await page.goto('/');
    await page.waitForLoadState('networkidle');
    const loadTime = Date.now() - startTime;

    expect(loadTime).toBeLessThan(3000);

    // 验证核心 Web 指标
    const metrics = await page.evaluate(() => {
      return new Promise((resolve) => {
        new PerformanceObserver((list) => {
          const entries = list.getEntries();
          resolve(entries.map((e) => ({ name: e.name, value: e.value })));
        }).observe({ type: 'largest-contentful-paint', buffered: true });
      });
    });
    console.log('性能指标:', metrics);
  });
});
```

**Playwright 配置文件：**

```typescript
// playwright.config.ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests/e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: [
    ['html'],
    ['junit', { outputFile: 'test-results/e2e-results.xml' }],
  ],
  use: {
    baseURL: 'http://localhost:5173',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
  },
  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
    { name: 'firefox', use: { ...devices['Desktop Firefox'] } },
    { name: 'webkit', use: { ...devices['Desktop Safari'] } },
    { name: 'mobile-chrome', use: { ...devices['Pixel 5'] } },
    { name: 'mobile-safari', use: { ...devices['iPhone 13'] } },
  ],
  webServer: {
    command: 'pnpm dev',
    url: 'http://localhost:5173',
    reuseExistingServer: !process.env.CI,
  },
});
```

---

## 5. Lighthouse CI 自动化性能测试

### 5.1 基础 Lighthouse CI 配置

```yaml
# .github/workflows/lighthouse.yml
name: Lighthouse CI

on:
  pull_request:
    branches: [main]

jobs:
  lighthouse:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: pnpm/action-setup@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'pnpm'

      - run: pnpm install --frozen-lockfile && pnpm build

      - name: 启动本地服务器
        run: pnpm preview &
        env:
          PORT: 4173

      - name: 等待服务器就绪
        run: npx wait-on http://localhost:4173 --timeout 30000

      - name: 运行 Lighthouse CI
        uses: treosh/lighthouse-ci-action@v12
        with:
          urls: |
            http://localhost:4173/
            http://localhost:4173/login
            http://localhost:4173/products
          configPath: .lighthouserc.json
          uploadArtifacts: true
          temporaryPublicStorage: true
```

**Lighthouse CI 配置文件：**

```json
// .lighthouserc.json
{
  "ci": {
    "collect": {
      "numberOfRuns": 3,
      "settings": {
        "preset": "desktop",
        "chromeFlags": "--no-sandbox --headless"
      }
    },
    "assert": {
      "assertions": {
        "categories:performance": ["error", { "minScore": 0.8 }],
        "categories:accessibility": ["warn", { "minScore": 0.9 }],
        "categories:best-practices": ["warn", { "minScore": 0.9 }],
        "categories:seo": ["warn", { "minScore": 0.9 }],
        "first-contentful-paint": ["warn", { "maxNumericValue": 2000 }],
        "largest-contentful-paint": ["error", { "maxNumericValue": 2500 }],
        "cumulative-layout-shift": ["error", { "maxNumericValue": 0.1 }],
        "total-blocking-time": ["warn", { "maxNumericValue": 300 }]
      }
    },
    "upload": {
      "target": "temporary-public-storage"
    }
  }
}
```

### 5.2 Lighthouse CI 与 PR 评论集成

```yaml
- name: 运行 Lighthouse CI 并评论
  uses: treosh/lighthouse-ci-action@v12
  id: lighthouse
  with:
    urls: |
      http://localhost:4173/
    uploadArtifacts: true

- name: 生成 Lighthouse 报告评论
  uses: actions/github-script@v7
  with:
    script: |
      const fs = require('fs');
      const results = JSON.parse(fs.readFileSync('${{ steps.lighthouse.outputs.manifest }}', 'utf8'));

      let comment = '## 🔦 Lighthouse 性能报告\n\n';
      comment += '| 页面 | 性能 | 可访问性 | 最佳实践 | SEO |\n';
      comment += '|------|------|---------|---------|----|\n';

      for (const result of results) {
        const summary = result.summary;
        const url = result.url;
        const perf = (summary.performance * 100).toFixed(0);
        const a11y = (summary.accessibility * 100).toFixed(0);
        const bp = (summary['best-practices'] * 100).toFixed(0);
        const seo = (summary.seo * 100).toFixed(0);

        const score = (s) => s >= 90 ? '🟢' : s >= 50 ? '🟠' : '🔴';
        comment += `| ${url} | ${score(summary.performance)} ${perf} | ${score(summary.accessibility)} ${a11y} | ${score(summary['best-practices'])} ${bp} | ${score(summary.seo)} ${seo} |\n`;
      }

      github.rest.issues.createComment({
        issue_number: context.issue.number,
        owner: context.repo.owner,
        repo: context.repo.repo,
        body: comment
      });
```

---

## 6. 前端代码质量工具

### 6.1 ESLint 配置

现代前端项目推荐使用 ESLint Flat Config 格式：

```javascript
// eslint.config.js
import js from '@eslint/js';
import tsPlugin from '@typescript-eslint/eslint-plugin';
import tsParser from '@typescript-eslint/parser';
import reactPlugin from 'eslint-plugin-react';
import reactHooksPlugin from 'eslint-plugin-react-hooks';
import importPlugin from 'eslint-plugin-import';
import prettierConfig from 'eslint-config-prettier';

export default [
  js.configs.recommended,
  {
    files: ['**/*.{ts,tsx}'],
    languageOptions: {
      parser: tsParser,
      parserOptions: {
        ecmaVersion: 'latest',
        sourceType: 'module',
        ecmaFeatures: { jsx: true },
      },
    },
    plugins: {
      '@typescript-eslint': tsPlugin,
      'react': reactPlugin,
      'react-hooks': reactHooksPlugin,
      'import': importPlugin,
    },
    rules: {
      // TypeScript 规则
      '@typescript-eslint/no-unused-vars': ['error', { argsIgnorePattern: '^_' }],
      '@typescript-eslint/explicit-function-return-type': 'off',
      '@typescript-eslint/no-explicit-any': 'warn',

      // React 规则
      'react/react-in-jsx-scope': 'off',
      'react/prop-types': 'off',
      'react-hooks/rules-of-hooks': 'error',
      'react-hooks/exhaustive-deps': 'warn',

      // Import 规则
      'import/order': ['error', {
        groups: ['builtin', 'external', 'internal', 'parent', 'sibling', 'index'],
        'newlines-between': 'always',
        alphabetize: { order: 'asc' },
      }],

      // 通用规则
      'no-console': ['warn', { allow: ['warn', 'error'] }],
      'prefer-const': 'error',
      'no-var': 'error',
    },
    settings: {
      react: { version: 'detect' },
    },
  },
  prettierConfig,
];
```

### 6.2 Prettier 配置

```json
// .prettierrc
{
  "semi": true,
  "singleQuote": true,
  "tabWidth": 2,
  "trailingComma": "all",
  "printWidth": 100,
  "bracketSpacing": true,
  "arrowParens": "always",
  "endOfLine": "lf",
  "plugins": ["prettier-plugin-tailwindcss"],
  "overrides": [
    {
      "files": "*.md",
      "options": { "printWidth": 80 }
    }
  ]
}
```

### 6.3 Stylelint 配置

```javascript
// .stylelintrc.js
module.exports = {
  extends: [
    'stylelint-config-standard',
    'stylelint-config-standard-scss',
    'stylelint-config-prettier',
  ],
  rules: {
    'selector-class-pattern': '^[a-z][a-zA-Z0-9]+$',
    'no-empty-source': null,
    'scss/operator-no-newline-after': null,
    'declaration-block-no-redundant-longhand-properties': null,
  },
};
```

### 6.4 Husky + lint-staged 配置

```json
// package.json
{
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged",
      "commit-msg": "commitlint -E HUSKY_GIT_PARAMS"
    }
  },
  "lint-staged": {
    "*.{ts,tsx}": [
      "eslint --fix",
      "prettier --write"
    ],
    "*.{css,scss}": [
      "stylelint --fix",
      "prettier --write"
    ],
    "*.{json,md}": [
      "prettier --write"
    ]
  }
}
```

### 6.5 GitHub Actions 中的代码质量检查

```yaml
# .github/workflows/quality.yml
name: Code Quality

on: [pull_request]

jobs:
  quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: pnpm/action-setup@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'pnpm'

      - run: pnpm install --frozen-lockfile

      - name: 检查代码格式
        run: |
          pnpm eslint . --format json --output-file eslint.json || true
          pnpm prettier --check "src/**/*" || true

      - name: 代码质量报告
        uses: actions/github-script@v7
        if: always()
        with:
          script: |
            const fs = require('fs');
            try {
              const eslintResults = JSON.parse(fs.readFileSync('eslint.json', 'utf8'));
              const errors = eslintResults.reduce((sum, f) => sum + f.errorCount, 0);
              const warnings = eslintResults.reduce((sum, f) => sum + f.warningCount, 0);

              let report = `## 📊 代码质量报告\n\n`;
              report += `- ❌ 错误: ${errors}\n`;
              report += `- ⚠️ 警告: ${warnings}\n`;
              report += `- 📁 检查文件数: ${eslintResults.length}\n`;

              if (errors > 0) {
                report += `\n### 错误详情\n`;
                for (const file of eslintResults.filter(f => f.errorCount > 0)) {
                  report += `\n**${file.filePath}**\n`;
                  for (const msg of file.messages.filter(m => m.severity === 2)) {
                    report += `- L${msg.line}: ${msg.message} (${msg.ruleId})\n`;
                  }
                }
              }

              github.rest.issues.createComment({
                issue_number: context.issue.number,
                owner: context.repo.owner,
                repo: context.repo.repo,
                body: report
              });
            } catch (e) {
              console.log('无法生成报告:', e.message);
            }
```

---

## 7. 依赖管理与安全

### 7.1 Dependabot 配置

```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
      day: "monday"
      time: "09:00"
      timezone: "Asia/Shanghai"
    open-pull-requests-limit: 10
    reviewers:
      - "frontend-team"
    labels:
      - "dependencies"
      - "automated"
    groups:
      react:
        patterns:
          - "react*"
          - "@types/react*"
      testing:
        patterns:
          - "jest*"
          - "@testing-library/*"
          - "vitest*"
          - "playwright*"
      build:
        patterns:
          - "vite*"
          - "esbuild*"
          - "rollup*"
      lint:
        patterns:
          - "eslint*"
          - "prettier*"
          - "stylelint*"
    ignore:
      - dependency-name: "*"
        update-types: ["version-update:semver-major"]

  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "weekly"
    labels:
      - "ci"
      - "dependencies"
```

### 7.2 Renovate 配置

Renovate 是 Dependabot 的替代方案，提供更多自定义选项：

```json
// renovate.json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": [
    "config:best-practices",
    ":semanticCommits"
  ],
  "schedule": ["before 9am on monday"],
  "timezone": "Asia/Shanghai",
  "labels": ["dependencies"],
  "reviewers": ["team:frontend"],
  "packageRules": [
    {
      "matchUpdateTypes": ["patch"],
      "automerge": true,
      "automergeType": "pr"
    },
    {
      "matchPackagePatterns": ["^@testing-library"],
      "groupName": "testing libraries",
      "automerge": true
    },
    {
      "matchPackagePatterns": ["^react"],
      "groupName": "React"
    },
    {
      "matchDepTypes": ["devDependencies"],
      "matchUpdateTypes": ["minor", "patch"],
      "automerge": true
    }
  ],
  "vulnerabilityAlerts": {
    "enabled": true,
    "labels": ["security"]
  }
}
```

### 7.3 依赖安全审计

```yaml
# .github/workflows/security.yml
name: Dependency Security

on:
  schedule:
    - cron: '0 9 * * 1'  # 每周一早上9点
  push:
    branches: [main]

jobs:
  audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: pnpm/action-setup@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'pnpm'

      - run: pnpm install --frozen-lockfile

      - name: 安全审计
        run: pnpm audit --json > audit-report.json || true

      - name: 检查许可证
        run: |
          pnpm add -D license-checker
          npx license-checker --json --out licenses.json
          # 检查是否有 GPL 或 AGPL 许可证
          node -e "
            const licenses = require('./licenses.json');
            const problematic = Object.entries(licenses)
              .filter(([_, info]) => /GPL|AGPL/.test(info.licenses));
            if (problematic.length > 0) {
              console.error('发现受限许可证:');
              problematic.forEach(([pkg, info]) => {
                console.error('  ' + pkg + ': ' + info.licenses);
              });
              process.exit(1);
            }
          "

      - name: 创建 Issue（发现漏洞时）
        if: failure()
        uses: actions/github-script@v7
        with:
          script: |
            const fs = require('fs');
            const report = JSON.parse(fs.readFileSync('audit-report.json', 'utf8'));

            let body = '## 🔒 依赖安全漏洞报告\n\n';
            body += `发现 ${report.metadata.vulnerabilities.total} 个漏洞\n\n`;
            body += '| 严重程度 | 数量 |\n|---------|------|\n';

            const vulns = report.metadata.vulnerabilities;
            body += `| 🔴 严重 | ${vulns.critical} |\n`;
            body += `| 🟠 高 | ${vulns.high} |\n`;
            body += `| 🟡 中 | ${vulns.moderate} |\n`;
            body += `| 🟢 低 | ${vulns.low} |\n`;

            github.rest.issues.create({
              owner: context.repo.owner,
              repo: context.repo.repo,
              title: '🔒 发现依赖安全漏洞',
              body: body,
              labels: ['security', 'dependencies']
            });
```

---

## 8. Storybook 与组件库管理

### 8.1 Storybook 基础配置

```yaml
# .github/workflows/storybook.yml
name: Storybook

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build-storybook:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: pnpm/action-setup@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'pnpm'

      - run: pnpm install --frozen-lockfile

      - name: 构建 Storybook
        run: pnpm build-storybook

      - name: 部署 Storybook 到 Chromatic
        if: github.event_name == 'push'
        uses: chromaui/action@latest
        with:
          projectToken: ${{ secrets.CHROMATIC_PROJECT_TOKEN }}
          buildScriptName: build-storybook
          exitOnceUploaded: true

      - name: PR 预览部署
        if: github.event_name == 'pull_request'
        uses: chromaui/action@latest
        with:
          projectToken: ${{ secrets.CHROMATIC_PROJECT_TOKEN }}
          buildScriptName: build-storybook
          exitZeroOnChanges: true
```

### 8.2 Storybook 组件文档示例

```typescript
// src/components/Button/Button.stories.tsx
import type { Meta, StoryObj } from '@storybook/react';
import { Button } from './Button';

const meta: Meta<typeof Button> = title: 'Components/Button',
  component: Button,
  tags: ['autodocs'],
  argTypes: {
    variant: {
      control: 'select',
      options: ['primary', 'secondary', 'outline', 'ghost', 'danger'],
    },
    size: {
      control: 'select',
      options: ['sm', 'md', 'lg'],
    },
    isLoading: { control: 'boolean' },
    isDisabled: { control: 'boolean' },
  },
};

export default meta;
type Story = StoryObj<typeof meta>;

export const Primary: Story = {
  args: {
    children: '主要按钮',
    variant: 'primary',
  },
};

export const Secondary: Story = {
  args: {
    children: '次要按钮',
    variant: 'secondary',
  },
};

export const Loading: Story = {
  args: {
    children: '加载中...',
    variant: 'primary',
    isLoading: true,
  },
};

export const AllSizes: Story = {
  render: () => (
    <div style={{ display: 'flex', gap: '12px', alignItems: 'center' }}>
      <Button size="sm">小按钮</Button>
      <Button size="md">中按钮</Button>
      <Button size="lg">大按钮</Button>
    </div>
  ),
};
```

### 8.3 视觉回归测试

```yaml
# .github/workflows/visual-test.yml
name: Visual Regression Test

on: [pull_request]

jobs:
  visual-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: pnpm/action-setup@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'pnpm'

      - run: pnpm install --frozen-lockfile

      - name: 运行视觉回归测试
        uses: chromaui/action@latest
        with:
          projectToken: ${{ secrets.CHROMATIC_PROJECT_TOKEN }}
          buildScriptName: build-storybook
          onlyChanged: true
          diagnostics: true
```

---

## 9. 前端 Monorepo 管理

### 9.1 Turborepo 配置

```yaml
# .github/workflows/monorepo-ci.yml
name: Monorepo CI

on:
  push:
    branches: [main]
  pull_request:

jobs:
  ci:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Turborepo 需要完整 git 历史

      - uses: pnpm/action-setup@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'pnpm'

      - run: pnpm install --frozen-lockfile

      - name: 构建（仅变更的包）
        run: pnpm turbo build --filter=...[origin/main]

      - name: 测试（仅变更的包）
        run: pnpm turbo test --filter=...[origin/main]

      - name: Lint（仅变更的包）
        run: pnpm turbo lint --filter=...[origin/main]

      - name: 变更检测报告
        run: |
          echo "## 📦 变更包分析" >> $GITHUB_STEP_SUMMARY
          CHANGED=$(pnpm turbo --filter=...[origin/main] --dry=json 2>/dev/null | jq -r '.packages[]' || echo "无法检测")
          echo "变更的包: $CHANGED" >> $GITHUB_STEP_SUMMARY
```

**Turborepo 配置文件：**

```json
// turbo.json
{
  "$schema": "https://turbo.build/schema.json",
  "globalDependencies": ["**/.env.*local"],
  "tasks": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": ["dist/**", ".next/**", "!.next/cache/**"]
    },
    "test": {
      "dependsOn": ["build"],
      "outputs": ["coverage/**"],
      "inputs": ["src/**", "tests/**"]
    },
    "lint": {
      "outputs": []
    },
    "dev": {
      "cache": false,
      "persistent": true
    }
  }
}
```

### 9.2 Nx 配置

```yaml
# .github/workflows/nx-ci.yml
name: Nx Monorepo CI

on:
  push:
    branches: [main]
  pull_request:

jobs:
  ci:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: pnpm/action-setup@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'pnpm'

      - run: pnpm install --frozen-lockfile

      - name: Nx 受影响的项目
        run: npx nx affected -t lint test build --base=origin/main --head=HEAD

      - name: Nx Cloud 分布式缓存
        run: npx nx run-many -t build --parallel=3
        env:
          NX_CLOUD_ACCESS_TOKEN: ${{ secrets.NX_CLOUD_ACCESS_TOKEN }}
```

### 9.3 Monorepo 包发布

```yaml
# .github/workflows/monorepo-release.yml
name: Monorepo Release

on:
  push:
    branches: [main]

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: pnpm/action-setup@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'pnpm'
          registry-url: 'https://registry.npmjs.org'

      - run: pnpm install --frozen-lockfile

      - name: 创建 Release PR
        uses: changesets/action@v1
        with:
          publish: pnpm release
          version: pnpm changeset version
          title: "chore: 版本发布"
          commit: "chore: 版本发布"
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```

---

## 10. npm/pnpm 包发布流程

### 10.1 自动化包发布

```yaml
# .github/workflows/publish.yml
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

      - uses: pnpm/action-setup@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'pnpm'
          registry-url: 'https://registry.npmjs.org'

      - run: pnpm install --frozen-lockfile

      - name: 运行测试
        run: pnpm test

      - name: 构建
        run: pnpm build

      - name: 发布到 npm
        run: pnpm publish --no-git-checks --access public
        env:
          NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}

      - name: 发布到 GitHub Packages
        run: pnpm publish --no-git-checks --access public --registry https://npm.pkg.github.com
        env:
          NODE_AUTH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### 10.2 语义化版本管理

```yaml
# .github/workflows/release.yml
name: Release

on:
  push:
    branches: [main]

permissions:
  contents: write
  pull-requests: write

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: 安装 semantic-release
        run: npm install -g semantic-release @semantic-release/changelog @semantic-release/git

      - name: 执行发布
        run: npx semantic-release
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```

**semantic-release 配置：**

```javascript
// .releaserc.js
module.exports = {
  branches: ['main'],
  plugins: [
    '@semantic-release/commit-analyzer',
    '@semantic-release/release-notes-generator',
    '@semantic-release/changelog',
    '@semantic-release/npm',
    '@semantic-release/github',
    ['@semantic-release/git', {
      assets: ['CHANGELOG.md', 'package.json'],
      message: 'chore(release): ${nextRelease.version} [skip ci]',
    }],
  ],
};
```

---

## 11. 前端国际化（i18n）协作

### 11.1 国际化资源管理

```yaml
# .github/workflows/i18n.yml
name: i18n Management

on:
  push:
    branches: [main]
    paths:
      - 'src/locales/**'

jobs:
  validate-translations:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: 检查翻译完整性
        run: |
          node scripts/check-translations.js

      - name: 检查翻译格式
        run: |
          pnpm i18next-parser src/**/*.{ts,tsx} --config i18next-parser.config.js
          git diff --exit-code src/locales/ || (echo "翻译文件有未处理的键值" && exit 1)

  sync-translations:
    runs-on: ubuntu-latest
    needs: validate-translations
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v4

      - name: 同步翻译到 Crowdin
        uses: crowdin/github-action@v1
        with:
          upload_sources: true
          upload_translations: false
          download_translations: true
          create_pull_request: true
          pull_request_title: 'chore(i18n): 更新翻译文件'
          pull_request_labels: 'i18n'
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          CROWDIN_PROJECT_ID: ${{ secrets.CROWDIN_PROJECT_ID }}
          CROWDIN_PERSONAL_TOKEN: ${{ secrets.CROWDIN_PERSONAL_TOKEN }}
```

### 11.2 翻译完整性检查脚本

```javascript
// scripts/check-translations.js
const fs = require('fs');
const path = require('path');

const LOCALES_DIR = path.join(__dirname, '../src/locales');
const BASE_LANG = 'zh-CN';
const TARGET_LANGS = ['en-US', 'ja-JP'];

function loadTranslations(lang) {
  const filePath = path.join(LOCALES_DIR, `${lang}.json`);
  return JSON.parse(fs.readFileSync(filePath, 'utf8'));
}

function getKeys(obj, prefix = '') {
  let keys = [];
  for (const [key, value] of Object.entries(obj)) {
    const fullKey = prefix ? `${prefix}.${key}` : key;
    if (typeof value === 'object' && value !== null) {
      keys = keys.concat(getKeys(value, fullKey));
    } else {
      keys.push(fullKey);
    }
  }
  return keys;
}

const baseTranslations = loadTranslations(BASE_LANG);
const baseKeys = getKeys(baseTranslations).sort();

let hasError = false;

for (const lang of TARGET_LANGS) {
  const translations = loadTranslations(lang);
  const langKeys = getKeys(translations).sort();

  const missing = baseKeys.filter((k) => !langKeys.includes(k));
  const extra = langKeys.filter((k) => !baseKeys.includes(k));

  if (missing.length > 0) {
    console.error(`❌ ${lang} 缺少以下翻译键:`);
    missing.forEach((k) => console.error(`   - ${k}`));
    hasError = true;
  }

  if (extra.length > 0) {
    console.warn(`⚠️ ${lang} 有多余的翻译键:`);
    extra.forEach((k) => console.warn(`   - ${k}`));
  }

  console.log(`✅ ${lang}: ${langKeys.length}/${baseKeys.length} 个翻译键`);
}

if (hasError) {
  process.exit(1);
}

console.log('\n🎉 所有翻译文件完整！');
```

---

## 12. 设计稿与代码协作

### 12.1 Figma + GitHub 集成工作流

```yaml
# .github/workflows/design-sync.yml
name: Design Sync

on:
  issues:
    types: [labeled]

jobs:
  design-review:
    if: contains(github.event.label.name, 'design-review')
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: 提取 Figma 链接
        id: figma
        uses: actions/github-script@v7
        with:
          script: |
            const body = context.payload.issue.body;
            const figmaRegex = /https:\/\/www\.figma\.com\/file\/[^\s]+/g;
            const links = body.match(figmaRegex) || [];
            return links[0] || '';

      - name: 同步设计 Token
        run: |
          # 从 Figma 导出设计 Token
          npx style-dictionary build --config design-tokens.config.js

      - name: 生成设计规范 PR
        if: steps.figma.outputs.result != ''
        uses: peter-evans/create-pull-request@v5
        with:
          title: 'design: 同步设计 Token'
          body: |
            自动从 Figma 设计稿同步的设计 Token 变更。
            Figma 链接: ${{ steps.figma.outputs.result }}
          branch: design/token-sync
          labels: design, automated
```

### 12.2 设计 Token 管理

```javascript
// design-tokens.config.js
module.exports = {
  source: ['tokens/**/*.json'],
  platforms: {
    css: {
      transformGroup: 'css',
      buildPath: 'src/styles/tokens/',
      files: [
        {
          destination: '_variables.css',
          format: 'css/variables',
          options: {
            outputReferences: true,
          },
        },
      ],
    },
    scss: {
      transformGroup: 'scss',
      buildPath: 'src/styles/tokens/',
      files: [
        {
          destination: '_variables.scss',
          format: 'scss/variables',
        },
      ],
    },
    js: {
      transformGroup: 'js',
      buildPath: 'src/styles/tokens/',
      files: [
        {
          destination: 'tokens.js',
          format: 'javascript/es6',
        },
      ],
    },
  },
};
```

---

## 13. 国内前端团队 GitHub 使用经验

### 13.1 网络加速方案

国内访问 GitHub 经常遇到速度慢的问题，以下是几种解决方案：

**方案一：使用 GitHub Actions 镜像加速**

```yaml
# 使用国内 npm 镜像
- name: 设置 npm 镜像
  run: |
    pnpm config set registry https://registry.npmmirror.com

# 使用国内 pip 镜像（如果有 Python 工具）
- name: 设置 pip 镜像
  run: |
    pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```

**方案二：使用 Gitee 镜像同步**

```yaml
# .github/workflows/mirror.yml
name: Mirror to Gitee

on:
  push:
    branches: [main]

jobs:
  mirror:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: 同步到 Gitee
        uses: wearerequired/git-mirror-action@master
        env:
          SSH_PRIVATE_KEY: ${{ secrets.GITEE_RSA_PRIVATE_KEY }}
        with:
          source-repo: git@github.com:your-org/your-repo.git
          destination-repo: git@gitee.com:your-org/your-repo.git
```

**方案三：GitHub Proxy 加速**

```yaml
# 在 CI 中使用代理
- name: 设置 Git 代理
  run: |
    git config --global http.proxy http://your-proxy:port
    git config --global https.proxy http://your-proxy:port
```

### 13.2 国内部署方案集成

```yaml
# .github/workflows/deploy-china.yml
name: Deploy to China

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: pnpm/action-setup@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'pnpm'
          registry-url: 'https://registry.npmmirror.com'

      - run: pnpm install --frozen-lockfile
      - run: pnpm build

      - name: 上传到阿里云 OSS
        uses: manyuanrong/setup-ossutil@v3.0
        with:
          access-key-id: ${{ secrets.ALIYUN_ACCESS_KEY_ID }}
          access-key-secret: ${{ secrets.ALIYUN_ACCESS_KEY_SECRET }}
          endpoint: oss-cn-hangzhou.aliyuncs.com
          bucket: your-bucket-name

      - name: 部署到 OSS
        run: |
          ossutil cp -r dist/ oss://your-bucket-name/ --update
          ossutil set-meta oss://your-bucket-name/ --update \
            --header "Cache-Control:public,max-age=31536000" \
            --include "*.js" --include "*.css"

      - name: 刷新 CDN 缓存
        run: |
          # 使用阿里云 CDN API 刷新缓存
          aliyun cdn RefreshObjectCaches \
            --ObjectPath "https://www.example.com/" \
            --ObjectType "Directory"

  deploy-to-tencent:
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: 部署到腾讯云 COS
        uses: zkqiang/tencent-cos-action@v1.0.0
        with:
          args: delete -r -f / && upload -r dist/ /
          secret_id: ${{ secrets.TENCENT_SECRET_ID }}
          secret_key: ${{ secrets.TENCENT_SECRET_KEY }}
          bucket: your-bucket-name
          region: ap-guangzhou

      - name: 刷新腾讯云 CDN
        run: |
          pip install coscmd
          coscmd refresh-cdn --url "https://www.example.com/*"
```

### 13.3 国内前端团队最佳实践

**1. 代码规范统一**

```json
// .editorconfig
root = true

[*]
charset = utf-8
end_of_line = lf
indent_style = space
indent_size = 2
insert_final_newline = true
trim_trailing_whitespace = true

[*.md]
trim_trailing_whitespace = false
```

**2. 团队协作规范文档模板**

```markdown
# 前端团队 GitHub 协作规范

## 分支命名规范
- 功能分支: `feature/需求编号-简短描述`
- 修复分支: `fix/Bug编号-简短描述`
- 热修复: `hotfix/问题描述`

## PR 规范
1. 标题格式: `type(scope): 描述`
2. 必须关联 Issue
3. 必须通过 CI 检查
4. 至少 2 人 Code Review
5. 使用 PR 模板填写变更说明

## Code Review 关注点
- 代码风格是否符合规范
- 是否有潜在的性能问题
- 是否有安全风险
- 测试是否充分
- 文档是否更新
```

**3. 国内 CI/CD 优化**

| 优化项 | 方案 | 效果 |
|--------|------|------|
| npm 安装速度 | 使用 npmmirror 镜像 | 提速 5-10 倍 |
| Docker 镜像 | 使用阿里云镜像仓库 | 提速 3-5 倍 |
| 构建缓利 | GitHub Actions Cache | 减少 50% 构建时间 |
| 并行测试 | 分片测试 + 并行执行 | 减少 60% 测试时间 |
| 增量构建 | Turborepo/Nx 缓存 | 减少 70% 构建时间 |

### 13.4 常见问题与解决方案

**问题 1：GitHub Actions 下载依赖超时**

```yaml
# 解决方案：增加超时时间和重试机制
- name: 安装依赖
  run: |
    for i in 1 2 3; do
      pnpm install --frozen-lockfile && break
      echo "第 $i 次尝试失败，等待 10 秒后重试..."
      sleep 10
    done
  timeout-minutes: 15
```

**问题 2：Git LFS 大文件传输慢**

```yaml
# 解决方案：使用 CDN 托管静态资源
- name: 上传静态资源到 CDN
  run: |
    # 将大文件上传到 CDN，Git 仓库只存储引用
    curl -T dist/assets/vendor.js \
      -H "Authorization: ${{ secrets.CDN_TOKEN }}" \
      https://cdn.example.com/upload/
```

**问题 3：多团队协作冲突频繁**

```yaml
# 解决方案：使用 CODEOWNERS + 必须的 Review
# .github/CODEOWNERS
/src/components/  @component-team
/src/pages/       @page-team
/src/services/    @backend-team
/.github/         @devops-team
```

### 13.5 推荐的工具链组合

| 项目类型 | 包管理器 | 构建工具 | 测试框架 | 部署平台 |
|---------|---------|---------|---------|---------|
| React SPA | pnpm | Vite | Vitest + Playwright | Vercel |
| Vue 3 项目 | pnpm | Vite | Vitest + Cypress | Netlify |
| Next.js 项目 | pnpm | Next.js | Jest + Playwright | Vercel |
| Nuxt 项目 | pnpm | Nuxt | Vitest + Playwright | Cloudflare Pages |
| 组件库 | pnpm | Vite/Storybook | Vitest + Chromatic | npm + Storybook |
| Monorepo | pnpm | Turborepo | Vitest | 各平台独立部署 |
| 文档站点 | pnpm | VitePress | - | GitHub Pages |

---

## 总结

本文档涵盖了前端开发者在 GitHub 上进行高效协作的完整工作流。从仓库结构设计、CI/CD 流水线搭建、自动化测试、性能监控到代码质量保障，每个环节都提供了详细的配置示例和最佳实践。

**核心要点：**

1. **自动化优先**：将代码检查、测试、构建、部署全部自动化，减少人工操作
2. **质量门禁**：通过 CI 流水线设置质量门禁，确保代码质量
3. **安全第一**：使用 Dependabot/Renovate 管理依赖安全
4. **团队协作**：通过 CODEOWNERS、PR 模板、Commit 规范提升协作效率
5. **国内优化**：使用镜像加速、CDN 部署等方案优化国内使用体验

通过合理运用这些工具和流程，前端团队可以显著提升开发效率和代码质量，实现快速迭代和稳定发布。
