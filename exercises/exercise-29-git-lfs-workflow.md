# 练习 29：Git LFS 大文件管理实战

## 学习目标

完成本练习后，你将能够：

- 理解 Git LFS（Large File Storage）的工作原理
- 配置 Git LFS 跟踪大文件
- 将现有仓库迁移到 Git LFS
- 在 CI/CD 流程中正确使用 Git LFS
- 管理 LFS 存储配额和优化存储使用

## 前置条件

- 已安装 Git 2.15+
- 已安装 Git LFS 客户端
- 拥有 GitHub 仓库
- 了解基本的 Git 操作

## 背景知识

### 什么是 Git LFS

Git LFS（Large File Storage）是 Git 的扩展，用于管理大文件。它通过以下方式工作：

1. **指针文件**：在 Git 仓库中存储小的指针文件
2. **外部存储**：实际的大文件存储在 LFS 服务器上
3. **按需下载**：只在需要时下载大文件

### 为什么使用 Git LFS

- **仓库大小**：避免仓库变得过大
- **克隆速度**：加快克隆和拉取速度
- **版本控制**：对大文件进行版本控制
- **协作效率**：减少网络传输量

### Git LFS 限制

- GitHub 免费账户：每月 1GB 存储 + 1GB 带宽
- GitHub Pro：每月 2GB 存储 + 2GB 带宽
- GitHub Team：每月 4GB 存储 + 4GB 带宽
- 文件大小限制：单个文件最大 2GB

---

## 练习步骤

### 第一部分：安装和配置 Git LFS

#### 步骤 1：安装 Git LFS

**macOS：**

```bash
# 使用 Homebrew
brew install git-lfs

# 或使用 MacPorts
port install git-lfs
```

**Linux（Ubuntu/Debian）：**

```bash
# 使用包管理器
sudo apt-get install git-lfs

# 或从 GitHub 下载
curl -s https://packagecloud.io/install/repositories/github/git-lfs/script.deb.sh | sudo bash
sudo apt-get install git-lfs
```

**Windows：**

```bash
# 使用 Chocolatey
choco install git-lfs

# 或使用 Scoop
scoop install git-lfs

# 或从官网下载安装程序
# https://git-lfs.github.com/
```

**验证安装：**

```bash
git lfs version
# 输出示例：git-lfs/3.4.0 (GitHub; linux amd64; go 1.21.5)
```

#### 步骤 2：初始化 Git LFS

```bash
# 在仓库中初始化 LFS
git lfs install

# 验证初始化
git lfs env
```

预期输出：

```
git-lfs/3.4.0 (GitHub; linux amd64; go 1.21.5)
git version 2.43.0

LocalWorkingDir=/home/user/my-repo
LocalGitDir=/home/user/my-repo/.git
LocalGitStorageDir=/home/user/my-repo/.git
LocalMediaDir=/home/user/my-repo/.git/lfs/objects
LocalReferenceDirs=
TempDir=/home/user/my-repo/.git/lfs/tmp
ConcurrentTransfers=8
TusTransfers=false
BasicTransfers=true
SkipFetchError=false
FetchRecentAlways=false
FetchRecentRefsDays=7
FetchRecentCommitsDays=0
FetchRecentIncludeRemotes=1
PruneOffsetDays=3
PruneVerifyRemotesAlways=false
PruneRemoteName=origin
LfsStorageDir=/home/user/my-repo/.git/lfs
AccessDownload=none
AccessUpload=none
DownloadTransfers=basic
UploadTransfers=basic
GIT_EXEC_PATH=/usr/lib/git-core
GIT_LFS_PATH=/usr/bin/git-lfs
```

#### 步骤 3：配置 LFS 跟踪规则

```bash
# 跟踪特定文件类型
git lfs track "*.psd"
git lfs track "*.zip"
git lfs track "*.tar.gz"
git lfs track "*.mp4"
git lfs track "*.mov"
git lfs track "*.avi"
git lfs track "*.png"
git lfs track "*.jpg"
git lfs track "*.jpeg"
git lfs track "*.gif"
git lfs track "*.bmp"
git lfs track "*.tiff"
git lfs track "*.webp"
git lfs track "*.mp3"
git lfs track "*.wav"
git lfs track "*.flac"
git lfs track "*.ogg"
git lfs track "*.unitypackage"
git lfs track "*.fbx"
git lfs track "*.obj"
git lfs track "*.blend"
git lfs track "*.max"
git lfs track "*.ma"
git lfs track "*.mb"

# 跟踪特定目录下的文件
git lfs track "assets/**"
git lfs track "media/**"
git lfs track "resources/**"

# 跟踪特定大小的文件（需要配置 pre-commit 钩子）
git lfs track "*.bin"
```

