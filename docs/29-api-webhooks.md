# GitHub API 与 Webhooks 完全指南

## 第一章：GitHub API 概述

### 1.1 REST API 与 GraphQL API 简介

GitHub 提供两种风格的 API 供开发者使用：REST API 和 GraphQL API。这两种 API 各有特点，适用于不同的使用场景。理解它们的区别和适用场景，对于选择合适的 API 至关重要。

**REST API** 是 GitHub 最早提供的 API 风格，遵循 RESTful 架构设计原则。它使用多个端点（endpoint）来访问不同的资源，每个端点对应一个特定的 URL。REST API 使用标准的 HTTP 方法（GET、POST、PATCH、PUT、DELETE）来执行不同的操作，使用 HTTP 状态码来表示操作结果。

**GraphQL API** 是 GitHub 在 2016 年推出的 API 风格，由 Facebook 开发。它使用单一端点（/graphql）来访问所有资源，客户端可以精确指定需要获取的数据字段。GraphQL API 使用查询语言来描述数据需求，可以一次请求获取多个关联资源。

### 1.2 REST API 与 GraphQL API 对比

| 特性 | REST API | GraphQL API |
|------|----------|-------------|
| 查询方式 | 多个端点 | 单一端点 |
| 数据获取 | 固定结构 | 按需获取 |
| 过载传输 | 可能 | 不会 |
| 学习难度 | 低 | 中 |
| 版本管理 | 简单 | 内置 |
| 缓存支持 | 好 | 需要额外配置 |
| 实时更新 | 需要轮询 | 支持订阅 |

**REST API 的优势**：
- 学习曲线低，易于上手
- 使用标准 HTTP 协议，工具支持广泛
- 缓存机制完善，性能优秀
- 文档丰富，社区支持强大

**GraphQL API 的优势**：
- 精确获取需要的数据，避免过载传输
- 一次请求获取多个关联资源
- 强类型系统，减少错误
- 支持实时订阅

### 1.3 API 版本控制

GitHub API 使用版本控制来管理 API 的演进：

```bash
# REST API 版本（通过 Accept 头部）
curl -H "Accept: application/vnd.github+json" \
  https://api.github.com/repos/octocat/Hello-World

# 指定 API 版本
curl -H "Accept: application/vnd.github.v3+json" \
  https://api.github.com/repos/octocat/Hello-World

# 使用预览功能
curl -H "Accept: application/vnd.github.antiope-preview+json" \
  https://api.github.com/repos/octocat/Hello-World/check-runs
```

### 1.4 API 端点结构

REST API 的端点结构遵循以下模式：

```
https://api.github.com/{resource}
```

常见的资源路径：
- `/users/{username}`：用户信息
- `/repos/{owner}/{repo}`：仓库信息
- `/repos/{owner}/{repo}/issues`：Issue 列表
- `/repos/{owner}/{repo}/pulls`：Pull Request 列表
- `/repos/{owner}/{repo}/releases`：Release 列表
- `/orgs/{org}`：组织信息
- `/orgs/{org}/teams`：团队列表

## 第二章：REST API 基础用法

REST API 是 GitHub 最常用的 API 风格，本章将详细介绍 REST API 的基础用法，包括获取资源、创建资源、更新资源和删除资源等操作。通过学习本章内容，开发者可以掌握使用 REST API 与 GitHub 交互的基本技能。

### 2.1 获取仓库信息

获取仓库信息是最常见的 API 调用之一。通过 REST API，开发者可以获取仓库的详细信息，包括仓库名称、描述、星标数、分支列表等。这些信息可以用于构建各种工具和集成应用。

```bash
# 使用 curl
curl -H "Authorization: token YOUR_TOKEN" \
  https://api.github.com/repos/octocat/Hello-World

# 使用 gh CLI
gh api repos/octocat/Hello-World

# 使用 Python
import requests

response = requests.get(
    'https://api.github.com/repos/octocat/Hello-World',
    headers={'Authorization': 'token YOUR_TOKEN'}
)
repo = response.json()
print(f"Stars: {repo['stargazers_count']}")
```

### 2.2 响应格式

GitHub API 返回 JSON 格式的响应数据。响应数据包含资源的详细信息，开发者可以根据需要解析和使用这些信息。了解响应数据的结构对于正确使用 API 至关重要。以下是仓库信息 API 的响应示例，包含了常用的字段。

```json
{
  "id": 1296269,
  "node_id": "MDEwOlJlcG9zaXRvcnkxMjk2MjY5",
  "name": "Hello-World",
  "full_name": "octocat/Hello-World",
  "private": false,
  "owner": {
    "login": "octocat",
    "id": 583231,
    "avatar_url": "https://avatars.githubusercontent.com/u/583231?v=4"
  },
  "html_url": "https://github.com/octocat/Hello-World",
  "description": "This your first repo!",
  "fork": false,
  "stargazers_count": 2500,
  "watchers_count": 2500,
  "forks_count": 1500,
  "open_issues_count": 5,
  "default_branch": "main",
  "created_at": "2011-01-26T19:01:12Z",
  "updated_at": "2023-01-01T00:00:00Z"
}
```

### 2.3 分页处理

当返回结果较多时，GitHub API 会使用分页机制。默认情况下，每页返回 30 条记录，最多可以设置为 100 条。开发者需要了解分页机制，以便获取所有数据。GitHub API 在响应头部中包含分页信息，开发者可以根据这些信息获取下一页数据。

```bash
# REST API 分页
# 第一页
curl "https://api.github.com/repos/octocat/Hello-World/issues?page=1&per_page=100"

# 响应头部包含 Link
# Link: <https://api.github.com/repos/octocat/Hello-World/issues?page=2>; rel="next",
#        <https://api.github.com/repos/octocat/Hello-World/issues?page=5>; rel="last"

# 使用 gh CLI 自动处理分页
gh api repos/octocat/Hello-World/issues --paginate

# Python 分页处理
import requests

def get_all_issues(owner, repo):
    issues = []
    page = 1
    while True:
        response = requests.get(
            f'https://api.github.com/repos/{owner}/{repo}/issues',
            params={'page': page, 'per_page': 100},
            headers={'Authorization': 'token YOUR_TOKEN'}
        )
        page_issues = response.json()
        if not page_issues:
            break
        issues.extend(page_issues)
        page += 1
    return issues
```

### 2.4 过滤和排序

GitHub API 支持丰富的过滤和排序参数，帮助开发者精确获取需要的数据。通过使用这些参数，可以减少不必要的数据传输，提高 API 调用效率。不同的 API 端点支持不同的过滤和排序参数，开发者应该查阅 API 文档了解具体支持的参数。

```bash
# 过滤 Issues
curl "https://api.github.com/repos/octocat/Hello-World/issues?state=open&labels=bug&sort=created&direction=desc"

# 过滤 PR
curl "https://api.github.com/repos/octocat/Hello-World/pulls?state=open&base=main&head=octocat:feature"

# 搜索
curl "https://api.github.com/search/repositories?q=language:python&sort=stars&order=desc"

# 搜索 Issues
curl "https://api.github.com/search/issues?q=repo:octocat/Hello-World+is:issue+is:open"
```

### 2.5 创建和更新资源

除了获取数据，REST API 还可以用于创建和更新资源。通过发送 POST、PATCH、PUT 或 DELETE 请求，开发者可以管理 GitHub 上的各种资源。创建和更新资源时，需要在请求体中提供相应的数据。以下是创建和更新 Issue、评论等资源的示例。

