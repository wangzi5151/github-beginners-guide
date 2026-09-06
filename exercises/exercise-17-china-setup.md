# 练习 17：国内环境配置实战

## 学习目标

- 配置国内加速方案
- 设置国内包管理器镜像
- 优化 Git 连接

## 步骤

### 步骤 1：配置 GitHub520

```bash
# 安装 GitHub520
pip install github520

# 验证 hosts 文件
cat /etc/hosts | grep github
```

### 步骤 2：配置 npm 镜像

```bash
# 永久设置淘宝镜像
npm config set registry https://registry.npmmirror.com

# 验证
npm config get registry
# 输出: https://registry.npmmirror.com

# 测试速度
npm install -g typescript
```

### 步骤 3：配置 pip 镜像

```bash
# 永久设置阿里云镜像
pip config set global.index-url https://mirrors.aliyun.com/pypi/simple/
pip config set global.trusted-host mirrors.aliyun.com

# 验证
pip config list

# 测试速度
pip install requests
```

### 步骤 4：配置 Git SSH

```bash
# 转换为 SSH
git config --global url."git@github.com:".insteadOf "https://github.com/"

# 验证
git config --global --get-regexp url
```

### 步骤 5：配置 Go 镜像

```bash
# 设置代理
go env -w GOPROXY=https://goproxy.cn,direct

# 验证
go env GOPROXY
```

### 步骤 6：配置 Docker 镜像

```bash
# 创建配置目录
sudo mkdir -p /etc/docker

# 写入配置
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": [
    "https://docker.1ms.run",
    "https://docker.xuanyuan.me"
  ]
}
EOF

# 重启 Docker
sudo systemctl daemon-reload
sudo systemctl restart docker

# 验证
docker info | grep -A 5 "Registry Mirrors"
```

## 实战任务

1. 安装并配置 GitHub520
2. 配置 npm/pip/Docker 国内镜像
3. 测试各工具的下载速度
4. 对比配置前后的速度差异

## 验证清单

- [ ] GitHub520 已安装并运行
- [ ] npm 镜像已配置
- [ ] pip 镜像已配置
- [ ] Git SSH 已配置
- [ ] Go 镜像已配置
- [ ] Docker 镜像已配置
- [ ] 能够正常访问 GitHub

## 下一步

恭喜完成所有练习！你现在可以在国内环境下高效使用 GitHub。