#### 步骤 4：查看和管理跟踪规则

```bash
# 查看当前跟踪规则
git lfs track

# 输出示例：
# Listing tracked patterns
#     *.psd (.gitattributes)
#     *.zip (.gitattributes)
#     *.mp4 (.gitattributes)
#     *.png (.gitattributes)
#     *.jpg (.gitattributes)

# 查看 LFS 跟踪的文件
git lfs ls-files

# 输出示例：
# 2a83d6f5e4 * assets/images/logo.png
# 8b7c6d5e4a * assets/videos/intro.mp4
# 5c4b3a2d1e * assets/models/character.fbx

# 查看 LFS 状态
git lfs status

# 查看 LFS 文件大小
git lfs ls-files --size
```

### 第二部分：使用 Git LFS 管理大文件

#### 步骤 5：创建测试仓库

```bash
# 创建新仓库
mkdir lfs-demo && cd lfs-demo
git init

# 初始化 LFS
git lfs install

# 配置跟踪规则
git lfs track "*.bin"
git lfs track "*.dat"
git lfs track "*.large"

# 创建 .gitattributes 文件
cat > .gitattributes << 'EOF'
# Git LFS 跟踪规则
*.bin filter=lfs diff=lfs merge=lfs -text
*.dat filter=lfs diff=lfs merge=lfs -text
*.large filter=lfs diff=lfs merge=lfs -text
*.psd filter=lfs diff=lfs merge=lfs -text
*.zip filter=lfs diff=lfs merge=lfs -text
*.mp4 filter=lfs diff=lfs merge=lfs -text
*.png filter=lfs diff=lfs merge=lfs -text
*.jpg filter=lfs diff=lfs merge=lfs -text
EOF

# 提交 .gitattributes
git add .gitattributes
git commit -m "初始化 Git LFS 配置"
```

#### 步骤 6：添加大文件

```bash
# 创建测试大文件
dd if=/dev/urandom of=test-file-1.bin bs=1M count=10
dd if=/dev/urandom of=test-file-2.bin bs=1M count=20
dd if=/dev/urandom of=test-data.dat bs=1M count=15

# 创建普通文本文件
echo "这是一个普通文本文件" > readme.txt

# 查看文件状态
git status

# 添加文件
git add .

# 查看 LFS 状态
git lfs status

# 预期输出：
# On branch main
#
# Git LFS objects to be pushed to origin/main:
#
#     test-file-1.bin (10 MB)
#     test-file-2.bin (20 MB)
#     test-data.dat (15 MB)

# 提交
git commit -m "添加大文件测试"
```

#### 步骤 7：查看 LFS 指针文件

```bash
# 查看 LFS 指针文件内容
cat test-file-1.bin

# 预期输出：
# version https://git-lfs.github.com/spec/v1
# oid sha256:1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef
# size 10485760

# 查看 Git 对象类型
git cat-file -t HEAD:test-file-1.bin
# 输出：blob

# 查看 Git 对象大小
git cat-file -s HEAD:test-file-1.bin
# 输出：133（指针文件的大小）

# 对比普通文件
git cat-file -s HEAD:readme.txt
# 输出：实际文件大小
```

#### 步骤 8：推送到远程仓库

```bash
# 添加远程仓库
git remote add origin https://github.com/yourusername/lfs-demo.git

# 推送（LFS 文件会自动上传）
git push -u origin main

# 预期输出：
# Uploading LFS objects: 100% (3/3), 45 MB | 0 B/s
# Enumerating objects: 6
# Counting objects: 100% (6/6)
# Delta compression using up to 8 threads
# Compressing objects: 100% (4/4)
# Writing objects: 100% (5/5)
# Total 5 (delta 0), reused 0 (delta 0), pack-reused 0
# To https://github.com/yourusername/lfs-demo.git
#  * [new branch]      main -> main
```

### 第三部分：迁移现有仓库到 Git LFS

#### 步骤 9：安装 git-lfs-migrate 工具

