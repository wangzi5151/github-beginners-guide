# 国内 Git 托管平台

## 平台对比

| 平台 | 地址 | 特点 | 免费私有仓库 |
|------|------|------|-------------|
| Gitee | gitee.com | 国内最大，中文友好 | ✅ |
| GitCode | gitcode.com | CSDN 旗下，社区活跃 | ✅ |
| CODING | coding.net | 腾讯云 DevOps | ✅ |
| Coding.net | coding.net | 企业级 DevOps | ✅ |
| 华为云 CodeHub | codehub.huaweicloud.com | 华为云生态 | ✅ |
| 阿里云 Codeup | codeup.aliyun.com | 阿里云生态 | ✅ |

---

## Gitee（码云）

### 简介
Gitee 是国内最大的代码托管平台，提供 Git 仓库托管、代码协作等功能。

### 优势
- 访问速度快
- 中文界面
- 支持 Markdown
- 提供 CI/CD
- 支持 Pages 服务

### 注册和使用

1. 访问 https://gitee.com
2. 注册账号
3. 创建仓库
4. 推送代码

### 基本操作

```bash
# 克隆仓库
git clone https://gitee.com/user/repo.git

# 推送代码
git remote add origin https://gitee.com/user/repo.git
git push -u origin main
```

### Gitee Pages

```bash
# 部署静态网站
# 1. 在仓库设置中启用 Pages
# 2. 选择部署分支和目录
# 3. 点击更新
```

---

## GitCode

### 简介
GitCode 是 CSDN 旗下的代码托管平台，专注于开发者社区。

### 优势
- 与 CSDN 社区集成
- 支持 Copilot
- 提供代码托管
- 支持 Pages

### 基本操作

```bash
# 克隆仓库
git clone https://gitcode.com/user/repo.git
```

---

## CODING

### 简介
CODING 是腾讯云旗下的 DevOps 平台，提供代码托管、CI/CD、项目管理等功能。

### 优势
- 企业级功能
- 完整的 DevOps 工具链
- 与腾讯云集成
- 支持 Kubernetes

### 基本操作

```bash
# 克隆仓库
git clone https://coding.net/user/project/repo.git
```

---

## 华为云 CodeHub

### 简介
华为云 CodeHub 是华为云提供的代码托管服务。

### 优势
- 与华为云集成
- 支持 DevOps
- 企业级安全

### 基本操作

```bash
# 克隆仓库
git clone https://codehub.huaweicloud.com/user/repo.git
```

---

## 阿里云 Codeup

### 简介
阿里云 Codeup 是阿里云提供的代码托管服务。

### 优势
- 与阿里云集成
- 支持企业级协作
- 提供代码扫描

### 基本操作

```bash
# 克隆仓库
git clone https://codeup.aliyun.com/user/repo.git
```

---

## 选择建议

### 个人开发者
- **推荐**: Gitee
- **理由**: 中文友好，访问快，功能完善

### 开源项目
- **推荐**: GitHub + Gitee 双托管
- **理由**: GitHub 面向国际，Gitee 面向国内

### 企业团队
- **推荐**: CODING 或阿里云 Codeup
- **理由**: 企业级功能，安全可靠

### 学习练习
- **推荐**: Gitee
- **理由**: 简单易用，中文文档

---

## 同步到国内平台

### 方法一：手动同步

```bash
# 添加国内平台为远程仓库
git remote add gitee https://gitee.com/user/repo.git

# 推送到国内平台
git push gitee main
```

### 方法二：使用 Gitee 同步功能

1. 在 Gitee 导入 GitHub 仓库
2. 设置自动同步
3. 定期更新

### 方法三：使用 GitHub Actions

```yaml
name: Sync to Gitee

on:
  push:
    branches: [main]

jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
      with:
        fetch-depth: 0
        
    - name: Push to Gitee
      uses: wearerequired/git-mirror-action@master
      env:
        SSH_PRIVATE_KEY: ${{ secrets.GITEE_RSA_PRIVATE_KEY }}
      with:
        source-repo: git@github.com:user/repo.git
        destination-repo: git@gitee.com:user/repo.git
```

---

## 相关资源

- [Gitee 官网](https://gitee.com)
- [GitCode 官网](https://gitcode.com)
- [CODING 官网](https://coding.net)
