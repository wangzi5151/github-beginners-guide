# 练习 23：使用 GitHub Codespaces 开发

## 学习目标

通过本次练习，你将掌握以下技能：

- 理解 GitHub Codespaces 的工作原理
- 创建和配置 Codespace
- 使用 VS Code 网页版进行开发
- 创建和自定义 devcontainer 配置
- 管理 Codespace 的生命周期
- 在团队中共享开发环境配置

## 前置要求

- 拥有 GitHub 账号
- 了解 GitHub Codespaces 的使用额度（免费用户每月有一定小时数）
- 基本的 Git 操作知识
- 基本的 Docker 概念了解（可选，但有帮助）

## 第一部分：了解 GitHub Codespaces

### 什么是 GitHub Codespaces？

GitHub Codespaces 是一种云端开发环境，它提供以下能力：

1. **即时可用**：几秒钟内启动一个完整的开发环境
2. **配置一致**：团队成员使用相同的开发环境配置
3. **云端运行**：代码在云端运行，本地机器无需高配置
4. **预配置环境**：预装常用开发工具和运行时
5. **无缝集成**：与 GitHub 仓库深度集成

### 使用场景

Codespaces 特别适合以下场景：

- 快速开始新项目开发，无需本地配置环境
- 为开源项目贡献代码，避免本地环境冲突
- 在不同设备间无缝切换开发环境
- 确保团队成员使用一致的开发环境
- 进行代码审查时快速运行和测试代码

## 第二部分：创建你的第一个 Codespace

在这一部分中，你将亲手创建一个 GitHub Codespace 并体验云端开发的便利性。创建 Codespace 的过程非常简单直观，GitHub 提供了多种创建方式，包括从仓库页面创建、从命令行创建、以及从已有模板创建。无论你选择哪种方式，GitHub 都会在云端为你分配一个完整的开发环境，其中包含了你项目所需的所有工具和依赖。创建完成后，你可以通过浏览器直接访问 VS Code 网页版，或者通过本地的 VS Code 客户端远程连接到你的 Codespace。整个过程通常只需要几分钟时间，相比传统的本地开发环境搭建，这大大节省了时间和精力。

### 步骤 1：从 GitHub 仓库创建

1. 打开你的 GitHub 仓库页面
2. 点击绿色的 "Code" 按钮
3. 切换到 "Codespaces" 标签页
4. 点击 "Create codespace on main"

```
仓库页面布局：
┌─────────────────────────────────────┐
│ [Code] [Issues] [Pull requests] ... │
├─────────────────────────────────────┤
│                                     │
│  ┌─────────┐                        │
│  │ Code ▼  │  ← 点击这里            │
│  └─────────┘                        │
│  ┌─────────────────────────────┐    │
│  │ Local │ GitHub Codespaces │  │    │
│  │       │ ───────────────── │  │    │
│  │       │ Create on main    │  │    │
│  │       │ ───────────────── │  │    │
│  └─────────────────────────────┘    │
└─────────────────────────────────────┘
```

### 步骤 2：等待 Codespace 启动

创建 Codespace 后，GitHub 会执行以下步骤：

1. 分配云端虚拟机
2. 克隆你的仓库代码
3. 根据配置安装开发工具
4. 启动 VS Code 网页版

这个过程通常需要30秒到2分钟。

### 步骤 3：熟悉 VS Code 网页版界面

Codespace 启动后，你会看到熟悉的 VS Code 界面：

```
┌────────┬──────────────────────────────────────────┐
│        │  explorer  │  editor area                │
│ 文件   │  ┌───────┐ │  ┌──────────────────────┐   │
│ 浏览   │  │ 文件树 │ │  │  代码编辑区域        │   │
│ 器     │  │       │ │  │                      │   │
│        │  └───────┘ │  └──────────────────────┘   │
│        │            │                              │
│        │            │  ┌──────────────────────┐   │
│        │            │  │  终端/输出区域        │   │
│        │            │  └──────────────────────┘   │
├────────┴────────────┴──────────────────────────────┤
│ 状态栏：显示连接状态、分支信息等                     │
└─────────────────────────────────────────────────────┘
```

### 步骤 4：在终端中运行命令

在 Codespace 的终端中，你可以像在本地一样运行命令。Codespace 提供了完整的终端环境，支持常用的 Shell 命令和开发工具。你可以在终端中安装依赖、运行测试、启动服务、执行脚本等操作。由于 Codespace 运行在云端 Linux 环境中，你需要注意一些与本地环境的差异。例如，文件路径可能与你的本地操作系统不同，某些系统级的操作可能需要特定的权限。此外，Codespace 的终端还支持多标签页和分屏功能，方便你同时运行多个命令或查看不同的输出内容：

