# GitHub 性能优化

## 仓库优化

### 清理历史

```bash
# 垃圾回收
git gc --aggressive --prune=now

# 清理未跟踪文件
git clean -fd

# 压缩历史
git repack -a -d
```

### 减小仓库体积

```bash
# 查看仓库大小
du -sh .git

# 清理大文件历史
git filter-branch --force --index-filter \
  'git rm --cached --ignore-unmatch large-file.zip' \
  --prune-empty --tag-name-filter cat -- --all

# 使用 BFG 清理
bfg --strip-blobs-bigger-than 100M
```

### 使用 .gitignore

```gitignore
# 编辑器
.vscode/
.idea/
*.swp

# 依赖
node_modules/
vendor/

# 构建产物
dist/
build/
*.log

# 环境变量
.env
.env.local

# 系统文件
.DS_Store
Thumbs.db
```

## CI/CD 优化

### 缓存策略

```yaml
# .github/workflows/ci.yml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    
    - name: Cache Node.js
      uses: actions/cache@v4
      with:
        path: ~/.npm
        key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
        restore-keys: |
          ${{ runner.os }}-node-
    
    - name: Cache Docker layers
      uses: actions/cache@v4
      with:
        path: /tmp/.buildx-cache
        key: ${{ runner.os }}-buildx-${{ github.sha }}
        restore-keys: |
          ${{ runner.os }}-buildx-
```

### 并行执行

```yaml
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - run: npm run lint
  
  test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - run: npm test
  
  build:
    needs: [lint, test]
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - run: npm run build
```

### 条件执行

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
    - uses: actions/checkout@v4
    - run: npm run deploy
```

## 克隆优化

### 浅克隆

```bash
# 浅克隆（最近 1 次提交）
git clone --depth 1 https://github.com/user/repo.git

# 浅克隆特定分支
git clone --depth 1 --single-branch --branch main https://github.com/user/repo.git
```

### 部分克隆

```bash
# Blobless 克隆
git clone --filter=blob:none https://github.com/user/repo.git

# Treeless 克隆
git clone --filter=tree:0 https://github.com/user/repo.git
```

### Sparse Checkout

```bash
# 启用 sparse checkout
git sparse-checkout init --cone

# 设置要检出的目录
git sparse-checkout set src/docs src/utils

# 查看 sparse checkout 配置
git sparse-checkout list
```

## GitHub Actions 性能

### 使用更快的 Runner

```yaml
jobs:
  build:
    runs-on: ubuntu-latest  # 更快
    # runs-on: ubuntu-22.04  # 指定版本
```

### 使用矩阵策略

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [18, 20, 22]
      fail-fast: false
```

### 使用并发控制

```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```

## API 性能

### 分页

```javascript
// 使用分页获取所有数据
async function getAllItems() {
  const items = [];
  let page = 1;
  
  while (true) {
    const response = await octokit.rest.issues.listForRepo({
      owner,
      repo,
      per_page: 100,
      page,
    });
    
    if (response.data.length === 0) break;
    items.push(...response.data);
    page++;
  }
  
  return items;
}
```

### 缓存

```javascript
// 使用缓存减少 API 调用
const cache = new Map();

async function getCachedData(key, fetchFn) {
  if (cache.has(key)) {
    return cache.get(key);
  }
  
  const data = await fetchFn();
  cache.set(key, data);
  return data;
}
```

### GraphQL

```graphql
# 使用 GraphQL 减少 API 调用
query {
  repository(owner: "your-org", name: "your-repo") {
    issues(first: 100) {
      edges {
        node {
          title
          state
          labels(first: 5) {
            edges {
              node {
                name
              }
            }
          }
        }
      }
    }
  }
}
```

## 监控和优化

### 性能监控

```yaml
# .github/workflows/performance.yml
name: Performance Monitor

on:
  schedule:
    - cron: '0 * * * *'  # 每小时

jobs:
  monitor:
    runs-on: ubuntu-latest
    steps:
    - name: Check build time
      run: |
        # 记录构建时间
        echo "Build time: ${{ github.run_duration }}"
```

### 优化建议

```markdown
## 优化清单

### 仓库优化
- [ ] 使用 .gitignore
- [ ] 清理大文件历史
- [ ] 使用浅克隆

### CI/CD 优化
- [ ] 使用缓存
- [ ] 并行执行
- [ ] 条件执行

### API 优化
- [ ] 使用分页
- [ ] 使用缓存
- [ ] 使用 GraphQL
```

## 最佳实践

1. **定期清理**：定期清理仓库历史
2. **使用缓存**：充分利用缓存
3. **并行执行**：尽可能并行执行任务
4. **条件执行**：只在必要时执行任务
5. **监控性能**：持续监控和优化性能

## 相关资源

- [Git 性能优化](https://git-scm.com/book/en/v2/Git-Tools-Maintenance-and-Data-Recovery)
- [GitHub Actions 性能](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)
- [GitHub API 最佳实践](https://docs.github.com/en/rest/overview/resources-in-the-rest-api#rate-limiting)

---

**上一篇：[GitHub 项目管理](W27-project-management.md) | 下一篇：[GitHub 开源指南](W29-open-source-guide.md)**