```bash
# 使用 Homebrew 安装（macOS）
brew install git-lfs-migrate

# 或从 GitHub 下载
# https://github.com/git-lfs/git-lfs/releases

# 验证安装
git lfs migrate --version
```

#### 步骤 10：迁移现有仓库

```bash
# 进入现有仓库
cd existing-repo

# 初始化 LFS
git lfs install

# 查看哪些文件适合迁移到 LFS
git lfs migrate info --include="*.psd,*.zip,*.mp4,*.png,*.jpg"

# 预期输出：
# migrate: Fetching remote refs: ..., done
# migrate: Sorting commits: ..., done
#
# *.psd   2 MB   2/2 files(s)   100%
# *.zip   50 MB  5/5 files(s)   100%
# *.mp4   200 MB 3/3 files(s)   100%
# *.png   10 MB  50/50 files(s) 100%
# *.jpg   5 MB   30/30 files(s) 100%

# 执行迁移（不重写历史，只迁移未来的提交）
git lfs migrate import --include="*.psd,*.zip,*.mp4,*.png,*.jpg"

# 如果需要重写历史（谨慎使用！）
# git lfs migrate import --include="*.psd,*.zip,*.mp4,*.png,*.jpg" --everything

# 提交迁移结果
git add .gitattributes
git commit -m "迁移到 Git LFS"

# 强制推送（如果重写了历史）
# git push --force origin main
```

#### 步骤 11：迁移后的验证

```bash
# 验证 LFS 文件
git lfs ls-files

# 检查仓库大小
git count-objects -vH

# 预期输出（应该比迁移前小很多）：
# count: 50
# size: 2.50 MiB
# in-pack: 100
# packs: 1
# size-pack: 3.00 MiB
# prunepackable: 0
# garbage: 0
# size-garbage: 0 bytes

# 验证文件完整性
git lfs fsck
```

### 第四部分：在 CI/CD 中使用 Git LFS

#### 步骤 12：GitHub Actions 配置

创建文件 `.github/workflows/lfs-build.yml`：

```yaml
name: Build with LFS

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
      - name: 检出代码（包含 LFS）
        uses: actions/checkout@v4
        with:
          lfs: true
      
      - name: 验证 LFS 文件
        run: |
          echo "检查 LFS 文件..."
          git lfs ls-files
          
          echo "验证 LFS 文件完整性..."
          git lfs fsck
      
      - name: 构建项目
        run: |
          echo "使用 LFS 文件进行构建..."
          # 这里可以添加实际的构建命令
          ls -la assets/
      
      - name: 运行测试
        run: |
          echo "运行测试..."
          # 使用 LFS 文件进行测试
```

#### 步骤 13：高级 CI/CD 配置

创建文件 `.github/workflows/lfs-pipeline.yml`：

```yaml
name: LFS Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  GIT_LFS_SKIP_SMUDGE: 0  # 确保下载 LFS 文件

jobs:
  # 测试作业
  test:
    runs-on: ubuntu-latest
    
    steps:
      - name: 检出代码
        uses: actions/checkout@v4
        with:
          lfs: true
      
      - name: 缓存 LFS 文件
        uses: actions/cache@v4
        with:
          path: .git/lfs
          key: ${{ runner.os }}-lfs-${{ hashFiles('.lfs-assets-id') }}
          restore-keys: |
            ${{ runner.os }}-lfs-
      
      - name: 拉取 LFS 文件
        run: git lfs pull
      
      - name: 验证 LFS 文件
        run: |
          git lfs ls-files --size
          git lfs fsck
      
      - name: 运行测试
        run: |
          # 使用 LFS 文件运行测试
          echo "运行测试..."
  
  # 构建作业
  build:
    needs: test
    runs-on: ubuntu-latest
    
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
    
    steps:
      - name: 检出代码
        uses: actions/checkout@v4
        with:
          lfs: true
      
      - name: 构建项目
        run: |
          echo "在 ${{ matrix.os }} 上构建..."
          # 构建逻辑
  
  # 部署作业
  deploy:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
      - name: 检出代码
        uses: actions/checkout@v4
        with:
          lfs: true
      
      - name: 部署到服务器
        run: |
          echo "部署包含 LFS 文件的项目..."
          # 部署逻辑
```

#### 步骤 14：创建 LFS 缓存脚本

