# Docker + GitHub Actions 容器化工作流

## 基础 Docker 构建

```yaml
# .github/workflows/docker.yml
name: Docker Build

on:
  push:
    branches: [main]
    tags: ['v*']

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
          type=ref,event=tag
          type=sha,prefix=
          type=raw,value=latest,enable={{is_default_branch}}
    
    - name: Build and push
      uses: docker/build-push-action@v5
      with:
        context: .
        push: true
        tags: ${{ steps.meta.outputs.tags }}
        labels: ${{ steps.meta.outputs.labels }}
```

## 多阶段构建

```dockerfile
# Dockerfile
# Stage 1: Build
FROM node:20-alpine AS builder

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY . .
RUN npm run build

# Stage 2: Production
FROM node:20-alpine AS production

WORKDIR /app

RUN addgroup -g 1001 -S nodejs
RUN adduser -S nextjs -u 1001

COPY --from=builder --chown=nextjs:nodejs /app/dist ./dist
COPY --from=builder --chown=nextjs:nodejs /app/node_modules ./node_modules
COPY --from=builder --chown=nextjs:nodejs /app/package.json ./package.json

USER nextjs

EXPOSE 3000

CMD ["node", "dist/main.js"]
```

## 多平台构建

```yaml
# .github/workflows/docker-multiplatform.yml
name: Docker Multi-Platform

on:
  push:
    tags: ['v*']

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Set up QEMU
      uses: docker/setup-qemu-action@v3
    
    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v3
    
    - name: Login to Container Registry
      uses: docker/login-action@v3
      with:
        registry: ghcr.io
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}
    
    - name: Build and push
      uses: docker/build-push-action@v5
      with:
        context: .
        platforms: linux/amd64,linux/arm64,linux/arm/v7
        push: true
        tags: ghcr.io/${{ github.repository }}:${{ github.ref_name }}
        cache-from: type=gha
        cache-to: type=gha,mode=max
```

## Docker Compose 测试

```yaml
# .github/workflows/docker-compose-test.yml
name: Docker Compose Test

on:
  pull_request:

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Start services
      run: |
        docker-compose up -d
        sleep 30
    
    - name: Run tests
      run: |
        docker-compose exec -T app npm test
    
    - name: Stop services
      if: always()
      run: docker-compose down
```

## 安全扫描

```yaml
# .github/workflows/docker-scan.yml
name: Docker Security Scan

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
    
    - name: Run Trivy vulnerability scanner
      uses: aquasecurity/trivy-action@master
      with:
        image-ref: 'my-app:scan'
        format: 'sarif'
        output: 'trivy-results.sarif'
        severity: 'CRITICAL,HIGH'
    
    - name: Upload scan results
      uses: github/codeql-action/upload-sarif@v3
      with:
        sarif_file: 'trivy-results.sarif'
```

## 镜像优化技巧

### 1. 使用多阶段构建

```dockerfile
# 构建阶段
FROM node:20 AS builder
WORKDIR /app
COPY . .
RUN npm ci && npm run build

# 运行阶段
FROM node:20-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
CMD ["node", "dist/main.js"]
```

### 2. 优化层缓存

```dockerfile
# 先复制依赖文件
COPY package*.json ./
RUN npm ci

# 再复制源代码
COPY . .
RUN npm run build
```

### 3. 使用 `.dockerignore`

```dockerignore
# .dockerignore
node_modules
.git
.github
*.md
.env
.env.local
dist
coverage
```

## 最佳实践

1. **使用多阶段构建**：减小镜像体积
2. **利用缓存**：加速构建过程
3. **扫描漏洞**：确保镜像安全
4. **使用非 root 用户**：提高安全性
5. **添加健康检查**：监控容器状态

## 相关资源

- [Docker 官方文档](https://docs.docker.com/)
- [GitHub Actions Docker 文档](https://docs.github.com/en/actions/use-cases-and-examples/publishing-packages/publishing-docker-images)
- [Trivy 文档](https://trivy.dev/)

---

**上一篇：[GitHub Enterprise 企业版](W16-github-enterprise.md) | 下一篇：[GitHub Code Search 高级搜索](W18-code-search.md)**
