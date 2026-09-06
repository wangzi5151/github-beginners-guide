# GitHub 开源指南

## 为什么参与开源？

| 好处 | 说明 |
|------|------|
| 技术提升 | 学习最佳实践 |
| 建立声誉 | 展示技术能力 |
| 职业发展 | 增加就业机会 |
| 社区贡献 | 回馈开源社区 |
| 人脉拓展 | 结识优秀开发者 |

## 找到适合的项目

### 寻找项目

| 平台 | 说明 |
|------|------|
| GitHub Explore | https://github.com/explore |
| Good First Issues | https://goodfirstissue.dev/ |
| Up For Grabs | https://up-for-grabs.net/ |
| First Timers Only | https://www.firsttimersonly.com/ |

### 选择标准

```markdown
## 适合新手的项目

- 有 good-first-issue 标签
- 文档完善
- 活跃维护
- 响应及时
- 有贡献指南
```

## 贡献流程

### 1. Fork 仓库

```bash
# Fork 仓库
gh repo fork owner/repo

# 克隆 Fork
git clone https://github.com/your-username/repo.git

# 添加上游
git remote add upstream https://github.com/owner/repo.git
```

### 2. 创建分支

```bash
# 从 main 创建分支
git checkout -b feature/your-feature

# 或修复 bug
git checkout -b fix/your-bug-fix
```

### 3. 开发和测试

```bash
# 安装依赖
npm install

# 运行测试
npm test

# 本地开发
npm run dev
```

### 4. 提交代码

```bash
# 添加更改
git add .

# 提交
git commit -m "feat: add your feature"

# 推送
git push origin feature/your-feature
```

### 5. 创建 PR

```bash
# 使用 CLI 创建 PR
gh pr create \
  --title "feat: add your feature" \
  --body "## Description\n描述你的更改\n\n## Related Issue\nCloses #123"
```

## 贡献类型

### 代码贡献

```markdown
## 代码贡献

- 新功能
- Bug 修复
- 性能优化
- 代码重构
```

### 文档贡献

```markdown
## 文档贡献

- 修复错别字
- 添加示例
- 完善说明
- 翻译文档
```

### 其他贡献

```markdown
## 其他贡献

- 报告 Bug
- 提出建议
- 回答问题
- 审查 PR
```

## 开源项目运营

### 创建项目

```markdown
## 项目创建清单

- [ ] 编写 README
- [ ] 添加 LICENSE
- [ ] 创建 CONTRIBUTING.md
- [ ] 创建 CODE_OF_CONDUCT.md
- [ ] 设置 Issue 模板
- [ ] 设置 PR 模板
- [ ] 配置 CI/CD
```

### 社区建设

```markdown
## 社区建设

- 及时响应 Issue 和 PR
- 欢迎新贡献者
- 定期发布更新
- 维护文档
- 建立沟通渠道
```

## 开源许可

| 许可证 | 说明 |
|--------|------|
| MIT | 最宽松，允许任何使用 |
| Apache 2.0 | 宽松，需要注明修改 |
| GPL | 需要开源衍生作品 |
| LGPL | 允许库被闭源使用 |
| BSD | 类似 MIT，有附加限制 |

## 常见问题

### Q: 如何开始参与开源？

A:
1. 从文档贡献开始
2. 修复简单的 Bug
3. 选择活跃的项目
4. 遵循贡献指南

### Q: PR 被拒绝怎么办？

A:
1. 不要气馁
2. 阅读反馈
3. 根据建议修改
4. 重新提交或放弃

### Q: 如何找到适合自己的项目？

A:
1. 选择自己使用过的工具
2. 查看 good-first-issue 标签
3. 阅读贡献指南
4. 了解项目活跃度

## 相关资源

- [开源指南](https://opensource.guide/)
- [GitHub 开源指南](https://docs.github.com/en/get-started/exploring-projects-on-github)
- [First Contributions](https://firstcontributions.github.io/)

---

**上一篇：[性能优化](W28-performance.md) | 下一篇：[GitHub 认证考试](W30-certification.md)**
