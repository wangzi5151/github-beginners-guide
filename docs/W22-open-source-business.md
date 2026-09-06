# 开源项目商业化运营

## 商业化模式

| 模式 | 说明 | 示例 |
|------|------|------|
| 开源核心 + 商业版 | 基础功能开源，高级功能收费 | GitLab, Grafana |
| 托管服务 | 提供托管版本 | Redis Cloud, MongoDB Atlas |
| 专业支持 | 提供付费支持和咨询 | Red Hat, Canonical |
| SaaS 服务 | 基于开源的 SaaS 产品 | WordPress.com, Ghost |
| 双重许可 | 开源 + 商业许可 | MySQL, Elasticsearch |

## GitHub Sponsors

### 配置 FUNDING.yml

```yaml
# .github/FUNDING.yml
github: [your-username]
patreon: [your-patreon]
open_collective: [your-project]
ko_fi: [your-username]
custom: ['https://your-domain.com/sponsor']
```

### 赞助等级

```yaml
# .github/SPONSORS.yml
# 赞助等级配置
tiers:
  - name: Bronze
    amount: 5
    description: "感谢支持"
  - name: Silver
    amount: 20
    description: "获得项目徽章"
  - name: Gold
    amount: 100
    description: "获得优先支持"
```

## 企业版功能

### 功能对比表

```markdown
# README.md

| 功能 | 社区版 | 企业版 |
|------|--------|--------|
| 核心功能 | ✅ | ✅ |
| 多用户支持 | 10 人 | 无限 |
| SSO/SAML | ❌ | ✅ |
| 审计日志 | ❌ | ✅ |
| 优先支持 | ❌ | ✅ |
| 自定义集成 | 有限 | 无限 |
| SLA 保证 | ❌ | 99.9% |
```

### 定价页面

```markdown
# PRICING.md

## 社区版（免费）
- 核心功能
- 社区支持
- 基础文档

## 专业版（$29/月）
- 所有社区版功能
- 50 用户
- 邮件支持
- 高级文档

## 企业版（联系销售）
- 所有专业版功能
- 无限用户
- SSO/SAML
- 审计日志
- 24/7 支持
- SLA 保证
- 定制开发
```

## 商业文档

### LICENSE（商业版）

```markdown
# 商业许可协议

版权所有 (c) 2024 Your Company

未经授权，不得复制、修改或分发本软件。

购买商业许可请联系：sales@your-domain.com
```

### CONTRIBUTING.md（商业化）

```markdown
# 贡献指南

## 社区贡献
欢迎贡献代码、文档和 bug 报告。

## 商业功能
企业版功能不接受外部贡献。

## 贡献者许可协议 (CLA)
提交 PR 前需要签署 CLA。
```

## 营销策略

### README 优化

```markdown
# 项目名称

> 一句话描述项目价值

## 为什么选择我们？

- ✅ 功能 1
- ✅ 功能 2
- ✅ 功能 3

## 快速开始

[快速开始指南链接]

## 商业版

需要更多功能？查看 [商业版](PRICING.md)

## 社区

- [Discord](https://discord.gg/xxx)
- [Twitter](https://twitter.com/xxx)
- [博客](https://blog.xxx.com)
```

### 徽章和指标

```markdown
# 徽章

![GitHub stars](https://img.shields.io/github/stars/your-org/your-repo)
![GitHub forks](https://img.shields.io/github/forks/your-org/your-repo)
![GitHub issues](https://img.shields.io/github/issues/your-org/your-repo)
![License](https://img.shields.io/github/license/your-org/your-repo)
```

## GitHub 功能利用

### 1. GitHub Sponsors

```bash
# 设置赞助
gh api user/sponsorship --method POST \
  -f sponsorable=your-username \
  -f tier_id=1
```

### 2. GitHub Marketplace

```yaml
# 创建 GitHub App
name: your-app
description: "Your app description"
url: https://your-domain.com
```

### 3. GitHub Discussions

```markdown
# 讨论分类

- 💡 Ideas - 功能建议
- 🙋 Q&A - 问题解答
- 💬 General - 一般讨论
- 📢 Announcements - 公告
- 🎉 Show and Tell - 成果展示
```

## 收入来源

| 来源 | 说明 |
|------|------|
| SaaS 订阅 | 托管服务费用 |
| 企业许可 | 商业使用许可 |
| 咨询服务 | 定制开发和培训 |
| 赞助 | 个人和企业赞助 |
| 广告 | 开源项目中的广告 |
| 培训认证 | 官方培训和认证 |

## 最佳实践

1. **明确价值主张**：清楚说明开源和商业版的区别
2. **保护商业功能**：确保商业代码不被开源版使用
3. **提供优质支持**：商业客户期望更好的支持
4. **持续创新**：保持开源项目的活跃度
5. **建立社区**：培养忠实的用户和贡献者

## 相关资源

- [GitHub Sponsors 文档](https://docs.github.com/en/sponsors)
- [开源商业化指南](https://opensource.guide)
- [GitHub Marketplace](https://github.com/marketplace)

---

**上一篇：[GitHub REST/GraphQL API 实战](W21-api-advanced.md) | 下一篇：[练习 12：CI/CD 流水线实战](exercise-12-cicd-pipeline.md)**