```bash
# 创建 Issue
curl -X POST \
  -H "Authorization: token YOUR_TOKEN" \
  -H "Accept: application/vnd.github.v3+json" \
  https://api.github.com/repos/octocat/Hello-World/issues \
  -d '{"title":"Bug Report","body":"Description of the bug"}'

# 更新 Issue
curl -X PATCH \
  -H "Authorization: token YOUR_TOKEN" \
  -H "Accept: application/vnd.github.v3+json" \
  https://api.github.com/repos/octocat/Hello-World/issues/1 \
  -d '{"state":"closed"}'

# 添加评论
curl -X POST \
  -H "Authorization: token YOUR_TOKEN" \
  -H "Accept: application/vnd.github.v3+json" \
  https://api.github.com/repos/octocat/Hello-World/issues/1/comments \
  -d '{"body":"This is a comment"}'
```

## 第三章：认证方式

认证是访问 GitHub API 的前提。GitHub 支持多种认证方式，每种方式适用于不同的场景。选择合适的认证方式对于安全和便捷性都非常重要。本章将详细介绍各种认证方式的特点和使用方法。

### 3.1 Personal Access Token (PAT)

Personal Access Token（PAT）是最简单的认证方式，适用于个人脚本和 CI/CD 流程。PAT 可以替代密码使用，提供与密码相同的访问权限。开发者应该为不同的用途创建不同的 PAT，并设置合理的过期时间。

```bash
# 创建 PAT
# Settings → Developer settings → Personal access tokens

# 使用 PAT 认证
curl -H "Authorization: token ghp_xxxxxxxxxxxx" \
  https://api.github.com/user

# 或使用 Bearer
curl -H "Authorization: Bearer ghp_xxxxxxxxxxxx" \
  https://api.github.com/user

# gh CLI 认证
gh auth login
gh auth status
```

### 3.2 GitHub App 认证

GitHub App 是更安全的认证方式，适用于集成应用和自动化工具。GitHub App 使用 JWT（JSON Web Token）进行认证，令牌有效期短（1小时），安全性更高。GitHub App 可以安装到仓库或组织，拥有细粒度的权限控制。以下是使用 Python 实现 GitHub App 认证的示例代码。

```python
import jwt
import time
import requests

class GitHubApp:
    def __init__(self, app_id, private_key_path):
        self.app_id = app_id
        with open(private_key_path, 'r') as f:
            self.private_key = f.read()
    
    def get_jwt(self):
        now = int(time.time())
        payload = {
            'iat': now - 60,
            'exp': now + 600,
            'iss': self.app_id
        }
        return jwt.encode(payload, self.private_key, algorithm='RS256')
    
    def get_installation_token(self, installation_id):
        jwt_token = self.get_jwt()
        headers = {
            'Authorization': f'Bearer {jwt_token}',
            'Accept': 'application/vnd.github.v3+json'
        }
        
        response = requests.post(
            f'https://api.github.com/app/installations/{installation_id}/access_tokens',
            headers=headers
        )
        return response.json()['token']
```

### 3.3 OAuth App 认证

OAuth App 适用于需要用户授权的第三方应用。通过 OAuth 流程，用户可以授权应用访问其 GitHub 账户的特定资源，而无需共享密码。OAuth App 的授权流程包括用户授权、获取授权码、换取访问令牌等步骤。以下是 OAuth 认证流程的详细说明。

```bash
# 1. 创建 OAuth App
# Settings → Developer settings → OAuth Apps → New OAuth App

# 2. 获取授权码
# 重定向用户到：
https://github.com/login/oauth/authorize?client_id=CLIENT_ID&redirect_uri=CALLBACK_URL&scope=repo

# 3. 用授权码换取 Token
curl -X POST https://github.com/login/oauth/access_token \
  -H "Accept: application/json" \
  -d "client_id=CLIENT_ID" \
  -d "client_secret=CLIENT_SECRET" \
  -d "code=AUTHORIZATION_CODE"

# 4. 使用 Token
curl -H "Authorization: token ACCESS_TOKEN" \
  https://api.github.com/user
```

### 3.4 认证方式对比

选择合适的认证方式需要考虑多个因素，包括安全性、便捷性、适用场景等。以下是各种认证方式的详细对比，帮助开发者根据实际需求做出选择。一般来说，个人脚本和 CI/CD 使用 PAT，集成应用使用 GitHub App，需要用户授权的第三方应用使用 OAuth App。

| 方式 | 适用场景 | 有效期 | 权限范围 |
|------|---------|--------|---------|
| PAT | 个人脚本、CI/CD | 可设置过期 | 用户级别 |
| GitHub App | 集成应用、机器人 | 1小时（可续期） | 安装级别 |
| OAuth App | 第三方应用 | 无过期（可撤销） | 用户授权 |
| GITHUB_TOKEN | GitHub Actions | 任务期间 | 仓库级别 |

## 第四章：常用 API 端点示例

GitHub API 提供了丰富的端点，覆盖了 GitHub 的各种功能。本章将介绍最常用的 API 端点，包括用户、仓库、Issue、Pull Request、Actions 和 Releases 等。通过这些示例，开发者可以快速上手使用 GitHub API。

### 4.1 用户相关 API

用户 API 用于获取用户信息、用户的仓库、用户的组织等。通过这些 API，可以构建用户资料页面、分析用户活动等。以下是常用的用户 API 端点示例。

```bash
# 获取当前用户信息
gh api user

# 获取特定用户信息
gh api users/{username}

# 获取用户的仓库
gh api users/{username}/repos

# 获取用户的组织
gh api users/{username}/orgs

# 获取用户的 Gists
gh api users/{username}/gists
```

### 4.2 仓库相关 API

仓库 API 用于管理仓库，包括获取仓库信息、创建仓库、删除仓库、获取分支列表等。这些 API 是构建仓库管理工具的基础。以下是常用的仓库 API 端点示例。

```bash
# 获取仓库信息
gh api repos/{owner}/{repo}

# 创建仓库
gh api user/repos -f name="new-repo" -f description="New repository"

# 删除仓库
gh api -X DELETE repos/{owner}/{repo}

# 获取仓库语言
gh api repos/{owner}/{repo}/languages

# 获取仓库贡献者
gh api repos/{owner}/{repo}/contributors

# 获取仓库标签
gh api repos/{owner}/{repo}/tags

# 获取仓库分支
gh api repos/{owner}/{repo}/branches
```

### 4.3 Issue 相关 API

Issue API 用于管理 Issue，包括创建、更新、关闭 Issue，以及管理标签和评论。Issue 是项目管理的重要工具，通过 API 可以实现自动化的 Issue 管理。以下是常用的 Issue API 端点示例。

```bash
# 列出 Issues
gh api repos/{owner}/{repo}/issues

# 创建 Issue
gh api repos/{owner}/{repo}/issues \
  -f title="Bug: Something broken" \
  -f body="Description of the bug" \
  -f 'labels[]="bug"' \
  -f 'assignees[]="username"'

# 更新 Issue
gh api -X PATCH repos/{owner}/{repo}/issues/{issue_number} \
  -f state="closed"

# 添加评论
gh api repos/{owner}/{repo}/issues/{issue_number}/comments \
  -f body="This is a comment"

# 添加标签
gh api -X POST repos/{owner}/{repo}/issues/{issue_number}/labels \
  -f 'labels[]="enhancement"'

# 移除标签
gh api -X DELETE repos/{owner}/{repo}/issues/{issue_number}/labels/{label_name}
```