```bash
# 查看当前目录
pwd
# 输出: /workspaces/your-repo-name

# 查看文件列表
ls -la

# 查看 Git 状态
git status

# 安装项目依赖（以 Node.js 项目为例）
npm install

# 运行项目
npm start

# 运行测试
npm test
```

### 步骤 5：编辑和提交代码

在 Codespace 中修改代码并提交：

```bash
# 创建新分支
git checkout -b feature/new-feature

# 编辑文件（在编辑器中直接修改）
# ...

# 查看修改
git diff

# 提交修改
git add .
git commit -m "Add new feature"

# 推送到远程
git push origin feature/new-feature
```

## 第三部分：配置 devcontainer

开发容器（Development Container）配置是 GitHub Codespaces 的核心特性之一。通过使用开发容器配置，你可以将整个开发环境定义为代码，这意味着你可以像管理应用程序代码一样管理你的开发环境配置。这种方式带来了许多好处：首先，它可以确保团队中的每个开发者都使用完全相同的开发环境，避免了"在我机器上可以运行"的问题；其次，新加入团队的成员可以在几分钟内搭建好开发环境，无需花费大量时间进行手动配置；最后，开发环境配置可以版本化管理，任何环境变更都可以通过代码审查流程来把控质量。

### 什么是 devcontainer？

`devcontainer` 是一个 JSON 配置文件，定义了 Codespace 的开发环境。它位于 `.devcontainer/devcontainer.json`。这个配置文件可以指定基础镜像、需要安装的开发工具、VS Code 扩展、端口转发规则、环境变量、以及项目初始化命令等信息。当 Codespace 启动时，系统会读取这个配置文件并按照其中的定义来配置开发环境。devcontainer 规范是由微软和 GitHub 共同制定的开放标准，不仅可以在 GitHub Codespaces 中使用，还可以在 VS Code 的远程容器开发功能中使用。这意味着你可以在本地开发时也使用相同的容器化环境，确保开发环境的一致性。

### 步骤 1：创建基本配置

在项目根目录下创建 devcontainer 配置：

```bash
# 创建 .devcontainer 目录
mkdir -p .devcontainer

# 创建配置文件
touch .devcontainer/devcontainer.json
```

### 步骤 2：编写 Node.js 项目配置

对于 Node.js 项目，使用以下配置：

```json
{
  "name": "Node.js Development Environment",
  "image": "mcr.microsoft.com/devcontainers/javascript-node:20",
  "features": {
    "ghcr.io/devcontainers/features/git:1": {},
    "ghcr.io/devcontainers/features/github-cli:1": {},
    "ghcr.io/devcontainers/features/docker-in-docker:2": {}
  },
  "forwardPorts": [3000, 5000, 8080],
  "postCreateCommand": "npm install",
  "customizations": {
    "vscode": {
      "extensions": [
        "dbaeumer.vscode-eslint",
        "esbenp.prettier-vscode",
        "ms-vscode.vscode-typescript-next",
        "GitHub.copilot"
      ],
      "settings": {
        "editor.formatOnSave": true,
        "editor.defaultFormatter": "esbenp.prettier-vscode",
        "editor.codeActionsOnSave": {
          "source.fixAll.eslint": "explicit"
        }
      }
    }
  }
}
```

### 步骤 3：编写 Python 项目配置

对于 Python 项目，使用以下配置：

```json
{
  "name": "Python Development Environment",
  "image": "mcr.microsoft.com/devcontainers/python:3.12",
  "features": {
    "ghcr.io/devcontainers/features/git:1": {},
    "ghcr.io/devcontainers/features/github-cli:1": {},
    "ghcr.io/devcontainers/features/common-utils:2": {}
  },
  "forwardPorts": [5000, 8000, 8080],
  "postCreateCommand": "pip install -r requirements.txt",
  "customizations": {
    "vscode": {
      "extensions": [
        "ms-python.python",
        "ms-python.vscode-pylance",
        "ms-python.black-formatter",
        "charliermarsh.ruff",
        "GitHub.copilot"
      ],
      "settings": {
        "python.defaultInterpreterPath": "/usr/local/bin/python",
        "[python]": {
          "editor.defaultFormatter": "ms-python.black-formatter",
          "editor.formatOnSave": true
        }
      }
    }
  }
}
```

