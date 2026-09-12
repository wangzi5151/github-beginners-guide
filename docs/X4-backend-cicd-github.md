# 后端开发者的 GitHub CI/CD 指南

> 本文档为后端开发者提供完整的 GitHub CI/CD 实践指南，涵盖主流后端语言的自动化流水线搭建、容器化部署、数据库迁移、API 测试、性能测试等核心环节，帮助后端团队构建稳定可靠的持续集成和持续部署体系。

---

## 目录

1. [后端项目 GitHub 仓库结构](#1-后端项目-github-仓库结构)
2. [Java/Spring Boot 项目的 GitHub Actions](#2-javaspring-boot-项目的-github-actions)
3. [Python/Django/Flask 项目的 GitHub Actions](#3-pythondjangoflask-项目的-github-actions)
4. [Go 项目的 GitHub Actions](#4-go-项目的-github-actions)
5. [Node.js/Nest.js 项目的 GitHub Actions](#5-nodejsnestjs-项目的-github-actions)
6. [Rust 项目的 GitHub Actions](#6-rust-项目的-github-actions)
7. [数据库迁移自动化](#7-数据库迁移自动化)
8. [API 测试自动化](#8-api-测试自动化)
9. [容器化部署](#9-容器化部署)
10. [Kubernetes 部署自动化](#10-kubernetes-部署自动化)
11. [后端代码质量检查](#11-后端代码质量检查)
12. [性能测试自动化](#12-性能测试自动化)
13. [多环境部署策略](#13-多环境部署策略)
14. [国内服务器部署方案](#14-国内服务器部署方案)

---

## 1. 后端项目 GitHub 仓库结构

### 1.1 标准后端项目目录结构

一个规范的后端项目仓库应具备清晰的分层结构，便于团队协作、自动化构建和部署。以下是不同语言的推荐目录结构：

**Java/Spring Boot 项目结构：**

```
my-spring-app/
├── .github/
│   ├── workflows/
│   │   ├── ci.yml                    # 持续集成
│   │   ├── deploy.yml                # 部署流水线
│   │   └── release.yml               # 发布流水线
│   ├── ISSUE_TEMPLATE/
│   │   ├── bug_report.md
│   │   └── feature_request.md
│   ├── PULL_REQUEST_TEMPLATE.md
│   ├── CODEOWNERS
│   └── dependabot.yml
├── src/
│   ├── main/
│   │   ├── java/com/example/app/
│   │   │   ├── controller/           # 控制器层
│   │   │   ├── service/              # 服务层
│   │   │   ├── repository/           # 数据访问层
│   │   │   ├── model/                # 实体类
│   │   │   ├── dto/                  # 数据传输对象
│   │   │   ├── config/               # 配置类
│   │   │   ├── exception/            # 异常处理
│   │   │   ├── util/                 # 工具类
│   │   │   └── Application.java
│   │   └── resources/
│   │       ├── application.yml
│   │       ├── application-dev.yml
│   │       ├── application-prod.yml
│   │       ├── db/migration/         # 数据库迁移脚本
│   │       └── static/
│   └── test/
│       ├── java/com/example/app/
│       │   ├── controller/
│       │   ├── service/
│       │   └── integration/          # 集成测试
│       └── resources/
│           └── application-test.yml
├── docker/
│   ├── Dockerfile
│   ├── docker-compose.yml
│   └── docker-compose.test.yml
├── docs/
│   ├── api/                          # API 文档
│   └── architecture/                 # 架构文档
├── scripts/
│   ├── build.sh
│   ├── deploy.sh
│   └── migration.sh
├── pom.xml                           # Maven 配置
├── .gitignore
├── README.md
└── CHANGELOG.md
```

**Python/Django 项目结构：**

```
my-django-app/
├── .github/
│   └── workflows/
├── myproject/
│   ├── settings/
│   │   ├── __init__.py
│   │   ├── base.py
│   │   ├── development.py
│   │   ├── production.py
│   │   └── testing.py
│   ├── urls.py
│   ├── wsgi.py
│   └── asgi.py
├── apps/
│   ├── users/
│   │   ├── models.py
│   │   ├── views.py
│   │   ├── serializers.py
│   │   ├── tests/
│   │   └── migrations/
│   └── orders/
│       ├── models.py
│       ├── views.py
│       ├── serializers.py
│       ├── tests/
│       └── migrations/
├── requirements/
│   ├── base.txt
│   ├── development.txt
│   ├── production.txt
│   └── testing.txt
├── docker/
├── scripts/
├── manage.py
├── pyproject.toml
├── .env.example
└── README.md
```

**Go 项目结构（遵循 golang-standards/project-layout）：**

```
my-go-app/
├── .github/
│   └── workflows/
├── cmd/
│   └── server/
│       └── main.go                   # 应用入口
├── internal/
│   ├── handler/                      # HTTP 处理器
│   ├── service/                      # 业务逻辑
│   ├── repository/                   # 数据访问
│   ├── model/                        # 数据模型
│   ├── middleware/                    # 中间件
│   └── config/                       # 配置
├── pkg/                              # 可复用的公共库
│   ├── logger/
│   ├── database/
│   └── response/
├── api/
│   └── openapi/                      # OpenAPI 规范
├── migrations/                       # 数据库迁移
├── deployments/
│   ├── docker/
│   └── k8s/
├── scripts/
├── test/
│   ├── integration/
│   └── e2e/
├── go.mod
├── go.sum
├── Makefile
├── Dockerfile
└── README.md
```

### 1.2 后端项目 `.github` 目录配置

**CODEOWNERS 文件：**

```
# 后端项目代码所有者
*                           @backend-team

# API 接口变更需要架构师审批
/api/                       @architect-zhang
/src/controller/            @architect-zhang

# 数据库迁移需要 DBA 审批
/src/main/resources/db/     @dba-team
/migrations/                @dba-team

# 配置文件变更需要运维审批
/docker/                    @devops-team
/.github/                   @devops-team
/deployments/               @devops-team

# 安全相关代码需要安全团队审批
/src/**/security/           @security-team
/src/**/auth/               @security-team
```

**Pull Request 模板：**

```markdown
## 变更说明

### 变更类型
- [ ] 新功能 (feat)
- [ ] Bug 修复 (fix)
- [ ] 重构 (refactor)
- [ ] 性能优化 (perf)
- [ ] 文档更新 (docs)
- [ ] 测试 (test)
- [ ] 构建/CI (chore)

### 变更描述
<!-- 请描述你的变更内容 -->

### 数据库变更
- [ ] 包含数据库迁移
- [ ] 无数据库变更

### API 变更
- [ ] 包含 API 变更（请更新 API 文档）
- [ ] 无 API 变更

### 测试
- [ ] 已添加单元测试
- [ ] 已添加集成测试
- [ ] 已添加 API 测试
- [ ] 手动测试通过

### 部署注意事项
<!-- 是否有特殊的部署要求？ -->

### 关联 Issue
<!-- 请关联相关的 Issue 编号 -->
```

### 1.3 后端项目分支管理策略

后端项目通常需要更严格的分支管理，特别是涉及数据库迁移和多环境部署时：

```
main (生产) ← release/v1.2.0 ← develop (开发) ← feature/user-auth
                                 ↑
                                 hotfix/critical-bug
```

**环境与分支对应关系：**

| 环境 | 分支 | 触发方式 | 用途 |
|------|------|---------|------|
| development | develop | 自动部署 | 日常开发测试 |
| staging | release/* | 自动部署 | 预发布验证 |
| production | main | 手动审批 | 生产环境 |
| hotfix | hotfix/* | 手动部署 | 紧急修复 |

---

## 2. Java/Spring Boot 项目的 GitHub Actions

### 2.1 完整的 Spring Boot CI 流水线

```yaml
# .github/workflows/spring-boot-ci.yml
name: Spring Boot CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

env:
  JAVA_VERSION: '21'
  GRADLE_VERSION: '8.5'

jobs:
  code-quality:
    name: 代码质量检查
    runs-on: ubuntu-latest
    steps:
      - name: 检出代码
        uses: actions/checkout@v4
        with:
          fetch-depth: 0  # SonarQube 需要完整历史

      - name: 设置 JDK
        uses: actions/setup-java@v4
        with:
          java-version: ${{ env.JAVA_VERSION }}
          distribution: 'temurin'
          cache: 'gradle'

      - name: Gradle 构建缓存
        uses: actions/cache@v4
        with:
          path: |
            ~/.gradle/caches
            ~/.gradle/wrapper
          key: ${{ runner.os }}-gradle-${{ hashFiles('**/*.gradle*', '**/gradle-wrapper.properties') }}
          restore-keys: |
            ${{ runner.os }}-gradle-

      - name: 编译检查
        run: ./gradlew compileJava compileTestJava

      - name: Checkstyle 检查
        run: ./gradlew checkstyleMain checkstyleTest
        continue-on-error: true

      - name: SpotBugs 静态分析
        run: ./gradlew spotbugsMain
        continue-on-error: true

      - name: SonarQube 分析
        if: github.event_name == 'pull_request'
        run: ./gradlew sonarqube
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
          SONAR_HOST_URL: ${{ secrets.SONAR_HOST_URL }}

  test:
    name: 自动化测试
    runs-on: ubuntu-latest
    needs: code-quality
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_DB: testdb
          POSTGRES_USER: testuser
          POSTGRES_PASSWORD: testpass
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

      redis:
        image: redis:7
        ports:
          - 6379:6379
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-java@v4
        with:
          java-version: ${{ env.JAVA_VERSION }}
          distribution: 'temurin'
          cache: 'gradle'

      - name: 运行单元测试
        run: ./gradlew test
        env:
          SPRING_DATASOURCE_URL: jdbc:postgresql://localhost:5432/testdb
          SPRING_DATASOURCE_USERNAME: testuser
          SPRING_DATASOURCE_PASSWORD: testpass
          SPRING_REDIS_HOST: localhost
          SPRING_REDIS_PORT: 6379

      - name: 生成测试报告
        if: always()
        run: ./gradlew jacocoTestReport

      - name: 检查代码覆盖率
        run: ./gradlew jacocoTestCoverageVerification

      - name: 上传测试报告
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: test-reports
          path: build/reports/tests/
          retention-days: 30

      - name: 上传覆盖率报告
        uses: actions/upload-artifact@v4
        with:
          name: coverage-report
          path: build/reports/jacoco/

      - name: 发布测试结果
        if: always()
        uses: dorny/test-reporter@v1
        with:
          name: 单元测试结果
          path: '**/build/test-results/test/TEST-*.xml'
          reporter: java-junit

  integration-test:
    name: 集成测试
    runs-on: ubuntu-latest
    needs: test
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_DB: integrationdb
          POSTGRES_USER: testuser
          POSTGRES_PASSWORD: testpass
        ports:
          - 5432:5432

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-java@v4
        with:
          java-version: ${{ env.JAVA_VERSION }}
          distribution: 'temurin'
          cache: 'gradle'

      - name: 运行集成测试
        run: ./gradlew integrationTest
        env:
          SPRING_PROFILES_ACTIVE: integration-test
          SPRING_DATASOURCE_URL: jdbc:postgresql://localhost:5432/integrationdb

      - name: 上传集成测试报告
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: integration-test-reports
          path: build/reports/tests/integrationTest/

  build:
    name: 构建与打包
    runs-on: ubuntu-latest
    needs: [test, integration-test]
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-java@v4
        with:
          java-version: ${{ env.JAVA_VERSION }}
          distribution: 'temurin'
          cache: 'gradle'

      - name: 构建可执行 JAR
        run: ./gradlew bootJar

      - name: 构建 Docker 镜像
        run: |
          docker build -t myapp:${{ github.sha }} .
          docker tag myapp:${{ github.sha }} myapp:latest

      - name: 保存 Docker 镜像
        run: docker save myapp:${{ github.sha }} | gzip > myapp-image.tar.gz

      - name: 上传构建产物
        uses: actions/upload-artifact@v4
        with:
          name: build-artifacts
          path: |
            build/libs/*.jar
            myapp-image.tar.gz
          retention-days: 7

  security-scan:
    name: 安全扫描
    runs-on: ubuntu-latest
    needs: build
    steps:
      - uses: actions/checkout@v4

      - name: 下载构建产物
        uses: actions/download-artifact@v4
        with:
          name: build-artifacts

      - name: OWASP 依赖检查
        run: ./gradlew dependencyCheckAnalyze
        continue-on-error: true

      - name: Trivy 容器扫描
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: myapp:${{ github.sha }}
          format: 'sarif'
          output: 'trivy-results.sarif'

      - name: 上传扫描结果
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: 'trivy-results.sarif'
```

### 2.2 Spring Boot 项目发布流程

```yaml
# .github/workflows/spring-boot-release.yml
name: Spring Boot Release

on:
  push:
    tags:
      - 'v*'

jobs:
  release:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      packages: write
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-java@v4
        with:
          java-version: '21'
          distribution: 'temurin'
          cache: 'gradle'

      - name: 构建
        run: ./gradlew bootJar

      - name: 发布到 GitHub Packages
        run: ./gradlew publish
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

      - name: 创建 GitHub Release
        uses: softprops/action-gh-release@v1
        with:
          files: build/libs/*.jar
          generate_release_notes: true
```

---

## 3. Python/Django/Flask 项目的 GitHub Actions

### 3.1 Django 项目 CI 流水线

```yaml
# .github/workflows/django-ci.yml
name: Django CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

env:
  PYTHON_VERSION: '3.12'
  DJANGO_SETTINGS_MODULE: 'myproject.settings.testing'

jobs:
  lint:
    name: 代码检查
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: 设置 Python
        uses: actions/setup-python@v5
        with:
          python-version: ${{ env.PYTHON_VERSION }}
          cache: 'pip'

      - name: 安装依赖
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements/testing.txt

      - name: Ruff 代码检查
        run: ruff check .

      - name: Ruff 格式检查
        run: ruff format --check .

      - name: MyPy 类型检查
        run: mypy .
        continue-on-error: true

      - name: Django 系统检查
        run: python manage.py check --deploy

  test:
    name: 自动化测试
    runs-on: ubuntu-latest
    needs: lint
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_DB: testdb
          POSTGRES_USER: testuser
          POSTGRES_PASSWORD: testpass
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

      redis:
        image: redis:7-alpine
        ports:
          - 6379:6379

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-python@v5
        with:
          python-version: ${{ env.PYTHON_VERSION }}
          cache: 'pip'

      - name: 安装依赖
        run: pip install -r requirements/testing.txt

      - name: 数据库迁移
        run: python manage.py migrate
        env:
          DATABASE_URL: postgres://testuser:testpass@localhost:5432/testdb
          REDIS_URL: redis://localhost:6379/0

      - name: 运行测试
        run: |
          pytest --cov=. --cov-report=xml --cov-report=html --junitxml=test-results.xml
        env:
          DATABASE_URL: postgres://testuser:testpass@localhost:5432/testdb
          REDIS_URL: redis://localhost:6379/0
          DJANGO_SETTINGS_MODULE: myproject.settings.testing

      - name: 上传测试结果
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: test-results
          path: |
            test-results.xml
            htmlcov/

      - name: 上传覆盖率到 Codecov
        uses: codecov/codecov-action@v4
        with:
          file: ./coverage.xml
          token: ${{ secrets.CODECOV_TOKEN }}

  build:
    name: 构建镜像
    runs-on: ubuntu-latest
    needs: test
    steps:
      - uses: actions/checkout@v4

      - name: 构建 Docker 镜像
        run: |
          docker build -t mydjangoapp:${{ github.sha }} .

      - name: 运行安全扫描
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: mydjangoapp:${{ github.sha }}
          format: 'table'
          exit-code: '1'
          severity: 'CRITICAL,HIGH'
```

### 3.2 Flask/FastAPI 项目 CI 流水线

```yaml
# .github/workflows/fastapi-ci.yml
name: FastAPI CI

on:
  push:
    branches: [main]
  pull_request:

jobs:
  test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_DB: testdb
          POSTGRES_USER: testuser
          POSTGRES_PASSWORD: testpass
        ports:
          - 5432:5432

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-python@v5
        with:
          python-version: '3.12'
          cache: 'pip'

      - name: 安装依赖
        run: |
          pip install uv
          uv pip install -r requirements.txt --system

      - name: 代码检查
        run: |
          ruff check src/
          ruff format --check src/

      - name: 运行测试
        run: |
          pytest tests/ -v --cov=src --cov-report=xml
        env:
          DATABASE_URL: postgresql://testuser:testpass@localhost:5432/testdb
          TESTING: true

      - name: 上传覆盖率
        uses: codecov/codecov-action@v4
        with:
          file: ./coverage.xml
```

### 3.3 Python 项目依赖管理

```yaml
# .github/workflows/python-dependencies.yml
name: Python Dependencies

on:
  schedule:
    - cron: '0 9 * * 1'
  workflow_dispatch:

jobs:
  update:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-python@v5
        with:
          python-version: '3.12'

      - name: 检查依赖更新
        run: |
          pip install pip-audit safety
          pip-audit
          safety check -r requirements/production.txt

      - name: 创建依赖更新 PR
        uses: peter-evans/create-pull-request@v5
        with:
          title: 'chore(deps): 更新 Python 依赖'
          body: |
            自动检查并更新的依赖变更。

            请在合并前确认：
            - [ ] 所有测试通过
            - [ ] 没有破坏性变更
          branch: chore/update-dependencies
          labels: dependencies, automated
```

---

## 4. Go 项目的 GitHub Actions

### 4.1 Go 项目完整 CI 流水线

```yaml
# .github/workflows/go-ci.yml
name: Go CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

env:
  GO_VERSION: '1.22'

jobs:
  lint:
    name: 代码检查
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: 设置 Go
        uses: actions/setup-go@v5
        with:
          go-version: ${{ env.GO_VERSION }}
          cache: true

      - name: golangci-lint
        uses: golangci/golangci-lint-action@v4
        with:
          version: latest
          args: --timeout=5m

      - name: go vet
        run: go vet ./...

      - name: 检查 go mod tidy
        run: |
          go mod tidy
          git diff --exit-code go.mod go.sum

  test:
    name: 单元测试
    runs-on: ubuntu-latest
    needs: lint
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_DB: testdb
          POSTGRES_USER: testuser
          POSTGRES_PASSWORD: testpass
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

      redis:
        image: redis:7-alpine
        ports:
          - 6379:6379

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-go@v5
        with:
          go-version: ${{ env.GO_VERSION }}
          cache: true

      - name: 运行测试
        run: |
          go test -v -race -coverprofile=coverage.out -covermode=atomic ./...
        env:
          DATABASE_URL: postgres://testuser:testpass@localhost:5432/testdb?sslmode=disable
          REDIS_URL: redis://localhost:6379

      - name: 生成覆盖率报告
        run: go tool cover -html=coverage.out -o coverage.html

      - name: 上传覆盖率
        uses: codecov/codecov-action@v4
        with:
          files: ./coverage.out

      - name: 上传测试报告
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: coverage-report
          path: coverage.html

  build:
    name: 构建
    runs-on: ubuntu-latest
    needs: test
    strategy:
      matrix:
        goos: [linux, darwin, windows]
        goarch: [amd64, arm64]
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-go@v5
        with:
          go-version: ${{ env.GO_VERSION }}
          cache: true

      - name: 构建二进制文件
        run: |
          CGO_ENABLED=0 GOOS=${{ matrix.goos }} GOARCH=${{ matrix.goarch }} \
            go build -ldflags="-s -w -X main.version=${{ github.sha }}" \
            -o bin/myapp-${{ matrix.goos }}-${{ matrix.goarch }} ./cmd/server
        env:
          GOOS: ${{ matrix.goos }}
          GOARCH: ${{ matrix.goarch }}

      - name: 上传构建产物
        uses: actions/upload-artifact@v4
        with:
          name: binary-${{ matrix.goos }}-${{ matrix.goarch }}
          path: bin/

  security:
    name: 安全扫描
    runs-on: ubuntu-latest
    needs: build
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-go@v5
        with:
          go-version: ${{ env.GO_VERSION }}

      - name: govulncheck 漏洞检查
        run: |
          go install golang.org/x/vuln/cmd/govulncheck@latest
          govulncheck ./...

      - name: gosec 安全扫描
        uses: securego/gosec@master
        with:
          args: ./...

  benchmark:
    name: 性能基准测试
    runs-on: ubuntu-latest
    needs: test
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-go@v5
        with:
          go-version: ${{ env.GO_VERSION }}
          cache: true

      - name: 运行基准测试
        run: go test -bench=. -benchmem ./... | tee benchmark.txt

      - name: 存储基准结果
        uses: actions/cache@v4
        with:
          path: ./benchmark.txt
          key: ${{ runner.os }}-benchmark-${{ github.sha }}

      - name: 比较基准结果
        if: github.event_name == 'pull_request'
        run: |
          go install golang.org/x/perf/cmd/benchstat@latest
          # 获取基准分支的结果进行比较
          git stash
          git checkout ${{ github.base_ref }}
          go test -bench=. -benchmem ./... > benchmark-base.txt || true
          git checkout -
          git stash pop
          benchstat benchmark-base.txt benchmark.txt || true
```

### 4.2 Go 项目 Makefile

```makefile
# Makefile
.PHONY: all build test lint clean docker

APP_NAME := myapp
VERSION := $(shell git describe --tags --always --dirty)
BUILD_TIME := $(shell date -u '+%Y-%m-%d_%H:%M:%S')
LDFLAGS := -ldflags "-s -w -X main.version=$(VERSION) -X main.buildTime=$(BUILD_TIME)"

all: lint test build

build:
	CGO_ENABLED=0 go build $(LDFLAGS) -o bin/$(APP_NAME) ./cmd/server

build-all:
	GOOS=linux GOARCH=amd64 CGO_ENABLED=0 go build $(LDFLAGS) -o bin/$(APP_NAME)-linux-amd64 ./cmd/server
	GOOS=darwin GOARCH=arm64 CGO_ENABLED=0 go build $(LDFLAGS) -o bin/$(APP_NAME)-darwin-arm64 ./cmd/server

test:
	go test -v -race -coverprofile=coverage.out ./...

test-integration:
	go test -v -tags=integration ./test/integration/...

lint:
	golangci-lint run --timeout=5m

bench:
	go test -bench=. -benchmem ./...

docker:
	docker build -t $(APP_NAME):$(VERSION) .

clean:
	rm -rf bin/
	rm -f coverage.out coverage.html

migrate-up:
	migrate -path migrations -database "$(DATABASE_URL)" up

migrate-down:
	migrate -path migrations -database "$(DATABASE_URL)" down

migrate-create:
	migrate create -ext sql -dir migrations -seq $(name)
```

---

## 5. Node.js/Nest.js 项目的 GitHub Actions

### 5.1 NestJS 项目 CI 流水线

```yaml
# .github/workflows/nestjs-ci.yml
name: NestJS CI

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
      - uses: actions/checkout@v4

      - uses: pnpm/action-setup@v4
        with:
          version: ${{ env.PNPM_VERSION }}

      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'pnpm'

      - run: pnpm install --frozen-lockfile

      - name: ESLint
        run: pnpm lint

      - name: TypeScript 类型检查
        run: pnpm tsc --noEmit

      - name: 格式检查
        run: pnpm format:check

  test:
    name: 自动化测试
    runs-on: ubuntu-latest
    needs: lint
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_DB: testdb
          POSTGRES_USER: testuser
          POSTGRES_PASSWORD: testpass
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

      redis:
        image: redis:7-alpine
        ports:
          - 6379:6379

    steps:
      - uses: actions/checkout@v4

      - uses: pnpm/action-setup@v4
        with:
          version: ${{ env.PNPM_VERSION }}

      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'pnpm'

      - run: pnpm install --frozen-lockfile

      - name: 单元测试
        run: pnpm test:cov
        env:
          DATABASE_HOST: localhost
          DATABASE_PORT: 5432
          DATABASE_USER: testuser
          DATABASE_PASSWORD: testpass
          DATABASE_NAME: testdb
          REDIS_HOST: localhost
          REDIS_PORT: 6379

      - name: E2E 测试
        run: pnpm test:e2e
        env:
          DATABASE_HOST: localhost
          DATABASE_PORT: 5432
          DATABASE_USER: testuser
          DATABASE_PASSWORD: testpass
          DATABASE_NAME: testdb
          REDIS_HOST: localhost
          REDIS_PORT: 6379

      - name: 上传测试报告
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: test-reports
          path: coverage/

      - name: 上传覆盖率
        uses: codecov/codecov-action@v4
        with:
          directory: ./coverage

  build:
    name: 构建
    runs-on: ubuntu-latest
    needs: test
    steps:
      - uses: actions/checkout@v4

      - uses: pnpm/action-setup@v4
        with:
          version: ${{ env.PNPM_VERSION }}

      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'pnpm'

      - run: pnpm install --frozen-lockfile

      - name: 构建
        run: pnpm build

      - name: 验证构建产物
        run: |
          ls -la dist/
          node -e "require('./dist/main.js')"

      - name: 上传构建产物
        uses: actions/upload-artifact@v4
        with:
          name: build
          path: dist/

  api-test:
    name: API 测试
    runs-on: ubuntu-latest
    needs: build
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_DB: testdb
          POSTGRES_USER: testuser
          POSTGRES_PASSWORD: testpass
        ports:
          - 5432:5432

    steps:
      - uses: actions/checkout@v4

      - uses: pnpm/action-setup@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'pnpm'

      - run: pnpm install --frozen-lockfile

      - name: 启动应用
        run: |
          pnpm build
          pnpm start:prod &
          sleep 10
        env:
          DATABASE_HOST: localhost
          DATABASE_PORT: 5432
          DATABASE_USER: testuser
          DATABASE_PASSWORD: testpass
          DATABASE_NAME: testdb

      - name: 运行 API 测试
        run: |
          npm install -g newman
          newman run test/api/postman_collection.json \
            --environment test/api/environment.json \
            --reporters cli,junit \
            --reporter-junit-export api-test-results.xml

      - name: 上传 API 测试结果
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: api-test-results
          path: api-test-results.xml
```

### 5.2 Express/Fastify 项目 CI

```yaml
# .github/workflows/express-ci.yml
name: Express CI

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [18, 20, 22]
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'

      - run: npm ci

      - run: npm run lint

      - run: npm test
        env:
          CI: true

      - run: npm run build
```

---

## 6. Rust 项目的 GitHub Actions

### 6.1 Rust 项目完整 CI 流水线

```yaml
# .github/workflows/rust-ci.yml
name: Rust CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

env:
  CARGO_TERM_COLOR: always
  RUST_VERSION: '1.78'

jobs:
  check:
    name: 代码检查
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: 设置 Rust
        uses: dtolnay/rust-toolchain@stable
        with:
          components: clippy, rustfmt

      - name: Cargo 缓存
        uses: actions/cache@v4
        with:
          path: |
            ~/.cargo/bin/
            ~/.cargo/registry/index/
            ~/.cargo/registry/cache/
            ~/.cargo/git/db/
            target/
          key: ${{ runner.os }}-cargo-${{ hashFiles('**/Cargo.lock') }}

      - name: 格式检查
        run: cargo fmt --all -- --check

      - name: Clippy 静态分析
        run: cargo clippy --all-targets --all-features -- -D warnings

      - name: 检查文档
        run: cargo doc --no-deps --document-private-items
        env:
          RUSTDOCFLAGS: "-D warnings"

  test:
    name: 测试
    runs-on: ubuntu-latest
    needs: check
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_DB: testdb
          POSTGRES_USER: testuser
          POSTGRES_PASSWORD: testpass
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      - uses: actions/checkout@v4

      - uses: dtolnay/rust-toolchain@stable

      - uses: actions/cache@v4
        with:
          path: |
            ~/.cargo/
            target/
          key: ${{ runner.os }}-cargo-${{ hashFiles('**/Cargo.lock') }}

      - name: 运行测试
        run: cargo test --all-features --verbose
        env:
          DATABASE_URL: postgres://testuser:testpass@localhost:5432/testdb
          RUST_LOG: debug

      - name: 运行集成测试
        run: cargo test --all-features --test '*'
        env:
          DATABASE_URL: postgres://testuser:testpass@localhost:5432/testdb

      - name: 代码覆盖率
        run: |
          cargo install cargo-tarpaulin
          cargo tarpaulin --all-features --out xml
        env:
          DATABASE_URL: postgres://testuser:testpass@localhost:5432/testdb

      - name: 上传覆盖率
        uses: codecov/codecov-action@v4
        with:
          file: ./cobertura.xml

  build:
    name: 构建
    runs-on: ${{ matrix.os }}
    needs: test
    strategy:
      matrix:
        os: [ubuntu-latest, macos-latest, windows-latest]
        include:
          - os: ubuntu-latest
            artifact: linux-amd64
          - os: macos-latest
            artifact: macos-arm64
          - os: windows-latest
            artifact: windows-amd64
    steps:
      - uses: actions/checkout@v4

      - uses: dtolnay/rust-toolchain@stable

      - uses: actions/cache@v4
        with:
          path: |
            ~/.cargo/
            target/
          key: ${{ runner.os }}-cargo-${{ hashFiles('**/Cargo.lock') }}

      - name: 构建
        run: cargo build --release

      - name: 上传构建产物
        uses: actions/upload-artifact@v4
        with:
          name: binary-${{ matrix.artifact }}
          path: target/release/myapp*

  security:
    name: 安全审计
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Cargo 审计
        uses: actions-rs/audit-check@v1
        with:
          token: ${{ secrets.GITHUB_TOKEN }}

      - name: 依赖漏洞检查
        run: |
          cargo install cargo-audit
          cargo audit

  benchmark:
    name: 性能基准测试
    runs-on: ubuntu-latest
    needs: test
    steps:
      - uses: actions/checkout@v4

      - uses: dtolnay/rust-toolchain@stable

      - name: 运行基准测试
        run: cargo bench --all-features -- --output-format bencher | tee output.txt

      - name: 存储基准结果
        uses: actions/cache@v4
        with:
          path: output.txt
          key: ${{ runner.os }}-bench-${{ github.sha }}
```

### 6.2 Rust 项目发布流程

```yaml
# .github/workflows/rust-release.yml
name: Rust Release

on:
  push:
    tags:
      - 'v*'

jobs:
  release:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, macos-latest, windows-latest]
    steps:
      - uses: actions/checkout@v4

      - uses: dtolnay/rust-toolchain@stable

      - name: 构建
        run: cargo build --release

      - name: 打包
        run: |
          cd target/release
          tar czf myapp-${{ github.ref_name }}-${{ runner.os }}.tar.gz myapp*
        if: runner.os != 'Windows'

      - name: 打包 (Windows)
        if: runner.os == 'Windows'
        run: |
          Compress-Archive -Path target/release/myapp.exe -DestinationPath myapp-${{ github.ref_name }}-Windows.zip

      - name: 发布到 crates.io
        if: runner.os == 'ubuntu-latest'
        run: cargo publish
        env:
          CARGO_REGISTRY_TOKEN: ${{ secrets.CARGO_REGISTRY_TOKEN }}

      - name: 创建 GitHub Release
        uses: softprops/action-gh-release@v1
        with:
          files: |
            target/release/*.tar.gz
            target/release/*.zip
```

---

## 7. 数据库迁移自动化

### 7.1 Flyway 数据库迁移

```yaml
# .github/workflows/db-migration.yml
name: Database Migration

on:
  push:
    branches: [main, develop]
    paths:
      - 'src/main/resources/db/migration/**'
      - 'migrations/**'

jobs:
  validate-migration:
    name: 验证迁移脚本
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_DB: migrationdb
          POSTGRES_USER: testuser
          POSTGRES_PASSWORD: testpass
        ports:
          - 5432:5432

    steps:
      - uses: actions/checkout@v4

      - name: 安装 Flyway
        run: |
          wget -qO- https://repo1.maven.org/maven2/org/flywaydb/flyway-commandline/10.4.0/flyway-commandline-10.4.0-linux-x64.tar.gz | tar xz
          echo "$PWD/flyway-10.4.0" >> $GITHUB_PATH

      - name: 验证迁移脚本
        run: |
          flyway -url=jdbc:postgresql://localhost:5432/migrationdb \
            -user=testuser -password=testpass \
            -locations=filesystem:src/main/resources/db/migration \
            validate

      - name: 执行迁移
        run: |
          flyway -url=jdbc:postgresql://localhost:5432/migrationdb \
            -user=testuser -password=testpass \
            -locations=filesystem:src/main/resources/db/migration \
            migrate

      - name: 检查迁移状态
        run: |
          flyway -url=jdbc:postgresql://localhost:5432/migrationdb \
            -user=testuser -password=testpass \
            -locations=filesystem:src/main/resources/db/migration \
            info

  deploy-migration:
    name: 部署迁移
    needs: validate-migration
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/checkout@v4

      - name: 执行生产迁移
        run: |
          flyway -url=${{ secrets.PROD_DATABASE_URL }} \
            -user=${{ secrets.PROD_DATABASE_USER }} \
            -password=${{ secrets.PROD_DATABASE_PASSWORD }} \
            -locations=filesystem:src/main/resources/db/migration \
            migrate

      - name: 通知迁移结果
        uses: slackapi/slack-github-action@v1
        with:
          payload: |
            {
              "text": "数据库迁移已执行完成\n分支: ${{ github.ref }}\n提交: ${{ github.sha }}"
            }
```

### 7.2 Liquibase 数据库迁移

```yaml
# .github/workflows/liquibase-migration.yml
name: Liquibase Migration

on:
  push:
    branches: [main]
    paths:
      - 'db/changelog/**'

jobs:
  migrate:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_DB: testdb
          POSTGRES_USER: testuser
          POSTGRES_PASSWORD: testpass
        ports:
          - 5432:5432

    steps:
      - uses: actions/checkout@v4

      - name: 验证变更日志
        uses: liquibase/liquibase-github-action@v4
        with:
          operation: validate
          changelog-file: db/changelog/db.changelog-master.xml
          url: jdbc:postgresql://localhost:5432/testdb
          username: testuser
          password: testpass

      - name: 执行迁移
        uses: liquibase/liquibase-github-action@v4
        with:
          operation: update
          changelog-file: db/changelog/db.changelog-master.xml
          url: jdbc:postgresql://localhost:5432/testdb
          username: testuser
          password: testpass

      - name: 生成迁移报告
        uses: liquibase/liquibase-github-action@v4
        with:
          operation: status
          changelog-file: db/changelog/db.changelog-master.xml
          url: jdbc:postgresql://localhost:5432/testdb
          username: testuser
          password: testpass
```

### 7.3 Python Alembic 迁移

```yaml
# .github/workflows/alembic-migration.yml
name: Alembic Migration

on:
  push:
    branches: [main]
    paths:
      - 'alembic/versions/**'

jobs:
  migrate:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_DB: testdb
          POSTGRES_USER: testuser
          POSTGRES_PASSWORD: testpass
        ports:
          - 5432:5432

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-python@v5
        with:
          python-version: '3.12'

      - name: 安装依赖
        run: pip install -r requirements.txt alembic

      - name: 验证迁移
        run: alembic check
        env:
          DATABASE_URL: postgresql://testuser:testpass@localhost:5432/testdb

      - name: 执行迁移
        run: alembic upgrade head
        env:
          DATABASE_URL: postgresql://testuser:testpass@localhost:5432/testdb

      - name: 生成迁移脚本
        run: alembic history > migration-history.txt

      - name: 上传迁移记录
        uses: actions/upload-artifact@v4
        with:
          name: migration-history
          path: migration-history.txt
```

### 7.4 Go migrate 迁移

```yaml
# .github/workflows/go-migrate.yml
name: Go Migrate

on:
  push:
    branches: [main]
    paths:
      - 'migrations/**'

jobs:
  migrate:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_DB: testdb
          POSTGRES_USER: testuser
          POSTGRES_PASSWORD: testpass
        ports:
          - 5432:5432

    steps:
      - uses: actions/checkout@v4

      - name: 安装 golang-migrate
        run: |
          curl -L https://github.com/golang-migrate/migrate/releases/download/v4.17.0/migrate.linux-amd64.tar.gz | tar xz
          sudo mv migrate /usr/local/bin/

      - name: 验证迁移文件
        run: migrate -path migrations -database postgres://testuser:testpass@localhost:5432/testdb?sslmode=disable validate

      - name: 执行迁移
        run: migrate -path migrations -database postgres://testuser:testpass@localhost:5432/testdb?sslmode=disable up

      - name: 检查迁移版本
        run: migrate -path migrations -database postgres://testuser:testpass@localhost:5432/testdb?sslmode=disable version

      - name: 生成迁移差异
        if: failure()
        run: |
          echo "迁移失败，请检查迁移脚本"
          migrate -path migrations -database postgres://testuser:testpass@localhost:5432/testdb?sslmode=disable version
```

---

## 8. API 测试自动化

### 8.1 Postman/Newman API 测试

```yaml
# .github/workflows/api-test.yml
name: API Test

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  api-test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_DB: testdb
          POSTGRES_USER: testuser
          POSTGRES_PASSWORD: testpass
        ports:
          - 5432:5432

    steps:
      - uses: actions/checkout@v4

      - name: 启动应用
        run: |
          # 根据项目类型启动应用
          docker-compose -f docker-compose.test.yml up -d
          sleep 30

      - name: 运行 Newman API 测试
        uses: anthropics/newman-action@v1
        with:
          collection: test/api/collection.json
          environment: test/api/environment.json
          reporters: cli,junit,htmlextra
          reporter-junit-export: api-test-results.xml
          reporter-htmlextra-export: api-test-report.html

      - name: 上传测试报告
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: api-test-reports
          path: |
            api-test-results.xml
            api-test-report.html

      - name: 发布测试结果
        if: always()
        uses: dorny/test-reporter@v1
        with:
          name: API 测试结果
          path: api-test-results.xml
          reporter: java-junit
```

### 8.2 REST Assured (Java) API 测试

```java
// src/test/java/com/example/api/UserApiTest.java
package com.example.api;

import io.restassured.RestAssured;
import io.restassured.http.ContentType;
import org.junit.jupiter.api.*;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.web.server.LocalServerPort;

import static io.restassured.RestAssured.*;
import static org.hamcrest.Matchers.*;

@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@TestMethodOrder(MethodOrderer.OrderAnnotation.class)
class UserApiTest {

    @LocalServerPort
    private int port;

    @BeforeEach
    void setUp() {
        RestAssured.port = port;
        RestAssured.basePath = "/api/v1";
    }

    @Test
    @Order(1)
    void shouldCreateUser() {
        given()
            .contentType(ContentType.JSON)
            .body("""
                {
                    "name": "张三",
                    "email": "zhangsan@example.com",
                    "password": "securePassword123"
                }
                """)
        .when()
            .post("/users")
        .then()
            .statusCode(201)
            .body("name", equalTo("张三"))
            .body("email", equalTo("zhangsan@example.com"))
            .body("id", notNullValue());
    }

    @Test
    @Order(2)
    void shouldGetUserById() {
        given()
            .pathParam("id", 1)
        .when()
            .get("/users/{id}")
        .then()
            .statusCode(200)
            .body("name", equalTo("张三"));
    }

    @Test
    @Order(3)
    void shouldReturn404ForNonExistentUser() {
        given()
            .pathParam("id", 999)
        .when()
            .get("/users/{id}")
        .then()
            .statusCode(404)
            .body("message", equalTo("用户不存在"));
    }

    @Test
    @Order(4)
    void shouldValidateInput() {
        given()
            .contentType(ContentType.JSON)
            .body("""
                {
                    "name": "",
                    "email": "invalid-email"
                }
                """)
        .when()
            .post("/users")
        .then()
            .statusCode(400)
            .body("errors", hasSize(greaterThan(0)));
    }
}
```

### 8.3 pytest API 测试 (Python)

```python
# tests/api/test_users.py
import pytest
from httpx import AsyncClient
from app.main import app

@pytest.fixture
async def client():
    async with AsyncClient(app=app, base_url="http://test") as ac:
        yield ac

@pytest.fixture
async def auth_headers(client):
    response = await client.post("/api/v1/auth/login", json={
        "username": "testuser",
        "password": "testpass"
    })
    token = response.json()["token"]
    return {"Authorization": f"Bearer {token}"}

class TestUserAPI:
    @pytest.mark.asyncio
    async def test_create_user(self, client, auth_headers):
        response = await client.post("/api/v1/users", json={
            "name": "张三",
            "email": "zhangsan@example.com",
            "password": "securePassword123"
        }, headers=auth_headers)

        assert response.status_code == 201
        data = response.json()
        assert data["name"] == "张三"
        assert data["email"] == "zhangsan@example.com"
        assert "id" in data

    @pytest.mark.asyncio
    async def test_get_user(self, client, auth_headers):
        # 创建用户
        create_response = await client.post("/api/v1/users", json={
            "name": "李四",
            "email": "lisi@example.com",
            "password": "securePassword123"
        }, headers=auth_headers)
        user_id = create_response.json()["id"]

        # 获取用户
        response = await client.get(f"/api/v1/users/{user_id}", headers=auth_headers)
        assert response.status_code == 200
        assert response.json()["name"] == "李四"

    @pytest.mark.asyncio
    async def test_user_not_found(self, client, auth_headers):
        response = await client.get("/api/v1/users/999", headers=auth_headers)
        assert response.status_code == 404

    @pytest.mark.asyncio
    async def test_validation_error(self, client, auth_headers):
        response = await client.post("/api/v1/users", json={
            "name": "",
            "email": "invalid"
        }, headers=auth_headers)
        assert response.status_code == 422

    @pytest.mark.asyncio
    async def test_list_users(self, client, auth_headers):
        response = await client.get("/api/v1/users", headers=auth_headers)
        assert response.status_code == 200
        assert isinstance(response.json()["items"], list)

    @pytest.mark.asyncio
    async def test_update_user(self, client, auth_headers):
        # 创建用户
        create_response = await client.post("/api/v1/users", json={
            "name": "王五",
            "email": "wangwu@example.com",
            "password": "securePassword123"
        }, headers=auth_headers)
        user_id = create_response.json()["id"]

        # 更新用户
        response = await client.put(f"/api/v1/users/{user_id}", json={
            "name": "王五（已更新）"
        }, headers=auth_headers)
        assert response.status_code == 200
        assert response.json()["name"] == "王五（已更新）"

    @pytest.mark.asyncio
    async def test_delete_user(self, client, auth_headers):
        # 创建用户
        create_response = await client.post("/api/v1/users", json={
            "name": "赵六",
            "email": "zhaoliu@example.com",
            "password": "securePassword123"
        }, headers=auth_headers)
        user_id = create_response.json()["id"]

        # 删除用户
        response = await client.delete(f"/api/v1/users/{user_id}", headers=auth_headers)
        assert response.status_code == 204

        # 验证已删除
        response = await client.get(f"/api/v1/users/{user_id}", headers=auth_headers)
        assert response.status_code == 404
```

### 8.4 Go API 测试

```go
// tests/api/user_test.go
package api_test

import (
    "bytes"
    "encoding/json"
    "net/http"
    "net/http/httptest"
    "testing"

    "github.com/stretchr/testify/assert"
    "github.com/stretchr/testify/require"
)

func TestCreateUser(t *testing.T) {
    app := setupTestApp()

    body := map[string]string{
        "name":     "张三",
        "email":    "zhangsan@example.com",
        "password": "securePassword123",
    }
    jsonBody, _ := json.Marshal(body)

    req := httptest.NewRequest(http.MethodPost, "/api/v1/users", bytes.NewBuffer(jsonBody))
    req.Header.Set("Content-Type", "application/json")
    resp, err := app.Test(req)

    require.NoError(t, err)
    assert.Equal(t, http.StatusCreated, resp.StatusCode)

    var result map[string]interface{}
    json.NewDecoder(resp.Body).Decode(&result)
    assert.Equal(t, "张三", result["name"])
    assert.NotNil(t, result["id"])
}

func TestGetUser(t *testing.T) {
    app := setupTestApp()

    // 创建用户
    createBody := map[string]string{
        "name":     "李四",
        "email":    "lisi@example.com",
        "password": "securePassword123",
    }
    jsonBody, _ := json.Marshal(createBody)
    createReq := httptest.NewRequest(http.MethodPost, "/api/v1/users", bytes.NewBuffer(jsonBody))
    createReq.Header.Set("Content-Type", "application/json")
    createResp, _ := app.Test(createReq)

    var created map[string]interface{}
    json.NewDecoder(createResp.Body).Decode(&created)
    userID := created["id"]

    // 获取用户
    req := httptest.NewRequest(http.MethodGet, "/api/v1/users/"+fmt.Sprintf("%v", userID), nil)
    resp, err := app.Test(req)

    require.NoError(t, err)
    assert.Equal(t, http.StatusOK, resp.StatusCode)

    var user map[string]interface{}
    json.NewDecoder(resp.Body).Decode(&user)
    assert.Equal(t, "李四", user["name"])
}

func TestUserNotFound(t *testing.T) {
    app := setupTestApp()

    req := httptest.NewRequest(http.MethodGet, "/api/v1/users/999", nil)
    resp, err := app.Test(req)

    require.NoError(t, err)
    assert.Equal(t, http.StatusNotFound, resp.StatusCode)
}
```

---

## 9. 容器化部署

### 9.1 多阶段 Docker 构建

**Java/Spring Boot Dockerfile：**

```dockerfile
# docker/Dockerfile
# 构建阶段
FROM eclipse-temurin:21-jdk-alpine AS builder
WORKDIR /app

COPY gradle/ gradle/
COPY gradlew build.gradle.kts settings.gradle.kts ./
RUN ./gradlew dependencies --no-daemon

COPY src/ src/
RUN ./gradlew bootJar --no-daemon

# 运行阶段
FROM eclipse-temurin:21-jre-alpine AS runtime
WORKDIR /app

RUN addgroup -g 1001 -S appgroup && \
    adduser -u 1001 -S appuser -G appgroup

COPY --from=builder /app/build/libs/*.jar app.jar

RUN chown -R appuser:appgroup /app
USER appuser

EXPOSE 8080

HEALTHCHECK --interval=30s --timeout=3s --start-period=30s --retries=3 \
    CMD wget --no-verbose --tries=1 --spider http://localhost:8080/actuator/health || exit 1

ENTRYPOINT ["java", "-XX:+UseContainerSupport", "-XX:MaxRAMPercentage=75.0", "-jar", "app.jar"]
```

**Python/Django Dockerfile：**

```dockerfile
# docker/Dockerfile
FROM python:3.12-slim AS builder
WORKDIR /app

RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential libpq-dev && \
    rm -rf /var/lib/apt/lists/*

COPY requirements/ requirements/
RUN pip install --no-cache-dir --prefix=/install -r requirements/production.txt

FROM python:3.12-slim AS runtime
WORKDIR /app

RUN apt-get update && apt-get install -y --no-install-recommends \
    libpq5 && \
    rm -rf /var/lib/apt/lists/*

COPY --from=builder /install /usr/local

RUN groupadd -r appgroup && useradd -r -g appgroup appuser

COPY . .

RUN chown -R appuser:appgroup /app
USER appuser

EXPOSE 8000

HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
    CMD python -c "import urllib.request; urllib.request.urlopen('http://localhost:8000/health/')" || exit 1

CMD ["gunicorn", "myproject.wsgi:application", "--bind", "0.0.0.0:8000", "--workers", "4"]
```

**Go Dockerfile：**

```dockerfile
# docker/Dockerfile
FROM golang:1.22-alpine AS builder
WORKDIR /app

COPY go.mod go.sum ./
RUN go mod download

COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -ldflags="-s -w" -o /app/server ./cmd/server

FROM alpine:3.19 AS runtime
WORKDIR /app

RUN apk --no-cache add ca-certificates tzdata && \
    addgroup -g 1001 -S appgroup && \
    adduser -u 1001 -S appuser -G appgroup

COPY --from=builder /app/server .

RUN chown appuser:appgroup server
USER appuser

EXPOSE 8080

HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
    CMD wget --no-verbose --tries=1 --spider http://localhost:8080/health || exit 1

ENTRYPOINT ["./server"]
```

**Node.js/NestJS Dockerfile：**

```dockerfile
# docker/Dockerfile
FROM node:20-alpine AS builder
WORKDIR /app

RUN corepack enable && corepack prepare pnpm@9 --activate

COPY package.json pnpm-lock.yaml ./
RUN pnpm install --frozen-lockfile

COPY . .
RUN pnpm build

FROM node:20-alpine AS runtime
WORKDIR /app

RUN corepack enable && corepack prepare pnpm@9 --activate

RUN addgroup -g 1001 -S appgroup && \
    adduser -u 1001 -S appuser -G appgroup

COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/package.json ./

RUN chown -R appuser:appgroup /app
USER appuser

EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
    CMD wget --no-verbose --tries=1 --spider http://localhost:3000/health || exit 1

CMD ["node", "dist/main.js"]
```

**Rust Dockerfile：**

```dockerfile
# docker/Dockerfile
FROM rust:1.78-alpine AS builder
WORKDIR /app

RUN apk add --no-cache musl-dev

COPY Cargo.toml Cargo.lock ./
RUN mkdir src && echo "fn main() {}" > src/main.rs && \
    cargo build --release && \
    rm -rf src

COPY src/ src/
RUN touch src/main.rs && cargo build --release

FROM alpine:3.19 AS runtime
WORKDIR /app

RUN apk --no-cache add ca-certificates tzdata && \
    addgroup -g 1001 -S appgroup && \
    adduser -u 1001 -S appuser -G appgroup

COPY --from=builder /app/target/release/myapp .

RUN chown appuser:appgroup myapp
USER appuser

EXPOSE 8080

HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
    CMD wget --no-verbose --tries=1 --spider http://localhost:8080/health || exit 1

ENTRYPOINT ["./myapp"]
```

### 9.2 Docker Compose 配置

```yaml
# docker/docker-compose.yml
version: '3.8'

services:
  app:
    build:
      context: ..
      dockerfile: docker/Dockerfile
    ports:
      - "8080:8080"
    environment:
      - DATABASE_URL=postgresql://postgres:postgres@postgres:5432/mydb
      - REDIS_URL=redis://redis:6379
      - SPRING_PROFILES_ACTIVE=docker
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    networks:
      - app-network

  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - app-network

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - app-network

  pgadmin:
    image: dpage/pgadmin4:latest
    environment:
      PGADMIN_DEFAULT_EMAIL: admin@example.com
      PGADMIN_DEFAULT_PASSWORD: admin
    ports:
      - "5050:80"
    depends_on:
      - postgres
    networks:
      - app-network

volumes:
  postgres_data:
  redis_data:

networks:
  app-network:
    driver: bridge
```

### 9.3 GitHub Actions Docker 构建与推送

```yaml
# .github/workflows/docker-build.yml
name: Docker Build & Push

on:
  push:
    branches: [main]
    tags: ['v*']
  pull_request:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    steps:
      - uses: actions/checkout@v4

      - name: 设置 Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: 登录 GitHub Container Registry
        if: github.event_name != 'pull_request'
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: 提取元数据
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=ref,event=branch
            type=ref,event=pr
            type=semver,pattern={{version}}
            type=semver,pattern={{major}}.{{minor}}
            type=sha

      - name: 构建并推送 Docker 镜像
        uses: docker/build-push-action@v5
        with:
          context: .
          file: docker/Dockerfile
          push: ${{ github.event_name != 'pull_request' }}
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

      - name: 运行 Trivy 安全扫描
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}
          format: 'sarif'
          output: 'trivy-results.sarif'

      - name: 上传扫描结果
        if: always()
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: 'trivy-results.sarif'
```

---

## 10. Kubernetes 部署自动化

### 10.1 Kubernetes 清单文件

```yaml
# deployments/k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  namespace: production
  labels:
    app: myapp
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: myapp
          image: ghcr.io/myorg/myapp:latest
          ports:
            - containerPort: 8080
          env:
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef:
                  name: myapp-secrets
                  key: database-url
            - name: REDIS_URL
              valueFrom:
                secretKeyRef:
                  name: myapp-secrets
                  key: redis-url
          resources:
            requests:
              memory: "256Mi"
              cpu: "250m"
            limits:
              memory: "512Mi"
              cpu: "500m"
          livenessProbe:
            httpGet:
              path: /actuator/health/liveness
              port: 8080
            initialDelaySeconds: 30
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /actuator/health/readiness
              port: 8080
            initialDelaySeconds: 20
            periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
  namespace: production
spec:
  selector:
    app: myapp
  ports:
    - port: 80
      targetPort: 8080
  type: ClusterIP
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
  namespace: production
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  ingressClassName: nginx
  tls:
    - hosts:
        - api.example.com
      secretName: myapp-tls
  rules:
    - host: api.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: myapp-service
                port:
                  number: 80
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: myapp-hpa
  namespace: production
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
  minReplicas: 3
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 80
```

### 10.2 GitHub Actions Kubernetes 部署

```yaml
# .github/workflows/k8s-deploy.yml
name: Kubernetes Deploy

on:
  push:
    branches: [main]
    tags: ['v*']

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/checkout@v4

      - name: 设置 kubectl
        uses: azure/setup-kubectl@v3

      - name: 配置 kubeconfig
        run: |
          mkdir -p $HOME/.kube
          echo "${{ secrets.KUBE_CONFIG }}" | base64 -d > $HOME/.kube/config

      - name: 登录容器仓库
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: 更新镜像版本
        run: |
          kubectl set image deployment/myapp \
            myapp=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }} \
            -n production

      - name: 等待部署完成
        run: |
          kubectl rollout status deployment/myapp -n production --timeout=300s

      - name: 验证部署
        run: |
          kubectl get pods -n production -l app=myapp
          kubectl get svc -n production

      - name: 回滚（部署失败时）
        if: failure()
        run: |
          kubectl rollout undo deployment/myapp -n production

  smoke-test:
    name: 冒烟测试
    needs: deploy
    runs-on: ubuntu-latest
    steps:
      - name: 健康检查
        run: |
          for i in {1..10}; do
            STATUS=$(curl -s -o /dev/null -w "%{http_code}" https://api.example.com/health)
            if [ "$STATUS" = "200" ]; then
              echo "健康检查通过"
              exit 0
            fi
            echo "等待服务就绪... ($i/10)"
            sleep 10
          done
          echo "健康检查失败"
          exit 1

      - name: API 冒烟测试
        run: |
          # 基本 API 测试
          curl -f https://api.example.com/api/v1/health
          curl -f https://api.example.com/api/v1/version
```

### 10.3 Kustomize 多环境管理

```yaml
# deployments/k8s/base/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - deployment.yaml
  - service.yaml
  - ingress.yaml
  - hpa.yaml

commonLabels:
  app: myapp

---
# deployments/k8s/overlays/production/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
bases:
  - ../../base
namespace: production
replicas:
  - name: myapp
    count: 3
patches:
  - target:
      kind: Deployment
      name: myapp
    patch: |
      - op: replace
        path: /spec/template/spec/containers/0/resources/requests/memory
        value: "512Mi"
      - op: replace
        path: /spec/template/spec/containers/0/resources/limits/memory
        value: "1Gi"

---
# deployments/k8s/overlays/staging/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
bases:
  - ../../base
namespace: staging
replicas:
  - name: myapp
    count: 1
```

---

## 11. 后端代码质量检查

### 11.1 SonarQube 集成

```yaml
# .github/workflows/sonarqube.yml
name: SonarQube Analysis

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  sonarqube:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: SonarQube 扫描
        uses: sonarqube-quality-gate-action@master
        with:
          scanMetadataReportFile: target/sonar/report-task.txt
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}

      - name: SonarQube 质量门禁
        uses: sonarqube-quality-gate-action@master
        timeout-minutes: 5
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
```

**sonar-project.properties 配置：**

```properties
# sonar-project.properties
sonar.projectKey=my-backend-app
sonar.projectName=My Backend App
sonar.projectVersion=1.0

sonar.sources=src/main
sonar.tests=src/test
sonar.java.binaries=build/classes
sonar.java.libraries=build/libs

sonar.coverage.jacoco.xmlReportPaths=build/reports/jacoco/test/jacocoTestReport.xml
sonar.junit.reportPaths=build/test-results/test

sonar.qualitygate.wait=true
sonar.qualitygate.timeout=300

sonar.exclusions=**/generated/**,**/test/**
```

### 11.2 Codecov 集成

```yaml
# .github/workflows/codecov.yml
name: Codecov

on: [push, pull_request]

jobs:
  coverage:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: 运行测试并生成覆盖率
        run: |
          # 根据项目类型运行测试
          ./gradlew test jacocoTestReport

      - name: 上传到 Codecov
        uses: codecov/codecov-action@v4
        with:
          files: build/reports/jacoco/test/jacocoTestReport.xml
          token: ${{ secrets.CODECOV_TOKEN }}
          fail_ci_if_error: true
          verbose: true

      - name: Codecov 覆盖率检查
        uses: codecov/codecov-action@v4
        with:
          token: ${{ secrets.CODECOV_TOKEN }}
          flags: unittests
          name: codecov-umbrella
```

### 11.3 代码质量报告集成

```yaml
# .github/workflows/quality-report.yml
name: Quality Report

on: [pull_request]

jobs:
  report:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: 运行测试和检查
        run: |
          ./gradlew test jacocoTestReport checkstyleMain

      - name: 生成质量报告
        uses: actions/github-script@v7
        with:
          script: |
            const fs = require('fs');

            // 读取测试结果
            const testResults = fs.readFileSync('build/reports/tests/test/index.html', 'utf8');

            // 读取覆盖率
            const coverage = fs.readFileSync('build/reports/jacoco/test/jacocoTestReport.csv', 'utf8');

            // 读取 Checkstyle 结果
            let checkstyle = '无 Checkstyle 结果';
            try {
              checkstyle = fs.readFileSync('build/reports/checkstyle/main.xml', 'utf8');
            } catch (e) {}

            let report = `## 📊 代码质量报告\n\n`;
            report += `### 测试结果\n`;
            report += `详见 [测试报告](https://github.com/${context.repo.owner}/${context.repo.repo}/actions/runs/${context.runId})\n\n`;

            report += `### 代码覆盖率\n`;
            const lines = coverage.split('\n');
            if (lines.length > 1) {
              const headers = lines[0].split(',');
              const values = lines[1].split(',');
              report += `| 指标 | 覆盖率 |\n|------|--------|\n`;
              for (let i = 0; i < headers.length; i++) {
                if (headers[i].includes('COVERED') || headers[i].includes('MISSED')) {
                  report += `| ${headers[i]} | ${values[i]} |\n`;
                }
              }
            }

            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: report
            });
```

---

## 12. 性能测试自动化

### 12.1 JMeter 性能测试

```yaml
# .github/workflows/performance-test.yml
name: Performance Test

on:
  schedule:
    - cron: '0 2 * * 0'  # 每周日凌晨2点
  workflow_dispatch:
    inputs:
      threads:
        description: '并发线程数'
        default: '100'
      duration:
        description: '测试持续时间（秒）'
        default: '300'

jobs:
  performance:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: 安装 JMeter
        run: |
          wget https://archive.apache.org/dist/jmeter/binaries/apache-jmeter-5.6.3.tgz
          tar xzf apache-jmeter-5.6.3.tgz
          echo "$PWD/apache-jmeter-5.6.3/bin" >> $GITHUB_PATH

      - name: 运行性能测试
        run: |
          THREADS=${{ github.event.inputs.threads || '100' }}
          DURATION=${{ github.event.inputs.duration || '300' }}

          jmeter -n -t test/performance/load-test.jmx \
            -Jthreads=$THREADS \
            -Jduration=$DURATION \
            -l results.jtl \
            -e -o report/

      - name: 上传测试报告
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: performance-report
          path: report/

      - name: 分析结果
        run: |
          # 提取关键指标
          TOTAL=$(grep -c "true" results.jtl || echo 0)
          SUCCESS=$(grep -c "true,true" results.jtl || echo 0)
          ERROR_RATE=$(echo "scale=2; ($TOTAL - $SUCCESS) * 100 / $TOTAL" | bc)

          echo "## 🚀 性能测试报告" >> $GITHUB_STEP_SUMMARY
          echo "" >> $GITHUB_STEP_SUMMARY
          echo "| 指标 | 值 |" >> $GITHUB_STEP_SUMMARY
          echo "|------|-----|" >> $GITHUB_STEP_SUMMARY
          echo "| 总请求数 | $TOTAL |" >> $GITHUB_STEP_SUMMARY
          echo "| 成功请求数 | $SUCCESS |" >> $GITHUB_STEP_SUMMARY
          echo "| 错误率 | ${ERROR_RATE}% |" >> $GITHUB_STEP_SUMMARY

          if (( $(echo "$ERROR_RATE > 5" | bc -l) )); then
            echo "::error::错误率 ${ERROR_RATE}% 超过阈值 5%"
            exit 1
          fi
```

### 12.2 k6 性能测试

```javascript
// test/performance/load-test.js
import http from 'k6/http';
import { check, sleep } from 'k6';
import { Rate, Trend } from 'k6/metrics';

const errorRate = new Rate('errors');
const latency = new Trend('latency');

export const options = {
  stages: [
    { duration: '2m', target: 100 },   // 升压到 100 用户
    { duration: '5m', target: 100 },   // 保持 100 用户
    { duration: '2m', target: 200 },   // 升压到 200 用户
    { duration: '5m', target: 200 },   // 保持 200 用户
    { duration: '2m', target: 0 },     // 降到 0
  ],
  thresholds: {
    http_req_duration: ['p(95)<500'],  // 95% 请求 < 500ms
    errors: ['rate<0.05'],             // 错误率 < 5%
  },
};

const BASE_URL = __ENV.BASE_URL || 'http://localhost:8080';

export default function () {
  // 测试用户登录
  const loginRes = http.post(`${BASE_URL}/api/v1/auth/login`, JSON.stringify({
    username: 'testuser',
    password: 'testpass',
  }), {
    headers: { 'Content-Type': 'application/json' },
  });

  check(loginRes, {
    'login status is 200': (r) => r.status === 200,
    'login has token': (r) => r.json('token') !== undefined,
  }) || errorRate.add(1);

  const token = loginRes.json('token');

  // 测试获取用户列表
  const usersRes = http.get(`${BASE_URL}/api/v1/users`, {
    headers: { Authorization: `Bearer ${token}` },
  });

  check(usersRes, {
    'users status is 200': (r) => r.status === 200,
    'users has items': (r) => r.json('items') !== undefined,
  }) || errorRate.add(1);

  latency.add(usersRes.timings.duration);

  // 测试创建用户
  const createRes = http.post(`${BASE_URL}/api/v1/users`, JSON.stringify({
    name: `User ${Date.now()}`,
    email: `user${Date.now()}@example.com`,
    password: 'testpass123',
  }), {
    headers: {
      'Content-Type': 'application/json',
      Authorization: `Bearer ${token}`,
    },
  });

  check(createRes, {
    'create status is 201': (r) => r.status === 201,
  }) || errorRate.add(1);

  sleep(1);
}

export function handleSummary(data) {
  return {
    'performance-results.json': JSON.stringify(data, null, 2),
    stdout: textSummary(data, { indent: ' ', enableColors: true }),
  };
}
```

```yaml
# .github/workflows/k6-performance.yml
name: k6 Performance Test

on:
  schedule:
    - cron: '0 2 * * 0'
  workflow_dispatch:

jobs:
  performance:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: 安装 k6
        run: |
          sudo gpg -k
          sudo gpg --no-default-keyring --keyring /usr/share/keyrings/k6-archive-keyring.gpg --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys C5AD17C747E3415A3642D57D77C6C491D6AC1D69
          echo "deb [signed-by=/usr/share/keyrings/k6-archive-keyring.gpg] https://dl.k6.io/deb stable main" | sudo tee /etc/apt/sources.list.d/k6.list
          sudo apt-get update && sudo apt-get install k6

      - name: 运行 k6 测试
        run: |
          k6 run test/performance/load-test.js \
            --out json=results.json \
            --summary-export=summary.json

      - name: 上传测试结果
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: k6-results
          path: |
            results.json
            summary.json

      - name: 性能报告
        if: always()
        run: |
          echo "## 🚀 k6 性能测试报告" >> $GITHUB_STEP_SUMMARY
          echo "" >> $GITHUB_STEP_SUMMARY
          cat summary.json | jq -r '
            "| 指标 | 值 |\n|------|-----|\n" +
            "| 总请求数 | " + .metrics.http_reqs.values.count + " |\n" +
            "| 平均响应时间 | " + (.metrics.http_req_duration.values.avg | tostring) + "ms |\n" +
            "| P95 响应时间 | " + (.metrics.http_req_duration.values["p(95)"] | tostring) + "ms |\n" +
            "| 错误率 | " + ((.metrics.http_req_failed.values.rate * 100) | tostring) + "% |"
          ' >> $GITHUB_STEP_SUMMARY
```

---

## 13. 多环境部署策略

### 13.1 环境配置管理

```yaml
# .github/workflows/multi-env-deploy.yml
name: Multi Environment Deploy

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: 构建 Docker 镜像
        run: |
          docker build -t myapp:${{ github.sha }} .
          docker tag myapp:${{ github.sha }} myapp:latest

      - name: 推送镜像
        run: |
          echo ${{ secrets.REGISTRY_PASSWORD }} | docker login -u ${{ secrets.REGISTRY_USERNAME }} --password-stdin
          docker push myapp:${{ github.sha }}
          docker push myapp:latest

  deploy-dev:
    name: 部署到开发环境
    needs: build
    if: github.ref == 'refs/heads/develop'
    runs-on: ubuntu-latest
    environment:
      name: development
      url: https://dev.example.com
    steps:
      - uses: actions/checkout@v4

      - name: 部署到开发环境
        run: |
          kubectl config use-context dev-cluster
          kubectl set image deployment/myapp myapp=myapp:${{ github.sha }} -n development
          kubectl rollout status deployment/myapp -n development

      - name: 运行冒烟测试
        run: |
          curl -f https://dev.example.com/health

  deploy-staging:
    name: 部署到预发布环境
    needs: build
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment:
      name: staging
      url: https://staging.example.com
    steps:
      - uses: actions/checkout@v4

      - name: 部署到预发布环境
        run: |
          kubectl config use-context staging-cluster
          kubectl set image deployment/myapp myapp=myapp:${{ github.sha }} -n staging
          kubectl rollout status deployment/myapp -n staging

      - name: 运行集成测试
        run: |
          npm install -g newman
          newman run test/api/staging-collection.json --environment test/api/staging-env.json

      - name: 性能基准测试
        run: |
          k6 run test/performance/smoke-test.js -e BASE_URL=https://staging.example.com

  deploy-production:
    name: 部署到生产环境
    needs: deploy-staging
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://api.example.com
    steps:
      - uses: actions/checkout@v4

      - name: 部署到生产环境（金丝雀发布）
        run: |
          kubectl config use-context prod-cluster

          # 金丝雀发布：先更新 1 个 Pod
          kubectl set image deployment/myapp-canary myapp=myapp:${{ github.sha }} -n production
          kubectl rollout status deployment/myapp-canary -n production

          # 等待 5 分钟观察
          sleep 300

          # 检查金丝雀 Pod 的健康状态
          if kubectl get pods -n production -l app=myapp,version=canary | grep -q Running; then
            echo "金丝雀发布健康，开始全量发布"
            kubectl set image deployment/myapp myapp=myapp:${{ github.sha }} -n production
            kubectl rollout status deployment/myapp -n production
          else
            echo "金丝雀发布失败，回滚"
            kubectl rollout undo deployment/myapp-canary -n production
            exit 1
          fi

      - name: 验证生产部署
        run: |
          curl -f https://api.example.com/health
          curl -f https://api.example.com/api/v1/version | jq .

      - name: 通知部署结果
        if: always()
        uses: slackapi/slack-github-action@v1
        with:
          payload: |
            {
              "text": "生产部署${{ job.status == 'success' ? '成功' : '失败' }}\n版本: ${{ github.sha }}\n时间: ${{ github.event.head_commit.timestamp }}"
            }
```

### 13.2 蓝绿部署策略

```yaml
# .github/workflows/blue-green-deploy.yml
name: Blue-Green Deploy

on:
  workflow_dispatch:
    inputs:
      target:
        description: '目标环境 (blue/green)'
        required: true
        type: choice
        options:
          - blue
          - green

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/checkout@v4

      - name: 确定当前和目标环境
        id: env
        run: |
          CURRENT=$(kubectl get svc myapp-active -n production -o jsonpath='{.spec.selector.slot}')
          if [ "$CURRENT" = "blue" ]; then
            echo "current=blue" >> $GITHUB_OUTPUT
            echo "target=green" >> $GITHUB_OUTPUT
          else
            echo "current=green" >> $GITHUB_OUTPUT
            echo "target=blue" >> $GITHUB_OUTPUT
          fi

      - name: 部署到目标环境
        run: |
          TARGET=${{ steps.env.outputs.target }}
          kubectl set image deployment/myapp-$TARGET myapp=myapp:${{ github.sha }} -n production
          kubectl rollout status deployment/myapp-$TARGET -n production

      - name: 切换流量
        run: |
          TARGET=${{ steps.env.outputs.target }}
          kubectl patch svc myapp-active -n production -p "{\"spec\":{\"selector\":{\"slot\":\"$TARGET\"}}}"

      - name: 验证切换
        run: |
          sleep 30
          curl -f https://api.example.com/health

      - name: 回滚（如果失败）
        if: failure()
        run: |
          CURRENT=${{ steps.env.outputs.current }}
          kubectl patch svc myapp-active -n production -p "{\"spec\":{\"selector\":{\"slot\":\"$CURRENT\"}}}"
```

### 13.3 版本回滚策略

```yaml
# .github/workflows/rollback.yml
name: Rollback

on:
  workflow_dispatch:
    inputs:
      version:
        description: '回滚版本（留空回滚到上一个版本）'
        required: false
      environment:
        description: '目标环境'
        required: true
        type: choice
        options:
          - staging
          - production

jobs:
  rollback:
    runs-on: ubuntu-latest
    environment: ${{ inputs.environment }}
    steps:
      - name: 配置 kubectl
        run: |
          echo "${{ secrets.KUBE_CONFIG }}" | base64 -d > $HOME/.kube/config

      - name: 执行回滚
        run: |
          if [ -z "${{ inputs.version }}" ]; then
            kubectl rollout undo deployment/myapp -n ${{ inputs.environment }}
          else
            kubectl rollout undo deployment/myapp --to-revision=${{ inputs.version }} -n ${{ inputs.environment }}
          fi
          kubectl rollout status deployment/myapp -n ${{ inputs.environment }}

      - name: 验证回滚
        run: |
          sleep 30
          curl -f https://${{ inputs.environment }}.example.com/health

      - name: 通知回滚
        uses: slackapi/slack-github-action@v1
        with:
          payload: |
            {
              "text": "⚠️ 已执行回滚\n环境: ${{ inputs.environment }}\n版本: ${{ inputs.version || '上一个版本' }}\n操作人: ${{ github.actor }}"
            }
```

---

## 14. 国内服务器部署方案

### 14.1 阿里云容器服务部署

```yaml
# .github/workflows/aliyun-deploy.yml
name: Aliyun Deploy

on:
  push:
    branches: [main]

env:
  REGISTRY: registry.cn-hangzhou.aliyuncs.com
  NAMESPACE: my-namespace
  IMAGE_NAME: myapp

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: 登录阿里云容器镜像服务
        run: |
          docker login -u ${{ secrets.ALIYUN_CR_USERNAME }} -p ${{ secrets.ALIYUN_CR_PASSWORD }} ${{ env.REGISTRY }}

      - name: 构建并推送镜像
        run: |
          FULL_IMAGE=${{ env.REGISTRY }}/${{ env.NAMESPACE }}/${{ env.IMAGE_NAME }}
          docker build -t $FULL_IMAGE:${{ github.sha }} .
          docker tag $FULL_IMAGE:${{ github.sha }} $FULL_IMAGE:latest
          docker push $FULL_IMAGE:${{ github.sha }}
          docker push $FULL_IMAGE:latest

  deploy-ack:
    name: 部署到阿里云 ACK
    needs: build-and-push
    runs-on: ubuntu-latest
    steps:
      - name: 配置 kubeconfig
        run: |
          mkdir -p $HOME/.kube
          echo "${{ secrets.ALIYUN_KUBE_CONFIG }}" | base64 -d > $HOME/.kube/config

      - name: 部署到 ACK
        run: |
          FULL_IMAGE=${{ env.REGISTRY }}/${{ env.NAMESPACE }}/${{ env.IMAGE_NAME }}
          kubectl set image deployment/myapp myapp=$FULL_IMAGE:${{ github.sha }} -n production
          kubectl rollout status deployment/myapp -n production --timeout=300s

      - name: 验证部署
        run: |
          kubectl get pods -n production
          kubectl get svc -n production
```

### 14.2 腾讯云容器服务部署

```yaml
# .github/workflows/tencent-deploy.yml
name: Tencent Cloud Deploy

on:
  push:
    branches: [main]

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: 登录腾讯云容器镜像服务
        run: |
          docker login -u ${{ secrets.TCR_USERNAME }} -p ${{ secrets.TCR_PASSWORD }} ccr.ccs.tencentyun.com

      - name: 构建并推送镜像
        run: |
          IMAGE=ccr.ccs.tencentyun.com/my-namespace/myapp
          docker build -t $IMAGE:${{ github.sha }} .
          docker push $IMAGE:${{ github.sha }}

  deploy-tke:
    name: 部署到腾讯云 TKE
    needs: build-and-push
    runs-on: ubuntu-latest
    steps:
      - name: 配置 kubeconfig
        run: |
          mkdir -p $HOME/.kube
          echo "${{ secrets.TKE_KUBE_CONFIG }}" | base64 -d > $HOME/.kube/config

      - name: 部署到 TKE
        run: |
          IMAGE=ccr.ccs.tencentyun.com/my-namespace/myapp
          kubectl set image deployment/myapp myapp=$IMAGE:${{ github.sha }} -n production
          kubectl rollout status deployment/myapp -n production
```

### 14.3 华为云容器服务部署

```yaml
# .github/workflows/huawei-deploy.yml
name: Huawei Cloud Deploy

on:
  push:
    branches: [main]

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: 登录华为云 SWR
        run: |
          docker login -u ${{ secrets.HW_SWR_USERNAME }} -p ${{ secrets.HW_SWR_PASSWORD }} swr.cn-north-4.myhuaweicloud.com

      - name: 构建并推送镜像
        run: |
          IMAGE=swr.cn-north-4.myhuaweicloud.com/my-namespace/myapp
          docker build -t $IMAGE:${{ github.sha }} .
          docker push $IMAGE:${{ github.sha }}

  deploy-cce:
    name: 部署到华为云 CCE
    needs: build-and-push
    runs-on: ubuntu-latest
    steps:
      - name: 配置 kubeconfig
        run: |
          mkdir -p $HOME/.kube
          echo "${{ secrets.CCE_KUBE_CONFIG }}" | base64 -d > $HOME/.kube/config

      - name: 部署到 CCE
        run: |
          IMAGE=swr.cn-north-4.myhuaweicloud.com/my-namespace/myapp
          kubectl set image deployment/myapp myapp=$IMAGE:${{ github.sha }} -n production
          kubectl rollout status deployment/myapp -n production
```

### 14.4 国内部署优化建议

**1. 镜像加速配置**

```yaml
# 在 CI 中使用国内镜像源
- name: 配置 npm 镜像
  run: npm config set registry https://registry.npmmirror.com

- name: 配置 pip 镜像
  run: pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple

- name: 配置 Docker 镜像加速
  run: |
    sudo mkdir -p /etc/docker
    sudo tee /etc/docker/daemon.json <<-'EOF'
    {
      "registry-mirrors": [
        "https://mirror.ccs.tencentyun.com",
        "https://registry.docker-cn.com"
      ]
    }
    EOF
    sudo systemctl daemon-reload
    sudo systemctl restart docker
```

**2. 多区域部署策略**

```yaml
# 多区域部署矩阵
strategy:
  matrix:
    region:
      - name: cn-hangzhou
        endpoint: registry.cn-hangzhou.aliyuncs.com
      - name: cn-beijing
        endpoint: registry.cn-beijing.aliyuncs.com
      - name: cn-shanghai
        endpoint: registry.cn-shanghai.aliyuncs.com
```

**3. CDN 静态资源加速**

```yaml
- name: 上传到阿里云 OSS + CDN
  run: |
    # 上传静态资源到 OSS
    ossutil cp -r dist/ oss://my-bucket/assets/ --update

    # 刷新 CDN 缓存
    aliyun cdn RefreshObjectCaches \
      --ObjectPath "https://cdn.example.com/assets/" \
      --ObjectType "Directory"
```

**4. 国内监控集成**

```yaml
- name: 部署监控
  run: |
    # 配置阿里云 ARMS 监控
    kubectl apply -f - <<EOF
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: arms-agent-config
    data:
      APP_NAME: myapp
      LICENSE_KEY: ${{ secrets.ARMS_LICENSE_KEY }}
    EOF
```

### 14.5 国内部署方案对比

| 云服务商 | 容器服务 | 镜像仓库 | CI/CD 集成 | 优势 |
|---------|---------|---------|-----------|------|
| 阿里云 | ACK | ACR | 云效 | 生态完善、文档丰富 |
| 腾讯云 | TKE | TCR | CODING | 游戏/社交场景优化 |
| 华为云 | CCE | SWR | DevCloud | 企业级安全、混合云 |
| AWS 中国 | EKS | ECR | CodeStar | 全球化部署能力 |

---

## 总结

本文档为后端开发者提供了完整的 GitHub CI/CD 实践指南，涵盖了从代码提交到生产部署的全流程自动化。

**核心要点：**

1. **自动化优先**：将代码检查、测试、构建、部署全部自动化，减少人工操作和人为错误
2. **多语言支持**：针对 Java、Python、Go、Node.js、Rust 等主流后端语言提供详细的 CI 配置
3. **容器化部署**：使用 Docker 多阶段构建优化镜像大小，Kubernetes 编排实现弹性伸缩
4. **质量门禁**：通过 SonarQube、Codecov 等工具确保代码质量，自动化性能测试保障系统稳定性
5. **多环境管理**：实现 dev/staging/prod 多环境自动化部署，支持金丝雀发布和蓝绿部署
6. **国内优化**：提供阿里云、腾讯云、华为云等国内主流云服务商的部署方案

通过合理运用这些工具和流程，后端团队可以构建稳定可靠的 CI/CD 体系，实现快速迭代和高质量交付。
