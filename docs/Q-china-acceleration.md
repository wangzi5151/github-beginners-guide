# GitHub 国内加速指南

## 为什么需要加速？

由于网络原因，国内访问 GitHub 有时会较慢。本文介绍几种加速方法。

## 方法一：修改 Hosts 文件

### 原理
通过修改 hosts 文件，直接指定 GitHub 的 IP 地址，避免 DNS 污染。

### 步骤

1. **获取 GitHub IP 地址**

访问以下网站获取最新 IP：
- https://github.com/ipaddresses
- https://www.ipaddress.com/

需要获取的域名：
- `github.com`
- `github.global.ssl.fastly.net`
- `assets-cdn.github.com`

2. **修改 hosts 文件**

**Windows**: `C:\Windows\System32\drivers\etc\hosts`

**macOS/Linux**: `/etc/hosts`

添加以下内容：
```
# GitHub
140.82.114.4 github.com
199.232.69.194 github.global.ssl.fastly.net
185.199.108.153 assets-cdn.github.com
140.82.114.20 github.io
140.82.113.22 api.github.com
140.82.114.6 nodeload.github.com
199.232.69.194 github.global.ssl.fastly.net
```

3. **刷新 DNS 缓存**

**Windows**:
```cmd
ipconfig /flushdns
```

**macOS**:
```bash
sudo dscacheutil -flushcache
sudo killall -HUP mDNSResponder
```

**Linux**:
```bash
sudo systemd-resolve --flush-caches
```

## 方法二：使用 GitHub 加速代理

### 公共代理服务

| 服务 | 地址 | 说明 |
|------|------|------|
| GHProxy | https://ghproxy.com | 免费，支持多种操作 |
| GitHub Mirror | https://mirror.ghproxy.com | 稳定可靠 |
| Dev-sidecar | https://github.com/docmirror/dev-sidecar | 开源客户端 |

### 使用 GHProxy 加速下载

```bash
# 加速克隆
git clone https://ghproxy.com/https://github.com/user/repo.git

# 加速下载文件
wget https://ghproxy.com/https://github.com/user/repo/raw/main/file.zip
```

### 使用 Dev-sidecar

1. 下载安装 Dev-sidecar
2. 启动服务
3. 自动加速 GitHub 访问

## 方法三：配置 Git 代理

如果你有代理服务器：

```bash
# 配置 HTTP 代理
git config --global http.proxy http://127.0.0.1:7890
git config --global https.proxy http://127.0.0.1:7890

# 配置 SOCKS5 代理
git config --global http.proxy socks5://127.0.0.1:7890
git config --global https.proxy socks5://127.0.0.1:7890

# 取消代理
git config --global --unset http.proxy
git config --global --unset https.proxy
```

## 方法四：使用国内镜像

### GitHub 镜像站

| 镜像 | 地址 | 说明 |
|------|------|------|
| Gitee | https://gitee.com | 国内最大代码托管平台 |
| GitCode | https://gitcode.com | CSDN 旗下 |
| CODING | https://coding.net | 腾讯云 DevOps |

### 从镜像克隆

```bash
# 从 Gitee 镜像克隆
git clone https://gitee.com/mirrors/user-repo.git
```

## 方法五：使用 SSH 连接

SSH 连接通常比 HTTPS 更稳定：

```bash
# 使用 SSH 克隆
git clone git@github.com:user/repo.git
```

## 方法六：配置 Git 使用 SSH

```bash
# 将 HTTPS 转换为 SSH
git config --global url."git@github.com:".insteadOf "https://github.com/"
```

## 常见问题

### Q: 克隆速度很慢？
**A:** 尝试以下方法：
1. 使用 SSH 代替 HTTPS
2. 使用浅克隆：`git clone --depth 1`
3. 使用加速代理

### Q: push 失败？
**A:** 检查：
1. SSH 密钥是否正确配置
2. 网络连接是否正常
3. 尝试使用代理

### Q: GitHub Pages 无法访问？
**A:** 可能是被墙，尝试：
1. 使用代理访问
2. 配置自定义域名
3. 使用 CDN 加速

## 推荐工具

| 工具 | 说明 |
|------|------|
| Dev-sidecar | 开源 GitHub 加速工具 |
| Watt Toolkit | 原 Steam++，支持 GitHub 加速 |
| Proxifier | 代理客户端 |

## 相关资源

- [GitHub 官方文档](https://docs.github.com)
- [GitHub 状态页面](https://www.githubstatus.com)