### 4.4 Pull Request 相关 API

Pull Request API 用于管理 Pull Request，包括创建、审查、合并 PR。Pull Request 是代码协作的核心功能，通过 API 可以实现自动化的代码审查和合并流程。以下是常用的 Pull Request API 端点示例。

```bash
# 列出 PR
gh api repos/{owner}/{repo}/pulls

# 创建 PR
gh api repos/{owner}/{repo}/pulls \
  -f title="New feature" \
  -f body="Description" \
  -f head="feature-branch" \
  -f base="main"

# 合并 PR
gh api -X PUT repos/{owner}/{repo}/pulls/{pull_number}/merge

# 获取 PR 文件变更
gh api repos/{owner}/{repo}/pulls/{pull_number}/files

# 审查 PR
gh api -X POST repos/{owner}/{repo}/pulls/{pull_number}/reviews \
  -f event="APPROVE" \
  -f body="Looks good!"

# 获取 PR 评论
gh api repos/{owner}/{repo}/pulls/{pull_number}/comments
```

### 4.5 Actions 相关 API

Actions API 用于管理 GitHub Actions，包括查看工作流、触发运行、下载日志等。通过 Actions API，可以实现 CI/CD 流程的自动化管理。以下是常用的 Actions API 端点示例。

```bash
# 列出工作流
gh api repos/{owner}/{repo}/actions/workflows

# 触发工作流
gh api -X POST repos/{owner}/{repo}/actions/workflows/{workflow_id}/dispatches \
  -f ref="main"

# 获取运行列表
gh api repos/{owner}/{repo}/actions/runs

# 下载日志
gh api repos/{owner}/{repo}/actions/runs/{run_id}/logs

# 获取制品
gh api repos/{owner}/{repo}/actions/runs/{run_id}/artifacts

# 取消运行
gh api -X POST repos/{owner}/{repo}/actions/runs/{run_id}/cancel
```

### 4.6 Releases 相关 API

Releases API 用于管理版本发布，包括创建、编辑、删除 Release，以及上传发布资产。版本发布是软件交付的重要环节，通过 API 可以实现自动化的发布流程。以下是常用的 Releases API 端点示例。

```bash
# 列出 Releases
gh api repos/{owner}/{repo}/releases

# 创建 Release
gh api repos/{owner}/{repo}/releases \
  -f tag_name="v1.0.0" \
  -f name="Release v1.0.0" \
  -f body="Release notes" \
  -f draft=false \
  -f prerelease=false

# 上传 Release Asset
gh api repos/{owner}/{repo}/releases/{release_id}/assets \
  -X POST \
  -H "Content-Type: application/zip" \
  --data-binary "@release.zip" \
  -f name="release.zip"

# 删除 Release
gh api -X DELETE repos/{owner}/{repo}/releases/{release_id}
```

## 第五章：GraphQL API 基础

GraphQL 是一种用于 API 的查询语言，由 Facebook 开发并开源。GitHub 在 2016 年推出了 GraphQL API，允许客户端精确指定需要获取的数据。与 REST API 相比，GraphQL API 可以减少数据传输量，提高查询效率。本章将介绍 GraphQL API 的基础知识和使用方法。

### 5.1 查询语法

GraphQL 使用查询语言来描述数据需求。查询是一个 JSON 格式的字符串，定义了需要获取的资源和字段。GraphQL 查询支持嵌套，可以一次获取多个关联资源。以下是 GraphQL 查询的基本语法示例。

```graphql
# 基本查询
query {
  viewer {
    login
    name
    email
  }
}

# 带参数的查询
query {
  repository(owner: "octocat", name: "Hello-World") {
    name
    description
    stargazerCount
    forkCount
  }
}

# 嵌套查询
query {
  repository(owner: "octocat", name: "Hello-World") {
    issues(first: 10, states: OPEN) {
      nodes {
        title
        body
        author {
          login
        }
        createdAt
      }
    }
  }
}
```

### 5.2 变更操作

除了查询数据，GraphQL 还支持变更操作（Mutation）。变更操作用于创建、更新或删除资源。变更操作的语法与查询类似，但使用 `mutation` 关键字。以下是 GraphQL 变更操作的示例。

```graphql
# 创建 Issue
mutation {
  createIssue(input: {
    repositoryId: "REPO_ID"
    title: "Bug Report"
    body: "Description of the bug"
  }) {
    issue {
      number
      title
      url
    }
  }
}

# 添加评论
mutation {
  addComment(input: {
    subjectId: "ISSUE_ID"
    body: "This is a comment"
  }) {
    commentEdge {
      node {
        body
        author {
          login
        }
      }
    }
  }
}
```

### 5.3 使用 gh CLI 执行 GraphQL

gh CLI 是使用 GraphQL API 最简单的方式。gh CLI 已经内置了认证和格式化功能，开发者可以直接使用 `gh api graphql` 命令执行 GraphQL 查询。以下是使用 gh CLI 执行 GraphQL 查询的示例。

```bash
# 基本查询
gh api graphql -f query='
{
  viewer {
    login
    name
  }
}'

# 带变量的查询
gh api graphql -f query='
query($repo: String!) {
  repository(owner: "octocat", name: $repo) {
    stargazerCount
  }
}' -f repo="Hello-World"

# 复杂查询
gh api graphql -f query='
{
  viewer {
    repositories(first: 10, orderBy: {field: STARGAZERS, direction: DESC}) {
      nodes {
        name
        stargazerCount
        primaryLanguage {
          name
        }
      }
    }
  }
}'
```

### 5.4 GraphQL 分页

GraphQL 使用游标（Cursor）进行分页。游标是一个不透明的字符串，标识列表中的位置。通过使用游标，可以获取列表的下一页或上一页数据。GraphQL 的分页模型使用 `pageInfo` 对象来描述分页状态，包括是否有下一页、是否有上一页、下一页游标、上一页游标等信息。

```graphql
# 使用游标分页
query {
  repository(owner: "octocat", name: "Hello-World") {
    issues(first: 10, after: "Y3Vyc29yOnYyOpK5MjAyMy0wMS0wMVQwMDowMDowMFo=") {
      pageInfo {
        hasNextPage
        endCursor
      }
      nodes {
        title
      }
    }
  }
}
```

### 5.5 GraphQL 与 REST 选择建议

选择使用 GraphQL 还是 REST API 取决于具体的使用场景。两种 API 风格各有优势，开发者应该根据实际需求做出选择。以下是选择建议，帮助开发者做出明智的决定。

**使用 GraphQL 的场景**：
- 需要获取多个关联资源
- 只需要部分字段，减少数据传输
- 移动端应用，带宽有限
- 复杂的嵌套查询需求

**使用 REST 的场景**：
- 简单的 CRUD 操作
- 需要缓存支持
- 文件上传/下载操作
- Webhook 处理

## 第六章：Octokit 官方 SDK 使用

为了简化 GitHub API 的使用，GitHub 提供了官方的 SDK 库 Octokit。Octokit 支持多种编程语言，包括 JavaScript/TypeScript、Python、Go 等。使用 Octokit 可以避免手动处理 HTTP 请求、认证、分页等复杂逻辑，让开发者专注于业务逻辑。本章将介绍如何使用 Octokit 与 GitHub API 交互。

