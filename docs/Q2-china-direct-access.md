# GitHub 国内直连完全指南

## 方案总览

| 方案 | 难度 | 稳定性 | 适用场景 |
|------|------|--------|----------|
| GitHub520 自动更新 Hosts | ⭐ | ⭐⭐⭐ | 日常使用 |
| Dev-sidecar 开发者边车 | ⭐ | ⭐⭐⭐⭐ | 日常使用 |
| Watt Toolkit | ⭐ | ⭐⭐⭐ | 临时加速 |
| Git 代理配置 | ⭐⭐ | ⭐⭐⭐⭐⭐ | 所有场景 |
| 国内镜像加速 | ⭐ | ⭐⭐⭐ | 克隆下载 |
| 云服务器中转 | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | 企业/团队 |

## 方案一：GitHub520（推荐）

### 原理
自动更新 GitHub 的 hosts 文件，绕过 DNS 污染。

### 安装使用

```bash
# macOS/Linux
pip install github520

# 或直接下载脚本
curl -fsSL https://raw.githubusercontent.com/521xueweihan/GitHub520/master/main.py -o github520.py
python3 github520.py
```

### 自动更新（crontab）

```bash
# 每小时更新一次
crontab -e
# 添加：
0 * * * * python3 /path/to/github520.py
```

### 配合 SwitchHosts 使用

1. 下载 [SwitchHosts](https://github.com/oldj/SwitchHosts)
2. 添加远程 hosts
3. URL: `https://raw.githubusercontent.com/521xueweihan/GitHub520/main/hosts`
4. 开启自动刷新

## 方案二：Dev-sidecar（开发者边车）

### 功能特点
- 自动加速 GitHub 访问
- 支持 GitHub 镜像
- 支持加速下载 Release
- 支持加速 raw 文件
- 支持加速 fork

### 安装

```bash
# 下载安装包
# https://github.com/docmirror/dev-sidecar/releases

# 或使用 npm 安装
npm install -g dev-sidecar
```

### 使用

```bash
# 启动服务
dev-sidecar

# 或后台运行
dev-sidecar --background
```

### 支持的功能

| 功能 | 说明 |
|------|------|
| GitHub 网页加速 | 访问 GitHub 更快 |
| Release 下载加速 | 下载 Release 文件 |
| raw 文件加速 | 加速 raw.githubusercontent.com |
| clone/push 加速 | Git 操作加速 |
| 头像加速 | GitHub 头像显示 |

## 方案三：Watt Toolkit（原 Steam++）

### 下载安装

访问 https://steampp.net/

### 使用步骤

1. 安装并打开 Watt Toolkit
2. 选择 **网络加速**
3. 勾选 **GitHub**
4. 点击 **一键加速**

## 方案四：Git 代理配置

### 临时使用代理

```bash
# HTTP 代理
git config --global http.proxy http://127.0.0.1:7890
git config --global https.proxy http://127.0.0.1:7890

# SOCKS5 代理
git config --global http.proxy socks5://127.0.0.1:7890
git config --global https.proxy socks5://127.0.0.1:7890

# 仅对 GitHub 生效
git config --global http.https://github.com.proxy http://127.0.0.1:7890
```

### 取消代理

```bash
git config --global --unset http.proxy
git config --global --unset https.proxy
```

### 使用环境变量

```bash
# 临时设置
export http_proxy=http://127.0.0.1:7890
export https_proxy=http://127.0.0.1:7890

# 取消
unset http_proxy
unset https_proxy
```

## 方案五：国内镜像站

### GitHub 镜像

| 镜像 | 地址 | 功能 |
|------|------|------|
| GitHub 镜像 | https://ghproxy.com | clone/download |
| gh-proxy.com | https://gh-proxy.com | 完整代理 |
| GitHub Proxy | https://github.moeyy.xyz | clone/download |
| kkgithub | https://kkgithub.com | 完整镜像 |

### 使用方法

```bash
# 克隆加速
git clone https://ghproxy.com/https://github.com/user/repo.git

# 下载 Release 加速
wget https://ghproxy.com/https://github.com/user/repo/releases/download/v1.0/file.zip

# raw 文件加速
curl https://ghproxy.com/https://github.com/user/repo/raw/main/file.txt
```

### GitHub 镜像站（完整功能）

| 镜像 | 地址 | 说明 |
|------|------|------|
| kkgithub | https://kkgithub.com | 完整功能镜像 |
| github.91chi.fun | https://github.91chi.fun | 代理访问 |

## 方案六：国内包管理器镜像

### npm 淘宝镜像

```bash
# 临时使用
npm install --registry https://registry.npmmirror.com

# 永久设置
npm config set registry https://registry.npmmirror.com

# 验证
npm config get registry
```

### Python pip 镜像

```bash
# 临时使用
pip install -i https://mirrors.aliyun.com/pypi/simple/ package

# 永久设置
pip config set global.index-url https://mirrors.aliyun.com/pypi/simple/
pip config set global.trusted-host mirrors.aliyun.com
```

### Docker 镜像

```json
// /etc/docker/daemon.json
{
  "registry-mirrors": [
    "https://docker.1ms.run",
    "https://docker.xuanyuan.me",
    "https://docker.m.daocloud.io"
  ]
}
```

```bash
# 重启 Docker
sudo systemctl daemon-reload
sudo systemctl restart docker
```

### Homebrew 镜像（macOS）

```bash
# 替换 Homebrew 源
export HOMEBREW_BREW_GIT_REMOTE="https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/brew.git"
export HOMEBREW_CORE_GIT_REMOTE="https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-core.git"
export HOMEBREW_BOTTLE_DOMAIN="https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles"

# 安装
brew install wget
```

### Go 模块镜像

```bash
# 设置代理
go env -w GOPROXY=https://goproxy.cn,direct

# 验证
go env GOPROXY
```

## 方案七：云服务器中转

### 搭建反向代理

```nginx
# nginx 配置
server {
    listen 443 ssl;
    server_name github-proxy.your-domain.com;
    
    location / {
        proxy_pass https://github.com;
        proxy_set_header Host github.com;
        proxy_ssl_server_name on;
    }
}
```

### 使用 cloudflare worker

```javascript
// Cloudflare Worker 脚本
addEventListener('fetch', event => {
  event.respondWith(handleRequest(event.request))
})

async function handleRequest(request) {
  const url = new URL(request.url)
  const githubUrl = `https://github.com${url.pathname}`
  
  return fetch(githubUrl, {
    headers: request.headers,
    method: request.method,
  })
}
```

## 方案八：SSH 优化

### 使用 SSH 代替 HTTPS

```bash
# 转换为 SSH
git config --global url."git@github.com:".insteadOf "https://github.com/"

