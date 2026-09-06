# GitHub + Azure DevOps 集成

## 为什么集成 Azure DevOps？

Azure DevOps 是微软的企业级 DevOps 平台，与 GitHub 深度集成，提供完整的 ALM（应用生命周期管理）。

## 集成方式

| 方式 | 说明 |
|------|------|
| Azure Pipelines | 直接使用 Azure DevOps 构建和部署 |
| GitHub Actions | 在 GitHub 中使用 Azure 任务 |
| Boards 集成 | Azure Boards 关联 GitHub Issue |
| Artifacts | 包管理与 GitHub Packages 联动 |
| Test Plans | 测试管理与 PR 关联 |

## Azure Pipelines + GitHub

### 基础配置

```yaml
# azure-pipelines.yml
trigger:
  branches:
    include:
      - main
      - develop

pool:
  vmImage: 'ubuntu-latest'

variables:
  buildConfiguration: 'Release'

steps:
- task: UseNode@1
  inputs:
    version: '20'

- script: |
    npm ci
    npm run build
    npm test
  displayName: 'Build and Test'

- task: PublishTestResults@2
  inputs:
    testResultsFormat: 'JUnit'
    testResultsFiles: 'test-results.xml'
```

### 部署到 Azure

```yaml
# 部署到 Azure App Service
- task: AzureWebApp@1
  inputs:
    azureSubscription: 'Azure-Connection'
    appType: 'webApp'
    appName: 'my-app'
    package: '$(Build.ArtifactStagingDirectory)/**/*.zip'
```

### 部署到 Azure Container Instances

```yaml
- task: AzureCLI@2
  inputs:
    azureSubscription: 'Azure-Connection'
    scriptType: 'bash'
    scriptLocation: 'inlineScript'
    inlineScript: |
      az container create \
        --resource-group myRG \
        --name myContainer \
        --image myregistry.azurecr.io/myapp:$(Build.BuildId) \
        --dns-name-label myapp \
        --ports 80
```

## GitHub Actions for Azure

### 登录 Azure

```yaml
# .github/workflows/azure-deploy.yml
name: Deploy to Azure

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    
    - name: Azure Login
      uses: azure/login@v1
      with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}
    
    - name: Deploy to Azure Web App
      uses: azure/webapps-deploy@v2
      with:
        app-name: 'my-app'
        publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE }}
        package: './dist'
```

### 使用 Azure CLI

```yaml
- name: Azure CLI Action
  uses: azure/cli@v1
  with:
    inlineScript: |
      az webapp deployment source config \
        --name my-app \
        --resource-group myRG \
        --repo-url ${{ github.repository }} \
        --branch main \
        --manual-integration
```

### 创建 Azure 资源

```yaml
- name: Create Azure Resource Group
  uses: azure/cli@v1
  with:
    inlineScript: |
      az group create --name myRG --location eastasia

- name: Create Azure Container Registry
  uses: azure/cli@v1
  with:
    inlineScript: |
      az acr create \
        --resource-group myRG \
        --name myregistry \
        --sku Basic
```

## Azure Boards + GitHub

### 关联 Issue

```bash
# 在 Azure Boards 中关联 GitHub Issue
# 1. 在 Azure Boards 设置中启用 GitHub 集成
# 2. 选择要关联的 GitHub 仓库
# 3. 在 Issue 中使用 #ID 关联工作项
```

### 自动关联

```yaml
# 在 GitHub Issue 中使用特殊语法
# AB#123 - 关联 Azure Boards 工作项
# Fixes AB#123 - 关联并自动关闭
```

## Azure Artifacts + GitHub

### 发布到 Azure Artifacts

```yaml
- task: Npm@1
  inputs:
    command: 'publish'
    publishRegistry: 'useFeed'
    publishFeed: 'my-feed'
    workingDir: '.'
    verbose: true
```

### 从 GitHub Packages 迁移

```bash
# 使用 Azure Artifacts
npm config set registry https://pkgs.dev.azure.com/myorg/_packaging/myfeed/npm/registry/

# 发布包
npm publish
```

## Azure DevOps GitHub App

### 安装

1. 访问 https://github.com/marketplace/azure-devops
2. 点击 Install
3. 选择仓库
4. 授权访问

### 功能

| 功能 | 说明 |
|------|------|
| 工作项关联 | PR 关联 Azure Boards |
| 状态检查 | 构建状态更新到 PR |
| 部署跟踪 | 部署状态显示在 GitHub |
| 安全扫描 | 代码安全检查 |

## 最佳实践

1. **统一身份**：使用 Azure AD 与 GitHub 同一账号
2. **环境分离**：Staging 和 Production 分开
3. **安全存储**：使用 Azure Key Vault 管理密钥
4. **监控告警**：使用 Azure Monitor 监控应用
5. **成本优化**：使用 Azure Cost Management

## 相关资源

- [Azure DevOps 文档](https://docs.microsoft.com/en-us/azure/devops/)
- [GitHub Actions for Azure](https://github.com/Azure/actions)
- [Azure GitHub 集成](https://docs.microsoft.com/en-us/azure/developer/github/)

---

**上一篇：[DevOps 实战](W15-devops.md) | 下一篇：[GitHub Copilot 企业培训](W24-copilot-training.md)**
