# Fork 与开源贡献

## 什么是 Fork？

Fork 是将别人的仓库复制到你的账户下，可以自由修改而不影响原项目。

```
原仓库 (upstream) → Fork (origin) → 你的修改
```

## Fork 工作流程

### 1. Fork 仓库
在 GitHub 上点击 **Fork** 按钮

### 2. 克隆你的 Fork

```bash
git clone git@github.com:your-username/repo.git
cd repo
```

### 3. 添加上游仓库

```bash
git remote add upstream git@github.com:original-owner/repo.git
```

### 4. 保持同步

```bash
# 获取上游更新
git fetch upstream

# 合并到 main 分支
git checkout main
git merge upstream/main

# 推送到你的 Fork
git push origin main
```

### 5. 创建功能分支

```bash
git checkout -b feature-new-thing
```

### 6. 修改并提交

```bash
git add .
git commit -m "feat: 新功能描述"
git push origin feature-new-thing
```

### 7. 创建 Pull Request
在 GitHub 上从你的 Fork 向原仓库创建 PR

## 寻找开源项目

### 标签搜索
- `good first issue`：适合新手
- `help wanted`：需要帮助
- `bug`：Bug 修复

### 浏览方式
- GitHub Explore：https://github.com/explore
- GitHub Trending：https://github.com/trending
- 专案网站：如 [firsttimersonly.com](https://www.firsttimersonly.com/)

## 贡献代码步骤

### 1. 阅读贡献指南
查找 `CONTRIBUTING.md` 文件

### 2. 设置开发环境

```bash
git clone git@github.com:your-fork/repo.git
cd repo
npm install  # 或其他安装命令
```

### 3. 了解代码规范
- 代码风格
- 提交信息格式
- 测试要求

### 4. 修改代码
- 小步提交
- 遵循规范
- 添加测试

### 5. 推送并创建 PR

```bash
git push origin feature-branch
```

在 GitHub 上创建 PR，关联相关 Issue

## 开源礼仪

1. **尊重维护者**：理解他们的时间有限
2. **先讨论**：大改动先开 Issue 讨论
3. **保持小 PR**：一个 PR 只做一件事
4. **耐心等待**：审查可能需要时间
5. **接受反馈**：可能需要修改多次

## 常见贡献类型

- 修复 Bug
- 添加功能
- 改进文档
- 优化性能
- 添加测试
- 翻译内容

## 下一步

[团队协作最佳实践 →](23-team-collaboration.md)
