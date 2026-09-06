# GitHub Packages 介绍

## 什么是 GitHub Packages？

GitHub Packages 是一个包管理服务，允许你安全地发布和消费软件包。它与 GitHub 生态系统深度集成。

## 支持的包管理器

| 包管理器 | 语言/平台 | 仓库类型 |
|----------|-----------|----------|
| npm | JavaScript/Node.js | npm |
| NuGet | .NET | nuget |
| RubyGems | Ruby | gem |
| Maven | Java | maven |
| Gradle | Java/Kotlin | gradle |
| Docker | 容器 | docker |
| Helm | Kubernetes | helm |
| Swift | iOS/macOS | swift |

## 使用场景

1. **私有包**：发布公司内部使用的包
2. **公共包**：发布开源包供社区使用
3. **依赖管理**：在 CI/CD 中使用 GitHub Packages 作为包源
4. **容器镜像**：存储和分发 Docker 镜像

## 安装包

### npm

```bash
# 配置 npm 使用 GitHub Packages
echo "@你的用户名:registry=https://npm.pkg.github.com" > .npmrc

# 安装包
npm install @你的用户名/包名
```

### NuGet

```bash
# 添加 GitHub Packages 源
dotnet nuget add source "https://nuget.pkg.github.com/你的用户名/index.json" \
  --name "GitHub" \
  --username "你的用户名" \
  --password "你的令牌"

# 安装包
dotnet add package 包名
```

### Docker

```bash
# 登录到 GitHub Container Registry
echo "你的令牌" | docker login ghcr.io -u 你的用户名 --password-stdin

# 拉取镜像
docker pull ghcr.io/用户名/镜像名:标签
```

## 发布包

### npm

```bash
# 登录
npm login --registry=https://npm.pkg.github.com

# 发布
npm publish
```

### Docker

```bash
# 构建镜像
docker build -t ghcr.io/用户名/镜像名:标签 .

# 推送镜像
docker push ghcr.io/用户名/镜像名:标签
```

## 配置 package.json

```json
{
  "name": "@你的用户名/包名",
  "version": "1.0.0",
  "description": "我的包",
  "main": "index.js",
  "publishConfig": {
    "registry": "https://npm.pkg.github.com"
  }
}
```

## GitHub Actions 集成

### 发布 npm 包

```yaml
name: Publish Package

on:
  release:
    types: [created]

jobs:
  publish:
    runs-on: ubuntu-latest
    
    permissions:
      contents: read
      packages: write
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '20'
        registry-url: 'https://npm.pkg.github.com'
    
    - run: npm ci
    - run: npm publish
      env:
        NODE_AUTH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### 发布 Docker 镜像

```yaml
name: Publish Docker Image

on:
  push:
    tags:
      - 'v*'

jobs:
  publish:
    runs-on: ubuntu-latest
    
    permissions:
      contents: read
      packages: write
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Log in to Container Registry
      uses: docker/login-action@v3
      with:
        registry: ghcr.io
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}
    
    - name: Build and push
      uses: docker/build-push-action@v5
      with:
        push: true
        tags: ghcr.io/${{ github.repository }}:${{ github.ref_name }}
```

## 权限管理

### 仓库级别

在仓库 **Settings** → **Actions** → **General** 中配置：
- **Read**: 读取包
- **Write**: 读取和发布包
- **Admin**: 管理包权限

### 组织级别

在组织 **Settings** → **Packages** 中配置默认权限。

## 版本管理

### 语义化版本

```
MAJOR.MINOR.PATCH
```

- **MAJOR**: 不兼容的 API 变更
- **MINOR**: 向后兼容的新功能
- **PATCH**: 向后兼容的 bug 修复

### 标签

使用 Git 标签来标记版本：
```bash
git tag -a v1.0.0 -m "Release version 1.0.0"
git push origin v1.0.0
```

## 最佳实践

1. **使用语义化版本**：让用户了解变更的性质
2. **编写清晰的发布说明**：说明新功能和修复
3. **自动化发布流程**：使用 GitHub Actions
4. **设置适当的权限**：最小权限原则
5. **定期更新依赖**：保持包的安全性

## 限制

- **存储限制**：
  - GitHub Free: 500 MB
  - GitHub Pro: 2 GB
  - GitHub Team: 2 GB
  - GitHub Enterprise: 50 GB

- **带宽限制**：
  - GitHub Free: 1 GB/月
  - GitHub Pro: 2 GB/月

## 相关资源

- [GitHub Packages 官方文档](https://docs.github.com/en/packages)
- [使用 GitHub Packages 发布包](https://docs.github.com/en/packages/working-with-a-github-packages-registry)
- [GitHub Container Registry](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry)