### 6.1 JavaScript/TypeScript Octokit

JavaScript/TypeScript 是最常用的编程语言之一，Octokit 提供了完善的 JavaScript/TypeScript 支持。通过 npm 安装 `@octokit/rest` 包，即可在项目中使用 Octokit。以下是使用 JavaScript Octokit 的示例代码。

```bash
# 安装
npm install @octokit/rest
```

```javascript
const { Octokit } = require("@octokit/rest");

// 初始化
const octokit = new Octokit({
  auth: "YOUR_TOKEN"
});

// 获取仓库信息
async function getRepo() {
  const { data } = await octokit.repos.get({
    owner: "octocat",
    repo: "Hello-World"
  });
  console.log(`Stars: ${data.stargazers_count}`);
}

// 创建 Issue
async function createIssue() {
  const { data } = await octokit.issues.create({
    owner: "octocat",
    repo: "Hello-World",
    title: "Bug Report",
    body: "Description of the bug"
  });
  console.log(`Issue created: ${data.html_url}`);
}

// 使用 GraphQL
async function getRepoWithGraphQL() {
  const { data } = await octokit.graphql(`
    query {
      repository(owner: "octocat", name: "Hello-World") {
        stargazerCount
        issues(first: 10, states: OPEN) {
          nodes {
            title
          }
        }
      }
    }
  `);
  console.log(data);
}
```

### 6.2 Python PyGithub

Python 是数据科学和自动化脚本开发的首选语言。PyGithub 是 Python 社区最流行的 GitHub API 库，提供了完整的 GitHub API 封装。通过 pip 安装 PyGithub，即可在 Python 项目中使用。以下是使用 PyGithub 的示例代码。

```bash
# 安装
pip install PyGithub
```

```python
from github import Github

# 初始化
g = Github("YOUR_TOKEN")

# 获取仓库
repo = g.get_repo("octocat/Hello-World")
print(f"Stars: {repo.stargazers_count}")

# 创建 Issue
issue = repo.create_issue(
    title="Bug Report",
    body="Description of the bug"
)
print(f"Issue created: {issue.html_url}")

# 获取 Issues
for issue in repo.get_issues(state="open"):
    print(f"#{issue.number}: {issue.title}")

# 创建 PR
pr = repo.create_pull(
    title="New feature",
    body="Description",
    head="feature-branch",
    base="main"
)
print(f"PR created: {pr.html_url}")
```

### 6.3 Go go-github

Go 是构建高性能服务端应用的流行语言。go-github 是 Go 社区的 GitHub API 库，由 Google 维护。go-github 提供了完整的 GitHub API 封装，支持类型安全的 API 调用。以下是使用 go-github 的示例代码。

```go
package main

import (
    "context"
    "fmt"
    "github.com/google/go-github/v58/github"
    "golang.org/x/oauth2"
)

func main() {
    ctx := context.Background()
    ts := oauth2.StaticTokenSource(
        &oauth2.Token{AccessToken: "YOUR_TOKEN"},
    )
    tc := oauth2.NewClient(ctx, ts)
    client := github.NewClient(tc)

    // 获取仓库
    repo, _, err := client.Repositories.Get(ctx, "octocat", "Hello-World")
    if err != nil {
        fmt.Println(err)
        return
    }
    fmt.Printf("Stars: %d\n", repo.GetStargazersCount())

    // 创建 Issue
    issueRequest := &github.IssueRequest{
        Title: github.String("Bug Report"),
        Body:  github.String("Description of the bug"),
    }
    issue, _, err := client.Issues.Create(ctx, "octocat", "Hello-World", issueRequest)
    if err != nil {
        fmt.Println(err)
        return
    }
    fmt.Printf("Issue created: %s\n", issue.GetHTMLURL())
}
```

### 6.4 使用 gh CLI

gh CLI 是 GitHub 官方提供的命令行工具，是使用 GitHub API 最简单的方式。gh CLI 已经内置了认证、格式化、分页等功能，开发者无需处理复杂的 API 细节。以下是使用 gh CLI 的常用命令示例。

```bash
# 获取仓库信息
gh api repos/{owner}/{repo}

# 创建 Issue
gh issue create --repo {owner}/{repo} --title "Bug" --body "Description"

# 列出 PR
gh pr list --repo {owner}/{repo}

# 触发 Action
gh workflow run workflow.yml --ref main

# GraphQL 查询
gh api graphql -f query='{ viewer { login } }'

# 格式化输出
gh api repos/{owner}/{repo} --jq '.stargazers_count'
```

## 第七章：Webhooks 概述与配置

Webhook 是 GitHub 提供的实时事件通知机制。通过配置 Webhook，当仓库发生特定事件时，GitHub 会向指定的 URL 发送 HTTP POST 请求。Webhook 是构建 GitHub 集成应用的基础，可以实现自动化的事件处理。本章将介绍 Webhook 的基本概念和配置方法。

### 7.1 什么是 Webhook

Webhook 是一种 HTTP 回调机制，当 GitHub 仓库发生特定事件时，会向预先配置的 URL 发送 HTTP POST 请求。与传统的轮询方式相比，Webhook 提供了实时性更好、效率更高的事件通知机制。开发者可以在接收端处理这些事件，实现各种自动化功能。

Webhook 的工作原理如下：首先在 GitHub 仓库中配置 Webhook，指定接收事件的 URL 和要监听的事件类型。当仓库发生事件时（如代码推送、Issue 创建等），GitHub 会向配置的 URL 发送 HTTP POST 请求，请求体包含事件的详细信息。接收服务器处理请求并返回 HTTP 200 响应，表示事件已成功处理。

### 7.2 Webhook 的优势

Webhook 相比传统的轮询方式有多个显著优势，使其成为构建实时集成应用的首选方案。

**实时性**：事件发生后立即通知，不需要等待轮询间隔。这对于需要实时响应的场景（如自动部署、即时通知）非常重要。

**减少轮询**：不需要定期检查 API，节省 API 调用配额。GitHub API 有速率限制，使用 Webhook 可以避免不必要的 API 调用。

**节省资源**：只在事件发生时才发送请求，减少服务器负载。轮询方式需要定期发送请求，即使没有新事件也会消耗资源。

**自动化**：可以触发自动化流程，如部署、通知、代码审查等。Webhook 是构建 CI/CD 流水线和自动化工具的基础。

### 7.3 配置 Webhook

Webhook 可以通过 GitHub CLI 或网页界面进行配置。配置 Webhook 时需要指定接收事件的 URL、要监听的事件类型、内容格式等信息。以下是使用 CLI 创建和管理 Webhook 的示例。

```bash
# 使用 CLI 创建 Webhook
gh api repos/{owner}/{repo}/hooks \
  -X POST \
  -f '{
    "name": "web",
    "active": true,
    "events": ["push", "pull_request", "issues"],
    "config": {
      "url": "https://your-server.com/webhook",
      "content_type": "json",
      "secret": "your-webhook-secret",
      "insecure_ssl": "0"
    }
  }'

# 查看 Webhook 列表
gh api repos/{owner}/{repo}/hooks

# 更新 Webhook
gh api -X PATCH repos/{owner}/{repo}/hooks/{hook_id} \
  -f '{
    "active": true,
    "events": ["push", "pull_request"]
  }'

# 删除 Webhook
gh api -X DELETE repos/{owner}/{repo}/hooks/{hook_id}
```