创建文件 `.github/scripts/cache-lfs.sh`：

```bash
#!/bin/bash

# Git LFS 缓存管理脚本

set -e

CACHE_DIR="${HOME}/.git-lfs-cache"
REPO_URL=$(git remote get-url origin)
REPO_HASH=$(echo -n "$REPO_URL" | sha256sum | cut -d' ' -f1)
CACHE_PATH="${CACHE_DIR}/${REPO_HASH}"

# 创建缓存目录
mkdir -p "$CACHE_PATH"

# 设置 LFS 缓存路径
export GIT_LFS_CACHE_PATH="$CACHE_PATH"

# 检查缓存是否存在
if [ -d "$CACHE_PATH" ] && [ "$(ls -A $CACHE_PATH)" ]; then
    echo "使用现有 LFS 缓存: $CACHE_PATH"
    
    # 从缓存复制 LFS 文件
    if [ -d ".git/lfs" ]; then
        cp -r "$CACHE_PATH/objects" ".git/lfs/" 2>/dev/null || true
    fi
else
    echo "创建新的 LFS 缓存"
fi

# 拉取 LFS 文件
git lfs pull

# 更新缓存
if [ -d ".git/lfs/objects" ]; then
    echo "更新 LFS 缓存..."
    rsync -a ".git/lfs/objects/" "$CACHE_PATH/objects/"
fi

echo "LFS 缓存路径: $CACHE_PATH"
echo "缓存大小: $(du -sh $CACHE_PATH | cut -f1)"
```

### 第五部分：LFS 存储优化

#### 步骤 15：清理 LFS 缓存

```bash
# 查看 LFS 存储使用情况
git lfs ls-files --size | awk '{sum += $2} END {print "总大小:", sum/1024/1024, "MB"}'

# 清理本地 LFS 缓存
git lfs prune

# 预期输出：
# prune: 5 local objects removed, 3 retained

# 查看清理后的状态
git lfs status

# 强制清理（包括最近使用的文件）
# git lfs prune --force

# 清理特定时间之前的文件
git lfs prune --dry-run  # 先预览会删除什么
git lfs prune --recent 7d  # 保留最近7天的文件
```

#### 步骤 16：配置 LFS 存储策略

创建文件 `.lfsconfig`：

```ini
[lfs]
    # 存储路径
    storage = /path/to/lfs/storage
    
    # 并发传输数
    concurrenttransfers = 8
    
    # 是否跳过 smudge（用于 CI/CD）
    # skip_smudge = true
    
    # 批量处理大小
    batchsize = 100

[lfs "fetch"]
    # 获取最近的引用
    recentrefs = 7d
    
    # 获取最近的提交
    recentcommits = 0
    
    # 包含远程引用
    includeremotes = origin

[lfs "prune"]
    # 保留偏移天数
    offsetdays = 3
    
    # 验证远程
    verifyremotesalways = false
    
    # 远程名称
    remotename = origin
```

#### 步骤 17：创建 LFS 管理脚本

创建文件 `scripts/lfs-manager.sh`：

