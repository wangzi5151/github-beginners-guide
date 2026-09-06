# Git 可视化工具介绍

## 为什么使用可视化工具？

- 更直观地查看代码历史
- 更方便地操作分支
- 更容易解决冲突
- 提高工作效率

## 桌面客户端

### 1. GitHub Desktop

**特点**：
- GitHub 官方客户端
- 简单易用
- 跨平台（Windows/macOS）
- 与 GitHub 深度集成

**安装**：
- Windows: https://desktop.github.com
- macOS: `brew install --cask github`

**基本操作**：
```
1. 克隆仓库：File → Clone Repository
2. 提交更改：填写描述 → Commit to main
3. 推送：Push origin
4. 拉取：Pull origin
```

### 2. SourceTree

**特点**：
- 免费
- 功能强大
- 支持 Git 和 Mercurial
- 图形化操作

**下载**：https://www.sourcetreeapp.com

**主要功能**：
- 可视化分支图
- 拖拽操作
- 冲突解决工具
- 支持 Git Flow

### 3. GitKraken

**特点**：
- 界面美观
- 功能丰富
- 跨平台
- 支持 GitHub/GitLab/Bitbucket

**下载**：https://www.gitkraken.com

**主要功能**：
- 美观的提交图
- 内置代码编辑器
- 合并冲突工具
- 子模块支持

### 4. TortoiseGit

**特点**：
- Windows 专用
- 集成到资源管理器
- 右键菜单操作
- 中文支持好

**下载**：https://tortoisegit.org

**使用方式**：
```
右键点击文件夹 → Git Clone / Git Commit / etc.
```

### 5. SmartGit

**特点**：
- 跨平台
- 功能强大
- 支持 GitHub/GitLab
- 商业软件（免费版可用）

**下载**：https://www.syntevo.com/smartgit

## VS Code 集成

### 内置 Git 功能

VS Code 内置了 Git 支持：

1. **源代码管理**：左侧边栏的 Source Control 图标
2. **提交**：输入消息 → 点击提交
3. **推送**：点击同步按钮
4. **拉取**：点击拉取按钮
5. **分支**：底部状态栏切换分支

### 推荐扩展

| 扩展 | 说明 |
|------|------|
| GitLens | 增强 Git 功能 |
| Git Graph | 提交图可视化 |
| Git History | 查看文件历史 |
| GitHub Pull Requests | 管理 PR |

### GitLens 功能

- 悬停查看提交信息
- 行内 blame
- 提交历史
- 分支比较

## 命令行工具

### tig

终端 Git 可视化工具：

```bash
# 安装
brew install tig  # macOS
sudo apt install tig  # Ubuntu

# 使用
tig  # 打开 tig
tig log  # 查看日志
tig blame  # 查看 blame
```

### lazygit

交互式 Git 工具：

```bash
# 安装
brew install lazygit  # macOS
go install github.com/jesseduffield/lazygit@latest

# 使用
lazygit
```

### gitui

快速 Git UI：

```bash
# 安装
brew install gitui  # macOS
cargo install gitui

# 使用
gitui
```

## IDE 集成

### IntelliJ IDEA / WebStorm

- 内置 Git 支持
- 图形化操作
- 冲突解决工具
- 分支管理

### Eclipse

- EGit 插件
- 图形化操作
- 历史查看

### Sublime Text

- Git 插件
- 命令面板操作

## 选择建议

| 场景 | 推荐工具 |
|------|----------|
| 新手入门 | GitHub Desktop |
| 专业开发 | GitKraken 或 SourceTree |
| Windows 用户 | TortoiseGit 或 SourceTree |
| VS Code 用户 | GitLens 扩展 |
| 终端用户 | tig 或 lazygit |
| 企业团队 | GitKraken Pro |

## 最佳实践

1. **选择适合自己的工具**：不要盲目追求功能多
2. **掌握基本命令**：可视化工具是辅助，不是替代
3. **保持同步**：定期拉取最新代码
4. **善用分支**：使用可视化工具管理分支
5. **解决冲突**：使用工具的冲突解决功能

## 相关资源

- [GitHub Desktop](https://desktop.github.com)
- [SourceTree](https://www.sourcetreeapp.com)
- [GitKraken](https://www.gitkraken.com)
- [TortoiseGit](https://tortoisegit.org)
- [tig](https://github.com/jonas/tig)
- [lazygit](https://github.com/jesseduffield/lazygit)
