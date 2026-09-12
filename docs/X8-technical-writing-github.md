# GitHub 技术写作指南

> **目标读者**：中国开发者、技术文档工程师、开源项目维护者
> **预计阅读时间**：50 分钟
> **前置知识**：基本的 Git/GitHub 使用、Markdown 基础

---

## 目录

1. [技术写作风格指南](#1-技术写作风格指南)
2. [中文技术文档排版规范](#2-中文技术文档排版规范)
3. [Markdown 高级技巧](#3-markdown-高级技巧)
4. [AsciiDoc 介绍](#4-asciidoc-介绍)
5. [静态文档站点](#5-静态文档站点)
6. [API 文档自动化](#6-api-文档自动化)
7. [文档版本管理策略](#7-文档版本管理策略)
8. [多语言文档管理](#8-多语言文档管理)
9. [文档自动化测试](#9-文档自动化测试)
10. [GitHub Wiki 使用指南](#10-github-wiki-使用指南)
11. [GitHub Discussions 作为知识库](#11-github-discussions-作为知识库)
12. [技术博客写作](#12-技术博客写作)
13. [文档贡献工作流](#13-文档贡献工作流)
14. [中英混合排版最佳实践](#14-中英混合排版最佳实践)

---

## 1. 技术写作风格指南

### 1.1 技术写作的核心原则

技术写作的目标是**让读者快速理解和使用技术内容**。与文学写作不同，技术写作强调：

**清晰性（Clarity）**：
- 使用简单直接的句子
- 避免歧义和模糊表达
- 一个句子表达一个意思

**准确性（Accuracy）**：
- 技术细节必须准确
- 代码示例必须可运行
- 版本号、路径、命令必须正确

**简洁性（Conciseness）**：
- 删除不必要的修饰词
- 避免重复信息
- 使用列表代替长段落

**一致性（Consistency）**：
- 术语使用统一
- 格式风格统一
- 文档结构统一

### 1.2 句子结构

**主动语态优先**：
```markdown
# 不推荐
The configuration file is edited by the user.

# 推荐
Edit the configuration file.
```

**使用现在时态**：
```markdown
# 不推荐
The system will return a JSON response.

# 推荐
The system returns a JSON response.
```

**使用第二人称（你/您）**：
```markdown
# 不推荐
Users should configure the environment.

# 推荐
你需要配置环境变量。
```

### 1.3 段落写作

**段落长度**：3-5 句为宜，避免超过 8 句

**段落结构**：
1. 主题句：说明本段核心内容
2. 支撑句：提供细节和解释
3. 过渡句：连接下一段（可选）

**示例**：
```markdown
GitHub Actions 是 GitHub 提供的 CI/CD 服务。你可以在仓库中定义工作流，
当代码推送到特定分支时自动运行测试和部署。工作流使用 YAML 格式定义，
存放在 `.github/workflows` 目录中。
```

### 1.4 术语管理

建立项目术语表（Glossary）：

```markdown
## 术语表

| 术语 | 英文 | 定义 |
|------|------|------|
| 仓库 | Repository | 存储代码和资源的容器 |
| 分支 | Branch | 代码的独立开发线 |
| 拉取请求 | Pull Request | 请求合并代码的机制 |
| 工作流 | Workflow | 自动化任务的定义文件 |
```

### 1.5 常见写作错误

| 错误类型 | 错误示例 | 正确示例 |
|----------|----------|----------|
| 冗长 | 在这里我们将会要讨论的是... | 本节讨论... |
| 被动语态 | 代码被提交到仓库 | 提交代码到仓库 |
| 模糊 | 一些配置可能需要修改 | 修改 `config.yml` 中的 `timeout` 字段 |
| 口语化 | 这个功能超好用 | 这个功能可以提高开发效率 |
| 术语不一致 | 有时叫"仓库"，有时叫"代码库" | 统一使用"仓库" |

---

## 2. 中文技术文档排版规范

### 2.1 标点符号规范

**中文标点**：
- 使用全角标点：，。！？；：""''（）
- 不使用半角标点：,.!?;:""''()

**特殊情况**：
- 代码中的标点使用半角
- 英文缩写后的句点使用半角（如 `e.g.`、`i.e.`）
- 数字和单位之间不加空格（如 `100MB`）

### 2.2 空格规范

**中英文之间加空格**：
```markdown
# 不推荐
使用GitHub进行版本控制

# 推荐
使用 GitHub 进行版本控制
```

**数字和中文之间加空格**：
```markdown
# 不推荐
仓库有100个星标

# 推荐
仓库有 100 个星标
```

**标点符号前后不加空格**：
```markdown
# 不推荐
使用 GitHub ，进行版本控制 。

# 推荐
使用 GitHub，进行版本控制。
```

**例外情况**：
- 代码块中的空格按代码规范
- 英文单词之间的空格保留

### 2.3 数字和单位

**数字**：
- 一般使用阿拉伯数字：3 个步骤、10 个文件
- 大数字使用逗号分隔：1,000、1,000,000
- 百分比使用符号：50%（而非 50 percent）

**单位**：
- 与数字之间不加空格：100MB、2GHz
- 使用国际单位制：KB、MB、GB、TB
- 时间单位：秒、分钟、小时、天

### 2.4 标题规范

**标题层级**：
- 一级标题（#）：文档标题，每个文件只有一个
- 二级标题（##）：主要章节
- 三级标题（###）：子章节
- 四级标题（####）：细分内容
- 不建议使用五级及以下标题

**标题格式**：
```markdown
# 文档标题

## 一级章节

### 二级章节

#### 三级章节
```

**标题大小写**：
- 中文标题：首字母大写（如"Git 基础教程"）
- 英文标题：Title Case（如"Getting Started with Git"）
- 专有名词保持原样：GitHub、JavaScript、API

### 2.5 列表规范

**有序列表**：用于有先后顺序的步骤
```markdown
1. 克隆仓库
2. 安装依赖
3. 启动服务
```

**无序列表**：用于并列的项目
```markdown
- 优点一
- 优点二
- 优点三
```

**嵌套列表**：
```markdown
- 前端技术栈
  - React
  - Vue.js
  - Angular
- 后端技术栈
  - Node.js
  - Python
  - Go
```

---

## 3. Markdown 高级技巧

### 3.1 表格进阶

**对齐方式**：
```markdown
| 左对齐 | 居中 | 右对齐 |
|:------|:----:|-------:|
| 内容 | 内容 | 内容 |
```

**表格中的代码**：
```markdown
| 命令 | 说明 |
|------|------|
| `git add` | 暂存文件 |
| `git commit` | 提交更改 |
| `git push` | 推送代码 |
```

**复杂表格**：
```markdown
| 功能 | 免费版 | 专业版 | 企业版 |
|------|:------:|:------:|:------:|
| 私有仓库 | ✅ 无限 | ✅ 无限 | ✅ 无限 |
| 协作者 | 3 人 | 无限 | 无限 |
| CI/CD 分钟数 | 2,000 | 3,000 | 50,000 |
| 存储空间 | 500MB | 2GB | 50GB |
```

### 3.2 代码块进阶

**带行号的代码块**（部分平台支持）：
```markdown
```python {.numberLines}
def hello():
    print("Hello, World!")

hello()
`` `
```

**高亮特定行**（部分平台支持）：
```markdown
```python {2,3}
def calculate_sum(a, b):
    result = a + b  # 这行会被高亮
    return result   # 这行也会被高亮
`` `
```

**代码块中的注释**：
```python
# 这是一个 Python 函数
def greet(name: str) -> str:
    """
    生成问候语
    
    Args:
        name: 用户名
        
    Returns:
        问候语字符串
    """
    return f"Hello, {name}!"
```

### 3.3 折叠内容

使用 `<details>` 标签创建可折叠内容：

```markdown
<details>
<summary>点击展开详细配置</summary>

```yaml
server:
  host: localhost
  port: 8080
  debug: true

database:
  host: localhost
  port: 5432
  name: mydb
```

</details>
```

### 3.4 警告和提示框

GitHub 支持以下提示框语法（GitHub Flavored Markdown）：

```markdown
> [!NOTE]
> 这是一个提示信息。

> [!TIP]
> 这是一个技巧提示。

> [!IMPORTANT]
> 这是一个重要信息。

> [!WARNING]
> 这是一个警告信息。

> [!CAUTION]
> 这是一个注意事项。
```

### 3.5 数学公式

GitHub 支持 LaTeX 数学公式：

**行内公式**：
```markdown
质能方程 $E = mc^2$ 是物理学中最著名的公式。
```

**块级公式**：
```markdown
$$
\sum_{i=1}^{n} i = \frac{n(n+1)}{2}
$$
```

**矩阵**：
```markdown
$$
\begin{pmatrix}
a & b \\
c & d
\end{pmatrix}
$$
```

### 3.6 Mermaid 图表

GitHub 原生支持 Mermaid 图表：

**流程图**：
```markdown
```mermaid
graph TD
    A[开始] --> B{是否已安装?}
    B -->|是| C[配置环境]
    B -->|否| D[安装依赖]
    D --> C
    C --> E[运行程序]
    E --> F[结束]
`` `
```

**序列图**：
```markdown
```mermaid
sequenceDiagram
    participant User as 用户
    participant Client as 客户端
    participant Server as 服务器
    
    User->>Client: 输入命令
    Client->>Server: 发送请求
    Server-->>Client: 返回响应
    Client-->>User: 显示结果
`` `
```

**甘特图**：
```markdown
```mermaid
gantt
    title 项目计划
    dateFormat  YYYY-MM-DD
    section 设计阶段
    需求分析       :done,    des1, 2024-01-01, 2024-01-15
    UI 设计        :active,  des2, 2024-01-10, 2024-01-25
    section 开发阶段
    前端开发       :         dev1, 2024-01-20, 2024-02-15
    后端开发       :         dev2, 2024-01-25, 2024-02-20
`` `
```

### 3.7 任务列表

```markdown
## 项目待办

- [x] 完成需求分析
- [x] 设计数据库 schema
- [ ] 实现用户认证
- [ ] 编写单元测试
- [ ] 部署到生产环境
```

### 3.8 脚注

```markdown
GitHub 是全球最大的代码托管平台[^1]，拥有超过 1 亿开发者[^2]。

[^1]: GitHub 官方网站 https://github.com
[^2]: 截至 2024 年的统计数据
```

---

## 4. AsciiDoc 介绍

### 4.1 AsciiDoc 简介

AsciiDoc 是一种轻量级标记语言，比 Markdown 更强大，特别适合编写大型技术文档。

**主要特点**：
- 支持复杂的文档结构
- 内置交叉引用和索引
- 支持条件编译
- 可扩展的宏系统
- 生成 HTML、PDF、EPUB 等格式

### 4.2 AsciiDoc 与 Markdown 对比

| 特性 | Markdown | AsciiDoc |
|------|----------|----------|
| 学习曲线 | 低 | 中等 |
| 表格支持 | 基础 | 强大 |
| 交叉引用 | 不支持 | 原生支持 |
| 条件编译 | 不支持 | 支持 |
| 目录生成 | 自动/手动 | 自动 |
| 代码块 | 基础 | 高级（含 callouts） |
| 适用场景 | 简单文档、README | 大型文档、书籍 |

### 4.3 AsciiDoc 基础语法

**标题**：
```asciidoc
= 文档标题
== 一级章节
=== 二级章节
==== 三级章节
```

**段落和换行**：
```asciidoc
这是第一段。

这是第二段。
段落内的换行使用 +
强制换行。
```

**列表**：
```asciidoc
* 无序列表项 1
* 无序列表项 2
** 嵌套项

. 有序列表项 1
. 有序列表项 2
.. 嵌套项
```

**代码块**：
```asciidoc
[source,python]
----
def hello():
    print("Hello, World!")
----
```

**带标注的代码块**：
```asciidoc
[source,python]
----
def hello():  # <1>
    print("Hello, World!")  # <2>
----
<1> 函数定义
<2> 打印语句
```

**表格**：
```asciidoc
[cols="1,2,1", options="header"]
|===
| 命令
| 说明
| 示例

| git add
| 暂存文件
| `git add .`

| git commit
| 提交更改
| `git commit -m "message"`
|===
```

**交叉引用**：
```asciidoc
参见 <<_installation>> 章节。

[[installation]]
== 安装指南

本章介绍如何安装软件。
```

### 4.4 AsciiDoc 工具链

**Asciidoctor**：主要的 AsciiDoc 处理器
```bash
# 安装
gem install asciidoctor

# 转换为 HTML
asciidoctor document.adoc

# 转换为 PDF
asciidoctor-pdf document.adoc

# 转换为 EPUB
asciidoctor-epub3 document.adoc
```

**Antora**：AsciiDoc 文档站点生成器
```bash
# 安装
npm install -g @antora/cli @antora/site-generator-default

# 生成站点
antora site.yml
```

### 4.5 何时选择 AsciiDoc

**选择 AsciiDoc**：
- 大型文档项目（超过 100 页）
- 需要交叉引用和索引
- 需要生成 PDF
- 技术书籍写作
- 企业级文档

**选择 Markdown**：
- 简单的 README
- 博客文章
- 快速笔记
- GitHub 项目文档

---

## 5. 静态文档站点

### 5.1 主流静态文档站点工具

| 工具 | 语言 | 特点 | 适用场景 |
|------|------|------|----------|
| MkDocs | Python | 简单易用、Material 主题 | 项目文档 |
| Docusaurus | React | Facebook 出品、版本管理 | 开源项目文档 |
| VitePress | Vue | 极速构建、Vue 生态 | Vue 项目文档 |
| Hugo | Go | 极速构建、功能强大 | 博客、文档 |
| Jekyll | Ruby | GitHub Pages 原生支持 | 博客 |
| Sphinx | Python | 学术文档、reStructuredText | Python 项目 |

### 5.2 MkDocs 快速上手

**安装**：
```bash
pip install mkdocs
pip install mkdocs-material  # Material 主题
```

**初始化项目**：
```bash
mkdocs new my-docs
cd my-docs
```

**目录结构**：
```
my-docs/
├── docs/
│   ├── index.md
│   ├── getting-started.md
│   └── api/
│       ├── overview.md
│       └── reference.md
└── mkdocs.yml
```

**配置文件（mkdocs.yml）**：
```yaml
site_name: 我的项目文档
site_description: 项目文档站点
site_url: https://example.com

theme:
  name: material
  language: zh
  palette:
    primary: indigo
    accent: indigo
  features:
    - navigation.tabs
    - navigation.sections
    - navigation.expand
    - search.suggest
    - content.code.copy

nav:
  - 首页: index.md
  - 快速开始: getting-started.md
  - API 文档:
    - 概述: api/overview.md
    - 参考: api/reference.md

markdown_extensions:
  - admonition
  - codehilite
  - toc:
      permalink: true
  - pymdownx.superfences
  - pymdownx.tabbed:
      alternate_style: true
```

**本地预览**：
```bash
mkdocs serve
# 访问 http://localhost:8000
```

**部署到 GitHub Pages**：
```bash
mkdocs gh-deploy
```

### 5.3 Docusaurus 快速上手

**初始化项目**：
```bash
npx create-docusaurus@latest my-website classic
cd my-website
```

**目录结构**：
```
my-website/
├── docs/
│   ├── intro.md
│   ├── tutorial-basics/
│   │   └── create-a-page.md
│   └── tutorial-extras/
│       └── translate-your-site.md
├── blog/
├── src/
│   ├── components/
│   └── css/
├── docusaurus.config.js
├── sidebars.js
└── package.json
```

**配置文件（docusaurus.config.js）**：
```javascript
module.exports = {
  title: '我的项目文档',
  tagline: '项目文档站点',
  url: 'https://example.com',
  baseUrl: '/',
  
  organizationName: 'your-org',
  projectName: 'your-project',
  
  i18n: {
    defaultLocale: 'zh-Hans',
    locales: ['zh-Hans', 'en'],
  },
  
  themeConfig: {
    navbar: {
      title: '我的项目',
      items: [
        { type: 'doc', position: 'left', docId: 'intro', label: '文档' },
        { to: '/blog', label: '博客', position: 'left' },
        { type: 'localeDropdown', position: 'right' },
        { href: 'https://github.com/your-org/your-project', label: 'GitHub', position: 'right' },
      ],
    },
    
    footer: {
      style: 'dark',
      links: [
        {
          title: '文档',
          items: [
            { label: '快速开始', to: '/docs/intro' },
          ],
        },
        {
          title: '社区',
          items: [
            { label: 'GitHub', href: 'https://github.com/your-org/your-project' },
          ],
        },
      ],
    },
  },
};
```

**运行和构建**：
```bash
# 本地开发
npm start

# 构建
npm run build

# 部署到 GitHub Pages
GIT_USER=your-username npm run deploy
```

### 5.4 VitePress 快速上手

**安装**：
```bash
npm init vitepress
```

**目录结构**：
```
docs/
├── .vitepress/
│   └── config.js
├── index.md
├── guide/
│   ├── getting-started.md
│   └── api.md
└── examples/
```

**配置文件（.vitepress/config.js）**：
```javascript
export default {
  title: '我的项目文档',
  description: '项目文档站点',
  
  themeConfig: {
    nav: [
      { text: '指南', link: '/guide/getting-started' },
      { text: 'API', link: '/guide/api' },
    ],
    
    sidebar: {
      '/guide/': [
        {
          text: '指南',
          items: [
            { text: '快速开始', link: '/guide/getting-started' },
            { text: 'API 参考', link: '/guide/api' },
          ],
        },
      ],
    },
    
    socialLinks: [
      { icon: 'github', link: 'https://github.com/your-org/your-project' },
    ],
    
    search: {
      provider: 'local',
    },
  },
};
```

**运行和构建**：
```bash
# 本地开发
npm run dev

# 构建
npm run build

# 预览构建结果
npm run preview
```

### 5.5 Hugo 快速上手

**安装**：
```bash
# macOS
brew install hugo

# Linux
sudo apt install hugo

# Windows
choco install hugo
```

**创建站点**：
```bash
hugo new site my-docs
cd my-docs
```

**安装主题**：
```bash
git init
git submodule add https://github.com/alex-shpak/hugo-book themes/hugo-book
```

**配置文件（hugo.toml）**：
```toml
baseURL = 'https://example.com/'
languageCode = 'zh-CN'
title = '我的项目文档'
theme = 'hugo-book'

[params]
  BookTheme = 'auto'
  BookToC = true
  BookSection = 'docs'
  BookRepo = 'https://github.com/your-org/your-project'

[menu]
  [[menu.after]]
    name = "GitHub"
    url = "https://github.com/your-org/your-project"
    weight = 10
```

**创建内容**：
```bash
hugo new docs/getting-started.md
```

**运行和构建**：
```bash
# 本地开发
hugo server -D

# 构建
hugo
```

---

## 6. API 文档自动化

### 6.1 API 文档工具概览

| 工具 | 特点 | 适用场景 |
|------|------|----------|
| Swagger UI | 交互式文档、在线测试 | REST API |
| Redoc | 美观的三栏布局 | 生产环境文档 |
| Stoplight Studio | 可视化编辑器 | API 设计 |
| Readme.io | 商业平台、协作功能 | 企业 API 文档 |
| Postman | API 测试 + 文档 | API 开发 |

### 6.2 OpenAPI 规范

OpenAPI（原 Swagger）是描述 REST API 的标准规范：

**OpenAPI 3.0 示例（YAML）**：
```yaml
openapi: 3.0.0
info:
  title: 用户管理 API
  description: 用户管理系统的 REST API 文档
  version: 1.0.0
  contact:
    name: API 支持
    email: support@example.com
  
servers:
  - url: https://api.example.com/v1
    description: 生产环境
  - url: https://staging-api.example.com/v1
    description: 测试环境

paths:
  /users:
    get:
      summary: 获取用户列表
      description: 返回所有用户的列表
      operationId: getUsers
      tags:
        - 用户管理
      parameters:
        - name: page
          in: query
          description: 页码
          required: false
          schema:
            type: integer
            default: 1
        - name: limit
          in: query
          description: 每页数量
          required: false
          schema:
            type: integer
            default: 20
      responses:
        '200':
          description: 成功返回用户列表
          content:
            application/json:
              schema:
                type: object
                properties:
                  users:
                    type: array
                    items:
                      $ref: '#/components/schemas/User'
                  total:
                    type: integer
        '401':
          description: 未授权
    
    post:
      summary: 创建用户
      description: 创建一个新用户
      operationId: createUser
      tags:
        - 用户管理
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateUserRequest'
      responses:
        '201':
          description: 用户创建成功
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
        '400':
          description: 请求参数错误

components:
  schemas:
    User:
      type: object
      properties:
        id:
          type: integer
          description: 用户 ID
          example: 1
        name:
          type: string
          description: 用户名
          example: "张三"
        email:
          type: string
          format: email
          description: 邮箱
          example: "zhangsan@example.com"
        created_at:
          type: string
          format: date-time
          description: 创建时间
    
    CreateUserRequest:
      type: object
      required:
        - name
        - email
      properties:
        name:
          type: string
          description: 用户名
          example: "张三"
        email:
          type: string
          format: email
          description: 邮箱
          example: "zhangsan@example.com"
```

### 6.3 Swagger UI 集成

**HTML 集成**：
```html
<!DOCTYPE html>
<html>
<head>
  <title>API 文档</title>
  <link rel="stylesheet" type="text/css" href="https://unpkg.com/swagger-ui-dist@5/swagger-ui.css">
</head>
<body>
  <div id="swagger-ui"></div>
  <script src="https://unpkg.com/swagger-ui-dist@5/swagger-ui-bundle.js"></script>
  <script>
    SwaggerUIBundle({
      url: "/openapi.yaml",
      dom_id: '#swagger-ui',
      presets: [
        SwaggerUIBundle.presets.apis,
        SwaggerUIBundle.SwaggerUIStandalonePreset
      ],
      layout: "BaseLayout"
    });
  </script>
</body>
</html>
```

**GitHub Pages 集成**：
```yaml
# .github/workflows/api-docs.yml
name: Deploy API Docs

on:
  push:
    branches: [main]
    paths:
      - 'openapi.yaml'

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          
      - name: Build docs
        run: |
          npm install -g redoc-cli
          redoc-cli build openapi.yaml -o docs/api/index.html
          
      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./docs
```

### 6.4 Redoc 集成

Redoc 提供更美观的三栏布局：

```bash
# 安装
npm install -g redoc-cli

# 本地预览
redoc-cli serve openapi.yaml

# 构建静态文件
redoc-cli build openapi.yaml -o api-docs.html
```

**自定义配置**：
```yaml
# redoc.yaml
theme:
  colors:
    primary:
      main: '#1976d2'
  typography:
    fontSize: '15px'
    fontFamily: 'Roboto, sans-serif'
  sidebar:
    width: '260px'
```

### 6.5 代码注释自动生成文档

**Python（使用 FastAPI）**：
```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
from typing import List

app = FastAPI(
    title="用户管理 API",
    description="用户管理系统的 REST API",
    version="1.0.0"
)

class User(BaseModel):
    """用户模型"""
    id: int
    name: str
    email: str

class CreateUserRequest(BaseModel):
    """创建用户请求"""
    name: str
    email: str

@app.get("/users", response_model=List[User], tags=["用户管理"])
async def get_users(page: int = 1, limit: int = 20):
    """
    获取用户列表
    
    - **page**: 页码（默认 1）
    - **limit**: 每页数量（默认 20）
    """
    # 实现代码
    pass

@app.post("/users", response_model=User, tags=["用户管理"])
async def create_user(request: CreateUserRequest):
    """
    创建用户
    
    创建一个新用户并返回用户信息
    """
    # 实现代码
    pass
```

**Node.js（使用 Express + Swagger JSDoc）**：
```javascript
const express = require('express');
const swaggerJsdoc = require('swagger-jsdoc');
const swaggerUi = require('swagger-ui-express');

const app = express();

const options = {
  definition: {
    openapi: '3.0.0',
    info: {
      title: '用户管理 API',
      version: '1.0.0',
    },
  },
  apis: ['./routes/*.js'],
};

const specs = swaggerJsdoc(options);
app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(specs));

/**
 * @swagger
 * /users:
 *   get:
 *     summary: 获取用户列表
 *     tags: [用户管理]
 *     parameters:
 *       - in: query
 *         name: page
 *         schema:
 *           type: integer
 *         description: 页码
 *     responses:
 *       200:
 *         description: 成功
 */
app.get('/users', (req, res) => {
  // 实现代码
});

app.listen(3000);
```

---

## 7. 文档版本管理策略

### 7.1 版本管理的重要性

随着项目迭代，文档也需要版本管理：

- 用户可能使用不同版本的软件
- API 可能在不同版本间有变化
- 需要保留旧版本文档供参考

### 7.2 Docusaurus 版本管理

Docusaurus 内置版本管理功能：

```bash
# 创建新版本
npm run docusaurus docs:version 2.0

# 目录结构变化
docs/           # 当前开发版本（next）
versioned_docs/
  version-1.0/  # 1.0 版本文档
  version-2.0/  # 2.0 版本文档
versioned_sidebars/
  version-1.0-sidebars.json
  version-2.0-sidebars.json
```

**配置版本下拉菜单**：
```javascript
// docusaurus.config.js
module.exports = {
  themeConfig: {
    navbar: {
      items: [
        {
          type: 'docsVersionDropdown',
          position: 'right',
        },
      ],
    },
  },
};
```

### 7.3 MkDocs 版本管理

使用 `mike` 工具管理 MkDocs 版本：

```bash
# 安装
pip install mike

# 设置默认版本
mike set-default latest

# 创建新版本
mike deploy 1.0
mike deploy 2.0

# 列出所有版本
mike list

# 设置别名
mike deploy 2.0 latest
```

**配置 mkdocs.yml**：
```yaml
extra:
  version:
    provider: mike
```

### 7.4 Git 分支策略

**分支命名规范**：
```
main           # 主分支，最新文档
docs/v1.0      # 1.0 版本文档
docs/v2.0      # 2.0 版本文档
docs/next       # 下一版本开发文档
```

**GitHub Actions 自动部署**：
```yaml
name: Deploy Docs

on:
  push:
    branches:
      - main
      - 'docs/**'

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Determine version
        id: version
        run: |
          BRANCH=${GITHUB_REF#refs/heads/}
          if [ "$BRANCH" = "main" ]; then
            echo "version=latest" >> $GITHUB_OUTPUT
          else
            VERSION=${BRANCH#docs/}
            echo "version=$VERSION" >> $GITHUB_OUTPUT
          fi
      
      - name: Deploy
        run: |
          # 部署到对应版本目录
          echo "Deploying version ${{ steps.version.outputs.version }}"
```

### 7.5 文档快照和归档

**创建文档快照**：
```bash
# 创建快照目录
mkdir -p snapshots/v1.0.0

# 复制文档
cp -r docs/* snapshots/v1.0.0/

# 提交快照
git add snapshots/v1.0.0
git commit -m "docs: snapshot v1.0.0"
git tag docs-v1.0.0
```

---

## 8. 多语言文档管理

### 8.1 多语言文档策略

**策略一：独立目录**
```
docs/
├── en/
│   ├── getting-started.md
│   └── api/
├── zh/
│   ├── getting-started.md
│   └── api/
└── ja/
    ├── getting-started.md
    └── api/
```

**策略二：文件后缀**
```
docs/
├── getting-started.md        # 默认语言
├── getting-started.zh.md     # 中文
├── getting-started.ja.md     # 日文
├── api/
│   ├── overview.md
│   ├── overview.zh.md
│   └── overview.ja.md
```

**策略三：i18n 框架**
```
docs/
├── getting-started.md
├── i18n/
│   ├── zh/
│   │   └── getting-started.md
│   └── ja/
│       └── getting-started.md
```

### 8.2 Docusaurus 多语言配置

```javascript
// docusaurus.config.js
module.exports = {
  i18n: {
    defaultLocale: 'zh-Hans',
    locales: ['zh-Hans', 'en', 'ja'],
    localeConfigs: {
      'zh-Hans': {
        label: '简体中文',
        htmlLang: 'zh-Hans',
      },
      en: {
        label: 'English',
      },
      ja: {
        label: '日本語',
      },
    },
  },
};
```

**翻译工作流**：
```bash
# 提取需要翻译的字符串
npm run write-translations -- --locale zh-Hans

# 翻译文件位于
# i18n/zh-Hans/docusaurus-plugin-content-docs/current/
# i18n/zh-Hans/docusaurus-theme-classic/
```

### 8.3 MkDocs 多语言配置

使用 `mkdocs-static-i18n` 插件：

```yaml
# mkdocs.yml
plugins:
  - i18n:
      default_language: en
      languages:
        en:
          name: English
          build: true
        zh:
          name: 中文
          build: true
```

**目录结构**：
```
docs/
├── index.md
├── getting-started.md
├── index.zh.md
├── getting-started.zh.md
```

### 8.4 Crowdin 集成

Crowdin 是专业的翻译管理平台，支持与 GitHub 集成：

**配置文件（crowdin.yml）**：
```yaml
project_id_env: CROWDIN_PROJECT_ID
api_token_env: CROWDIN_PERSONAL_TOKEN

files:
  - source: /docs/**/*.md
    translation: /docs/**/%file_name%.%two_letters_code%.md
```

**GitHub Actions 集成**：
```yaml
name: Crowdin

on:
  push:
    branches: [main]
    paths:
      - 'docs/**'

jobs:
  crowdin-upload:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Upload sources
        uses: crowdin/github-action@v1
        with:
          upload_sources: true
          download_translations: true
          config: crowdin.yml
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          CROWDIN_PROJECT_ID: ${{ secrets.CROWDIN_PROJECT_ID }}
          CROWDIN_PERSONAL_TOKEN: ${{ secrets.CROWDIN_PERSONAL_TOKEN }}
```

### 8.5 翻译最佳实践

1. **保持术语一致性**：建立多语言术语表
2. **避免硬编码字符串**：使用 i18n 框架管理文本
3. **提供语言切换**：在导航栏添加语言选择器
4. **定期同步**：确保翻译与原文保持同步
5. **社区翻译**：鼓励社区贡献翻译

---

## 9. 文档自动化测试

### 9.1 为什么需要文档测试

文档中的代码示例可能因为以下原因失效：

- API 变更
- 依赖版本更新
- 配置文件变化
- 环境差异

### 9.2 代码块测试

**Python doctest**：
```python
def add(a, b):
    """
    计算两数之和
    
    >>> add(1, 2)
    3
    >>> add(-1, 1)
    0
    """
    return a + b

if __name__ == "__main__":
    import doctest
    doctest.testmod()
```

**Markdown 代码块测试**（使用 `markdown-test`）：
```bash
# 安装
npm install -g markdown-test

# 运行测试
markdown-test docs/**/*.md
```

### 9.3 链接检查

**使用 markdown-link-check**：
```bash
# 安装
npm install -g markdown-link-check

# 检查单个文件
markdown-link-check README.md

# 检查目录
find docs -name "*.md" -exec markdown-link-check {} \;
```

**GitHub Actions 配置**：
```yaml
name: Check Links

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  link-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Check links
        uses: gaurav-nelson/github-action-markdown-link-check@v1
        with:
          use-quiet-mode: 'yes'
          config-file: '.mlc-config.json'
```

**配置文件（.mlc-config.json）**：
```json
{
  "retryOn429": true,
  "retryCount": 3,
  "aliveStatusCodes": [200, 206, 301, 302],
  "ignorePatterns": [
    {
      "pattern": "^http://localhost"
    }
  ]
}
```

### 9.4 拼写检查

**使用 cspell**：
```bash
# 安装
npm install -g cspell

# 检查文件
cspell "docs/**/*.md"

# 添加自定义词汇
cspell add words
```

**配置文件（cspell.json）**：
```json
{
  "version": "0.2",
  "language": "en",
  "words": [
    "GitHub",
    "API",
    "CLI",
    "Docusaurus",
    "VitePress"
  ],
  "ignorePaths": [
    "node_modules",
    "package-lock.json"
  ]
}
```

### 9.5 文档 CI/CD 流程

完整的文档 CI/CD 流程：

```yaml
name: Docs CI

on:
  push:
    branches: [main]
    paths:
      - 'docs/**'
  pull_request:
    branches: [main]
    paths:
      - 'docs/**'

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Check markdown
        uses: avto-dev/markdown-lint@v1
        with:
          args: './docs'
      
      - name: Check links
        uses: gaurav-nelson/github-action-markdown-link-check@v1
      
      - name: Spell check
        run: npx cspell "docs/**/*.md"
  
  build:
    needs: lint
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: '20'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Build docs
        run: npm run docs:build
      
      - name: Upload artifact
        uses: actions/upload-artifact@v4
        with:
          name: docs
          path: docs/dist
  
  deploy:
    needs: build
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - name: Download artifact
        uses: actions/download-artifact@v4
        with:
          name: docs
      
      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: .
```

---

## 10. GitHub Wiki 使用指南

### 10.1 Wiki 简介

GitHub Wiki 是每个仓库自带的文档功能，适合编写项目文档、知识库和教程。

**特点**：
- 独立的 Git 仓库
- 支持 Markdown 语法
- 可以本地克隆编辑
- 支持侧边栏导航

### 10.2 Wiki 创建和编辑

**启用 Wiki**：
1. 进入仓库 Settings
2. 在 Features 部分勾选 Wikis
3. 点击 Wiki 标签开始创建

**创建页面**：
1. 点击 "New Page"
2. 输入页面标题
3. 编写内容（Markdown 格式）
4. 点击 "Save Page"

### 10.3 Wiki 本地管理

**克隆 Wiki**：
```bash
# Wiki 地址格式
git clone https://github.com/owner/repo.wiki.git

# 示例
git clone https://github.com/octocat/Hello-World.wiki.git
```

**本地编辑**：
```bash
cd repo.wiki.git

# 编辑文件
vim Home.md

# 提交更改
git add .
git commit -m "Update wiki"
git push
```

### 10.4 Wiki 侧边栏

编辑 `_Sidebar.md` 文件来定义侧边栏：

```markdown
## 目录

* [首页](Home)
* [快速开始](Getting-Started)
* [安装指南](Installation)
  * [Windows](Installation-Windows)
  * [macOS](Installation-macOS)
  * [Linux](Installation-Linux)
* [API 参考](API-Reference)
* [常见问题](FAQ)
```

### 10.5 Wiki 脚注

编辑 `_Footer.md` 文件来定义页脚：

```markdown
---

**项目链接**
- [GitHub 仓库](https://github.com/owner/repo)
- [问题反馈](https://github.com/owner/repo/issues)
- [讨论区](https://github.com/owner/repo/discussions)
```

### 10.6 Wiki 最佳实践

1. **结构清晰**：使用侧边栏组织页面层级
2. **链接互通**：在页面之间添加链接
3. **定期维护**：保持 Wiki 内容与项目同步
4. **本地备份**：定期克隆 Wiki 仓库备份
5. **贡献指南**：说明如何贡献 Wiki 内容

### 10.7 Wiki 局限性

**不支持**：
- Pull Request 审核
- 代码审查
- 版本分支
- 复杂的目录结构

**替代方案**：
- 文档仓库（docs/ 目录）
- GitHub Pages
- 外部文档平台（如 Readme.io）

---

## 11. GitHub Discussions 作为知识库

### 11.1 Discussions 简介

GitHub Discussions 是 GitHub 提供的讨论功能，适合构建社区知识库：

**特点**：
- 分类管理讨论
- 支持问答格式
- 可以标记最佳答案
- 支持投票和反应

### 11.2 启用 Discussions

1. 进入仓库 Settings
2. 在 Features 部分勾选 Discussions
3. 点击 Discussions 标签开始配置

### 11.3 讨论分类

建议创建以下分类：

| 分类 | 用途 | 格式 |
|------|------|------|
| 📢 公告 | 项目公告 | 讨论 |
| 💡 想法 | 功能建议 | 讨论 |
| ❓ Q&A | 问题解答 | 问答 |
| 🐛 Bug 报告 | 问题反馈 | 讨论 |
| 📖 使用教程 | 教程分享 | 讨论 |
| 🎉 展示 | 项目展示 | 讨论 |

### 11.4 Discussions 配置

**分类配置文件**：
```yaml
# .github/DISCUSSION_TEMPLATE/announcement.yml
title: "📢 [公告] "
labels: ["announcement"]
body:
  - type: textarea
    id: content
    attributes:
      label: 公告内容
      description: 输入公告内容
    validations:
      required: true
```

**问答模板**：
```yaml
# .github/DISCUSSION_TEMPLATE/q-and-a.yml
title: "❓ [问题] "
labels: ["question"]
body:
  - type: textarea
    id: description
    attributes:
      label: 问题描述
      description: 详细描述你的问题
      placeholder: |
        1. 你想要做什么？
        2. 你尝试了什么？
        3. 你期望的结果是什么？
    validations:
      required: true
  
  - type: textarea
    id: environment
    attributes:
      label: 环境信息
      description: 提供你的环境信息
      placeholder: |
        - 操作系统：
        - Node.js 版本：
        - 包管理器：
    validations:
      required: false
```

### 11.5 Discussions 与 Issues 的区别

| 特性 | Issues | Discussions |
|------|--------|-------------|
| 用途 | Bug 修复、任务跟踪 | 讨论、问答、想法 |
| 状态 | 开启/关闭 | 无状态 |
| 最佳答案 | 不支持 | 支持 |
| 投票 | 不支持 | 支持 |
| 分类 | 标签 | 分类 |
| 格式 | 单一 | 多种（讨论、问答、投票等） |

### 11.6 将 Discussion 转换为 Issue

如果讨论中发现了一个 Bug，可以将其转换为 Issue：

1. 在 Discussion 中找到相关的回复
2. 点击 "..." 菜单
3. 选择 "Transfer to issue"
4. 填写 Issue 信息

### 11.7 Discussions 最佳实践

1. **建立模板**：为不同类型的讨论创建模板
2. **分类清晰**：明确每个分类的用途
3. **及时回复**：保持社区活跃
4. **标记答案**：对于 Q&A 分类，及时标记最佳答案
5. **链接相关**：在 Discussion 和 Issue 之间建立链接

---

## 12. 技术博客写作

### 12.1 GitHub Pages + Jekyll

**Jekyll 简介**：
Jekyll 是 GitHub Pages 原生支持的静态站点生成器。

**创建博客**：
```bash
# 创建新仓库
# 仓库名格式：username.github.io

# 克隆仓库
git clone https://github.com/username/username.github.io.git
cd username.github.io

# 初始化 Jekyll
jekyll new .
```

**目录结构**：
```
username.github.io/
├── _posts/
│   └── 2024-01-01-my-first-post.md
├── _config.yml
├── index.md
└── about.md
```

**配置文件（_config.yml）**：
```yaml
title: 我的技术博客
description: 分享技术心得
url: https://username.github.io

theme: minima

plugins:
  - jekyll-feed
  - jekyll-seo-tag

social:
  github: username
  twitter: username
```

**编写文章**：
```markdown
---
layout: post
title: "Git 工作流最佳实践"
date: 2024-01-01 12:00:00 +0800
categories: [Git, 工具]
tags: [git, workflow, best-practices]
---

## 为什么需要 Git 工作流

Git 工作流是团队协作的基础...

## 常见的 Git 工作流

### Git Flow

Git Flow 是最经典的工作流...
```

### 12.2 GitHub Pages + Hugo

**创建 Hugo 博客**：
```bash
# 创建新站点
hugo new site blog
cd blog

# 安装主题
git init
git submodule add https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod

# 创建文章
hugo new posts/my-first-post.md
```

**配置文件（hugo.toml）**：
```toml
baseURL = 'https://username.github.io/blog/'
languageCode = 'zh-CN'
title = '我的技术博客'
theme = 'PaperMod'

[params]
  author = "Your Name"
  description = "分享技术心得"
  defaultTheme = "auto"
  ShowReadingTime = true
  ShowShareButtons = true

[[menu.main]]
  name = "首页"
  url = "/"
  weight = 1

[[menu.main]]
  name = "文章"
  url = "/posts/"
  weight = 2

[[menu.main]]
  name = "标签"
  url = "/tags/"
  weight = 3

[[menu.main]]
  name = "关于"
  url = "/about/"
  weight = 4
```

### 12.3 GitHub Actions 自动部署

**Jekyll 部署**：
```yaml
name: Deploy Jekyll

on:
  push:
    branches: [main]

permissions:
  contents: read
  pages: write
  id-token: write

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Ruby
        uses: ruby/setup-ruby@v1
        with:
          ruby-version: '3.2'
          bundler-cache: true
      
      - name: Build
        run: bundle exec jekyll build
      
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./_site

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - name: Deploy
        id: deployment
        uses: actions/deploy-pages@v4
```

**Hugo 部署**：
```yaml
name: Deploy Hugo

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          submodules: true
          fetch-depth: 0
      
      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: 'latest'
          extended: true
      
      - name: Build
        run: hugo --minify
      
      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
```

### 12.4 技术博客写作技巧

**文章结构**：
```markdown
# 标题

## 概述
简要介绍文章内容（2-3 句）

## 前置知识
列出读者需要了解的内容

## 正文
分步骤讲解，配合代码示例

## 总结
回顾要点，提供下一步建议

## 参考资料
列出相关链接
```

**代码示例原则**：
1. 完整可运行
2. 有注释说明
3. 突出重点部分
4. 提供完整代码仓库链接

**SEO 优化**：
- 使用描述性标题
- 添加关键词
- 使用内部链接
- 添加图片 alt 文本
- 生成 sitemap

---

## 13. 文档贡献工作流

### 13.1 文档贡献流程

```
Fork 仓库
    ↓
创建分支
    ↓
编辑文档
    ↓
本地预览
    ↓
提交更改
    ↓
创建 Pull Request
    ↓
代码审查
    ↓
合并发布
```

### 13.2 文档贡献指南模板

在仓库中创建 `CONTRIBUTING.md`：

```markdown
# 贡献指南

感谢你对本项目的贡献！

## 如何贡献文档

### 1. Fork 仓库

点击仓库右上角的 "Fork" 按钮。

### 2. 克隆仓库

```bash
git clone https://github.com/your-username/project.git
cd project
```

### 3. 创建分支

```bash
git checkout -b docs/your-topic
```

### 4. 编辑文档

使用你喜欢的编辑器编辑文档。

### 5. 本地预览

```bash
# 安装依赖
npm install

# 本地预览
npm run docs:dev
```

### 6. 提交更改

```bash
git add .
git commit -m "docs: 描述你的更改"
```

### 7. 推送并创建 PR

```bash
git push origin docs/your-topic
```

然后在 GitHub 上创建 Pull Request。

## 文档规范

- 使用中文撰写
- 遵循 [中文技术文档排版规范](#中文技术文档排版规范)
- 代码示例必须可运行
- 图片存放在 `docs/assets` 目录

## 问题反馈

如有问题，请在 Discussions 中提问。
```

### 13.3 Pull Request 模板

创建 `.github/PULL_REQUEST_TEMPLATE/docs.md`：

```markdown
## 文档更改类型

- [ ] 新增文档
- [ ] 修复错误
- [ ] 更新内容
- [ ] 翻译

## 更改内容

<!-- 描述你的更改 -->

## 相关 Issue

<!-- 关联的 Issue 编号 -->

## 检查清单

- [ ] 文档格式正确
- [ ] 代码示例可运行
- [ ] 链接有效
- [ ] 图片显示正常
- [ ] 已添加必要的说明

## 截图（如适用）

<!-- 添加截图 -->
```

### 13.4 代码审查最佳实践

**作为审查者**：
1. 检查技术准确性
2. 验证代码示例
3. 检查格式和风格
4. 提供建设性反馈
5. 及时响应

**作为贡献者**：
1. 预览更改效果
2. 自我检查
3. 响应审查意见
4. 保持耐心
5. 学习改进

### 13.5 自动化贡献流程

**GitHub Actions 自动检查**：
```yaml
name: PR Docs Check

on:
  pull_request:
    paths:
      - 'docs/**'

jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Check markdown
        uses: avto-dev/markdown-lint@v1
      
      - name: Check links
        uses: gaurav-nelson/github-action-markdown-link-check@v1
      
      - name: Build preview
        run: |
          npm ci
          npm run docs:build
      
      - name: Comment preview URL
        uses: actions/github-script@v7
        with:
          script: |
            github.rest.issues.createComment({
              owner: context.repo.owner,
              repo: context.repo.repo,
              issue_number: context.issue.number,
              body: '✅ 文档预览构建成功！'
            })
```

---

## 14. 中英混合排版最佳实践

中英混合排版是中文技术文档中最常见的排版挑战之一。由于技术领域大量使用英文术语，如何在中文文档中正确处理英文内容，直接影响文档的专业性和可读性。

### 14.1 基本规则

**中英文之间加空格**：
这是中英混合排版最基本的规则。在中文和英文、数字之间应该添加一个半角空格，这样可以提高阅读体验，避免字符粘连。

```markdown
# 推荐
使用 GitHub 进行版本控制
安装 Node.js 依赖
项目有 100 个 Star

# 不推荐
使用GitHub进行版本控制
安装Node.js依赖
项目有100个Star
```

空格的添加不仅是为了美观，更是为了提高可读性。当英文单词紧邻中文字符时，读者的视线需要快速切换语言环境，空格可以提供一个视觉上的缓冲。

**数字和中文之间加空格**：
数字与中文之间同样需要添加空格，这样可以使数字更加醒目，便于阅读。

```markdown
# 推荐
仓库有 100 个星标
3 个提交记录
版本号为 2.0.1

# 不推荐
仓库有100个星标
3个提交记录
版本号为2.0.1
```

**全角标点与半角标点**：
中文语境中应使用全角标点符号，英文语境中应使用半角标点符号。

```markdown
# 推荐
这是一个示例，演示如何使用 GitHub。
请运行 `npm install` 命令。

# 不推荐
这是一个示例,演示如何使用GitHub.
请运行 `npm install` 命令
```

### 14.2 标点符号使用规范

**中文标点符号**：
- 逗号：，（全角）
- 句号：。（全角）
- 冒号：：（全角）
- 分号；（全角）
- 问号：？（全角）
- 感叹号：！（全角）
- 引号：""（全角）
- 括号：（）（全角）

**英文标点符号**：
- 逗号：,（半角）
- 句号：.（半角）
- 冒号：:（半角）
- 分号：;（半角）
- 问号：?（半角）
- 感叹号：!（半角）
- 引号：""（半角）
- 括号：()（半角）

**混合使用规则**：
当句子以中文为主，包含英文单词或代码时，使用中文标点：
```markdown
运行 `git commit` 命令，提交你的更改。
```

当句子以英文为主，包含中文说明时，使用英文标点：
```markdown
Use `git commit` to commit your changes.
```

### 14.3 专有名词处理

**保持原样，不翻译**：
技术领域的专有名词应该保持英文原样，不进行翻译。这是因为：
1. 专有名词的中文翻译可能不统一
2. 开发者更熟悉英文原词
3. 便于搜索和交流

```markdown
# 推荐
使用 GitHub Actions 进行 CI/CD
配置 Docker 容器
运行 Kubernetes 集群

# 不推荐
使用 GitHub 动作进行持续集成/持续部署
配置 Docker 容器
运行 Kubernetes 集群
```

**首次出现可加注释**：
对于不太常见的专有名词，首次出现时可以添加中文注释，帮助读者理解。

```markdown
持续集成（Continuous Integration，CI）是一种开发实践，要求开发者频繁地将代码集成到共享仓库中。
```

**常见技术术语对照表**：

| 英文术语 | 中文术语 | 建议使用 |
|----------|----------|----------|
| Repository | 仓库 | 英文或中文均可 |
| Branch | 分支 | 英文或中文均可 |
| Pull Request | 拉取请求 | 英文（PR） |
| Issue | 问题/议题 | 英文 |
| Fork | 复刻 | 英文 |
| Clone | 克隆 | 英文 |
| Commit | 提交 | 英文或中文均可 |
| Push | 推送 | 英文或中文均可 |
| Merge | 合并 | 英文或中文均可 |
| Deploy | 部署 | 中文 |
| Build | 构建 | 中文 |
| Test | 测试 | 中文 |
| Debug | 调试 | 中文 |
| API | 接口 | 英文 |
| SDK | 开发工具包 | 英文 |
| IDE | 集成开发环境 | 英文 |
| CLI | 命令行界面 | 英文 |
| GUI | 图形用户界面 | 英文 |
| URL | 网址 | 英文 |
| HTTP | 超文本传输协议 | 英文 |
| JSON | JavaScript 对象表示法 | 英文 |
| YAML | YAML 格式 | 英文 |
| Markdown | Markdown 格式 | 英文 |
| Docker | Docker 容器 | 英文 |
| Kubernetes | K8s | 英文 |
| Git | Git 版本控制 | 英文 |
| Node.js | Node.js 运行时 | 英文 |
| React | React 框架 | 英文 |
| Vue | Vue 框架 | 英文 |
| Angular | Angular 框架 | 英文 |
| TypeScript | TypeScript 语言 | 英文 |
| JavaScript | JavaScript 语言 | 英文 |
| Python | Python 语言 | 英文 |
| Java | Java 语言 | 英文 |
| Go | Go 语言 | 英文 |
| Rust | Rust 语言 | 英文 |

### 14.4 代码和命令处理

**代码使用英文标点**：
在代码块或行内代码中，始终使用英文标点符号。

```markdown
# 推荐
运行 `git commit -m "message"` 提交更改。
配置文件路径为 `/etc/config.yml`。

# 不推荐
运行 `git commit -m "message"` 提交更改。
配置文件路径为 `/etc/config.yml`。
```

**命令说明使用中文标点**：
当用中文描述命令或代码时，使用中文标点。

```markdown
# 推荐
这个命令用于提交更改。参数 `-m` 用于指定提交信息。

# 不推荐
这个命令用于提交更改.参数 `-m` 用于指定提交信息.
```

**代码注释的语言选择**：
根据项目受众选择注释语言：
- 面向国际开发者：使用英文注释
- 面向中国开发者：可以使用中文注释
- 开源项目：建议使用英文注释

```python
# 英文注释（面向国际开发者）
def calculate_sum(a, b):
    """Calculate the sum of two numbers."""
    return a + b

# 中文注释（面向中国开发者）
def calculate_sum(a, b):
    """计算两个数的和。"""
    return a + b
```

### 14.5 排版工具推荐

**中文排版检查工具**：

1. **pangu.js**：自动在中英文之间加空格
   ```bash
   npm install -g pangu
   pangu --help
   ```

2. **lint-md**：中文 Markdown 排版检查
   ```bash
   npm install -g lint-md
   lint-md docs/**/*.md
   ```

3. **zhlint**：中文排版格式化工具
   ```bash
   npm install -g zhlint
   zhlint --fix docs/**/*.md
   ```

**VS Code 插件**：
1. **Chinese Typography**：自动处理中英文间距
2. **Markdown Lint**：Markdown 格式检查
3. **Pangu Mark**：自动添加中英文间距

**GitHub Actions 集成**：
```yaml
name: Lint Markdown

on:
  push:
    paths:
      - '**/*.md'
  pull_request:
    paths:
      - '**/*.md'

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Run lint-md
        run: |
          npm install -g lint-md
          lint-md docs/**/*.md --config .lintmdrc.json
```

**lint-md 配置文件（.lintmdrc.json）**：
```json
{
  "rules": {
    "no-empty-code": true,
    "no-empty-blockquote": true,
    "no-empty-list": true,
    "no-fullwidth-number": true,
    "no-space-in-inline-code": true,
    "no-trailing-punctuation": true,
    "no-long-code": {
      "length": 100,
      "exclude": ["code"]
    },
    "space-round-number": true,
    "space-round-letter": true,
    "space-round-alphabet": true
  }
}
```

### 14.6 常见错误示例

| 错误类型 | 错误示例 | 正确示例 | 说明 |
|----------|----------|----------|------|
| 缺少空格 | 使用GitHub进行版本控制 | 使用 GitHub 进行版本控制 | 中英文之间加空格 |
| 标点错误 | 这是示例. | 这是示例。 | 中文语境用全角标点 |
| 代码标点 | 运行 `npm install`。 | 运行 `npm install`。 | 代码中的标点用半角 |
| 过度翻译 | 拉取请求 | Pull Request | 保持专有名词原样 |
| 格式混乱 | 运行`npm install`命令 | 运行 `npm install` 命令 | 代码前后加空格 |
| 数字格式 | 仓库有100个星标 | 仓库有 100 个星标 | 数字前后加空格 |
| 混用标点 | 使用GitHub,进行版本控制 | 使用 GitHub，进行版本控制 | 统一使用全角标点 |
| 缺少注释 | 使用 CI/CD | 使用 CI/CD（持续集成/持续部署） | 首次出现加注释 |

### 14.7 排版检查清单

在发布文档前，使用以下检查清单：

**基本格式检查**：
- [ ] 中英文之间是否有空格
- [ ] 数字和中文之间是否有空格
- [ ] 标点符号是否正确（全角/半角）
- [ ] 代码块中的标点是否使用半角
- [ ] 专有名词是否保持原样

**内容检查**：
- [ ] 术语使用是否一致
- [ ] 链接是否有效
- [ ] 图片是否显示正常
- [ ] 代码示例是否可运行
- [ ] 是否有拼写错误

**可读性检查**：
- [ ] 段落是否过长
- [ ] 标题层级是否合理
- [ ] 列表使用是否恰当
- [ ] 是否有足够的示例
- [ ] 是否有必要的说明

**国际化检查**：
- [ ] 英文内容是否需要翻译
- [ ] 是否需要添加中文注释
- [ ] 是否考虑了不同语言读者的需求
- [ ] 是否提供了多语言版本

### 14.8 最佳实践总结

**核心原则**：
1. **一致性**：在整个文档中保持一致的排版风格
2. **可读性**：排版应该服务于内容，提高阅读体验
3. **专业性**：技术文档应该体现专业水准
4. **国际化**：考虑不同语言背景读者的需求

**具体建议**：
1. 建立项目排版规范文档
2. 使用自动化工具检查排版
3. 在代码审查中关注排版问题
4. 定期更新排版规范
5. 收集读者反馈并改进

**常见问题解答**：

**问**：中英文之间一定要加空格吗？
**答**：是的，这是中英混合排版的基本规则。加空格可以提高可读性，避免字符粘连。

**问**：代码中的中文注释应该用什么标点？
**答**：代码注释通常使用英文标点，因为代码编辑器和 IDE 对英文标点的支持更好。但如果项目明确面向中国开发者，也可以使用中文标点。

**问**：专有名词要不要翻译？
**答**：一般不翻译。技术领域的专有名词保持英文原样更便于交流和搜索。首次出现时可以添加中文注释。

**问**：如何处理中英文混用的标题？
**答**：标题中的英文专有名词保持原样，其他部分使用中文。例如："GitHub Actions 使用指南"。

**问**：文档中的数字用阿拉伯数字还是中文数字？
**答**：一般使用阿拉伯数字，更便于阅读。例如："3 个步骤"而不是"三个步骤"。

---

## 15. 技术文档项目实战

技术文档不仅仅是代码的附属品，它是开发者与用户之间沟通的桥梁。一个优秀的技术文档项目需要系统化的规划、持续的维护和不断的优化。本章将通过实际案例，带你从零开始构建一个完整的技术文档体系。

### 15.1 从零开始创建项目文档

假设你正在开发一个名为 `awesome-cli` 的命令行工具，需要为其创建完整的文档体系。我们将从文档规划、内容编写到部署上线，一步步完成整个流程。

**第一步：规划文档结构**

良好的文档结构是成功的一半。在开始编写之前，需要仔细规划文档的组织方式：

```
awesome-cli-docs/
├── docs/
│   ├── index.md                 # 首页
│   ├── getting-started.md       # 快速开始
│   ├── installation.md          # 安装指南
│   ├── configuration.md         # 配置说明
│   ├── commands/                # 命令参考
│   │   ├── init.md
│   │   ├── build.md
│   │   └── deploy.md
│   ├── guides/                  # 使用指南
│   │   ├── basic-usage.md
│   │   ├── advanced-usage.md
│   │   └── best-practices.md
│   ├── api/                     # API 文档
│   │   ├── overview.md
│   │   └── reference.md
│   ├── faq.md                   # 常见问题
│   └── changelog.md             # 更新日志
├── mkdocs.yml
└── README.md
```

**第二步：编写首页内容**

首页是用户接触文档的第一印象，应该简洁明了地介绍项目：

```markdown
# Awesome CLI 文档

欢迎使用 Awesome CLI！这是一个强大的命令行工具，帮助你快速构建和部署应用。

## 功能特性

- **快速构建**：一键构建项目，支持多种框架
- **智能部署**：自动检测环境，一键部署到云端
- **插件系统**：丰富的插件生态，扩展功能无限可能
- **跨平台支持**：支持 Windows、macOS 和 Linux

## 快速开始

```bash
# 安装
npm install -g awesome-cli

# 初始化项目
awesome-cli init my-project

# 构建项目
awesome-cli build

# 部署项目
awesome-cli deploy
```

## 获取帮助

- [快速开始指南](getting-started.md)
- [命令参考](commands/init.md)
- [常见问题](faq.md)
- [GitHub Issues](https://github.com/your-username/awesome-cli/issues)
```

**第三步：编写快速开始指南**

快速开始指南应该让新手在 5 分钟内上手使用：

```markdown
# 快速开始

本指南将在 5 分钟内带你了解 Awesome CLI 的基本使用方法。

## 前置条件

- Node.js 18 或更高版本
- npm 9 或更高版本
- Git

## 安装

使用 npm 全局安装：

```bash
npm install -g awesome-cli
```

验证安装：

```bash
awesome-cli --version
# 输出: awesome-cli v1.0.0
```

## 创建第一个项目

```bash
# 初始化项目
awesome-cli init my-first-project

# 进入项目目录
cd my-first-project

# 查看项目结构
ls -la
```

项目结构如下：

```
my-first-project/
├── src/
│   └── index.js
├── tests/
│   └── index.test.js
├── package.json
├── awesome.config.js
└── README.md
```

## 构建项目

```bash
awesome-cli build
```

构建完成后，会在 `dist/` 目录生成构建产物。

## 部署项目

```bash
awesome-cli deploy
```

按照提示选择部署环境，完成部署。

## 下一步

- [配置说明](configuration.md) - 了解如何配置项目
- [命令参考](commands/init.md) - 查看所有可用命令
- [使用指南](guides/basic-usage.md) - 深入学习使用方法
```

### 15.2 文档维护工作流

建立文档与代码同步更新的工作流，确保文档始终与代码保持一致：

**GitHub Actions 配置**：
```yaml
name: Docs Update Reminder

on:
  push:
    branches: [main]
    paths:
      - 'src/**'
      - '!docs/**'

jobs:
  remind:
    runs-on: ubuntu-latest
    steps:
      - name: Check if docs need update
        uses: actions/github-script@v7
        with:
          script: |
            const { data: prs } = await github.rest.pulls.list({
              owner: context.repo.owner,
              repo: context.repo.repo,
              state: 'closed',
              sort: 'updated',
              direction: 'desc',
              per_page: 1
            });
            
            if (prs.length > 0) {
              const pr = prs[0];
              const { data: files } = await github.rest.pulls.listFiles({
                owner: context.repo.owner,
                repo: context.repo.repo,
                pull_number: pr.number
              });
              
              const hasCodeChanges = files.some(f => 
                f.filename.startsWith('src/') && 
                !f.filename.startsWith('docs/')
              );
              
              if (hasCodeChanges) {
                await github.rest.issues.create({
                  owner: context.repo.owner,
                  repo: context.repo.repo,
                  title: '提醒：代码变更，请检查文档是否需要更新',
                  body: `PR #${pr.number} 修改了代码，但未更新文档。\n\n请检查以下内容是否需要更新：\n- API 文档\n- 命令参考\n- 配置说明\n- 使用示例`,
                  labels: ['documentation']
                });
              }
            }
```

### 15.3 文档质量评估指标

建立文档质量评估体系，量化文档质量：

| 指标 | 说明 | 目标值 |
|------|------|--------|
| 覆盖率 | 已文档化的功能比例 | > 90% |
| 准确性 | 文档与实际功能的一致性 | 100% |
| 时效性 | 文档最后更新时间 | < 30 天 |
| 可读性 | 阅读难度评分 | 中等以下 |
| 完整性 | 必要章节的完整性 | 100% |
| 链接有效性 | 有效链接比例 | > 95% |

**自动化检查脚本**：
```bash
#!/bin/bash
# docs-quality-check.sh

echo "=== 文档质量检查报告 ==="
echo ""

# 检查文件数量
TOTAL_FILES=$(find docs -name "*.md" | wc -l)
echo "文档文件总数: $TOTAL_FILES"

# 检查文件大小
TOTAL_SIZE=$(find docs -name "*.md" -exec wc -c {} + | tail -1 | awk '{print $1}')
echo "文档总大小: $TOTAL_SIZE 字节"

# 检查链接有效性
echo ""
echo "检查链接有效性..."
broken_links=0
for file in docs/**/*.md; do
  while IFS= read -r line; do
    if [[ $line =~ \[.*\]\((http[s]?://[^)]+)\) ]]; then
      url="${BASH_REMATCH[1]}"
      status=$(curl -s -o /dev/null -w "%{http_code}" "$url")
      if [ "$status" != "200" ]; then
        echo "  失效链接: $url (在 $file)"
        ((broken_links++))
      fi
    fi
  done < "$file"
done
echo "失效链接数: $broken_links"

# 检查代码块
echo ""
echo "检查代码块..."
code_blocks=$(grep -r '```' docs --count | awk -F: '{sum+=$2} END {print sum}')
echo "代码块总数: $((code_blocks / 2))"

echo ""
echo "=== 检查完成 ==="
```

### 15.4 文档国际化实战

为项目添加多语言支持的完整流程：

**第一步：提取可翻译内容**

```bash
# 使用 Docusaurus 提取翻译字符串
npm run write-translations -- --locale zh-Hans
```

**第二步：翻译内容文件**

目录结构：
```
i18n/
├── zh-Hans/
│   ├── docusaurus-plugin-content-docs/
│   │   └── current/
│   │       ├── getting-started.md
│   │       └── commands/
│   │           └── init.md
│   └── docusaurus-theme-classic/
│       ├── navbar.json
│       └── footer.json
└── en/
    └── docusaurus-plugin-content-docs/
        └── current/
            └── getting-started.md
```

**第三步：配置语言切换**

```javascript
// docusaurus.config.js
module.exports = {
  i18n: {
    defaultLocale: 'zh-Hans',
    locales: ['zh-Hans', 'en'],
    localeConfigs: {
      'zh-Hans': {
        label: '简体中文',
        direction: 'ltr',
        htmlLang: 'zh-Hans',
      },
      en: {
        label: 'English',
        direction: 'ltr',
        htmlLang: 'en-US',
      },
    },
  },
};
```

**第四步：翻译工作流自动化**

```yaml
# .github/workflows/translate.yml
name: Translation Sync

on:
  push:
    branches: [main]
    paths:
      - 'docs/**'
      - '!i18n/**'

jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Detect changes
        id: changes
        run: |
          git diff HEAD~1 --name-only | grep -v i18n | grep docs > changed_files.txt
          if [ -s changed_files.txt ]; then
            echo "has_changes=true" >> $GITHUB_OUTPUT
          fi
      
      - name: Create issue
        if: steps.changes.outputs.has_changes == 'true'
        uses: actions/github-script@v7
        with:
          script: |
            const fs = require('fs');
            const changedFiles = fs.readFileSync('changed_files.txt', 'utf8');
            
            await github.rest.issues.create({
              owner: context.repo.owner,
              repo: context.repo.repo,
              title: '翻译提醒：文档内容已更新',
              body: `以下文档已更新，请同步翻译：\n\n\`\`\`\n${changedFiles}\n\`\`\`\n\n请更新对应的翻译文件。`,
              labels: ['translation']
            });
```

### 15.5 文档搜索优化

提升文档搜索体验的技巧：

**本地搜索配置（MkDocs）**：
```yaml
# mkdocs.yml
plugins:
  - search:
      lang: zh
      separator: '[\s\-\.]+'
```

**Algolia DocSearch 集成**：
```javascript
// docusaurus.config.js
module.exports = {
  themeConfig: {
    algolia: {
      appId: 'YOUR_APP_ID',
      apiKey: 'YOUR_SEARCH_API_KEY',
      indexName: 'your-index',
      contextualSearch: true,
      searchPagePath: 'search',
    },
  },
};
```

**搜索优化建议**：
1. 为每个页面添加描述性标题
2. 使用关键词丰富的章节标题
3. 添加元数据描述
4. 建立清晰的文档层级
5. 使用标签分类内容

### 15.6 文档性能优化

大型文档站点的性能优化：

**图片优化**：
```markdown
<!-- 使用 WebP 格式 -->
![示例图片](./assets/example.webp)

<!-- 添加尺寸属性 -->
<img src="./assets/example.png" width="600" height="400" alt="示例图片">

<!-- 使用懒加载 -->
<img src="./assets/example.png" loading="lazy" alt="示例图片">
```

**代码块优化**：
```markdown
<!-- 只展示关键代码 -->
```python
# 关键部分
def important_function():
    pass
```

<!-- 完整代码放在折叠区域 -->
<details>
<summary>查看完整代码</summary>

```python
# 完整实现
def important_function():
    # ... 100 行代码
    pass
```

</details>
```

**构建优化**：
```yaml
# GitHub Actions 缓存配置
- name: Cache docs
  uses: actions/cache@v4
  with:
    path: |
      docs/.cache
      node_modules
    key: docs-${{ hashFiles('package-lock.json') }}
    restore-keys: |
      docs-
```

### 15.7 文档自动化测试

文档中的代码示例可能因为各种原因失效，比如接口变更、依赖版本更新或者环境差异。建立自动化测试机制，可以及时发现并修复这些问题，确保文档的准确性和可靠性。

**测试策略**：
1. 代码块测试：提取文档中的代码块并执行验证
2. 链接检查：验证所有外部链接是否有效可访问
3. 拼写检查：检查文档中的拼写错误和格式问题
4. 格式检查：验证文档格式是否符合规范要求

**完整的测试流程**：
```yaml
# .github/workflows/docs-test.yml
name: Docs Testing

on:
  push:
    branches: [main]
    paths:
      - 'docs/**'
  pull_request:
    branches: [main]
    paths:
      - 'docs/**'

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Check markdown format
        run: |
          npm install -g markdownlint-cli
          markdownlint docs/**/*.md
      
      - name: Check links
        run: |
          npm install -g markdown-link-check
          find docs -name "*.md" -exec markdown-link-check {} \;
      
      - name: Spell check
        run: |
          npm install -g cspell
          cspell "docs/**/*.md"
      
      - name: Build docs
        run: npm run docs:build
```

### 15.8 文档贡献指南

为项目建立清晰的文档贡献指南，鼓励社区参与文档改进。良好的贡献指南可以降低参与门槛，提高文档质量：

```markdown
# 文档贡献指南

感谢你对本项目文档的贡献！以下是参与文档改进的指南。

## 如何贡献

### 报告问题

如果你发现文档中的错误或不清楚的地方，请通过以下方式反馈：
- 在 GitHub Issues 中创建新的问题
- 使用 "文档问题" 标签
- 详细描述问题所在和建议的改进方案

### 提交修改

1. Fork 项目仓库到你的账号
2. 创建文档分支：`git checkout -b docs/your-topic`
3. 修改文档内容并确保格式正确
4. 本地预览确认无误后提交
5. 创建 Pull Request 并描述修改内容

### 文档规范

- 使用中文撰写，保持语言简洁明了
- 遵循中英混合排版规范
- 代码示例必须可运行且有注释说明
- 图片存放在 `docs/assets` 目录
- 添加必要的注释和说明文字

## 文档结构

```
docs/
├── index.md           # 首页介绍
├── getting-started.md # 快速开始指南
├── guides/            # 详细使用指南
├── api/               # API 参考文档
└── faq.md             # 常见问题解答
```

## 本地开发

```bash
# 安装项目依赖
npm install

# 启动本地预览服务器
npm run docs:dev

# 构建生产版本文档
npm run docs:build
```

## 联系方式

如有任何疑问，请在 GitHub Discussions 中提问交流。
```

---

## 附录 A：推荐工具列表

| 类别 | 工具 | 用途 |
|------|------|------|
| 编辑器 | VS Code | 代码和文档编辑 |
| 编辑器 | Typora | Markdown 编辑 |
| 文档生成 | MkDocs | 项目文档 |
| 文档生成 | Docusaurus | 开源项目文档 |
| 文档生成 | VitePress | Vue 生态文档 |
| 文档生成 | Hugo | 博客和文档 |
| API 文档 | Swagger UI | REST API 文档 |
| API 文档 | Redoc | 美观的 API 文档 |
| 排版检查 | lint-md | 中文排版检查 |
| 排版检查 | zhlint | 中文格式化 |
| 链接检查 | markdown-link-check | 链接有效性检查 |
| 拼写检查 | cspell | 拼写检查 |
| 翻译管理 | Crowdin | 多语言翻译平台 |
| 版本管理 | mike | MkDocs 版本管理 |

## 附录 B：推荐阅读

- [Google 技术写作课程](https://developers.google.com/tech-writing)
- [Markdown 语法指南](https://www.markdownguide.org/)
- [中文文案排版指北](https://github.com/sparanoid/chinese-copywriting-guidelines)
- [OpenAPI 规范](https://swagger.io/specification/)
- [Docusaurus 文档](https://docusaurus.io/)
- [MkDocs 文档](https://www.mkdocs.org/)
- [Hugo 文档](https://gohugo.io/)

---

**最后更新**：2024 年 12 月

**许可协议**：本文档采用 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) 协议发布。