### 7.4 Webhook 配置选项

配置 Webhook 时需要了解各个选项的含义和作用。合理配置这些选项可以确保 Webhook 正常工作，同时保证安全性。以下是 Webhook 配置选项的详细说明。

| 选项 | 说明 |
|------|------|
| Payload URL | 接收事件的 URL |
| Content type | application/json 或 application/x-www-form-urlencoded |
| Secret | 用于验证签名的密钥 |
| SSL verification | 是否验证 SSL 证书 |
| Active | 是否启用 |
| Events | 触发事件列表 |

### 7.5 Webhook 事件列表

GitHub 支持多种 Webhook 事件，覆盖了仓库的各种活动。开发者可以根据需求选择监听特定的事件类型。以下是常用的 Webhook 事件类型及其触发时机的详细说明。

| 事件 | 触发时机 |
|------|---------|
| push | 代码推送到仓库 |
| pull_request | PR 创建、更新、合并、关闭 |
| issues | Issue 创建、更新、关闭、重新打开 |
| issue_comment | Issue 或 PR 上的评论 |
| release | Release 发布、编辑、删除 |
| create | 分支或标签创建 |
| delete | 分支或标签删除 |
| fork | 仓库被 fork |
| watch | 仓库被 star |
| workflow_run | GitHub Actions 运行完成 |
| workflow_job | GitHub Actions Job 状态变更 |
| check_run | Check Run 状态变更 |
| check_suite | Check Suite 状态变更 |
| deployment | 部署创建 |
| deployment_status | 部署状态变更 |
| gollum | Wiki 页面编辑 |
| member | 协作者添加或移除 |
| membership | 团队成员添加或移除 |
| organization | 组织设置变更 |
| page_build | GitHub Pages 构建完成 |
| project | Project 创建、更新、删除 |
| project_card | Project Card 创建、更新、删除 |
| project_column | Project Column 创建、更新、删除 |
| public | 仓库从私有变为公开 |
| pull_request_review | PR 审查提交 |
| pull_request_review_comment | PR 审查评论 |
| repository | 仓库创建、删除、归档 |
| status | Commit 状态变更 |
| team | 团队创建、更新、删除 |
| team_add | 团队添加仓库访问 |

## 第八章：Webhook 事件类型详解

每种 Webhook 事件类型都有其特定的数据结构和触发条件。了解事件的详细结构对于正确处理 Webhook 事件至关重要。本章将详细介绍几种常用事件类型的数据结构和处理方法。

### 8.1 Push 事件

Push 事件在代码推送到仓库时触发。这是最常用的 Webhook 事件之一，常用于触发 CI/CD 流水线、发送通知等。Push 事件包含推送的分支、提交信息、推送者等详细信息。以下是 Push 事件的数据结构示例。

```json
{
  "ref": "refs/heads/main",
  "before": "0000000000000000000000000000000000000000",
  "after": "1234567890abcdef1234567890abcdef12345678",
  "repository": {
    "id": 1296269,
    "name": "Hello-World",
    "full_name": "octocat/Hello-World"
  },
  "pusher": {
    "name": "octocat",
    "email": "octocat@github.com"
  },
  "commits": [
    {
      "id": "1234567890abcdef1234567890abcdef12345678",
      "message": "Fix bug",
      "timestamp": "2023-01-01T00:00:00Z",
      "author": {
        "name": "octocat",
        "email": "octocat@github.com"
      }
    }
  ]
}
```

### 8.2 Pull Request 事件

Pull Request 事件在 PR 状态变更时触发。PR 事件的 `action` 字段标识具体的变更类型，如 `opened`（创建）、`closed`（关闭）、`merged`（合并）、`reviewed`（审查）等。通过监听 PR 事件，可以实现自动化的代码审查、合并通知等功能。以下是 PR 事件的数据结构示例。

```json
{
  "action": "opened",
  "number": 42,
  "pull_request": {
    "id": 123456,
    "number": 42,
    "title": "New feature",
    "body": "Description of the feature",
    "state": "open",
    "user": {
      "login": "octocat"
    },
    "head": {
      "ref": "feature-branch",
      "sha": "abc123"
    },
    "base": {
      "ref": "main",
      "sha": "def456"
    }
  }
}
```

### 8.3 Issue 事件

Issue 事件在 Issue 状态变更时触发。Issue 事件的 `action` 字段标识具体的变更类型，如 `opened`（创建）、`closed`（关闭）、`reopened`（重新打开）、`edited`（编辑）等。通过监听 Issue 事件，可以实现自动化的 Issue 管理、通知等功能。以下是 Issue 事件的数据结构示例。

```json
{
  "action": "opened",
  "issue": {
    "id": 123456,
    "number": 1,
    "title": "Bug: Something broken",
    "body": "Description of the bug",
    "state": "open",
    "user": {
      "login": "octocat"
    },
    "labels": [
      {
        "name": "bug",
        "color": "fc2929"
      }
    ]
  }
}
```

### 8.4 Workflow Run 事件

Workflow Run 事件在 GitHub Actions 运行完成时触发。通过监听 Workflow Run 事件，可以实现 CI/CD 流程的监控和通知。例如，当工作流失败时发送告警通知，当工作流成功时触发后续流程。以下是 Workflow Run 事件的数据结构示例。

```json
{
  "action": "completed",
  "workflow_run": {
    "id": 123456,
    "name": "CI",
    "head_branch": "main",
    "head_sha": "abc123",
    "status": "completed",
    "conclusion": "success",
    "workflow_id": 789,
    "run_number": 42,
    "created_at": "2023-01-01T00:00:00Z",
    "updated_at": "2023-01-01T00:05:00Z"
  }
}
```

## 第九章：Webhook 安全（签名验证）

Webhook 安全是构建可靠集成应用的关键。如果不对 Webhook 请求进行验证，攻击者可能会伪造 Webhook 请求，触发未授权的操作。GitHub 使用 HMAC-SHA256 算法对 Webhook 请求进行签名，开发者应该在接收端验证签名，确保请求确实来自 GitHub。

### 9.1 为什么需要签名验证

签名验证是 Webhook 安全的基础。通过验证签名，可以确保以下几点：首先，请求确实来自 GitHub，而不是伪造的。其次，请求在传输过程中没有被篡改。最后，防止重放攻击（攻击者截获合法请求后重新发送）。开发者应该始终验证 Webhook 签名，不要跳过这个安全步骤。

### 9.2 签名生成算法

GitHub 使用 HMAC-SHA256 算法生成签名。HMAC（Hash-based Message Authentication Code）是一种基于哈希的消息认证码，使用密钥对消息进行哈希计算。GitHub 使用配置的 Webhook Secret 作为密钥，对请求体进行 HMAC-SHA256 计算，生成签名并附加到请求头中。签名通过 `X-Hub-Signature-256` 头部传递，格式为 `sha256=签名值`。

```
X-Hub-Signature-256: sha256=abc123...
```

### 9.3 验证签名示例

验证签名的实现因编程语言而异，但基本原理相同：使用 Webhook Secret 对请求体进行 HMAC-SHA256 计算，然后与请求头中的签名进行比较。比较时应该使用常量时间比较函数（如 Python 的 `hmac.compare_digest`），以防止时序攻击。以下是几种常用语言的签名验证实现。

**Python 验证**：

