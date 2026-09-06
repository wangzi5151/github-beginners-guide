# 创建和管理仓库

## 在 GitHub 上创建仓库

### 方法一：网页创建

1. 登录 GitHub
2. 点击右上角 **+** → **New repository**
3. 填写信息：
   - **Repository name**：仓库名称（必填）
   - **Description**：仓库描述（可选）
   - **Public/Private**：选择可见性
   - **Initialize this repository with**：
     - ✅ Add a README file
     - Add .gitignore：选择模板
     - Choose a license：选择许可证
4. 点击 **Create repository**

### 方法二：命令行创建

```bash
# 使用 GitHub CLI
gh repo create my-repo --public --description "我的项目"

# 创建私有仓库
gh repo create my-repo --private
```

## 仓库设置

### 基本设置
进入仓库 → **Settings** → **General**

- **Repository name**：修改仓库名称
- **Description**：修改描述
- **Website**：添加项目网站
- **Topics**：添加标签（便于发现）

### 分支设置
**Settings** → **Branches**

- 默认分支设置
- 分支保护规则

### 协作者管理
**Settings** → **Collaborators**

```bash
# 使用 CLI 添加协作者
gh repo add-collaborator user/repo username
```

### 删除仓库
**Settings** → **Danger Zone** → **Delete this repository**

## 仓库描述和 README

好的仓库应该包含：

```
my-repo/
├── README.md          # 项目说明
├── LICENSE            # 许可证
├── .gitignore         # 忽略文件
├── CONTRIBUTING.md    # 贡献指南
├── CHANGELOG.md       # 更新日志
└── src/              # 源代码
```

## 仓库克隆方式

| 协议 | URL 格式 | 特点 |
|------|---------|------|
| SSH | `git@github.com:user/repo.git` | 需要 SSH 密钥 |
| HTTPS | `https://github.com/user/repo.git` | 需要认证 |
| GitHub CLI | `gh repo clone user/repo` | 自动选择最佳方式 |

## 仓库模板

GitHub 提供仓库模板功能：
1. 创建仓库时选择 **Choose a repository template**
2. 基于模板创建新仓库

## 下一步

[README 与文档 →](15-readme-docs.md)