# 使用 SSH 克隆
git clone git@github.com:user/repo.git
```

### SSH 连接优化

```bash
# ~/.ssh/config
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519
    ServerAliveInterval 60
    ServerAliveCountMax 30
    TCPKeepAlive yes
```

## 测速工具

```bash
# 测试 GitHub 连通性
ping github.com

# 测试下载速度
curl -o /dev/null -w "speed: %{speed_download} bytes/s\n" https://github.com/user/repo/archive/main.zip

# 使用 speedtest
speedtest-cli
```

## 最佳实践

1. **组合使用**：Hosts + 代理双保险
2. **自动更新**：使用 GitHub520 自动更新 hosts
3. **镜像备份**：配置国内包管理器镜像
4. **SSH 优先**：使用 SSH 代替 HTTPS
5. **浅克隆**：大仓库使用 `--depth 1`

## 推荐配置

```bash
# 1. 安装 GitHub520
pip install github520

# 2. 配置 Git SSH
git config --global url."git@github.com:".insteadOf "https://github.com/"

# 3. 配置 npm 镜像
npm config set registry https://registry.npmmirror.com

# 4. 配置 pip 镜像
pip config set global.index-url https://mirrors.aliyun.com/pypi/simple/

# 5. 配置 Go 镜像
go env -w GOPROXY=https://goproxy.cn,direct
```

## 相关资源

- [GitHub520](https://github.com/521xueweihan/GitHub520)
- [Dev-sidecar](https://github.com/docmirror/dev-sidecar)
- [Watt Toolkit](https://steampp.net/)
- [淘宝 NPM 镜像](https://npmmirror.com/)
- [阿里云 pip 镜像](https://mirrors.aliyun.com/pypi/simple/)
- [DaoCloud Docker 镜像](https://docker.m.daocloud.io)

---

**上一篇：[GitHub 国内加速指南](Q-china-acceleration.md) | 下一篇：[国内 Git 托管平台](R-china-platforms.md)**