```python
import hmac
import hashlib

def verify_signature(payload_body, secret_token, signature_header):
    """验证 GitHub Webhook 签名"""
    if not signature_header:
        return False
    
    hash_object = hmac.new(
        secret_token.encode('utf-8'),
        msg=payload_body,
        digestmod=hashlib.sha256
    )
    expected_signature = "sha256=" + hash_object.hexdigest()
    return hmac.compare_digest(expected_signature, signature_header)

# Flask 示例
from flask import Flask, request, abort

app = Flask(__name__)
WEBHOOK_SECRET = "your-secret"

@app.route('/webhook', methods=['POST'])
def webhook():
    signature = request.headers.get('X-Hub-Signature-256')
    if not verify_signature(request.data, WEBHOOK_SECRET, signature):
        abort(401)
    
    payload = request.json
    # 处理 Webhook 事件
    return 'OK', 200
```

**Node.js 验证**：

```javascript
const crypto = require('crypto');

function verifySignature(payload, signature, secret) {
  const hmac = crypto.createHmac('sha256', secret);
  const digest = 'sha256=' + hmac.update(payload).digest('hex');
  return crypto.timingSafeEqual(
    Buffer.from(signature),
    Buffer.from(digest)
  );
}

// Express 示例
const express = require('express');
const app = express();
const WEBHOOK_SECRET = 'your-secret';

app.post('/webhook', express.raw({ type: 'application/json' }), (req, res) => {
  const signature = req.headers['x-hub-signature-256'];
  if (!verifySignature(req.body, signature, WEBHOOK_SECRET)) {
    return res.status(401).send('Invalid signature');
  }
  
  const payload = JSON.parse(req.body);
  // 处理 Webhook 事件
  res.status(200).send('OK');
});
```

**Go 验证**：

```go
package main

import (
    "crypto/hmac"
    "crypto/sha256"
    "encoding/hex"
    "io"
    "net/http"
)

func verifySignature(payload []byte, signature, secret string) bool {
    mac := hmac.New(sha256.New, []byte(secret))
    mac.Write(payload)
    expectedMAC := hex.EncodeToString(mac.Sum(nil))
    expectedSignature := "sha256=" + expectedMAC
    return hmac.Equal([]byte(signature), []byte(expectedSignature))
}

func webhookHandler(w http.ResponseWriter, r *http.Request) {
    payload, err := io.ReadAll(r.Body)
    if err != nil {
        http.Error(w, "Error reading body", http.StatusBadRequest)
        return
    }
    
    signature := r.Header.Get("X-Hub-Signature-256")
    if !verifySignature(payload, signature, "your-secret") {
        http.Error(w, "Invalid signature", http.StatusUnauthorized)
        return
    }
    
    // 处理 Webhook 事件
    w.WriteHeader(http.StatusOK)
}
```

### 9.4 安全最佳实践

Webhook 安全不仅仅是验证签名，还包括多个方面的安全措施。以下是 Webhook 安全的最佳实践，开发者应该在构建集成应用时严格遵守。这些最佳实践可以帮助开发者避免常见的安全风险，确保 Webhook 服务的安全性和可靠性。

1. **始终验证签名**：不要跳过签名验证步骤
2. **使用常量时间比较**：防止时序攻击，使用 `hmac.compare_digest` 等函数
3. **使用 HTTPS**：确保传输过程中数据不被窃听
4. **保护 Secret**：不要硬编码在代码中，使用环境变量
5. **限制 IP 白名单**：只允许 GitHub 的 IP 范围访问

## 第十章：实战：构建 GitHub 集成应用

本章将通过几个实际案例，展示如何使用 GitHub API 和 Webhooks 构建集成应用。这些案例涵盖了常见的使用场景，包括自动代码审查、Issue 自动标签、部署通知等。通过学习这些案例，开发者可以掌握构建 GitHub 集成应用的基本技能。

### 10.1 示例：自动代码审查机器人

自动代码审查机器人是一个常见的 GitHub 集成应用。当 Pull Request 创建时，机器人会自动分析代码变更，检测潜在的问题（如安全漏洞、代码风格问题等），并在 PR 上添加审查评论。以下是一个使用 Python Flask 构建的自动代码审查机器人示例。这个机器人会检测代码中的敏感信息和 TODO 注释，并在 PR 上添加相应的评论。

```python
from flask import Flask, request
import hmac
import hashlib
import requests
import os

app = Flask(__name__)

GITHUB_TOKEN = os.environ.get('GITHUB_TOKEN')
WEBHOOK_SECRET = os.environ.get('WEBHOOK_SECRET')

def verify_signature(payload, signature):
    if not signature:
        return False
    hash_object = hmac.new(
        WEBHOOK_SECRET.encode('utf-8'),
        msg=payload,
        digestmod=hashlib.sha256
    )
    expected = "sha256=" + hash_object.hexdigest()
    return hmac.compare_digest(expected, signature)

def review_code(owner, repo, pr_number):
    """自动审查代码"""
    headers = {
        'Authorization': f'token {GITHUB_TOKEN}',
        'Accept': 'application/vnd.github.v3+json'
    }
    
    # 获取 PR 文件
    url = f'https://api.github.com/repos/{owner}/{repo}/pulls/{pr_number}/files'
    response = requests.get(url, headers=headers)
    files = response.json()
    
    comments = []
    for file in files:
        filename = file['filename']
        patch = file.get('patch', '')
        
        # 简单的代码审查规则
        if 'password' in patch.lower() or 'secret' in patch.lower():
            comments.append({
                'path': filename,
                'position': 1,
                'body': 'Warning: Possible sensitive information detected.'
            })
        
        if 'TODO' in patch:
            comments.append({
                'path': filename,
                'position': 1,
                'body': 'Note: TODO comment found. Please track this task.'
            })
    
    # 提交审查评论
    if comments:
        review_url = f'https://api.github.com/repos/{owner}/{repo}/pulls/{pr_number}/reviews'
        review_data = {
            'event': 'COMMENT',
            'body': 'Automated code review by Bot',
            'comments': comments
        }
        requests.post(review_url, json=review_data, headers=headers)

@app.route('/webhook', methods=['POST'])
def webhook():
    signature = request.headers.get('X-Hub-Signature-256')
    if not verify_signature(request.data, signature):
        return 'Invalid signature', 401
    
    payload = request.json
    
    if payload.get('action') == 'opened' and 'pull_request' in payload:
        pr = payload['pull_request']
        owner = payload['repository']['owner']['login']
        repo = payload['repository']['name']
        pr_number = pr['number']
        
        review_code(owner, repo, pr_number)
    
    return 'OK', 200

if __name__ == '__main__':
    app.run(port=5000)
```

### 10.2 示例：Issue 自动标签机器人

Issue 自动标签机器人可以根据 Issue 的标题和内容自动添加标签。这有助于 Issue 的分类和管理，提高项目管理效率。以下是一个使用 Python Flask 构建的 Issue 自动标签机器人示例。这个机器人会分析 Issue 的标题和内容，根据关键词自动添加相应的标签（如 bug、enhancement、documentation 等）。