```bash
#!/bin/bash

# Git LFS 管理脚本

set -e

# 颜色定义
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

# 函数：打印帮助信息
show_help() {
    echo "Git LFS 管理工具"
    echo ""
    echo "用法: $0 [命令]"
    echo ""
    echo "命令:"
    echo "  status    - 显示 LFS 状态"
    echo "  info      - 显示 LFS 文件信息"
    echo "  cleanup   - 清理 LFS 缓存"
    echo "  migrate   - 迁移文件到 LFS"
    echo "  verify    - 验证 LFS 文件完整性"
    echo "  help      - 显示此帮助信息"
}

# 函数：显示状态
show_status() {
    echo -e "${GREEN}=== Git LFS 状态 ===${NC}"
    echo ""
    
    echo "LFS 版本:"
    git lfs version
    echo ""
    
    echo "跟踪的文件类型:"
    git lfs track
    echo ""
    
    echo "LFS 文件列表:"
    git lfs ls-files --size
    echo ""
    
    echo "存储使用情况:"
    if [ -d ".git/lfs" ]; then
        du -sh .git/lfs
    else
        echo "未找到 LFS 存储"
    fi
}

# 函数：显示文件信息
show_info() {
    echo -e "${GREEN}=== LFS 文件信息 ===${NC}"
    echo ""
    
    echo "按类型统计:"
    git lfs ls-files | awk -F'.' '{print $NF}' | sort | uniq -c | sort -rn
    echo ""
    
    echo "按大小排序（前10个）:"
    git lfs ls-files --size | sort -k2 -rn | head -10
    echo ""
    
    echo "总文件数: $(git lfs ls-files | wc -l)"
    echo "总大小: $(git lfs ls-files --size | awk '{sum += $2} END {printf "%.2f MB", sum/1024/1024}')"
}

# 函数：清理缓存
cleanup() {
    echo -e "${YELLOW}=== 清理 LFS 缓存 ===${NC}"
    echo ""
    
    echo "清理前:"
    du -sh .git/lfs 2>/dev/null || echo "未找到 LFS 存储"
    echo ""
    
    echo "执行清理..."
    git lfs prune
    echo ""
    
    echo "清理后:"
    du -sh .git/lfs 2>/dev/null || echo "未找到 LFS 存储"
}

# 函数：迁移文件
migrate_files() {
    echo -e "${YELLOW}=== 迁移文件到 LFS ===${NC}"
    echo ""
    
    read -p "请输入要迁移的文件扩展名（用逗号分隔，如 *.psd,*.zip）: " extensions
    
    if [ -z "$extensions" ]; then
        echo -e "${RED}未指定扩展名${NC}"
        return 1
    fi
    
    echo "将迁移以下类型的文件: $extensions"
    echo ""
    
    read -p "确认迁移？(y/N) " confirm
    if [[ $confirm =~ ^[Yy]$ ]]; then
        git lfs migrate import --include="$extensions"
        echo -e "${GREEN}迁移完成${NC}"
    else
        echo "取消迁移"
    fi
}

# 函数：验证文件
verify_files() {
    echo -e "${GREEN}=== 验证 LFS 文件完整性 ===${NC}"
    echo ""
    
    git lfs fsck
    echo ""
    
    echo -e "${GREEN}验证完成${NC}"
}

# 主逻辑
case "${1:-help}" in
    status)
        show_status
        ;;
    info)
        show_info
        ;;
    cleanup)
        cleanup
        ;;
    migrate)
        migrate_files
        ;;
    verify)
        verify_files
        ;;
    help|*)
        show_help
        ;;
esac
```

```bash
# 使脚本可执行
chmod +x scripts/lfs-manager.sh

# 使用脚本
./scripts/lfs-manager.sh status
./scripts/lfs-manager.sh info
./scripts/lfs-manager.sh cleanup
./scripts/lfs-manager.sh verify
```

---

## 验证练习结果

### 检查清单

完成练习后，请验证以下内容：

- [ ] Git LFS 已正确安装和初始化
- [ ] 跟踪规则已配置在 `.gitattributes` 中
- [ ] 大文件已正确添加到 LFS
- [ ] 推送到远程仓库成功
- [ ] CI/CD 工作流能正确处理 LFS 文件
- [ ] LFS 缓存清理功能正常

### 验证命令

```bash
# 检查 LFS 安装
git lfs version

# 查看跟踪规则
git lfs track

# 查看 LFS 文件
git lfs ls-files

# 验证文件完整性
git lfs fsck

# 检查仓库大小
git count-objects -vH

# 测试克隆（在新目录）
cd /tmp
git clone https://github.com/yourusername/lfs-demo.git lfs-test
cd lfs-test
git lfs ls-files
```

---

## 进阶挑战

### 挑战 1：创建 LFS 存储监控脚本

创建监控 LFS 存储使用情况的脚本：

```bash
#!/bin/bash

# 监控 LFS 存储使用情况

THRESHOLD_MB=100  # 警告阈值

# 获取 LFS 存储大小
get_lfs_size() {
    if [ -d ".git/lfs" ]; then
        du -sm .git/lfs | cut -f1
    else
        echo 0
    fi
}

# 检查大小
LFS_SIZE=$(get_lfs_size)

if [ "$LFS_SIZE" -gt "$THRESHOLD_MB" ]; then
    echo "警告: LFS 存储使用 ${LFS_SIZE}MB，超过阈值 ${THRESHOLD_MB}MB"
    
    # 发送通知
    # 可以集成 Slack、邮件等通知方式
fi
```

### 挑战 2：实现 LFS 文件去重

创建检测和删除重复 LFS 文件的工具：

