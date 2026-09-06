# DevOps 实战

## CI/CD 流水线

### 完整 CI/CD 配置

```yaml
# .github/workflows/ci-cd.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  # 代码质量检查
  lint:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    
    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '20'
        cache: 'npm'
    
    - run: npm ci
    - run: npm run lint
    - run: npm run type-check

  # 测试
  test:
    runs-on: ubuntu-latest
    needs: lint
    
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_PASSWORD: postgres
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
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '20'
        cache: 'npm'
    
    - run: npm ci
    
    - name: Run tests
      run: npm test
      env:
        DATABASE_URL: postgresql://postgres:postgres@localhost:5432/test
        REDIS_URL: redis://localhost:6379
    
    - name: Upload coverage
      uses: codecov/codecov-action@v4
      with:
        files: ./coverage/lcov.info

  # 构建 Docker 镜像
  build:
    runs-on: ubuntu-latest
    needs: test
    permissions:
      contents: read
      packages: write
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Login to Container Registry
      uses: docker/login-action@v3
      with:
        registry: ${{ env.REGISTRY }}
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}
    
    - name: Extract metadata
      id: meta
      uses: docker/metadata-action@v5
      with:
        images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
        tags: |
          type=ref,event=branch
          type=sha,prefix={{branch}}-
          type=semver,pattern={{version}}
    
    - name: Build and push
      uses: docker/build-push-action@v5
      with:
        context: .
        push: true
        tags: ${{ steps.meta.outputs.tags }}
        labels: ${{ steps.meta.outputs.labels }}

  # 部署到 Staging
  deploy-staging:
    runs-on: ubuntu-latest
    needs: build
    if: github.ref == 'refs/heads/develop'
    environment: staging
    
    steps:
    - name: Deploy to staging
      run: |
        echo "Deploying to staging..."
        # 部署逻辑
    
  # 部署到 Production
  deploy-production:
    runs-on: ubuntu-latest
    needs: build
    if: github.ref == 'refs/heads/main'
    environment: production
    
    steps:
    - name: Deploy to production
      run: |
        echo "Deploying to production..."
        # 部署逻辑
```

## GitOps 工作流

### 使用 ArgoCD

```yaml
# argocd-application.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/your-org/your-repo.git
    targetRevision: main
    path: k8s
  destination:
    server: https://kubernetes.default.svc
    namespace: my-app
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
    - CreateNamespace=true
```

### 镜像自动更新

```yaml
# .github/workflows/update-image.yml
name: Update Image

on:
  push:
    branches: [main]

jobs:
  update:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    
    - name: Update ArgoCD image
      run: |
        # 更新 Kubernetes 配置中的镜像版本
        sed -i "s|image: .*|image: ghcr.io/your-org/your-app:${{ github.sha }}|" k8s/deployment.yaml
        
        # 提交更改
        git config user.name "github-actions[bot]"
        git config user.email "github-actions[bot]@users.noreply.github.com"
        git add .
        git commit -m "chore: update image to ${{ github.sha }}"
        git push
```

## 基础设施即代码

### Terraform

```yaml
# .github/workflows/terraform.yml
name: Terraform

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  terraform:
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: terraform
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Setup Terraform
      uses: hashicorp/setup-terraform@v3
    
    - name: Terraform Init
      run: terraform init
    
    - name: Terraform Plan
      run: terraform plan
      if: github.event_name == 'pull_request'
    
    - name: Terraform Apply
      run: terraform apply -auto-approve
      if: github.ref == 'refs/heads/main' && github.event_name == 'push'
```

## 监控和可观测性

### 部署后验证

```yaml
# .github/workflows/post-deploy.yml
name: Post Deploy Verification

on:
  workflow_run:
    workflows: ["CI/CD Pipeline"]
    types: [completed]

jobs:
  verify:
    runs-on: ubuntu-latest
    if: ${{ github.event.workflow_run.conclusion == 'success' }}
    
    steps:
    - name: Health check
      run: |
        for i in {1..30}; do
          STATUS=$(curl -s -o /dev/null -w "%{http_code}" https://api.example.com/health)
          if [ "$STATUS" = "200" ]; then
            echo "Service is healthy"
            exit 0
          fi
          echo "Waiting for service... ($i/30)"
          sleep 10
        done
        echo "Service failed to become healthy"
        exit 1
    
    - name: Smoke test
      run: |
        # 运行关键路径测试
        curl -s https://api.example.com/api/users | jq .
```

## 最佳实践

1. **自动化一切**：从代码到部署全流程自动化
2. **环境隔离**：Staging 和 Production 分离
3. **快速反馈**：尽早发现并修复问题
4. **回滚能力**：确保可以快速回滚
5. **监控告警**：实时监控系统状态

## 相关资源

- [GitHub Actions 文档](https://docs.github.com/en/actions)
- [ArgoCD 文档](https://argo-cd.readthedocs.io/)
- [Terraform 文档](https://www.terraform.io/docs)

---

**上一篇：[GitHub App 开发](W14-github-app-dev.md) | 下一篇：[GitHub Enterprise 企业版](W16-github-enterprise.md)**
