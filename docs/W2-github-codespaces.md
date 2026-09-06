# GitHub Codespaces 介绍

## 什么是 GitHub Codespaces？

GitHub Codespaces 是一个云端开发环境，让你可以在浏览器中直接编写、运行和调试代码，无需在本地安装任何开发工具。

## 核心优势

| 优势 | 说明 |
|------|------|
| **即时可用** | 无需配置本地环境，秒级启动 |
| **一致性** | 团队成员使用相同的开发环境 |
| **便携性** | 在任何设备上开发，包括平板和手机 |
| **强大性能** | 可配置 2-32 核 CPU，最高 64GB 内存 |
| **免费额度** | 每月 120 核心小时免费使用 |

## 快速开始

### 启动 Codespace

1. 进入 GitHub 仓库页面
2. 点击 **Code** 按钮
3. 选择 **Codespaces** 标签
4. 点击 **Create codespace on main**

### 使用 VS Code 桌面版

1. 安装 [GitHub Codespaces 扩展](https://marketplace.visualstudio.com/items?itemName=GitHub.codespaces)
2. 在 VS Code 中打开命令面板（`Ctrl+Shift+P`）
3. 输入 `Codespaces: Open in VS Code`

## 自定义开发环境

### devcontainer.json 配置

在项目根目录创建 `.devcontainer/devcontainer.json`：

```json
{
  "name": "My Project",
  "image": "mcr.microsoft.com/devcontainers/javascript-node:20",
  "forwardPorts": [3000],
  "postCreateCommand": "npm install",
  "customizations": {
    "vscode": {
      "extensions": [
        "dbaeumer.vscode-eslint",
        "esbenp.prettier-vscode"
      ]
    }
  }
}
```

### 常用基础镜像

| 镜像 | 适用场景 |
|------|----------|
| `mcr.microsoft.com/devcontainers/javascript-node` | Node.js 项目 |
| `mcr.microsoft.com/devcontainers/python` | Python 项目 |
| `mcr.microsoft.com/devcontainers/java` | Java 项目 |
| `mcr.microsoft.com/devcontainers/go` | Go 项目 |
| `mcr.microsoft.com/devcontainers/rust` | Rust 项目 |
| `mcr.microsoft.com/devcontainers/cpp` | C++ 项目 |

## 高级功能

### 端口转发

```json
{
  "forwardPorts": [3000, 8080],
  "portsAttributes": {
    "3000": {
      "label": "Web App",
      "onAutoForward": "openBrowser"
    },
    "8080": {
      "label": "API",
      "onAutoForward": "silent"
    }
  }
}
```

### Git 配置

Codespace 会自动使用你的 GitHub 凭据，无需额外配置 SSH 密钥。

### Secrets 管理

在仓库设置中添加 Codespace Secrets：

1. 进入仓库 **Settings** → **Codespaces**
2. 点击 **New repository secret**
3. 添加环境变量

```bash
# 在 Codespace 中使用
echo $API_KEY
```

## 使用 GitHub CLI 管理

```bash
# 创建 Codespace
gh codespace create -r owner/repo

# 列出 Codespaces
gh codespace list

# 连接到 Codespace
gh codespace ssh

# 删除 Codespace
gh codespace delete -c codespace-name
```

## 定价

| 计划 | 免费额度 | 超出费用 |
|------|----------|----------|
| Free | 120 核心小时/月 | $0.18/小时 |
| Pro | 180 核心小时/月 | $0.18/小时 |
| Team | 180 核心小时/月 | $0.18/小时 |
| Enterprise | 无限制 | $0.18/小时 |

> 注意：2 核配置下，120 核心小时 = 60 小时使用时间

## 最佳实践

1. **使用 devcontainer.json**：确保团队环境一致
2. **配置端口转发**：方便本地访问服务
3. **利用预构建**：加速 Codespace 启动
4. **设置超时**：避免不必要的费用
5. **使用 Codespace Secrets**：安全存储敏感信息

## 常见问题

### Q: Codespace 支持哪些语言？
A: 支持所有主流编程语言，通过自定义 devcontainer.json 可以配置任何开发环境。

### Q: 如何停止 Codespace？
A: 关闭浏览器标签页后，Codespace 会在 30 分钟后自动停止。也可以在 GitHub 网页手动停止。

### Q: Codespace 中的数据会丢失吗？
A: Codespace 停止后数据会保留 30 天。如果删除 Codespace，数据将永久丢失。

## 相关资源

- [GitHub Codespaces 官方文档](https://docs.github.com/en/codespaces)
- [devcontainer.json 参考](https://containers.dev/implementors/json_reference/)
- [GitHub Codespaces 免费额度](https://github.com/features/codespaces)

---

**上一篇：[GitHub Packages 介绍](J-github-packages.md) | 下一篇：[GitHub Mobile 介绍](K-github-mobile.md)**