```python
from flask import Flask, request
import hmac
import hashlib
import requests
import os

app = Flask(__name__)

GITHUB_TOKEN = os.environ.get('GITHUB_TOKEN')
WEBHOOK_SECRET = os.environ.get('WEBHOOK_SECRET')

def verify_signature(payload, signature):
    if not signature:
        return False
    hash_object = hmac.new(
        WEBHOOK_SECRET.encode('utf-8'),
        msg=payload,
        digestmod=hashlib.sha256
    )
    expected = "sha256=" + hash_object.hexdigest()
    return hmac.compare_digest(expected, signature)

def analyze_issue(title, body):
    """分析 Issue 内容并返回标签"""
    labels = []
    
    # 检测 Bug
    if any(word in title.lower() for word in ['bug', 'error', 'crash', 'broken']):
        labels.append('bug')
    
    # 检测功能请求
    if any(word in title.lower() for word in ['feature', 'request', 'enhancement']):
        labels.append('enhancement')
    
    # 检测文档
    if any(word in title.lower() for word in ['doc', 'documentation', 'readme']):
        labels.append('documentation')
    
    # 检测优先级
    if any(word in title.lower() for word in ['urgent', 'critical', 'blocker']):
        labels.append('priority: high')
    
    return labels

def add_labels(owner, repo, issue_number, labels):
    """添加标签到 Issue"""
    headers = {
        'Authorization': f'token {GITHUB_TOKEN}',
        'Accept': 'application/vnd.github.v3+json'
    }
    
    url = f'https://api.github.com/repos/{owner}/{repo}/issues/{issue_number}/labels'
    requests.post(url, json={'labels': labels}, headers=headers)

@app.route('/webhook', methods=['POST'])
def webhook():
    signature = request.headers.get('X-Hub-Signature-256')
    if not verify_signature(request.data, signature):
        return 'Invalid signature', 401
    
    payload = request.json
    
    if payload.get('action') == 'opened' and 'issue' in payload:
        issue = payload['issue']
        owner = payload['repository']['owner']['login']
        repo = payload['repository']['name']
        
        title = issue['title']
        body = issue.get('body', '')
        issue_number = issue['number']
        
        labels = analyze_issue(title, body)
        if labels:
            add_labels(owner, repo, issue_number, labels)
    
    return 'OK', 200

if __name__ == '__main__':
    app.run(port=5000)
```

### 10.3 示例：部署通知机器人

部署通知机器人可以在 CI/CD 流程完成时发送通知。这有助于团队了解部署状态，及时发现和处理部署问题。以下是一个使用 Python Flask 构建的部署通知机器人示例。这个机器人会监听 GitHub Actions 的 workflow_run 事件，当工作流完成时，向 Slack 发送通知消息。

```python
from flask import Flask, request
import hmac
import hashlib
import requests
import os

app = Flask(__name__)

GITHUB_TOKEN = os.environ.get('GITHUB_TOKEN')
WEBHOOK_SECRET = os.environ.get('WEBHOOK_SECRET')
SLACK_WEBHOOK_URL = os.environ.get('SLACK_WEBHOOK_URL')

def verify_signature(payload, signature):
    if not signature:
        return False
    hash_object = hmac.new(
        WEBHOOK_SECRET.encode('utf-8'),
        msg=payload,
        digestmod=hashlib.sha256
    )
    expected = "sha256=" + hash_object.hexdigest()
    return hmac.compare_digest(expected, signature)

def send_slack_message(message):
    """发送 Slack 通知"""
    requests.post(SLACK_WEBHOOK_URL, json={'text': message})

def handle_workflow_run(payload):
    """处理 workflow_run 事件"""
    workflow_run = payload['workflow_run']
    
    status = workflow_run['status']
    conclusion = workflow_run.get('conclusion', 'in_progress')
    name = workflow_run['name']
    branch = workflow_run['head_branch']
    url = workflow_run['html_url']
    
    if status == 'completed':
        if conclusion == 'success':
            emoji = '✅'
            message = f"{emoji} {name} succeeded on {branch}"
        elif conclusion == 'failure':
            emoji = '❌'
            message = f"{emoji} {name} failed on {branch}"
        else:
            emoji = '⚠️'
            message = f"{emoji} {name} {conclusion} on {branch}"
        
        message += f"\n<{url}|View details>"
        send_slack_message(message)

@app.route('/webhook', methods=['POST'])
def webhook():
    signature = request.headers.get('X-Hub-Signature-256')
    if not verify_signature(request.data, signature):
        return 'Invalid signature', 401
    
    payload = request.json
    event = request.headers.get('X-GitHub-Event')
    
    if event == 'workflow_run':
        handle_workflow_run(payload)
    
    return 'OK', 200

if __name__ == '__main__':
    app.run(port=5000)
```

## 第十一章：API 速率限制与优化

GitHub 对 API 调用有速率限制，以防止滥用和确保服务稳定性。了解速率限制规则并采取优化措施，对于构建可靠的集成应用至关重要。本章将介绍速率限制的规则、检查方法和优化策略。

### 11.1 速率限制规则

GitHub API 的速率限制因认证方式而异。未认证的请求速率限制最低，GitHub App 的速率限制最高。开发者应该根据实际需求选择合适的认证方式，并合理规划 API 调用。以下是不同认证方式的速率限制详情。

| 认证方式 | 核心限制 | 搜索限制 | GraphQL 点数 |
|---------|---------|---------|-------------|
| 未认证 | 60/小时 | 10/分钟 | N/A |
| PAT | 5000/小时 | 30/分钟 | 5000/小时 |
| GitHub App | 15000/小时 | 30/分钟 | 5000/小时 |
| GITHUB_TOKEN | 1000/小时 | 30/分钟 | 1000/小时 |

### 11.2 检查速率限制

开发者可以通过 API 检查当前的速率限制状态。速率限制信息包含限制总数、已使用数、剩余数和重置时间。通过监控这些信息，可以避免超出速率限制。以下是检查速率限制的方法。

```bash
# 查看当前限制
gh api /rate_limit

# 响应示例
{
  "resources": {
    "core": {
      "limit": 5000,
      "used": 123,
      "remaining": 4877,
      "reset": 1609459200
    },
    "search": {
      "limit": 30,
      "used": 5,
      "remaining": 25,
      "reset": 1609459260
    },
    "graphql": {
      "limit": 5000,
      "used": 100,
      "remaining": 4900,
      "reset": 1609459200
    }
  }
}
```

### 11.3 处理速率限制

当 API 调用超出速率限制时，GitHub 会返回 403 状态码。开发者应该在代码中处理速率限制，当遇到速率限制时，等待一段时间后重试。以下是处理速率限制的 Python 示例代码，它会自动检测速率限制并等待重置时间。

```python
import requests
import time

def github_api_call(url, token):
    headers = {
        'Authorization': f'token {token}',
        'Accept': 'application/vnd.github.v3+json'
    }
    
    while True:
        response = requests.get(url, headers=headers)
        
        # 检查速率限制
        remaining = int(response.headers.get('X-RateLimit-Remaining', 0))
        reset_time = int(response.headers.get('X-RateLimit-Reset', 0))
        
        if remaining == 0:
            # 等待重置
            wait_time = reset_time - time.time() + 1
            if wait_time > 0:
                print(f"Rate limited. Waiting {wait_time} seconds...")
                time.sleep(wait_time)
                continue
        
        if response.status_code == 200:
            return response.json()
        elif response.status_code == 403 and 'rate limit' in response.text.lower():
            time.sleep(60)
            continue
        else:
            response.raise_for_status()
```

### 11.4 优化策略

为了减少 API 调用次数和提高效率，开发者可以采取多种优化策略。这些策略可以帮助应用在速率限制内完成更多的工作，同时提高响应速度和用户体验。以下是常用的 API 调用优化策略。

