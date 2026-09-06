# 练习 15：发布管理实战

## 学习目标

- 使用语义化版本
- 自动生成 Release Notes
- 自动发布到 npm

## 步骤

### 步骤 1：创建发布工作流

```yaml
# .github/workflows/release.yml
name: Release

on:
  push:
    tags: ['v*']

permissions:
  contents: write

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
      with:
        fetch-depth: 0
    
    - uses: actions/setup-node@v4
      with:
        node-version: '20'
        registry-url: 'https://registry.npmjs.org'
    
    - run: npm ci
    - run: npm test
    
    - name: Create Release
      uses: softprops/action-gh-release@v2
      with:
        generate_release_notes: true
    
    - run: npm publish
      env:
        NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
```

### 步骤 2：配置 Release Notes 模板

```yaml
# .github/release.yml
changelog:
  categories:
    - title: 🚀 Features
      labels:
        - enhancement
    - title: 🐛 Bug Fixes
      labels:
        - bug
    - title: 📝 Documentation
      labels:
        - documentation
    - title: 🔒 Security
      labels:
        - security
```

## 实战任务

1. 创建自动发布工作流
2. 配置 Release Notes 模板
3. 测试发布流程
4. 发布一个版本

## 验证清单

- [ ] 能够创建发布工作流
- [ ] 能够配置 Release Notes
- [ ] 能够发布到 npm
- [ ] 能够使用语义化版本

## 下一步

继续 [练习 16：Monorepo 管理实战](exercise-16-monorepo.md)