```python
#!/usr/bin/env python3

import hashlib
import os
from collections import defaultdict

def find_duplicate_lfs_files():
    """查找重复的 LFS 文件"""
    
    lfs_dir = ".git/lfs/objects"
    if not os.path.exists(lfs_dir):
        return {}
    
    # 按文件大小分组
    size_groups = defaultdict(list)
    
    for root, dirs, files in os.walk(lfs_dir):
        for file in files:
            filepath = os.path.join(root, file)
            size = os.path.getsize(filepath)
            size_groups[size].append(filepath)
    
    # 查找可能重复的文件
    duplicates = {}
    for size, files in size_groups.items():
        if len(files) > 1:
            # 计算文件哈希
            hash_groups = defaultdict(list)
            for filepath in files:
                with open(filepath, 'rb') as f:
                    file_hash = hashlib.sha256(f.read()).hexdigest()
                hash_groups[file_hash].append(filepath)
            
            # 保存重复文件
            for file_hash, hash_files in hash_groups.items():
                if len(hash_files) > 1:
                    duplicates[file_hash] = {
                        'size': size,
                        'files': hash_files
                    }
    
    return duplicates

if __name__ == "__main__":
    duplicates = find_duplicate_lfs_files()
    
    if duplicates:
        print("发现重复的 LFS 文件:")
        for file_hash, info in duplicates.items():
            print(f"\n哈希: {file_hash}")
            print(f"大小: {info['size']} bytes")
            print("文件:")
            for filepath in info['files']:
                print(f"  - {filepath}")
    else:
        print("未发现重复的 LFS 文件")
```

### 挑战 3：创建 LFS 备份工具

创建自动备份 LFS 文件的工具：

```bash
#!/bin/bash

# LFS 备份脚本

BACKUP_DIR="/path/to/backup"
REPO_NAME=$(basename $(git rev-parse --show-toplevel))
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_PATH="${BACKUP_DIR}/${REPO_NAME}_${TIMESTAMP}"

# 创建备份目录
mkdir -p "$BACKUP_PATH"

# 备份 LFS 文件
echo "备份 LFS 文件..."
if [ -d ".git/lfs/objects" ]; then
    cp -r ".git/lfs/objects" "$BACKUP_PATH/"
fi

# 备份配置
cp .gitattributes "$BACKUP_PATH/" 2>/dev/null || true
cp .lfsconfig "$BACKUP_PATH/" 2>/dev/null || true

# 压缩备份
echo "压缩备份..."
tar -czf "${BACKUP_PATH}.tar.gz" -C "$BACKUP_DIR" "$(basename $BACKUP_PATH)"

# 清理临时目录
rm -rf "$BACKUP_PATH"

echo "备份完成: ${BACKUP_PATH}.tar.gz"
echo "备份大小: $(du -sh ${BACKUP_PATH}.tar.gz | cut -f1)"
```

### 挑战 4：实现 LFS 文件版本管理

创建管理 LFS 文件版本的工具：

```python
#!/usr/bin/env python3

import subprocess
import json
from datetime import datetime

def get_lfs_file_versions(filepath):
    """获取 LFS 文件的所有版本"""
    
    # 获取文件的提交历史
    result = subprocess.run(
        ['git', 'log', '--pretty=format:%H|%ai|%s', '--follow', '--', filepath],
        capture_output=True,
        text=True
    )
    
    versions = []
    for line in result.stdout.strip().split('\n'):
        if line:
            commit_hash, date, message = line.split('|', 2)
            versions.append({
                'commit': commit_hash,
                'date': date,
                'message': message
            })
    
    return versions

def get_file_size_at_commit(filepath, commit):
    """获取特定提交时的文件大小"""
    
    result = subprocess.run(
        ['git', 'cat-file', '-s', f'{commit}:{filepath}'],
        capture_output=True,
        text=True
    )
    
    try:
        return int(result.stdout.strip())
    except ValueError:
        return 0

if __name__ == "__main__":
    filepath = input("请输入文件路径: ")
    versions = get_lfs_file_versions(filepath)
    
    print(f"\n文件 {filepath} 的版本历史:")
    print("-" * 60)
    
    for version in versions:
        size = get_file_size_at_commit(filepath, version['commit'])
        print(f"提交: {version['commit'][:8]}")
        print(f"日期: {version['date']}")
        print(f"说明: {version['message']}")
        print(f"大小: {size / 1024 / 1024:.2f} MB")
        print("-" * 60)
```