### 步骤 4：编写多语言项目配置

对于需要多种语言运行时的项目：

```json
{
  "name": "Full Stack Development Environment",
  "image": "mcr.microsoft.com/devcontainers/base:ubuntu",
  "features": {
    "ghcr.io/devcontainers/features/node:1": {
      "version": "20"
    },
    "ghcr.io/devcontainers/features/python:1": {
      "version": "3.12"
    },
    "ghcr.io/devcontainers/features/docker-in-docker:2": {},
    "ghcr.io/devcontainers/features/git:1": {},
    "ghcr.io/devcontainers/features/github-cli:1": {}
  },
  "forwardPorts": [3000, 5000, 8000],
  "postCreateCommand": "bash .devcontainer/setup.sh",
  "customizations": {
    "vscode": {
      "extensions": [
        "dbaeumer.vscode-eslint",
        "esbenp.prettier-vscode",
        "ms-python.python",
        "ms-azuretools.vscode-docker",
        "GitHub.copilot"
      ]
    }
  }
}
```

然后创建设置脚本：

```bash
#!/bin/bash
# .devcontainer/setup.sh

echo "Setting up development environment..."

# 安装前端依赖
if [ -f "frontend/package.json" ]; then
    echo "Installing frontend dependencies..."
    cd frontend && npm install && cd ..
fi

# 安装后端依赖
if [ -f "backend/requirements.txt" ]; then
    echo "Installing backend dependencies..."
    cd backend && pip install -r requirements.txt && cd ..
fi

# 安装 Git hooks
if [ -f ".husky/install.sh" ]; then
    echo "Installing Git hooks..."
    bash .husky/install.sh
fi

echo "Setup complete!"
```

## 第四部分：使用 Dockerfile 自定义容器

虽然 devcontainer 提供了丰富的预构建镜像和功能模块，但有时你的项目可能需要更特殊的环境配置。在这种情况下，你可以使用 Dockerfile 来完全自定义你的开发容器。通过 Dockerfile，你可以精确控制容器中安装的每一个软件包、每一个环境变量、以及每一个系统配置。这种方式给予了你最大的灵活性，可以满足任何复杂的开发环境需求。例如，如果你的项目需要特定版本的数据库驱动程序、特殊的编译工具链、或者自定义的安全配置，使用 Dockerfile 就是最佳选择。需要注意的是，编写 Dockerfile 时应遵循最佳实践，包括使用多阶段构建、最小化镜像层、合理利用缓存等，以确保容器构建的速度和效率。

### 步骤 1：创建 Dockerfile

当你需要更精细地控制开发环境的配置时，Dockerfile 提供了最大的灵活性。你可以在 Dockerfile 中定义从操作系统层面到应用层面的所有配置，包括系统软件包的安装、编程语言运行时的配置、开发工具的安装、以及各种环境变量的设置。以下是一个针对多语言开发环境的 Dockerfile 示例：

```dockerfile
# .devcontainer/Dockerfile
FROM mcr.microsoft.com/devcontainers/base:ubuntu

# 安装系统依赖
RUN apt-get update && apt-get install -y \
    build-essential \
    curl \
    wget \
    vim \
    htop \
    && rm -rf /var/lib/apt/lists/*

# 安装 Node.js 20
RUN curl -fsSL https://deb.nodesource.com/setup_20.x | bash - \
    && apt-get install -y nodejs

# 安装 Python 3.12
RUN apt-get update && apt-get install -y \
    python3.12 \
    python3.12-venv \
    python3-pip \
    && rm -rf /var/lib/apt/lists/*

# 安装 Docker CLI
RUN curl -fsSL https://get.docker.com | sh

# 安装常用开发工具
RUN npm install -g \
    typescript \
    ts-node \
    nodemon \
    prettier \
    eslint

# 设置 Git 配置
RUN git config --global core.autocrlf input \
    && git config --global pull.rebase true

# 创建非 root 用户（可选）
ARG USERNAME=vscode
RUN groupadd --gid 1000 ${USERNAME} \
    && useradd --uid 1000 --gid ${USERNAME} -m ${USERNAME}
```

### 步骤 2：更新 devcontainer.json 使用 Dockerfile

```json
{
  "name": "Custom Development Environment",
  "build": {
    "dockerfile": "Dockerfile",
    "context": ".."
  },
  "remoteUser": "vscode",
  "customizations": {
    "vscode": {
      "extensions": [
        "GitHub.copilot"
      ]
    }
  }
}
```