**使用条件请求**：

```bash
# 使用 ETag
curl -H "If-None-Match: \"abc123\"" \
  https://api.github.com/repos/octocat/Hello-World

# 使用 Last-Modified
curl -H "If-Modified-Since: Wed, 01 Jan 2023 00:00:00 GMT" \
  https://api.github.com/repos/octocat/Hello-World
```

**使用 GraphQL 减少请求**：

```graphql
# 一次请求获取多个资源
query {
  repository(owner: "octocat", name: "Hello-World") {
    issues(first: 100) {
      nodes {
        title
        comments(first: 10) {
          nodes {
            body
          }
        }
      }
    }
    pullRequests(first: 100) {
      nodes {
        title
        reviews(first: 10) {
          nodes {
            body
          }
        }
      }
    }
  }
}
```

**使用 Webhook 替代轮询**：

```python
# 不好的做法：轮询
while True:
    issues = get_issues()
    process_issues(issues)
    time.sleep(60)

# 好的做法：使用 Webhook
@app.route('/webhook', methods=['POST'])
def webhook():
    issue = request.json['issue']
    process_issue(issue)
    return 'OK', 200
```

**缓存响应**：

```python
import requests_cache

# 安装：pip install requests-cache
requests_cache.install_cache('github_cache', expire_after=300)

# 现在所有请求都会被缓存
response = requests.get('https://api.github.com/repos/octocat/Hello-World')
```

## 第十二章：GitHub App vs OAuth App 对比

GitHub App 和 OAuth App 是两种不同的应用类型，适用于不同的使用场景。理解它们的区别对于选择合适的应用类型至关重要。本章将详细对比两种应用类型的特点和适用场景。

### 12.1 核心区别

GitHub App 和 OAuth App 在身份模型、权限模型、速率限制等方面有显著区别。GitHub App 以独立身份运行，拥有自己的权限；OAuth App 以用户身份运行，继承用户的权限。这些区别决定了它们的适用场景。

| 特性 | GitHub App | OAuth App |
|------|------------|----------|
| 身份 | 独立身份 | 用户身份 |
| 安装 | 安装在仓库/组织 | 用户授权 |
| 权限 | 细粒度权限 | 用户范围权限 |
| 速率限制 | 15000/小时 | 5000/小时 |
| Webhook | 绑定到安装 | 需要单独配置 |
| 推荐场景 | 集成应用 | 用户授权应用 |

### 12.2 GitHub App 优势

GitHub App 是更现代的应用类型，具有多个优势。这些优势使 GitHub App 成为构建集成应用的首选。以下是 GitHub App 的主要优势，开发者应该在选择应用类型时考虑这些因素。

**细粒度权限**：GitHub App 可以精确配置需要的权限，只请求必要的访问权限。这比 OAuth App 的用户范围权限更安全。

**独立身份**：GitHub App 以独立身份运行，不依赖用户账户。这使得应用的行为更加可预测，也更容易管理。

**更高配额**：GitHub App 的 API 速率限制是 15000 次/小时，比 OAuth App 的 5000 次/小时高三倍。

**安装级令牌**：GitHub App 使用短期有效的安装令牌（1小时），比长期有效的 OAuth 令牌更安全。

**组织级控制**：组织管理员可以控制 GitHub App 的安装，可以限制哪些仓库可以安装应用。

### 12.3 OAuth App 优势

OAuth App 是传统的应用类型，虽然在某些方面不如 GitHub App，但仍有其独特的优势。在某些场景下，OAuth App 可能是更好的选择。以下是 OAuth App 的主要优势。

**用户身份**：OAuth App 以用户身份执行操作，可以访问用户有权限的所有资源。

**简单集成**：OAuth App 的集成流程相对简单，适合需要用户授权的场景。

**广泛支持**：OAuth App 支持所有 GitHub 功能，没有 GitHub App 的一些限制。

**长期令牌**：OAuth 令牌没有过期时间（除非手动撤销），适合需要长期访问的场景。

### 12.4 选择建议

选择使用 GitHub App 还是 OAuth App 取决于具体的使用场景。开发者应该根据应用的需求、安全要求和用户体验等因素做出选择。以下是选择建议，帮助开发者做出明智的决定。

**选择 GitHub App**：
- 构建 GitHub 集成应用
- 需要细粒度权限控制
- 需要高 API 配额
- 组织级别的集成

**选择 OAuth App**：
- 需要以用户身份操作
- 简单的用户授权场景
- 需要访问用户特定数据
- 传统的第三方应用

### 12.5 创建 GitHub App

创建 GitHub App 需要在 GitHub 设置中完成。创建过程中需要配置应用的基本信息、权限、Webhook 等。以下是创建 GitHub App 的详细步骤，开发者应该按照这些步骤完成应用的创建和配置。

```bash
# 1. 访问 Settings → Developer settings → GitHub Apps → New GitHub App

# 2. 配置基本信息
# - App name: My Integration
# - Homepage URL: https://example.com
# - Callback URL: https://example.com/callback

# 3. 配置权限
# Repository permissions:
#   - Issues: Read & Write
#   - Pull requests: Read & Write
#   - Contents: Read-only

# 4. 配置事件
# - Issues
# - Pull request

# 5. 生成私钥

# 6. 安装到仓库/组织
```

### 12.6 使用 GitHub App

使用 GitHub App 需要实现 JWT 认证和安装令牌获取。以下是使用 Python 实现 GitHub App 认证的完整示例代码。这个示例展示了如何生成 JWT、获取安装令牌、以及使用安装令牌调用 API。开发者可以基于这个示例构建自己的 GitHub App 应用。

```python
import jwt
import time
import requests

class GitHubApp:
    def __init__(self, app_id, private_key_path):
        self.app_id = app_id
        with open(private_key_path, 'r') as f:
            self.private_key = f.read()
    
    def get_jwt(self):
        now = int(time.time())
        payload = {
            'iat': now - 60,
            'exp': now + 600,
            'iss': self.app_id
        }
        return jwt.encode(payload, self.private_key, algorithm='RS256')
    
    def get_installation_token(self, installation_id):
        jwt_token = self.get_jwt()
        headers = {
            'Authorization': f'Bearer {jwt_token}',
            'Accept': 'application/vnd.github.v3+json'
        }
        
        response = requests.post(
            f'https://api.github.com/app/installations/{installation_id}/access_tokens',
            headers=headers
        )
        return response.json()['token']
    
    def api_call(self, method, url, installation_id, **kwargs):
        token = self.get_installation_token(installation_id)
        headers = {
            'Authorization': f'token {token}',
            'Accept': 'application/vnd.github.v3+json'
        }
        return requests.request(method, url, headers=headers, **kwargs)

# 使用示例
app = GitHubApp('APP_ID', 'private-key.pem')
token = app.get_installation_token('INSTALLATION_ID')
response = app.api_call('GET', 'https://api.github.com/repos/octocat/Hello-World', 'INSTALLATION_ID')
```

通过本指南，你应该已经掌握了 GitHub API 和 Webhooks 的完整知识。从 REST API 到 GraphQL，从 Webhook 配置到安全验证，从 Octokit SDK 到速率限制优化，GitHub 提供了强大的 API 生态系统。无论是构建自动化工具、集成应用，还是简单的脚本，都可以利用这些 API 实现你的需求。