---

## 常见问题

### Q1：Git LFS 和 Git 子模块有什么区别？

| 特性 | Git LFS | Git 子模块 |
|------|---------|-----------|
| 用途 | 管理大文件 | 引用其他仓库 |
| 存储 | 外部存储 | 独立仓库 |
| 克隆 | 按需下载 | 需要额外克隆 |
| 版本 | 与主仓库同步 | 独立版本 |

### Q2：如何查看 LFS 存储配额？

```bash
# 查看 GitHub LFS 使用情况
gh api user/settings/billing/summary --jq '.plans[].name'

# 或访问 GitHub 网页：
# Settings → Billing and plans → Usage
```

### Q3：LFS 文件被意外删除怎么办？

```bash
# 从 LFS 缓存恢复
git lfs checkout

# 从远程重新下载
git lfs pull

# 查看 LFS 文件状态
git lfs status
```

### Q4：如何优化 LFS 克隆速度？

```bash
# 浅克隆 + LFS
git clone --depth 1 --filter=blob:none https://github.com/user/repo.git

# 只克隆特定分支
git clone -b main --single-branch https://github.com/user/repo.git

# 跳过 LFS 文件（后续按需下载）
GIT_LFS_SKIP_SMUDGE=1 git clone https://github.com/user/repo.git
git lfs pull --include="specific-file"
```

---

## Git LFS 深入理解与最佳实践

### LFS 的内部工作机制

理解 Git LFS 的内部工作机制有助于更好地使用和排查问题。当你执行 `git add` 命令添加一个被 LFS 跟踪的文件时，Git LFS 的过滤器会自动介入。首先，LFS 会计算文件的 SHA-256 哈希值，这个哈希值作为文件的唯一标识。然后，LFS 将实际的文件内容存储到本地的 `.git/lfs/objects` 目录中，按照哈希值的前两位作为子目录进行组织。最后，在 Git 仓库中存储一个指针文件，这个指针文件非常小，通常只有一百多字节，包含了版本信息、哈希值和文件大小。

当你执行 `git push` 命令时，Git LFS 会将本地存储的大文件上传到 LFS 服务器。上传完成后，LFS 服务器会返回一个确认信息。当你执行 `git clone` 或 `git pull` 命令时，Git 会先获取指针文件，然后通过 smudge 过滤器自动下载对应的大文件。如果设置了 `GIT_LFS_SKIP_SMUDGE` 环境变量，LFS 文件不会自动下载，需要手动执行 `git lfs pull` 命令来获取。

### 何时使用 Git LFS

并非所有大文件都需要使用 Git LFS 管理。判断是否需要使用 LFS 可以从以下几个维度考虑。文件大小是一个重要因素，通常超过 1MB 的二进制文件建议使用 LFS 管理。文件变更频率也很关键，频繁变更的大文件会导致仓库体积快速增长，使用 LFS 可以避免这个问题。文件类型决定了压缩效率，已经压缩过的文件格式如 ZIP、JPEG、MP4 等，Git 的增量压缩效果很差，使用 LFS 更为合适。团队协作规模也影响决策，大团队中每个人克隆仓库的成本更高，使用 LFS 可以显著减少克隆时间。

适合使用 LFS 管理的文件类型包括图像素材文件，例如 PSD 源文件、AI 设计文件、TIFF 高清图片等。视频和音频文件，例如 MP4 视频、WAV 音频、工程项目的音频工程文件等。三维模型文件，例如 FBX、OBJ、Blend、Maya 等格式的模型文件。游戏资源文件，例如 Unity 的 AssetBundle、Unreal 的 Pak 文件等。编译产物，例如大型二进制安装包、编译后的动态链接库等。数据集文件，例如机器学习的训练数据、大型 CSV 数据文件等。

不太适合使用 LFS 的文件包括文本源代码文件，Git 对文本文件的版本管理效率很高。小型配置文件，这些文件体积小且压缩效果好。已经被 Git 高效管理的历史文件，除非仓库体积已经过大需要瘦身。

### 团队协作中的 LFS 工作流