## 第五部分：管理 Codespace 生命周期

了解如何有效地管理 Codespace 的生命周期对于控制成本和提高工作效率非常重要。每个 Codespace 都有自己的运行状态，包括可用状态、已停止状态和已删除状态。当 Codespace 处于运行状态时，它会消耗你的使用额度；当 Codespace 停止时，不会产生费用，但其中的数据会被保留。你需要根据实际使用情况来合理管理 Codespace 的状态，在不使用时及时停止以节省费用，在不再需要时及时删除以释放资源。GitHub 提供了自动停止的功能，可以在 Codespace 空闲一段时间后自动停止，这是一个非常好的节省成本的实践。

### 查看你的 Codespaces

通过 GitHub CLI 管理你的 Codespaces：

```bash
# 列出所有 Codespaces
gh codespace list

# 输出示例：
# NAME                          DISPLAY NAME      REPOSITORY             BRANCH   STATE
# owner-repo-abc123             owner/repo        owner/repo             main     Available
# owner-another-def456          owner/another     owner/another          dev      Shutdown
```

### 连接到现有 Codespace

```bash
# 使用 SSH 连接到 Codespace
gh codespace ssh

# 指定 Codespace 名称
gh codespace ssh --codespace owner-repo-abc123

# 使用端口转发
gh codespace ssh --codespace owner-repo-abc123 -- -L 3000:localhost:3000
```

### 管理端口转发

```bash
# 查看 Codespace 的端口转发
gh codespace ports forward 3000:3000 --codespace owner-repo-abc123

# 查看当前端口状态
gh codespace ports list --codespace owner-repo-abc123
```

### 停止和删除 Codespace

```bash
# 停止 Codespace
gh codespace stop --codespace owner-repo-abc123

# 删除 Codespace
gh codespace delete --codespace owner-repo-abc123

# 删除所有 Codespace
gh codespace delete --all
```

### 通过网页管理

访问 https://github.com/codespaces 可以在网页上管理你的所有 Codespaces。

## 第六部分：端口转发和 Web 应用预览

端口转发是 Codespaces 提供的一个强大功能，它允许你在云端运行 Web 应用服务，并通过浏览器在本地访问。当你在 Codespace 中启动一个开发服务器时，Codespace 会自动检测到服务监听的端口，并将其转发到一个可以通过浏览器访问的 URL。这对于前端开发、后端开发、以及全栈开发都非常有用。你可以实时预览你的代码修改效果，就像在本地开发一样流畅。此外，你还可以将端口设置为公共可见，这样你就可以分享预览链接给团队成员或客户，让他们提前体验你的应用功能。这在进行设计评审、功能演示、或者远程协作时非常方便。

### 步骤 1：启动 Web 应用

在 Codespace 中启动你的 Web 应用：

```bash
# 启动开发服务器
npm run dev
# 假设服务器运行在 http://localhost:3000
```

### 步骤 2：访问应用

启动服务器后，Codespace 会自动检测到端口并进行转发。你会看到一个通知，点击 "Open in Browser" 即可在浏览器中访问应用。

你也可以手动配置端口转发：

1. 打开终端底部的 "PORTS" 标签页
2. 点击 "Forward a Port"
3. 输入端口号（如 3000）
4. 选择可见性（Private 或 Public）

### 步骤 3：分享预览链接

将端口设置为 Public 后，你可以分享链接给其他人预览你的应用：

1. 在 PORTS 标签页中找到对应的端口
2. 右键点击，选择 "Port Visibility" → "Public"
3. 复制链接地址分享给他人

## 第七部分：进阶挑战

完成基础的 Codespace 使用后，你可以尝试以下进阶挑战来深入掌握 Codespaces 的高级功能。这些挑战将帮助你创建更完善的开发环境配置，满足复杂项目的需求，并学习如何与团队成员共享开发环境配置。通过这些实践，你将能够充分利用 Codespaces 的能力，打造高效、一致的开发体验。

### 挑战 1：为现有项目创建完整的 devcontainer 配置

选择你自己的一个项目，创建完整的开发环境配置：

