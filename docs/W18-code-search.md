# GitHub Code Search 高级搜索

## 搜索语法

### 基础搜索

```
# 搜索关键词
hello world

# 搜索特定语言
language:javascript

# 搜索特定路径
path:src/components

# 搜索特定文件
filename:package.json

# 组合搜索
language:typescript path:src/api
```

### 高级过滤

```
# 搜索函数定义
def calculate_total

# 搜索类定义
class User

# 搜索导入语句
import.*from.*react

# 搜索特定模式
TODO:.*fix
FIXME:.*
HACK:.*
```

### 正则表达式

```
# 使用正则表达式
/\b\d{3}-\d{4}\b/  # 匹配电话号码格式

# 使用 OR
error OR warning

# 使用 AND
function AND async

# 使用 NOT
TODO NOT test
```

## 搜索范围

### 个人搜索

```bash
# 搜索我的仓库
yourusername search-query

# 搜索特定仓库
owner:yourusername repo:your-repo search-query
```

### 组织搜索

```bash
# 搜索组织仓库
org:your-org search-query

# 搜索组织特定仓库
org:your-org repo:specific-repo search-query
```

### 全局搜索

```bash
# 搜索所有公开仓库
is:public search-query

# 搜索特定语言的所有仓库
is:public language:python search-query
```

## 使用 GitHub CLI 搜索

```bash
# 搜索仓库
gh search repos "machine learning" --language python --stars ">1000"

# 搜索代码
gh search code "function authenticate" --repo owner/repo

# 搜索 issues
gh search issues "bug label:urgent" --state open

# 搜索用户
gh search users "location:beijing" --type user
```

## Code Search API

### REST API

```bash
# 搜索代码
curl -H "Authorization: token $GITHUB_TOKEN" \
  "https://api.github.com/search/code?q=language:python+repo:owner/repo+filename:main.py"

# 搜索仓库
curl -H "Authorization: token $GITHUB_TOKEN" \
  "https://api.github.com/search/repositories?q=machine+learning+language:python+stars:>1000"

# 搜索 issues
curl -H "Authorization: token $GITHUB_TOKEN" \
  "https://api.github.com/search/issues?q=repo:owner/repo+is:issue+label:bug"
```

### GraphQL API

```graphql
query {
  search(query: "language:python stars:>1000", type: REPOSITORY, first: 10) {
    edges {
      node {
        ... on Repository {
          name
          description
          url
          stargazerCount
          primaryLanguage {
            name
          }
        }
      }
    }
  }
}
```

## 搜索技巧

### 1. 精确搜索

```
# 搜索精确短语
"exact phrase"

# 搜索特定文件类型
extension:py

# 搜索特定目录
path:src/components
```

### 2. 组合条件

```
# AND 条件
language:typescript AND path:src

# OR 条件
language:javascript OR language:typescript

# NOT 条件
language:python NOT test
```

### 3. 使用通配符

```
# 通配符搜索
test*.js
*.config.*
```

## 搜索自动化

### GitHub Action 自动搜索

```yaml
# .github/workflows/search.yml
name: Code Search

on:
  schedule:
    - cron: '0 0 * * *'

jobs:
  search:
    runs-on: ubuntu-latest
    steps:
    - name: Search for security issues
      run: |
        gh search code "password" --repo owner/repo --json path
        gh search code "secret" --repo owner/repo --json path
        gh search code "api_key" --repo owner/repo --json path
```

### 使用 Octokit 搜索

```javascript
const { Octokit } = require("@octokit/rest");

const octokit = new Octokit({ auth: process.env.GITHUB_TOKEN });

async function searchCode(query) {
  const results = await octokit.rest.search.code({
    q: query,
    per_page: 100,
  });
  
  return results.data.items;
}

// 搜索所有包含 TODO 的代码
const todos = await searchCode("TODO repo:owner/repo");
```

## 最佳实践

1. **使用精确搜索**：避免搜索结果过多
2. **组合多个条件**：缩小搜索范围
3. **使用正则表达式**：处理复杂模式
4. **保存常用搜索**：提高效率
5. **使用 API 自动化**：批量处理搜索结果

## 相关资源

- [GitHub Code Search 文档](https://docs.github.com/en/search-github/github-code-search)
- [搜索语法文档](https://docs.github.com/en/search-github/github-code-search/understanding-github-code-search-syntax)
- [搜索 API 文档](https://docs.github.com/en/rest/search)

---

**上一篇：[Docker + GitHub Actions 容器化](W17-docker-actions.md) | 下一篇：[数据库 CI/CD 工作流](W19-database-cicd.md)**
