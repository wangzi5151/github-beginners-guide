# 练习 13：Docker 部署实战

## 学习目标

- 编写 Dockerfile
- 使用 GitHub Actions 构建镜像
- 推送到容器注册表

## 步骤

### 步骤 1：创建 Dockerfile

```dockerfile
FROM node:20-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY . .

EXPOSE 3000

CMD ["node", "src/index.js"]
```

### 步骤 2：创建 .dockerignore

```
node_modules
.git
.github
*.md
.env
```

### 步骤 3：创建工作流

```yaml
# .github/workflows/docker.yml
name: Docker Build

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
    
    - name: Login to GHCR
      uses: docker/login-action@v3
      with:
        registry: ghcr.io
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}
    
    - name: Build and push
      uses: docker/build-push-action@v5
      with:
        context: .
        push: true
        tags: |
          ghcr.io/${{ github.repository }}:latest
          ghcr.io/${{ github.repository }}:${{ github.ref_name }}
```

## 实战任务

1. 创建 Dockerfile
2. 创建 .dockerignore
3. 创建 Docker 构建工作流
4. 测试镜像构建和推送

## 验证清单

- [ ] 能够编写 Dockerfile
- [ ] 能够配置 .dockerignore
- [ ] 能够创建 Docker 构建工作流
- [ ] 能够推送到 GHCR

## 下一步

继续 [练习 14：安全扫描实战](exercise-14-security-scan.md)