```json
{
  "name": "My Project Dev Environment",
  "image": "mcr.microsoft.com/devcontainers/base:ubuntu",
  "features": {
    "ghcr.io/devcontainers/features/node:1": { "version": "20" },
    "ghcr.io/devcontainers/features/python:1": { "version": "3.12" },
    "ghcr.io/devcontainers/features/docker-in-docker:2": {}
  },
  "forwardPorts": [3000, 5432, 6379],
  "postCreateCommand": "bash .devcontainer/setup.sh",
  "postAttachCommand": "echo 'Welcome to the dev environment!'",
  "customizations": {
    "vscode": {
      "extensions": [
        "dbaeumer.vscode-eslint",
        "ms-python.python",
        "ms-azuretools.vscode-docker",
        "GitHub.copilot",
        "eamodio.gitlens"
      ],
      "settings": {
        "files.autoSave": "afterDelay",
        "editor.formatOnSave": true
      }
    }
  }
}
```

### 挑战 2：配置数据库服务

使用 Docker Compose 配置开发所需的数据库服务：

```yaml
# .devcontainer/docker-compose.yml
version: '3.8'
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    volumes:
      - ..:/workspace:cached
    command: sleep infinity
    network_mode: service:db

  db:
    image: postgres:16
    restart: unless-stopped
    volumes:
      - postgres-data:/var/lib/postgresql/data
    environment:
      POSTGRES_USER: devuser
      POSTGRES_PASSWORD: devpass
      POSTGRES_DB: devdb

  redis:
    image: redis:7-alpine
    restart: unless-stopped

volumes:
  postgres-data:
```

对应的 `devcontainer.json`：

```json
{
  "name": "Project with Database",
  "dockerComposeFile": "docker-compose.yml",
  "service": "app",
  "workspaceFolder": "/workspace",
  "forwardPorts": [5432, 6379],
  "postCreateCommand": "npm install && npm run db:migrate",
  "customizations": {
    "vscode": {
      "extensions": [
        "ms-azuretools.vscode-docker",
        "mtxr.sqltools",
        "mtxr.sqltools-driver-pg"
      ]
    }
  }
}
```

### 挑战 3：配置团队共享的 Codespace 预构建

在仓库设置中配置预构建（Prebuild），加速 Codespace 的创建速度：

1. 进入仓库 Settings → Codespaces
2. 点击 "Add prebuild configuration"
3. 选择分支（通常是 main）
4. 配置区域和实例类型
5. 保存配置

预构建会在代码推送时自动更新，这样团队成员创建 Codespace 时可以直接使用预构建的环境，无需等待依赖安装。预构建功能对于大型项目特别有用，因为这些项目的依赖安装和环境配置可能需要花费较长时间。通过使用预构建，团队成员可以在几秒钟内启动一个完全配置好的开发环境，大大提高了开发效率。此外，预构建还可以减少网络带宽的消耗，因为依赖包只需要下载一次，而不是每个团队成员都需要单独下载。

## 验证清单

完成练习后，请确认以下事项：

- [ ] 成功创建了一个 Codespace
- [ ] 能够在 Codespace 中编辑和运行代码
- [ ] 创建了基本的 `devcontainer.json` 配置
- [ ] 了解如何管理 Codespace 的生命周期
- [ ] 配置了端口转发并能够预览 Web 应用
- [ ] 了解如何通过 CLI 管理 Codespaces

## 常见问题

### Q1：Codespaces 是免费的吗？

GitHub 为免费用户每月提供一定的免费额度（目前是120核心小时/月）。Pro 用户获得更多额度。超出额度后会按使用量计费。

### Q2：如何降低 Codespaces 的使用成本？

- 不使用时及时停止 Codespace
- 设置较短的自动停止超时时间
- 使用较小的机器类型
- 使用预构建减少启动时间

### Q3：Codespace 中的数据会保留吗？

Codespace 中的数据在 Codespace 存在期间会保留。如果 Codespace 被删除，未推送的代码会丢失。建议经常推送代码到远程仓库。

## 总结

通过本次练习，你学会了如何使用 GitHub Codespaces 进行云端开发。Codespaces 提供了一致、便捷的开发环境，特别适合团队协作和开源贡献。通过 devcontainer 配置，你可以将开发环境代码化，确保每个团队成员使用相同的环境。在实际工作中，建议将 devcontainer 配置纳入版本控制，与项目代码一起维护和更新。这样，当项目的技术栈或依赖发生变化时，开发环境也会自动更新，避免了环境不一致导致的各种问题。同时，合理利用 Codespaces 的预构建功能可以大幅缩短环境启动时间，提升团队的开发效率。随着云端开发技术的不断成熟，GitHub Codespaces 将成为越来越多开发团队的首选开发环境方案。掌握这项技能，将为你的职业发展带来新的机遇。