在团队协作中使用 LFS 需要建立统一的工作流规范。首先，团队所有成员都需要安装 Git LFS 客户端，可以在项目文档中说明安装方法。其次，LFS 的跟踪规则应该提交到仓库的 `.gitattributes` 文件中，确保所有人使用相同的配置。第三，在代码审查时注意检查是否有新的大文件被添加但未被 LFS 跟踪。第四，建立大文件的命名和组织规范，避免文件散落在仓库各处。

对于设计师和内容创作者等非开发人员，需要提供简化的操作指南。可以创建图形化的操作手册，说明如何安装 LFS、如何提交大文件和如何获取最新版本。考虑使用 Git 图形化工具，例如 GitKraken、SourceTree 或 GitHub Desktop，这些工具对 LFS 有良好的支持。建立定期同步机制，确保设计师的工作成果能够及时集成到项目中。

### LFS 存储成本管理

GitHub 对 LFS 存储和带宽有配额限制，合理管理存储成本非常重要。定期审查 LFS 中的文件，删除不再需要的历史版本。使用 `git lfs prune` 命令清理本地不再需要的 LFS 文件缓存。对于大型项目，考虑使用自建的 LFS 服务器或第三方存储服务。配置 LFS 服务器使用对象存储服务，例如 AWS S3、Azure Blob Storage 或阿里云 OSS，利用这些服务的成本优势。

监控 LFS 的使用情况，设置存储和带宽使用的告警阈值。在 CI/CD 流程中优化 LFS 文件的下载，只下载当前构建需要的文件。使用缓存机制避免重复下载相同的 LFS 文件。对于大型二进制文件，考虑是否可以在构建时生成，而不是存储在仓库中。

### 从 LFS 迁移回 Git

在某些情况下，可能需要将 LFS 管理的文件迁移回普通的 Git 管理。这通常发生在文件体积较小、变更不频繁或不再需要 LFS 的场景。迁移过程需要使用 `git lfs migrate export` 命令，这个命令会将 LFS 指针文件替换为实际的文件内容。迁移完成后需要强制推送到远程仓库，因为这会重写 Git 历史。

需要注意的是，迁移回 Git 会增加仓库体积，影响克隆和拉取速度。在迁移前应该评估仓库体积的变化，确保在可接受范围内。建议在迁移前创建备份分支，以便在出现问题时可以回滚。迁移后需要通知团队所有成员重新克隆仓库，以避免历史冲突。

### LFS 与 Git 子模块的配合

在使用 Git 子模块的项目中，LFS 的配置需要特别注意。子模块需要单独初始化 LFS，每个子模块都有独立的 LFS 配置。在克隆包含子模块的仓库时，需要使用 `git submodule update --init --recursive` 命令并确保 LFS 已初始化。如果子模块中使用了 LFS，需要在子模块目录中执行 `git lfs install` 和 `git lfs pull`。

建议在主仓库的文档中说明子模块的 LFS 配置要求。可以创建初始化脚本，自动化完成子模块的克隆和 LFS 初始化过程。在 CI/CD 流程中，确保工作流包含子模块和 LFS 的初始化步骤。

### LFS 文件的二进制差异

Git LFS 不支持二进制文件的增量差异，每次修改都会存储完整的文件副本。这意味着频繁修改的大文件会快速消耗存储配额。为了优化存储使用，可以考虑以下策略。对于可以拆分的大型文件，将其拆分为多个较小的文件，只更新变更的部分。对于图像文件，保留源文件的同时生成优化后的版本。对于视频文件，使用版本号管理而不是每次修改都提交。对于数据文件，考虑使用增量更新机制，只存储变化的数据。定期清理旧版本的 LFS 文件，只保留最近几个版本。

---

## 延伸阅读

- [Git LFS 官方文档](https://git-lfs.github.com/)
- [GitHub LFS 文档](https://docs.github.com/en/repositories/working-with-files/managing-large-files)
- [Git LFS 最佳实践](https://github.com/git-lfs/git-lfs/wiki/Tutorial)

---

## 练习总结

通过本练习，你已经学会了：

1. ✅ 安装和配置 Git LFS
2. ✅ 设置文件跟踪规则
3. ✅ 使用 LFS 管理大文件
4. ✅ 迁移现有仓库到 LFS
5. ✅ 在 CI/CD 中集成 LFS
6. ✅ 优化 LFS 存储使用

Git LFS 是管理大型二进制文件的重要工具，建议在项目早期就配置好 LFS，避免后期迁移的麻烦。
