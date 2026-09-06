# 双语 README 指南

## 为什么需要双语 README？

对于中国开发者来说，双语 README 可以：
- 吸引国际用户和贡献者
- 提高项目的可访问性
- 展示专业性
- 扩大项目影响力

## 推荐结构

```
project-root/
├── README.md           # 主 README（英文）
├── README.zh-CN.md     # 中文版
└── docs/
    └── translations/   # 其他语言版本
```

## 英文 README 示例

```markdown
# Project Name

A brief description of what this project does.

[中文文档](README.zh-CN.md)

## Features

- Feature 1
- Feature 2
- Feature 3

## Quick Start

### Installation

```bash
npm install my-project
```

### Usage

```javascript
import { myFunction } from 'my-project';

myFunction();
```

## Documentation

- [Getting Started](docs/getting-started.md)
- [API Reference](docs/api.md)
- [Examples](docs/examples.md)

## Contributing

Contributions are welcome! Please read our [Contributing Guide](CONTRIBUTING.md) first.

## License

[MIT](LICENSE)
```

## 中文 README 示例

```markdown
# 项目名称

项目简介

[English](README.md)

## 功能特性

- 功能 1
- 功能 2
- 功能 3

## 快速开始

### 安装

```bash
npm install my-project
```

### 使用方法

```javascript
import { myFunction } from 'my-project';

myFunction();
```

## 文档

- [快速开始](docs/getting-started.md)
- [API 参考](docs/api.md)
- [示例](docs/examples.md)

## 贡献指南

欢迎贡献！请先阅读[贡献指南](CONTRIBUTING.md)。

## 许可证

[MIT](LICENSE)
```

## 使用工具自动管理

### GitDocs

```yaml
# .gitdocs.yml
default_locale: en
locales:
  - zh-CN
  - ja
  - ko
```

### GitHub Action 自动同步

```yaml
name: Sync README translations

on:
  push:
    branches: [main]
    paths: ['README.md']

jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    
    - name: Sync translations
      uses: your-org/sync-readme-action@v1
      with:
        source: README.md
        targets: |
          README.zh-CN.md
          README.ja.md
```

## 最佳实践

### 1. 保持内容同步

```bash
# 检查翻译版本是否过时
diff README.md README.zh-CN.md
```

### 2. 使用清晰的语言

- 避免俚语和成语
- 使用简单的句子结构
- 提供上下文说明

### 3. 格式一致

- 保持标题层级相同
- 使用相同的代码块格式
- 保持链接结构一致

### 4. 添加语言切换器

```markdown
[English](README.md) | [中文](README.zh-CN.md) | [日本語](README.ja.md)
```

### 5. 使用徽章显示版本

```markdown
![English](https://img.shields.io/badge/language-English-blue)
![中文](https://img.shields.io/badge/language-中文-red)
```

## 常见问题

### Q: 如何处理图片和图表？
A: 使用通用的图片，或者为不同语言版本提供本地化的图片。

### Q: 代码注释需要翻译吗？
A: 通常不需要翻译代码注释，但可以在 README 中提供双语说明。

### Q: 如何处理长文档？
A: 考虑使用文档工具（如 Docusaurus）来管理多语言文档。

## 相关资源

- [GitHub 多语言文档最佳实践](https://docs.github.com/en/communities/documenting-your-project)
- [Markdown 多语言指南](https://www.markdownguide.org/basic-syntax/#images)

---

**上一篇：[GitHub 项目实战案例](V-practical-examples.md) | 下一篇：[GitHub 教育资源](W-education.md)**
