# 标签与发布

## 什么是标签？

标签用于标记特定的提交，通常用于版本发布。

## 标签操作

### 创建标签

```bash
# 轻量标签
git tag v1.0.0

# 附注标签（推荐）
git tag -a v1.0.0 -m "Release version 1.0.0"
```

### 查看标签

```bash
# 列出所有标签
git tag

# 查看标签详情
git show v1.0.0
```

### 推送标签

```bash
# 推送单个标签
git push origin v1.0.0

# 推送所有标签
git push origin --tags
```

### 删除标签

```bash
# 删除本地标签
git tag -d v1.0.0

# 删除远程标签
git push origin --delete v1.0.0
```

### 检出标签

```bash
git checkout v1.0.0
```

## 语义化版本 (SemVer)

```
MAJOR.MINOR.PATCH
```

| 版本号 | 说明 | 示例 |
|--------|------|------|
| MAJOR | 不兼容的 API 修改 | 2.0.0 |
| MINOR | 向下兼容的功能新增 | 1.1.0 |
| PATCH | 向下兼容的问题修正 | 1.0.1 |

### 示例
```
v1.0.0 → v1.0.1 (修复 bug)
v1.0.1 → v1.1.0 (新功能)
v1.1.0 → v2.0.0 (破坏性变更)
```

## GitHub Releases

### 创建 Release

#### 网页创建
1. 进入仓库 → **Releases**
2. 点击 **Create a new release**
3. 选择标签
4. 填写标题和描述
5. 上传附件（可选）
6. 发布

#### 命令行创建

```bash
# 使用 GitHub CLI
gh release create v1.0.0 --title "Release v1.0.0" --notes "首次发布"

# 上传文件
gh release create v1.0.0 --title "Release v1.0.0" --notes "发布说明" ./dist/*
```

### 自动生成 Release Notes

```bash
gh release create v1.0.0 --generate-notes
```

### Release 模板

创建 `.github/release.yml`：

```yaml
changelog:
  categories:
    - title: New Features
      labels:
        - enhancement
    - title: Bug Fixes
      labels:
        - bug
    - title: Other Changes
      labels:
        - '*'
```

## 自动发布

### 使用 GitHub Actions

```yaml
name: Release

on:
  push:
    tags:
      - 'v*'

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    
    - name: Create Release
      uses: softprops/action-gh-release@v2
      with:
        generate_release_notes: true
```

## 最佳实践

1. **使用语义化版本**：让版本号有意义
2. **编写清晰的发布说明**：让用户了解变更
3. **关联 Issue/PR**：追溯变更来源
4. **自动化发布流程**：减少人为错误

## 下一步

[GitHub CLI 命令行工具 →](26-github-cli.md)
