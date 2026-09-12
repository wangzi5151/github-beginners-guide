# Git 性能优化完全指南

> 本章将全面介绍 Git 性能优化的策略和技巧，从克隆到日常操作，从客户端到 CI/CD，帮助你在大型仓库中保持高效的开发体验。

---

## 目录

1. [大型仓库的性能挑战](#1-大型仓库的性能挑战)
2. [git clone 优化](#2-git-clone-优化)
3. [git fetch/pull 优化](#3-git-fetchpull-优化)
4. [git status 优化](#4-git-status-优化)
5. [git diff 优化](#5-git-diff-优化)
6. [git gc 与 git repack 配置](#6-git-gc-与-git-repack-配置)
7. [Git LFS 性能调优](#7-git-lfs-性能调优)
8. [Git Hooks 性能优化](#8-git-hooks-性能优化)
9. [CI/CD 中的 Git 性能优化](#9-cicd-中的-git-性能优化)
10. [Monorepo Git 性能策略](#10-monorepo-git-性能策略)
11. [Git 客户端性能对比](#11-git-客户端性能对比)
12. [Git 缓存与预加载策略](#12-git-缓存与预加载策略)
13. [网络层优化](#13-网络层优化)
14. [监控与诊断工具](#14-监控与诊断工具)

---

## 1. 大型仓库的性能挑战

### 1.1 什么是大型仓库

大型仓库通常具有以下特征：

```
┌──────────────────────────────────────────────────────────┐
│              大型仓库特征分析                               │
├──────────────────┬───────────────────────────────────────┤
│     特征         │           描述                         │
├──────────────────┼───────────────────────────────────────┤
│ 代码量大         │ 文件数量 > 10,000 或代码行数 > 1,000,000 │
│ 历史悠久         │ 提交历史 > 100,000 次                   │
│ 二进制文件多     │ 图片、视频、模型等大文件                   │
│ 分支众多         │ 活跃分支 > 100 个                       │
│ 子模块复杂       │ 多层嵌套子模块                           │
│ Monorepo        │ 多个项目共享一个仓库                      │
└──────────────────┴───────────────────────────────────────┘
```

典型的大型仓库案例包括：

- **Linux 内核仓库**：超过 100 万个提交，数十万个文件
- **Windows 操作系统仓库**：超过 300GB，数百万个文件
- **Google 的 Monorepo**：数十亿行代码
- **大型游戏项目**：大量纹理、模型、音频等二进制资产

### 1.2 性能瓶颈分析方法

要优化 Git 性能，首先需要识别瓶颈所在。以下是系统性的分析方法：

```bash
# 分析仓库大小
git count-objects -v --human-readable
# 输出示例:
# count: 150
# size: 620K
# in-pack: 12000
# packs: 3
# size-pack: 450M
# garbage: 0
# size-garbage: 0

# 分析 packfile 内容分布
git verify-pack -v .git/objects/pack/pack-*.idx | \
    awk '{print $2}' | sort | uniq -c | sort -rn
# 输出示例:
# 8000 blob
# 3000 tree
# 800 commit
# 200 tag

# 找出最大的文件（按对象大小排序）
git rev-list --objects --all | \
    git cat-file --batch-check='%(objecttype) %(objectname) %(objectsize) %(rest)' | \
    sed -n 's/^blob //p' | \
    sort -rnk2 | head -20

# 分析提交历史规模
echo "提交总数: $(git log --oneline | wc -l)"
echo "合并提交: $(git log --merges --oneline | wc -l)"
echo "分支总数: $(git branch -a | wc -l)"
echo "文件总数: $(git ls-files | wc -l)"
echo "未跟踪文件: $(git ls-files --others --exclude-standard | wc -l)"

# 分析 packfile 数量和大小
ls -lhS .git/objects/pack/
```

### 1.3 性能指标基准测试

```bash
# 测量 Git 操作耗时
time git status
time git diff
time git log --oneline -100
time git add .
time git commit -m "test"

# 使用 Git 内置追踪
GIT_TRACE=1 git status
GIT_TRACE=1 git add .
GIT_TRACE=1 git commit -m "test"

# 详细追踪（包含性能数据）
GIT_TRACE=1 GIT_TRACE_PERFORMANCE=1 git status

# 性能基准测试脚本
cat > git-benchmark.sh << 'EOF'
#!/bin/bash
echo "Git 性能基准测试"
echo "================"
echo "仓库: $(pwd)"
echo "时间: $(date)"
echo ""

echo "1. git status"
time git status > /dev/null 2>&1

echo ""
echo "2. git diff"
time git diff > /dev/null 2>&1

echo ""
echo "3. git log -100"
time git log --oneline -100 > /dev/null 2>&1

echo ""
echo "4. git add ."
time git add . > /dev/null 2>&1
git reset > /dev/null 2>&1

echo ""
echo "5. git branch -a"
time git branch -a > /dev/null 2>&1

echo ""
echo "================"
echo "测试完成"
EOF

chmod +x git-benchmark.sh
./git-benchmark.sh
```

### 1.4 性能问题的常见原因

```
┌──────────────────────────────────────────────────────────────┐
│              Git 性能问题常见原因                               │
├──────────────────────┬───────────────────────────────────────┤
│      原因类别        │           具体原因                      │
├──────────────────────┼───────────────────────────────────────┤
│ 仓库规模过大         │ 文件数量多、历史长、二进制文件大           │
│ 松散对象过多         │ 未及时 gc、频繁创建小文件                 │
│ 网络传输慢           │ 仓库远程、网络带宽低、协议低效            │
│ 索引缓存未启用       │ fsmonitor、untrackedCache 未开启        │
│ Hooks 耗时           │ pre-commit 运行完整测试、代码分析        │
│ 子模块嵌套深         │ 多层子模块、递归更新                     │
│ 大文件未用 LFS       │ 二进制文件直接存储在 Git 中               │
│ 配置不当             │ 压缩级别、并行度等配置不合理              │
└──────────────────────┴───────────────────────────────────────┘
```

---

## 2. git clone 优化

### 2.1 浅克隆（Shallow Clone）

浅克隆是减少克隆时间最直接有效的方法，它只获取最近的 N 次提交。

```bash
# 基本浅克隆 - 只获取最新提交
git clone --depth=1 https://github.com/user/repo.git

# 指定深度 - 获取最近 50 次提交
git clone --depth=50 https://github.com/user/repo.git

# 浅克隆特定分支
git clone --depth=1 --branch=main https://github.com/user/repo.git

# 浅克隆特定标签
git clone --depth=1 --branch=v1.0.0 https://github.com/user/repo.git

# 浅克隆的大小对比示例:
# 完整克隆: 450MB
# 浅克隆 (depth=1): 50MB
# 浅克隆 (depth=50): 120MB
```

```bash
# 后续增加克隆深度
cd repo
git fetch --deepen=50

# 获取完整历史（取消浅克隆）
git fetch --unshallow

# 检查当前是否为浅克隆
git rev-parse --is-shallow-repository

# 浅克隆的限制：
# - 无法进行某些三方合并操作
# - git blame 只能显示浅层历史
# - git rebase 可能有问题
# - 某些 CI/CD 工具可能不支持
```

### 2.2 部分克隆（Partial Clone）

部分克隆是 Git 2.19 引入的革命性特性，允许按需下载对象。

```bash
# 只克隆 commit 和 tree，blob 按需下载
git clone --filter=blob:none https://github.com/user/repo.git

# 按大小过滤 blob - 不下载大于 1MB 的 blob
git clone --filter=blob:limit=1m https://github.com/user/repo.git

# 按树过滤 - 不下载树对象（极度精简）
git clone --filter=tree:0 https://github.com/user/repo.git

# 组合过滤
git clone --filter=blob:none,tree:0 https://github.com/user/repo.git

# 过滤类型说明:
# blob:none      - 不下载任何 blob（按需获取）
# blob:limit=N   - 不下载大于 N 字节的 blob
# tree:N         - 不下载深度大于 N 的 tree
# tree:0         - 不下载任何 tree（最激进）
```

```bash
# 部分克隆的工作原理:
# 1. 客户端请求克隆，带上过滤条件
# 2. 服务端只传输符合过滤条件的对象
# 3. 客户端在需要时按需获取缺失的对象
# 4. 缺失的对象会缓存在本地

# 配置服务器支持部分克隆
git config uploadpack.allowFilter true
git config uploadpack.blobPackfileUri true

# 查看部分克隆配置
git config --list | grep partialclone

# 手动触发按需下载
git checkout -- src/large-file.bin  # 自动下载缺失的 blob
```

### 2.3 稀疏检出（Sparse Checkout）

稀疏检出允许只检出仓库中的特定目录，显著减少工作区大小。

```bash
# 方法 1: 克隆时启用稀疏检出
git clone --sparse https://github.com/user/repo.git
cd repo
git sparse-checkout init --cone
git sparse-checkout set src/ docs/

# 方法 2: 对已有仓库启用稀疏检出
git sparse-checkout init --cone
git sparse-checkout set src/ docs/ tests/

# 添加更多目录
git sparse-checkout add packages/core/

# 查看当前稀疏检出配置
git sparse-checkout list

# 禁用稀疏检出（检出所有文件）
git sparse-checkout disable

# 重新启用
git sparse-checkout init --cone
git sparse-checkout set .
```

```bash
# cone 模式（推荐）vs 传统模式

# cone 模式 - 只支持目录级别过滤，性能更好
git sparse-checkout init --cone
git sparse-checkout set src/ docs/

# 传统模式 - 支持通配符，更灵活但较慢
git sparse-checkout init
git sparse-checkout set "src/*.py" "docs/**/*.md" "!docs/internal/"

# cone 模式的性能优势:
# - 使用文件系统级别的目录匹配
# - 不需要逐个文件检查 .gitignore 规则
# - 大型仓库中快数倍
```

### 2.4 组合优化策略

将多种优化技术组合使用，可以获得最佳效果：

```bash
# 最佳组合: 浅克隆 + 部分克隆 + 稀疏检出
git clone \
    --depth=1 \
    --filter=blob:none \
    --sparse \
    --branch=main \
    https://github.com/user/repo.git

cd repo

# 初始化稀疏检出
git sparse-checkout init --cone
git sparse-checkout set src/ docs/ tests/

# 大小对比:
# 完整克隆: 450MB + 2GB 工作区 = 2.45GB
# 优化后:   50MB  + 500MB 工作区 = 550MB
```

```bash
# CI/CD 中的典型优化配置
# GitHub Actions
- name: Checkout
  uses: actions/checkout@v4
  with:
    fetch-depth: 1              # 浅克隆
    sparse-checkout: |
      src/
      docs/
      tests/
    sparse-checkout-cone-mode: true

# GitLab CI
variables:
  GIT_DEPTH: 1
  GIT_SPARSE_CHECKOUT_PATHS: "src/ docs/ tests/"

# 自定义 CI 脚本
git clone --depth=1 --filter=blob:none --sparse URL
cd repo
git sparse-checkout init --cone
git sparse-checkout set $(cat .ci-paths.txt)
```

### 2.5 克隆优化效果对比

```
┌──────────────────────────────────────────────────────────────────┐
│                    克隆优化效果对比                                │
├──────────────────┬──────────┬──────────┬──────────┬──────────────┤
│     方法         │ 下载大小  │ 工作区    │ 克隆时间  │ 适用场景      │
├──────────────────┼──────────┼──────────┼──────────┼──────────────┤
│ 完整克隆         │ 450MB    │ 2GB      │ 120s     │ 完整开发      │
│ 浅克隆(depth=1)  │ 50MB     │ 2GB      │ 15s      │ CI/CD        │
│ 部分克隆(blob:none)│ 80MB   │ 2GB      │ 20s      │ 按需开发      │
│ 稀疏检出         │ 450MB    │ 500MB    │ 100s     │ 特定模块开发  │
│ 组合优化         │ 50MB     │ 500MB    │ 10s      │ 最佳实践      │
└──────────────────┴──────────┴──────────┴──────────┴──────────────┘
```

---

## 3. git fetch/pull 优化

### 3.1 fetch 优化

```bash
# 只获取特定分支（而非所有分支）
git fetch origin main

# 获取所有分支
git fetch --all

# 配置并行获取
git config --global fetch.parallel 4

# 浅获取 - 增加克隆深度
git fetch --deepen=50

# 获取但不获取标签
git fetch --no-tags

# 获取并修剪已删除的远程分支
git fetch --prune

# 配置自动 prune
git config --global fetch.prune true
git config --global fetch.pruneTags true
```

```bash
# 使用 refspec 进行精确获取
# 只获取 main 和 develop 分支
git config --add remote.origin.fetch '+refs/heads/main:refs/remotes/origin/main'
git config --add remote.origin.fetch '+refs/heads/develop:refs/remotes/origin/develop'

# 使用协议 v2 优化（Git 2.18+）
git config --global protocol.version 2
# 协议 v2 的优势:
# - 只传输需要的引用
# - 支持引用过滤
# - 减少网络往返

# 查看 fetch 详情
GIT_TRACE=1 GIT_TRANSFER_TRACE=1 git fetch origin main
```

### 3.2 pull 优化

```bash
# 配置 pull 行为为 rebase（避免不必要的合并提交）
git config --global pull.rebase true

# 只允许快进合并
git config --global pull.ff only

# 自动 stash 未提交的更改
git config --global rebase.autoStash true

# 推荐的 pull 配置组合
git config --global pull.rebase true
git config --global pull.ff only
git config --global rebase.autoStash true
git config --global rebase.updateRefs true

# 使用 rebase + autostash 进行 pull
git pull --rebase --autostash origin main

# 查看 pull 的详细信息
git pull --verbose origin main
```

### 3.3 子模块优化

```bash
# 并行更新子模块
git submodule update --init --recursive --jobs=4

# 浅克隆子模块
git submodule update --init --depth=1

# 只更新特定子模块
git submodule update --init -- src/lib

# 配置子模块为浅克隆
cat > .gitmodules << 'EOF'
[submodule "src/lib"]
    path = src/lib
    url = https://github.com/user/lib.git
    shallow = true
    branch = main
EOF

# 全局子模块配置
git config --global submodule.recurse true
git config --global submodule.shallow true
git config --global submodule.fetchJobs 4

# 批量更新子模块
git submodule foreach --recursive 'git fetch --depth=1 && git checkout main && git pull'
```

### 3.4 增量更新策略

```bash
# 使用 refspec 进行增量更新
git fetch origin \
    refs/heads/main:refs/remotes/origin/main \
    refs/heads/develop:refs/remotes/origin/develop

# 只获取新提交（浅获取）
git fetch --depth=1 origin main

# 增加深度
git fetch --deepen=100 origin main

# 获取特定标签
git fetch origin tag v1.0.0

# 使用 prune 清理已删除的分支
git fetch --prune origin

# 配置自动 prune
git config --global fetch.prune true
git config --global fetch.pruneTags true
```

---

## 4. git status 优化

### 4.1 文件系统监控（fsmonitor）

fsmonitor 是 Git 最重要的性能优化之一，它利用操作系统级别的文件系统事件来避免全量扫描。

```bash
# 启用 fsmonitor（Git 2.37+）
git config core.fsmonitor true

# 使用 Watchman 作为 fsmonitor 后端
# 安装 Watchman:
# Ubuntu/Debian:
sudo apt-get install watchman

# macOS:
brew install watchman

# 配置 Git 使用 Watchman
git config core.fsmonitor "git-fsmonitor--daemon"

# 启动 fsmonitor 守护进程
git fsmonitor--daemon start

# 查看 fsmonitor 状态
git fsmonitor--daemon status

# 停止 fsmonitor 守护进程
git fsmonitor--daemon stop

# fsmonitor 的性能提升效果:
# 未启用: git status 耗时 2.5 秒
# 启用后: git status 耗时 0.1 秒
# 提升幅度: 25 倍
```

### 4.2 未跟踪文件缓存（untrackedCache）

```bash
# 启用 untracked cache
git config core.untrackedCache true

# untracked cache 的工作原理:
# 1. 首次运行时扫描所有未跟踪文件
# 2. 将结果缓存到 .git/untracked-cache/ 目录
# 3. 后续运行时只检查变更的目录
# 4. 使用 mtime 判断目录是否变更

# 查看 untracked cache 状态
git ls-files --untracked --debug

# 清理 untracked cache
git clean -fdx  # 删除未跟踪的文件和目录

# untracked cache 的性能提升效果:
# 未启用: git status 耗时 1.5 秒
# 启用后: git status 耗时 0.3 秒
# 提升幅度: 5 倍
```

### 4.3 综合状态优化配置

```bash
# 完整的 status 优化配置
git config core.fsmonitor true
git config core.untrackedCache true
git config feature.manyFiles true

# feature.manyFiles 启用以下优化:
# - core.untrackedCache = true
# - core.fsmonitor = true
# - index.version = 4
# - index.skipHash = false

# 查看当前配置
git config --get core.fsmonitor
git config --get core.untrackedCache
git config --get feature.manyFiles
```

```bash
# 性能对比测试
echo "优化前:"
git config --unset core.fsmonitor
git config --unset core.untrackedCache
time git status > /dev/null 2>&1

echo "优化后:"
git config core.fsmonitor true
git config core.untrackedCache true
time git status > /dev/null 2>&1
```

### 4.4 status 高级用法

```bash
# 快速查看状态（简洁格式）
git status --short
git status -s

# 只显示未跟踪的文件
git status --untracked-files
git status -u

# 忽略子模块的状态
git status --ignore-submodules=dirty

# 使用 porcelain 格式（适合脚本解析）
git status --porcelain
git status --porcelain=v2

# 查看分支信息
git status --branch
git status -b

# 性能优化的状态检查技巧
git diff --quiet           # 只检查是否有修改，返回退出码
git diff --cached --quiet  # 只检查暂存区是否有修改
```

```bash
# 批量状态检查脚本
cat > check-status.sh << 'EOF'
#!/bin/bash
# 快速检查仓库状态

# 检查是否有未提交的更改
if ! git diff --quiet; then
    echo "有未暂存的更改"
fi

# 检查是否有暂存的更改
if ! git diff --cached --quiet; then
    echo "有暂存的更改"
fi

# 检查是否有未跟踪的文件
if [ -n "$(git ls-files --others --exclude-standard)" ]; then
    echo "有未跟踪的文件"
fi

# 检查是否有冲突
if [ -n "$(git ls-files -u)" ]; then
    echo "有未解决的冲突"
fi
EOF

chmod +x check-status.sh
```

---

## 5. git diff 优化

### 5.1 diff 算法选择

Git 支持四种 diff 算法，各有优缺点：

```
┌──────────────────────────────────────────────────────────────┐
│                   Git Diff 算法对比                           │
├──────────────┬──────────┬──────────┬────────────────────────┤
│     算法     │   速度    │  质量    │        特点            │
├──────────────┼──────────┼──────────┼────────────────────────┤
│ myers        │ 最快     │ 一般     │ 默认算法               │
│ minimal      │ 最慢     │ 最小     │ 最小化 diff 行数       │
│ patience     │ 中等     │ 较好     │ 更好的重命名检测       │
│ histogram    │ 较快     │ 最好     │ 最佳的重命名检测       │
└──────────────┴──────────┴──────────┴────────────────────────┘
```

```bash
# 配置 diff 算法
git config diff.algorithm histogram

# 临时使用不同算法
git diff --diff-algorithm=patience
git diff --diff-algorithm=minimal

# 算法性能测试
echo "myers:"
time git diff --diff-algorithm=myers > /dev/null 2>&1

echo "patience:"
time git diff --diff-algorithm=patience > /dev/null 2>&1

echo "histogram:"
time git diff --diff-algorithm=histogram > /dev/null 2>&1
```

### 5.2 diff 输出优化

```bash
# 使用 stat 模式（只显示统计，不显示详细差异）
git diff --stat
git diff --shortstat

# 使用摘要模式
git diff --summary

# 限制 diff 的上下文行数（减少输出量）
git diff --unified=3  # 默认
git diff --unified=1  # 减少上下文

# 只比较特定类型的文件
git diff -- '*.py'
git diff -- 'src/*.js'

# 忽略空白字符差异
git diff --ignore-all-space
git diff -w

# 忽略空白字符变更
git diff --ignore-space-change
git diff -b

# 使用颜色高亮
git diff --color-words
git diff --color-moved
```

### 5.3 大文件 diff 优化

```bash
# 配置大文件 diff 驱动（只显示元数据差异）
git config diff.psd.textconv "identify -verbose"
git config diff.pdf.textconv "pdftotext"
git config diff.docx.textconv "pandoc -t plain"
git config diff.xlsx.textconv "xlsx2csv"

# .gitattributes 配置
*.psd  diff=psd
*.pdf  diff=pdf
*.docx diff=docx
*.xlsx diff=xlsx

# 跳过二进制文件的 diff
git diff --binary

# 只显示二进制文件的统计
git diff --stat --binary

# 配置外部 diff 工具
git config diff.tool vscode
git config difftool.vscode.cmd "code --wait --diff $LOCAL $REMOTE"
```

### 5.4 diff 性能优化配置

```bash
# diff 相关的性能优化配置
git config diff.algorithm histogram
git config diff.colorMoved default
git config diff.renames true
git config diff.submodule log

# 配置 rename 检测阈值
git config diff.renameLimit 10000

# 启用 diff 的索引缓存
git config diff.cached true
```

---

## 6. git gc 与 git repack 配置

### 6.1 自动 gc 配置

```bash
# 自动 gc 阈值
git config gc.auto 6700        # 松散对象数量阈值（默认值）
git config gc.autoPackLimit 50 # packfile 数量阈值（默认值）

# 自动 gc 策略
git config gc.autoDetach true       # 后台运行 gc
git config gc.writeCommitGraph true # 写入 commit-graph 加速 log

# 禁用自动 gc（适用于 CI/CD 环境）
git config gc.auto 0

# 手动触发 gc
git gc

# 查看当前 gc 配置
git config --get gc.auto
git config --get gc.autoPackLimit
git config --get gc.writeCommitGraph
```

### 6.2 激进 gc 策略

```bash
# 激进 gc（更彻底的压缩，但耗时更长）
git gc --aggressive

# 配置激进 gc 的参数
git config gc.aggressiveDepth 50
git config gc.aggressiveWindow 250

# 激进 gc 的适用场景:
# 1. 仓库大小显著增大
# 2. 首次 gc 后仍有大量松散对象
# 3. 需要最大化压缩率

# 激进 gc 的缺点:
# 1. 耗时较长（可能需要数分钟甚至数小时）
# 2. CPU 使用率高
# 3. 可能影响其他 Git 操作
```

### 6.3 repack 优化

```bash
# 基本 repack
git repack -a -d

# 参数说明:
# -a: 打包所有对象（包括不在任何 packfile 中的）
# -d: 删除多余的 packfile
# -l: 只打包本地引用
# -f: 强制重新打包
# -n: 不更新服务器信息

# 增量 repack（指定 delta 参数）
git repack -a -d --depth=250 --window=250

# 使用几何级数 repack（Git 2.24+，推荐）
git repack --geometric=2 -d

# 几何级数 repack 的优势:
# 1. 自动平衡 packfile 大小
# 2. 减少 packfile 数量
# 3. 优化查找效率
# 4. 避免单个超大 packfile
```

```bash
# repack 相关配置
git config pack.window 250       # delta 搜索窗口大小
git config pack.depth 50         # delta 链最大深度
git config pack.threads 4        # 并行压缩线程数
git config pack.windowMemory 1g  # delta 搜索内存限制
git config pack.packSizeLimit 2g # 单个 packfile 大小限制

# 定期 repack 脚本
cat > git-repack.sh << 'EOF'
#!/bin/bash
echo "开始 Git repack..."

# 基本 repack
echo "1. 基本 repack..."
git repack -a -d

# 几何级数 repack
echo "2. 几何级数 repack..."
git repack --geometric=2 -d

# 清理不可达对象
echo "3. 清理不可达对象..."
git prune

# 更新服务器信息
echo "4. 更新服务器信息..."
git update-server-info

# 写入 commit-graph
echo "5. 写入 commit-graph..."
git commit-graph write --reachable

echo "repack 完成"
EOF

chmod +x git-repack.sh
```

### 6.4 commit-graph 优化

```bash
# commit-graph 用于加速 git log 和 git merge-base

# 写入 commit-graph
git commit-graph write

# 只写入可达的提交
git commit-graph write --reachable

# 验证 commit-graph
git commit-graph verify

# 配置自动更新 commit-graph
git config gc.writeCommitGraph true

# commit-graph 的性能提升:
# 未启用: git log --oneline -1000 耗时 2 秒
# 启用后: git log --oneline -1000 耗时 0.1 秒
# 提升幅度: 20 倍

# 查看 commit-graph 文件
ls -la .git/objects/info/commit-graph*

# commit-graph 的链式结构
# 支持多个 commit-graph 文件链接
# 增量更新时只需写入新的 commit-graph
```

---

## 7. Git LFS 性能调优

### 7.1 LFS 基础配置

```bash
# 安装 Git LFS
git lfs install

# 跟踪大文件
git lfs track "*.psd"
git lfs track "*.zip"
git lfs track "*.mp4"
git lfs track "*.bin"
git lfs track "*.so"
git lfs track "*.dll"

# 查看跟踪规则
git lfs track

# 查看 LFS 文件列表
git lfs ls-files

# 查看 LFS 状态
git lfs status
```

### 7.2 LFS 性能优化配置

```bash
# 1. 增加并行传输数
git config lfs.concurrenttransfers 8  # 默认 3

# 2. 使用 LFS 本地缓存
git config lfs.storage ~/.git-lfs-cache

# 3. 只获取需要的 LFS 文件
git lfs pull --include="src/**"
git lfs pull --exclude="tests/**"

# 4. 配置 LFS smudge 过滤器
git config filter.lfs.smudge "git-lfs smudge -- %f"
git config filter.lfs.clean "git-lfs clean -- %f"
git config filter.lfs.required true

# 5. 延迟 LFS 下载（按需获取）
git config filter.lfs.smudge "git-lfs smudge --skip -- %f"
# 之后手动下载: git lfs pull

# 6. 配置 LFS 传输重试
git config lfs.transfer.maxretries 3
git config lfs.transfer.maxrequests 100

# 7. 查看 LFS 配置
git config --list | grep lfs
```

### 7.3 LFS 缓存策略

```bash
# 配置 LFS 缓存目录
git config lfs.storage ~/.git-lfs-cache

# 查看缓存大小
du -sh ~/.git-lfs-cache

# 清理旧的缓存
git lfs prune

# 共享 LFS 缓存（团队共享服务器）
git config lfs.storage /shared/git-lfs-cache

# LFS 缓存的目录结构:
# ~/.git-lfs-cache/
# ├── lfs/
# │   ├── objects/
# │   │   ├── ab/
# │   │   │   └── cd/
# │   │   │       └── abcdef1234567890...
# │   │   └── ...
# │   └── tmp/
# └── ...

# LFS 性能对比:
# 未缓存: git lfs pull 耗时 30 秒
# 缓存后: git lfs pull 耗时 2 秒
```

### 7.4 LFS 迁移

```bash
# 将现有大文件迁移到 LFS
git lfs migrate import --include="*.psd" --everything
git lfs migrate import --include="*.zip,*.mp4" --everything

# 从 LFS 迁移回来
git lfs migrate export --include="*.psd" --everything

# 查看迁移信息
git lfs migrate info
git lfs migrate info --include="*.psd"

# LFS 迁移注意事项:
# 1. 迁移会重写历史
# 2. 需要强制推送到远程
# 3. 团队成员需要重新克隆
```

---

## 8. Git Hooks 性能优化

### 8.1 Hooks 性能问题分析

```bash
# 常见的性能问题:
# 1. pre-commit hook 运行完整的测试套件
# 2. pre-push hook 运行耗时的代码分析
# 3. post-checkout hook 安装依赖

# 分析 hooks 耗时
GIT_TRACE=1 git commit -m "test" 2>&1 | grep -E "hook|time"

# 查看 hooks 执行时间
time git commit -m "test" 2>&1
```

### 8.2 优化 pre-commit hook

```bash
#!/bin/bash
# 优化后的 pre-commit hook

# 1. 只检查暂存的文件（而非所有文件）
STAGED_FILES=$(git diff --cached --name-only --diff-filter=ACM)

# 2. 如果没有暂存的文件，直接退出
if [ -z "$STAGED_FILES" ]; then
    exit 0
fi

# 3. 只运行相关的检查（按文件类型）
for file in $STAGED_FILES; do
    # Python 文件检查
    if [[ "$file" == *.py ]]; then
        python -m black --check "$file" || exit 1
    fi

    # JavaScript/TypeScript 文件检查
    if [[ "$file" == *.js ]] || [[ "$file" == *.ts ]]; then
        npx prettier --check "$file" || exit 1
    fi
done

# 4. 使用 lint 缓存避免重复检查
LINT_CACHE=".git/lint-cache"
if [ -f "$LINT_CACHE" ]; then
    LAST_LINT=$(cat "$LINT_CACHE")
    CURRENT_HASH=$(echo "$STAGED_FILES" | md5sum | cut -d' ' -f1)
    if [ "$LAST_LINT" = "$CURRENT_HASH" ]; then
        echo "Lint 缓存命中，跳过检查"
        exit 0
    fi
fi

# 5. 运行 lint
npm run lint

# 6. 保存缓存
echo "$STAGED_FILES" | md5sum | cut -d' ' -f1 > "$LINT_CACHE"

exit 0
```

### 8.3 使用 pre-commit 框架

```bash
# 安装 pre-commit 框架
pip install pre-commit

# 创建配置文件
cat > .pre-commit-config.yaml << 'EOF'
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.4.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
      - id: check-added-large-files
        args: ['--maxkb=1000']

  - repo: https://github.com/psf/black
    rev: 23.3.0
    hooks:
      - id: black
        language_version: python3

  - repo: https://github.com/PyCQA/flake8
    rev: 6.0.0
    hooks:
      - id: flake8
        args: ['--max-line-length=120']
EOF

# 安装 hooks
pre-commit install

# 运行所有 hooks
pre-commit run --all-files

# 跳过特定 hook
SKIP=flake8 git commit -m "test"

# pre-commit 的缓存机制:
# 1. 首次运行时下载和安装工具到缓存目录
# 2. 后续运行时使用缓存的工具
# 3. 只检查修改的文件（自动暂存后检查）
```

### 8.4 异步和条件化 Hooks

```bash
#!/bin/bash
# post-commit hook - 异步运行耗时任务

# 后台运行测试和构建
(
    npm test &
    npm run build &

    # 等待所有后台任务完成
    wait
) &

# 立即返回，不阻塞提交
exit 0
```

```bash
#!/bin/bash
# pre-push hook - 只在必要时运行测试

# 检查是否有测试文件被修改
MODIFIED_TESTS=$(git diff --name-only HEAD@{1}..HEAD 2>/dev/null | grep -E "test.*\.(py|js|ts)$")

if [ -n "$MODIFIED_TESTS" ]; then
    echo "检测到测试文件修改，运行测试..."
    npm test
fi

exit 0
```

---

## 9. CI/CD 中的 Git 性能优化

### 9.1 CI/CD 克隆优化

```yaml
# GitHub Actions 优化配置
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 1                    # 浅克隆
          sparse-checkout: |                # 稀疏检出
            src/
            docs/
            tests/
          sparse-checkout-cone-mode: true

      - name: Cache dependencies
        uses: actions/cache@v3
        with:
          path: |
            node_modules
            .git/lfs
          key: ${{ runner.os }}-deps-${{ hashFiles('**/package-lock.json') }}
```

```yaml
# GitLab CI 优化配置
variables:
  GIT_DEPTH: 1
  GIT_SUBMODULE_STRATEGY: none
  GIT_CLEAN_FLAGS: -ffdx
  GIT_SPARSE_CHECKOUT_PATHS: "src/ docs/ tests/"

build:
  cache:
    key: ${CI_COMMIT_REF_SLUG}
    paths:
      - node_modules/
      - .git/lfs/
  script:
    - npm install
    - npm run build
```

### 9.2 CI/CD 缓存策略

```yaml
# 多层缓存策略
- name: Cache Git LFS
  uses: actions/cache@v3
  with:
    path: .git/lfs
    key: ${{ runner.os }}-lfs-${{ hashFiles('.lfs-assets-id') }}
    restore-keys: |
      ${{ runner.os }}-lfs-

- name: Cache node modules
  uses: actions/cache@v3
  with:
    path: node_modules
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
    restore-keys: |
      ${{ runner.os }}-node-

- name: Cache build artifacts
  uses: actions/cache@v3
  with:
    path: dist
    key: ${{ runner.os }}-build-${{ github.sha }}
    restore-keys: |
      ${{ runner.os }}-build-
```

### 9.3 CI/CD 增量构建

```yaml
# 只在特定文件变更时运行作业
on:
  push:
    paths:
      - 'src/**'
      - 'package.json'
      - 'package-lock.json'

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Check changes
        uses: dorny/paths-filter@v2
        id: changes
        with:
          filters: |
            src:
              - 'src/**'
            tests:
              - 'tests/**'
            docs:
              - 'docs/**'

      - name: Build
        if: steps.changes.outputs.src == 'true'
        run: npm run build

      - name: Test
        if: steps.changes.outputs.tests == 'true'
        run: npm test

      - name: Deploy docs
        if: steps.changes.outputs.docs == 'true'
        run: npm run deploy-docs
```

### 9.4 CI/CD 并行化

```yaml
# 并行测试分片
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        shard: [1, 2, 3, 4]
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 1

      - name: Install dependencies
        run: npm ci

      - name: Run tests (shard ${{ matrix.shard }})
        run: npm test -- --shard=${{ matrix.shard }}/4
```

---

## 10. Monorepo Git 性能策略

### 10.1 Monorepo 的性能挑战

```bash
# Monorepo 的典型问题:
# 1. 仓库巨大（>10GB）
# 2. 文件数量多（>100,000）
# 3. 提交频繁（每天数百次）
# 4. 分支众多（数百个）
# 5. CI/CD 构建时间长

# 分析 Monorepo 仓库
echo "仓库大小: $(git count-objects -v --human-readable | grep size-pack)"
echo "文件总数: $(git ls-files | wc -l)"
echo "提交总数: $(git log --oneline | wc -l)"
echo "分支总数: $(git branch -a | wc -l)"
```

### 10.2 稀疏检出策略

```bash
# 初始化稀疏检出
git sparse-checkout init --cone

# 按团队/项目设置检出目录
git sparse-checkout set \
    packages/core \
    packages/shared \
    packages/team-a

# 动态添加目录
git sparse-checkout add packages/team-b

# 创建稀疏检出配置文件（可提交到仓库）
cat > .sparse-checkout << 'EOF'
packages/core/
packages/shared/
packages/team-a/
tools/
configs/
EOF

# 应用配置
git sparse-checkout set --stdin < .sparse-checkout
```

### 10.3 部分克隆策略

```bash
# 部分克隆 + 稀疏检出组合
git clone \
    --filter=blob:none \
    --sparse \
    --depth=1 \
    https://github.com/org/monorepo.git

cd monorepo

git sparse-checkout init --cone
git sparse-checkout set packages/team-a/

# 按需获取文件
git checkout -- packages/team-a/src/index.ts
```

### 10.4 工作区策略

```bash
# 使用 git worktree 管理多个工作区
git worktree add ../monorepo-feature-a feature-a
git worktree add ../monorepo-bugfix-123 bugfix-123

# 查看工作区列表
git worktree list

# 删除工作区
git worktree remove ../monorepo-feature-a

# 工作区的优势:
# 1. 可以同时在多个分支工作
# 2. 避免频繁的 checkout（大型仓库中很慢）
# 3. 每个工作区有独立的文件状态
# 4. 共享同一个 .git 目录
```

### 10.5 Monorepo 工具集成

```bash
# 使用 Nx（JavaScript/TypeScript Monorepo）
npx nx run-many --target=build --all
npx nx affected --target=test    # 只测试受影响的项目

# 使用 Turborepo
npx turbo run build
npx turbo run test

# 使用 Bazel（Google 的构建系统）
bazel build //packages/core:all
bazel test //packages/core:tests

# 使用 Lerna
npx lerna run build
npx lerna run test

# 这些工具的共同优势:
# 1. 增量构建（只构建变更的部分）
# 2. 任务缓存（避免重复构建）
# 3. 并行执行（利用多核 CPU）
# 4. 依赖分析（自动确定构建顺序）
```

---

## 11. Git 客户端性能对比

### 11.1 原生 Git

```bash
# 原生 Git 的优势:
# 1. 最新特性支持
# 2. 广泛的文档和社区支持
# 3. 跨平台兼容性
# 4. 无额外依赖

# 原生 Git 的性能特点:
# - 小型仓库: 优秀（毫秒级）
# - 中型仓库: 良好（秒级）
# - 大型仓库: 需要优化配置

# 原生 Git 优化配置
git config core.fsmonitor true
git config core.untrackedCache true
git config feature.manyFiles true
git config protocol.version 2
```

### 11.2 Git LFS

```bash
# Git LFS 的优势:
# 1. 大文件支持（GB 级别）
# 2. 与原生 Git 无缝集成
# 3. 广泛的平台支持
# 4. 按需下载

# Git LFS 的性能特点:
# - 大文件存储: 优秀
# - 网络传输: 良好（支持并行）
# - 缓存机制: 良好

# Git LFS 优化配置
git config lfs.concurrenttransfers 8
git config lfs.storage ~/.git-lfs-cache
```

### 11.3 Gitless

```bash
# Gitless 是 Git 的简化前端
# 安装: pip install gitless

# Gitless 的优势:
# 1. 简化的命令（不需要 index/stage）
# 2. 更直观的工作流
# 3. 自动处理常见操作

# Gitless 的性能特点:
# - 与原生 Git 相同（底层使用 Git）
# - 无额外性能开销
```

### 11.4 Jujutsu (jj)

```bash
# Jujutsu 是新一代版本控制系统
# 安装: cargo install --git https://github.com/martinvonz/jj

# Jujutsu 的优势:
# 1. 与 Git 仓库兼容
# 2. 更好的合并处理
# 3. 工作区概念（类似 worktree）
# 4. 操作日志和撤销

# Jujutsu 的性能特点:
# - 某些操作比 Git 更快
# - 内存使用更低
# - 更好的大仓库支持
```

### 11.5 性能对比测试方法

```bash
#!/bin/bash
# Git 客户端性能对比测试脚本

REPO_URL="https://github.com/user/large-repo.git"
TEST_DIR="/tmp/git-perf-test"

echo "Git 客户端性能对比测试"
echo "======================"

# 清理测试目录
rm -rf "$TEST_DIR"
mkdir -p "$TEST_DIR"

# 测试克隆时间
echo ""
echo "1. 克隆测试"
time git clone "$REPO_URL" "$TEST_DIR/repo" 2>&1

cd "$TEST_DIR/repo"

# 测试 status 时间
echo ""
echo "2. Status 测试"
time git status > /dev/null 2>&1

# 测试 diff 时间
echo ""
echo "3. Diff 测试"
time git diff > /dev/null 2>&1

# 测试 log 时间
echo ""
echo "4. Log 测试"
time git log --oneline -100 > /dev/null 2>&1

# 测试 add 时间
echo ""
echo "5. Add 测试"
echo "test" > test-file.txt
time git add . > /dev/null 2>&1
git reset > /dev/null 2>&1
rm test-file.txt

echo ""
echo "======================"
echo "测试完成"

# 清理
cd /
rm -rf "$TEST_DIR"
```

---

## 12. Git 缓存与预加载策略

### 12.1 Git 内置缓存

```bash
# untracked cache - 缓存未跟踪文件列表
git config core.untrackedCache true

# fsmonitor - 文件系统监控缓存
git config core.fsmonitor true

# commit-graph - 提交图缓存
git config gc.writeCommitGraph true

# packfile delta 缓存
git config pack.deltaCacheSize 1g
git config pack.deltaCacheLimit 1000

# packfile 窗口缓存
git config core.packedGitLimit 1g
git config core.packedGitWindowSize 1g

# 索引版本（v4 更快）
git config index.version 4
```

### 12.2 预加载脚本

```bash
#!/bin/bash
# Git 仓库预加载脚本

REPO_DIR="$1"

if [ -z "$REPO_DIR" ]; then
    echo "Usage: $0 <repo-dir>"
    exit 1
fi

cd "$REPO_DIR"

echo "预加载 Git 仓库: $REPO_DIR"
echo "开始时间: $(date)"

# 1. 预加载索引（触发 fsmonitor 和 untracked cache）
echo "1. 预加载索引..."
time git status > /dev/null 2>&1

# 2. 写入 commit-graph
echo "2. 写入 commit-graph..."
time git commit-graph write --reachable

# 3. 重新打包
echo "3. 重新打包..."
time git repack -a -d

# 4. 预加载 untracked cache
echo "4. 预加载 untracked cache..."
time git ls-files --others --exclude-standard > /dev/null 2>&1

# 5. 预加载 LFS 文件
if git lfs version > /dev/null 2>&1; then
    echo "5. 预加载 LFS 文件..."
    time git lfs pull > /dev/null 2>&1
fi

echo "预加载完成: $(date)"
```

### 12.3 缓存预热策略

```bash
#!/bin/bash
# Git 缓存预热脚本（适用于服务器）

REPOS=(
    "/srv/git/repo1.git"
    "/srv/git/repo2.git"
    "/srv/git/repo3.git"
)

for repo in "${REPOS[@]}"; do
    if [ -d "$repo" ]; then
        echo "预热: $repo"
        cd "$repo"

        # 更新服务器信息
        git update-server-info

        # 写入 commit-graph
        git commit-graph write --reachable

        # 重新打包
        git repack -a -d --geometric=2

        # 清理不可达对象
        git prune

        cd - > /dev/null
    fi
done

echo "缓存预热完成"
```

```bash
# 在 cron 中定期运行缓存预热
# 每天凌晨 2 点运行
# 0 2 * * * /path/to/git-cache-warmup.sh

# 在 systemd 中运行
cat > /etc/systemd/git-cache-warmup.service << 'EOF'
[Unit]
Description=Git Cache Warmup
After=network.target

[Service]
Type=oneshot
ExecStart=/path/to/git-cache-warmup.sh
User=git
Group=git

[Install]
WantedBy=multi-user.target
EOF

cat > /etc/systemd/git-cache-warmup.timer << 'EOF'
[Unit]
Description=Run Git Cache Warmup daily

[Timer]
OnCalendar=*-*-* 02:00:00
Persistent=true

[Install]
WantedBy=timers.target
EOF
```

### 12.4 内存缓存配置

```bash
# Git 的内存缓存配置

# packfile 内存限制
git config core.packedGitLimit 1g

# packfile 窗口大小
git config core.packedGitWindowSize 1g

# delta 缓存大小
git config pack.deltaCacheSize 1g

# delta 缓存条目限制
git config pack.deltaCacheLimit 1000

# 大文件阈值
git config core.bigFileThreshold 512m

# 性能对比测试
echo "未优化:"
git config --unset core.packedGitLimit
git config --unset core.packedGitWindowSize
time git log --oneline -1000 > /dev/null 2>&1

echo "优化后:"
git config core.packedGitLimit 1g
git config core.packedGitWindowSize 1g
time git log --oneline -1000 > /dev/null 2>&1
```

---

## 13. 网络层优化

### 13.1 压缩优化

```bash
# 配置压缩级别（1-9）
git config core.compression 6        # 默认级别
git config core.looseCompression 6   # 松散对象压缩
git config pack.compression 6        # packfile 压缩

# 压缩级别对比:
# 1: 最快，压缩率最低（适合快速网络）
# 6: 平衡（默认，推荐）
# 9: 最慢，压缩率最高（适合慢速网络）

# HTTP 传输缓冲区
git config http.postBuffer 524288000  # 500MB

# 大文件阈值
git config core.bigFileThreshold 512m
```

### 13.2 代理配置

```bash
# HTTP 代理
git config --global http.proxy http://proxy.example.com:8080
git config --global https.proxy https://proxy.example.com:8080

# SOCKS 代理
git config --global http.proxy socks5://proxy.example.com:1080

# 特定域名的代理
git config --global http.https://github.com.proxy http://proxy.example.com:8080

# 代理认证
git config --global http.proxy http://user:password@proxy.example.com:8080

# 禁用代理
git config --global --unset http.proxy
git config --global --unset https.proxy

# 环境变量代理
export http_proxy=http://proxy.example.com:8080
export https_proxy=https://proxy.example.com:8080
export no_proxy=github.com,gitlab.com
```

### 13.3 CDN 和镜像加速

```bash
# 使用 GitHub 镜像（中国大陆加速）
git config url."https://ghproxy.com/https://github.com/".insteadOf "https://github.com/"

# 使用 GitLab 镜像
git config url."https://gitlab.example.com/".insteadOf "https://gitlab.com/"

# 使用 Gitee 镜像
git config url."https://gitee.com/".insteadOf "https://github.com/"

# 查看镜像配置
git config --get-regexp url

# 移除镜像配置
git config --unset url."https://ghproxy.com/https://github.com/".insteadOf

# 为特定仓库设置镜像
git config url."https://mirror.example.com/".insteadOf "https://github.com/specific-org/"
```

### 13.4 网络超时配置

```bash
# 配置 HTTP 超时
git config --global http.lowSpeedLimit 1000    # 最低速度 (bytes/s)
git config --global http.lowSpeedTime 30       # 低速持续时间 (秒)

# 配置连接超时
git config --global http.connectTimeout 30     # 连接超时 (秒)

# 禁用证书验证（不推荐，仅用于测试）
git config --global http.sslVerify false

# 配置 DNS 缓存
git config --global http.dnsCacheTimeout 600   # DNS 缓存时间 (秒)
```

### 13.5 多通道传输

```bash
# 并行获取
git config --global fetch.parallel 4

# 并行推送
git config --global push.parallel 4

# 并行子模块更新
git config --global submodule.fetchJobs 4

# 并行 LFS 传输
git config lfs.concurrenttransfers 8

# 查看当前配置
echo "fetch.parallel: $(git config --get fetch.parallel)"
echo "push.parallel: $(git config --get push.parallel)"
echo "submodule.fetchJobs: $(git config --get submodule.fetchJobs)"
echo "lfs.concurrenttransfers: $(git config --get lfs.concurrenttransfers)"
```

---

## 14. 监控与诊断工具

### 14.1 Git 内置追踪

```bash
# 基本追踪（显示 Git 内部操作）
GIT_TRACE=1 git status

# 详细追踪（包含性能数据）
GIT_TRACE=1 GIT_TRACE_PERFORMANCE=1 git status

# 网络追踪
GIT_TRACE=1 GIT_TRANSFER_TRACE=1 git fetch origin

# 打包追踪
GIT_TRACE=1 GIT_PACK_TRACE=1 git gc

# 追踪输出到文件
GIT_TRACE=/tmp/git-trace.log git status
GIT_TRACE_PERFORMANCE=/tmp/git-perf.log git status

# 追踪特定操作
GIT_TRACE=1 git add .
GIT_TRACE=1 git commit -m "test"
GIT_TRACE=1 git push origin main
```

### 14.2 系统性能分析工具

```bash
# 使用 time 命令
time git status
time git diff
time git log --oneline -100

# 使用 strace 追踪系统调用（Linux）
strace -c git status

# 使用 dtruss 追踪系统调用（macOS）
sudo dtruss -c git status

# 使用 perf 进行性能分析（Linux）
perf record -g git status
perf report

# 使用火焰图分析
git clone https://github.com/brendangregg/FlameGraph.git
perf record -g git status
perf script | FlameGraph/stackcollapse-perf.pl | FlameGraph/flamegraph.pl > git-status.svg
```

### 14.3 仓库健康检查脚本

```bash
#!/bin/bash
# Git 仓库健康检查脚本

echo "Git 仓库健康检查"
echo "================"
echo "仓库: $(pwd)"
echo "时间: $(date)"
echo ""

# 1. 检查仓库大小
echo "1. 仓库大小:"
git count-objects -v --human-readable
echo ""

# 2. 检查对象数量
echo "2. 对象统计:"
git count-objects -v
echo ""

# 3. 检查 packfile
echo "3. Packfile 信息:"
ls -lhS .git/objects/pack/ 2>/dev/null || echo "无 packfile"
echo ""

# 4. 检查引用
echo "4. 引用统计:"
echo "  总引用: $(git show-ref | wc -l)"
echo "  分支: $(git branch | wc -l)"
echo "  远程分支: $(git branch -r | wc -l)"
echo "  标签: $(git tag | wc -l)"
echo ""

# 5. 检查未跟踪文件
echo "5. 文件统计:"
echo "  已跟踪: $(git ls-files | wc -l)"
echo "  未跟踪: $(git ls-files --others --exclude-standard | wc -l)"
echo ""

# 6. 检查大文件
echo "6. 大文件 (>10MB):"
git rev-list --objects --all | \
    git cat-file --batch-check='%(objecttype) %(objectname) %(objectsize) %(rest)' | \
    sed -n 's/^blob //p' | \
    awk '$2 > 10485760 {printf "  %s (%.1fMB)\n", $3, $2/1048576}' | \
    sort -k2 -rn | head -10
echo ""

# 7. 检查完整性
echo "7. 完整性检查:"
git fsck --no-reflogs --unreachable 2>&1 | head -20
echo ""

# 8. 检查配置
echo "8. 性能配置:"
echo "  core.fsmonitor: $(git config --get core.fsmonitor || echo '未设置')"
echo "  core.untrackedCache: $(git config --get core.untrackedCache || echo '未设置')"
echo "  feature.manyFiles: $(git config --get feature.manyFiles || echo '未设置')"
echo "  protocol.version: $(git config --get protocol.version || echo '未设置')"
echo ""

echo "================"
echo "检查完成"
```

### 14.4 性能监控脚本

```bash
#!/bin/bash
# Git 性能监控脚本

LOG_FILE="/var/log/git-performance.log"
REPO_DIR="${1:-.}"

log_metric() {
    local operation=$1
    local duration=$2
    local timestamp=$(date '+%Y-%m-%d %H:%M:%S')
    echo "$timestamp,$operation,$duration" >> "$LOG_FILE"
}

monitor_operation() {
    local operation=$1
    shift
    local start_time=$(date +%s%N)
    "$@" > /dev/null 2>&1
    local end_time=$(date +%s%N)
    local duration=$(( (end_time - start_time) / 1000000 ))
    log_metric "$operation" "$duration"
    echo "$operation: ${duration}ms"
}

cd "$REPO_DIR" || exit 1

echo "开始性能监控..."
echo "仓库: $(pwd)"
echo "时间: $(date)"
echo ""

# 监控常用操作
monitor_operation "status" git status
monitor_operation "diff" git diff
monitor_operation "log-100" git log --oneline -100
monitor_operation "branch" git branch -a
monitor_operation "add" git add .
git reset > /dev/null 2>&1

echo ""
echo "监控完成"
echo "日志文件: $LOG_FILE"
```

### 14.5 可视化监控

```bash
# 使用 git-stats 查看统计
pip install git-stats

# 生成统计报告
git-stats --since="2024-01-01" --until="2024-12-31"

# 使用 gitstats 生成 HTML 报告
# 安装: apt-get install gitstats
gitstats /path/to/repo /output/dir

# 使用 Gource 可视化提交历史
# 安装: apt-get install gource
gource --seconds-per-day 0.1

# 使用 GitKraken 查看仓库状态
# GitKraken 是一个图形化的 Git 客户端，提供直观的性能分析

# 使用 VS Code Git 扩展
# VS Code 内置了 Git 支持，可以查看状态、差异、历史等
```

---

## 总结

本章全面介绍了 Git 性能优化的策略和技巧：

- **克隆优化**：使用浅克隆、部分克隆、稀疏检出减少下载量和工作区大小
- **fetch/pull 优化**：配置并行获取、使用协议 v2、优化子模块更新
- **status 优化**：启用 fsmonitor 和 untrackedCache，提升状态检查速度
- **diff 优化**：选择合适的 diff 算法、配置缓存、优化输出格式
- **gc/repack 配置**：优化自动 gc 阈值、使用几何级数 repack、启用 commit-graph
- **LFS 调优**：配置并行传输、使用缓存、延迟下载
- **Hooks 优化**：只检查修改的文件、使用缓存、异步执行
- **CI/CD 优化**：使用浅克隆、配置缓存、并行化构建
- **Monorepo 策略**：使用稀疏检出、部分克隆、工作区管理
- **客户端对比**：了解不同客户端的性能特点和适用场景
- **缓存策略**：配置内置缓存、创建预加载脚本、定期预热
- **网络优化**：配置压缩、代理、CDN 加速
- **监控工具**：使用 Git 内置追踪、系统性能分析工具、仓库健康检查

通过合理配置和优化，可以在大型仓库中保持高效的开发体验。关键是要根据具体的仓库特征和工作流程，选择合适的优化策略，并持续监控和调整。

---

## 附录一：性能优化的系统方法论

性能优化不是简单地调整几个配置参数，而是一个系统性的工程过程。在进行 Git 性能优化之前，我们需要建立一套完整的方法论，包括问题识别、瓶颈分析、方案设计和效果验证四个阶段。

### 问题识别阶段

在问题识别阶段，我们需要收集性能数据，了解当前的性能状况。这包括测量各种 Git 操作的耗时，分析仓库的规模和结构，以及了解团队的工作流程。通过这些数据，我们可以识别出哪些操作是最耗时的，哪些仓库是最大的，哪些工作流程是最频繁的。

```bash
# 收集性能数据
echo "=== 仓库基本信息 ==="
echo "仓库大小: $(git count-objects -v --human-readable | grep 'size-pack' | awk '{print $2}')"
echo "文件数量: $(git ls-files | wc -l)"
echo "提交数量: $(git log --oneline | wc -l)"
echo "分支数量: $(git branch -a | wc -l)"

echo ""
echo "=== 操作耗时测试 ==="
echo -n "git status: "
time git status > /dev/null 2>&1

echo -n "git diff: "
time git diff > /dev/null 2>&1

echo -n "git log -100: "
time git log --oneline -100 > /dev/null 2>&1
```

### 瓶颈分析阶段

在瓶颈分析阶段，我们需要深入分析性能数据，找出导致性能问题的根本原因。这可能包括仓库过大、网络传输慢、配置不当等多种因素。通过使用 Git 内置的追踪工具和系统性能分析工具，我们可以精确定位瓶颈所在。

```bash
# 使用 Git 追踪工具分析瓶颈
GIT_TRACE=1 GIT_TRACE_PERFORMANCE=1 git status 2>&1 | grep -E "trace|time"

# 使用系统工具分析
strace -c git status 2>&1 | tail -20

# 分析文件系统性能
time find . -name "*.py" | wc -l
time git ls-files "*.py" | wc -l
```

### 方案设计阶段

在方案设计阶段，我们需要根据瓶颈分析的结果，设计合适的优化方案。这可能包括配置调整、工具升级、流程改进等多种措施。在设计方案时，我们需要考虑方案的可行性、成本和收益，确保方案能够有效地解决问题。

```bash
# 根据瓶颈设计优化方案
# 如果瓶颈是 git status 太慢:
git config core.fsmonitor true
git config core.untrackedCache true

# 如果瓶颈是网络传输太慢:
git config --global fetch.parallel 4
git config --global protocol.version 2
git clone --depth=1 --filter=blob:none URL

# 如果瓶颈是仓库太大:
git gc --aggressive
git repack -a -d --geometric=2
```

### 效果验证阶段

在效果验证阶段，我们需要测量优化后的性能数据，与优化前的数据进行对比，验证优化效果。如果效果不理想，我们需要重新分析瓶颈，调整优化方案。通过持续的监控和调整，我们可以确保仓库始终保持最佳性能。

```bash
# 验证优化效果
echo "=== 优化后性能测试 ==="
echo -n "git status: "
time git status > /dev/null 2>&1

echo -n "git diff: "
time git diff > /dev/null 2>&1

echo -n "git log -100: "
time git log --oneline -100 > /dev/null 2>&1

# 对比优化前后的数据
echo ""
echo "=== 性能对比 ==="
echo "优化前 git status: 2.5 秒"
echo "优化后 git status: 0.1 秒"
echo "提升幅度: 25 倍"
```

---

## 附录二：常见性能问题的诊断流程

在日常开发中，我们经常会遇到各种 Git 性能问题。以下是一些常见问题的诊断流程，可以帮助我们快速定位和解决问题。

### git status 很慢

当 `git status` 变慢时，通常是因为文件系统扫描耗时过长。这可能是因为仓库中文件数量过多，或者未跟踪文件过多。通过启用 fsmonitor 和 untracked cache，我们可以显著提高 `git status` 的性能。

```bash
# 诊断 git status 慢的问题
GIT_TRACE=1 git status 2>&1 | grep -E "trace|time"

# 检查文件数量
echo "已跟踪文件: $(git ls-files | wc -l)"
echo "未跟踪文件: $(git ls-files --others --exclude-standard | wc -l)"

# 启用优化
git config core.fsmonitor true
git config core.untrackedCache true

# 验证优化效果
time git status > /dev/null 2>&1
```

### git clone 很慢

当 `git clone` 变慢时，通常是因为仓库太大或网络传输太慢。通过使用浅克隆、部分克隆和稀疏检出，我们可以显著减少克隆时间和下载量。

```bash
# 诊断 git clone 慢的问题
GIT_TRACE=1 git clone URL 2>&1 | grep -E "trace|time|transfer"

# 检查仓库大小
git count-objects -v --human-readable

# 使用优化克隆
git clone --depth=1 --filter=blob:none --sparse URL
cd repo
git sparse-checkout init --cone
git sparse-checkout set src/ docs/

# 验证优化效果
time git clone --depth=1 --filter=blob:none URL
```

### git push 很慢

当 `git push` 变慢时，通常是因为要推送的对象太多或太大。通过使用增量推送和优化 packfile，我们可以显著提高推送速度。

```bash
# 诊断 git push 慢的问题
GIT_TRACE=1 git push origin main 2>&1 | grep -E "trace|time|transfer"

# 检查要推送的对象
git log origin/main..main --oneline

# 使用增量推送
git push origin main

# 优化 packfile
git repack -a -d
git gc --auto

# 验证优化效果
time git push origin main
```

### git log 很慢

当 `git log` 变慢时，通常是因为提交历史太长或需要遍历的提交太多。通过使用 commit-graph 和限制输出数量，我们可以显著提高日志查询速度。

```bash
# 诊断 git log 慢的问题
GIT_TRACE=1 git log --oneline -100 2>&1 | grep -E "trace|time"

# 检查提交数量
git log --oneline | wc -l

# 启用 commit-graph
git config gc.writeCommitGraph true
git commit-graph write --reachable

# 限制输出数量
git log --oneline -100

# 验证优化效果
time git log --oneline -100 > /dev/null 2>&1
```

---

## 附录三：团队协作中的性能优化

在团队协作中，Git 性能优化不仅影响个人效率，还影响整个团队的生产力。以下是一些团队协作中的性能优化策略。

### 统一的优化配置

团队应该统一使用相同的优化配置，以确保所有成员都能获得一致的性能体验。这可以通过在项目中提供配置脚本或使用配置管理工具来实现。

```bash
# 创建团队配置脚本
cat > setup-git-performance.sh << 'EOF'
#!/bin/bash
# 团队 Git 性能优化配置

echo "配置 Git 性能优化..."

# 全局性能优化
git config --global core.fsmonitor true
git config --global core.untrackedCache true
git config --global feature.manyFiles true
git config --global protocol.version 2
git config --global fetch.parallel 4
git config --global push.parallel 4

# 仓库级性能优化
git config core.fsmonitor true
git config core.untrackedCache true
git config gc.writeCommitGraph true

# 优化 packfile
git config pack.window 250
git config pack.depth 50
git config pack.threads 4

echo "配置完成"
EOF

chmod +x setup-git-performance.sh

# 在项目 README 中说明
echo "## 性能优化" >> README.md
echo "运行 ./setup-git-performance.sh 配置 Git 性能优化" >> README.md
```

### 代码审查中的性能考虑

在代码审查中，我们应该关注可能影响 Git 性能的问题。例如，大文件不应该直接提交到仓库中，而应该使用 Git LFS；频繁的小提交可以合并为较大的提交，以减少提交历史的长度。

```bash
# 在 pre-commit hook 中检查大文件
#!/bin/bash
# 检查是否有大文件被提交
STAGED_FILES=$(git diff --cached --name-only)
for file in $STAGED_FILES; do
    size=$(git cat-file -s ":$file" 2>/dev/null || echo 0)
    if [ "$size" -gt 10485760 ]; then  # 10MB
        echo "警告: 文件 $file 超过 10MB"
        echo "建议使用 Git LFS 存储大文件"
        exit 1
    fi
done

# 检查是否有二进制文件被提交
for file in $STAGED_FILES; do
    if file "$file" | grep -q "binary"; then
        echo "警告: 二进制文件 $file 被提交"
        echo "建议使用 Git LFS 存储二进制文件"
    fi
done

exit 0
```

### 共享的性能监控

团队应该建立共享的性能监控系统，以便及时发现和解决性能问题。这可以通过定期运行性能测试脚本并将结果上传到共享平台来实现。

```bash
#!/bin/bash
# 团队性能监控脚本

REPO_NAME=$(basename $(pwd))
LOG_FILE="/var/log/git-performance/${REPO_NAME}.log"

# 确保日志目录存在
mkdir -p $(dirname "$LOG_FILE")

# 记录性能数据
{
    echo "=== 性能监控报告 ==="
    echo "仓库: $(pwd)"
    echo "时间: $(date)"
    echo ""

    echo "--- 仓库信息 ---"
    git count-objects -v --human-readable
    echo ""

    echo "--- 操作耗时 ---"
    echo -n "git status: "
    time git status > /dev/null 2>&1

    echo -n "git diff: "
    time git diff > /dev/null 2>&1

    echo -n "git log -100: "
    time git log --oneline -100 > /dev/null 2>&1

    echo ""
    echo "========================"
} >> "$LOG_FILE" 2>&1

echo "性能数据已记录到 $LOG_FILE"
```

---

## 附录四：性能优化的持续改进

性能优化不是一次性的任务，而是一个持续的过程。我们需要建立一套持续改进的机制，以确保仓库始终保持最佳性能。

### 定期性能评估

团队应该定期进行性能评估，以了解仓库的性能状况和优化效果。这可以通过每月运行一次性能测试脚本来实现。

```bash
#!/bin/bash
# 月度性能评估脚本

echo "=== 月度性能评估 ==="
echo "仓库: $(pwd)"
echo "日期: $(date)"
echo ""

# 运行性能测试
./git-benchmark.sh

# 生成性能报告
echo ""
echo "=== 性能趋势分析 ==="
if [ -f /var/log/git-performance/$(basename $(pwd)).log ]; then
    echo "历史数据可用"
    tail -100 /var/log/git-performance/$(basename $(pwd)).log | \
        grep "git status" | \
        awk '{print $3}' | \
        sort -n | \
        head -1
else
    echo "暂无历史数据"
fi
```

### 性能优化的文档化

团队应该将性能优化的经验和最佳实践文档化，以便新成员可以快速了解和应用。这可以通过在项目中维护一个性能优化指南来实现。

```bash
# 创建性能优化指南
cat > PERFORMANCE.md << 'EOF'
# Git 性能优化指南

## 快速开始

运行以下命令配置性能优化：

```bash
./setup-git-performance.sh
```

## 常见问题

### git status 很慢

启用 fsmonitor 和 untracked cache：

```bash
git config core.fsmonitor true
git config core.untrackedCache true
```

### git clone 很慢

使用浅克隆和部分克隆：

```bash
git clone --depth=1 --filter=blob:none URL
```

## 最佳实践

1. 使用 Git LFS 存储大文件
2. 定期运行 git gc 和 git repack
3. 使用协议 v2 进行网络传输
4. 启用 commit-graph 加速日志查询

## 监控

运行性能监控脚本：

```bash
./git-performance-monitor.sh
```
EOF
```

### 性能优化的自动化

团队应该将性能优化的过程自动化，以减少人工干预和提高效率。这可以通过使用 CI/CD 工具和自动化脚本来实现。

```yaml
# GitHub Actions 自动化性能优化
name: Git Performance Optimization

on:
  schedule:
    - cron: '0 2 * * 0'  # 每周日凌晨 2 点

jobs:
  optimize:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Run GC
        run: git gc --aggressive

      - name: Run Repack
        run: git repack -a -d --geometric=2

      - name: Write Commit Graph
        run: git commit-graph write --reachable

      - name: Run Performance Test
        run: ./git-benchmark.sh

      - name: Upload Results
        uses: actions/upload-artifact@v3
        with:
          name: performance-report
          path: performance-report.txt
```

---

## 附录五：性能优化的工具链

在进行 Git 性能优化时，我们需要使用各种工具来收集数据、分析问题和验证效果。以下是一些常用的性能优化工具。

### Git 内置工具

Git 提供了多种内置工具来帮助我们进行性能优化。这些工具包括 `git count-objects`、`git verify-pack`、`git fsck` 等。

```bash
# 使用 git count-objects 分析仓库大小
git count-objects -v --human-readable

# 使用 git verify-pack 分析 packfile
git verify-pack -v .git/objects/pack/pack-*.idx

# 使用 git fsck 检查仓库完整性
git fsck --full --strict

# 使用 git reflog 分析引用历史
git reflog show --all
```

### 系统性能工具

除了 Git 内置工具外，我们还可以使用系统性能工具来分析 Git 的性能。这些工具包括 `time`、`strace`、`perf` 等。

```bash
# 使用 time 测量命令耗时
time git status

# 使用 strace 追踪系统调用
strace -c git status

# 使用 perf 进行性能分析
perf record -g git status
perf report

# 使用火焰图分析
perf script | FlameGraph/stackcollapse-perf.pl | FlameGraph/flamegraph.pl > git-status.svg
```

### 第三方工具

除了上述工具外，还有一些第三方工具可以帮助我们进行 Git 性能优化。这些工具包括 `git-lfs`、`git-filter-repo`、`git-stats` 等。

```bash
# 使用 git-lfs 管理大文件
git lfs install
git lfs track "*.psd"
git lfs track "*.zip"

# 使用 git-filter-repo 清理历史
pip install git-filter-repo
git filter-repo --path-glob '*.env' --invert-paths

# 使用 git-stats 生成统计报告
pip install git-stats
git-stats --since="2024-01-01" --until="2024-12-31"
```

### 自定义工具

除了使用现有工具外，我们还可以创建自定义工具来满足特定的需求。这些工具可以封装常用的操作，提高工作效率。

```bash
#!/bin/bash
# 自定义 Git 性能工具

case "$1" in
    "analyze")
        echo "分析仓库性能..."
        git count-objects -v --human-readable
        echo ""
        echo "文件统计:"
        echo "  已跟踪: $(git ls-files | wc -l)"
        echo "  未跟踪: $(git ls-files --others --exclude-standard | wc -l)"
        echo ""
        echo "提交统计:"
        echo "  总提交: $(git log --oneline | wc -l)"
        echo "  合并提交: $(git log --merges --oneline | wc -l)"
        ;;
    "optimize")
        echo "优化仓库性能..."
        git gc --auto
        git repack -a -d
        git commit-graph write --reachable
        echo "优化完成"
        ;;
    "benchmark")
        echo "运行性能基准测试..."
        time git status > /dev/null 2>&1
        time git diff > /dev/null 2>&1
        time git log --oneline -100 > /dev/null 2>&1
        ;;
    *)
        echo "用法: $0 {analyze|optimize|benchmark}"
        exit 1
        ;;
esac
```

---

## 附录六：性能优化的监控告警

为了及时发现和解决性能问题，我们需要建立一套监控告警系统。这个系统可以定期检查仓库的性能指标，并在指标超过阈值时发送告警通知。

### 监控指标

我们需要监控以下关键指标：

- 仓库大小：包括对象数量、packfile 大小等
- 操作耗时：包括 git status、git diff、git log 等常用操作的耗时
- 网络传输：包括克隆、推送、拉取等操作的传输时间和数据量
- 错误率：包括操作失败的次数和原因

```bash
#!/bin/bash
# 性能监控脚本

REPO_NAME=$(basename $(pwd))
METRICS_FILE="/var/log/git-metrics/${REPO_NAME}.json"

# 确保目录存在
mkdir -p $(dirname "$METRICS_FILE")

# 收集指标
{
    echo "{"
    echo "  \"timestamp\": \"$(date -Iseconds)\","
    echo "  \"repo\": \"$(pwd)\","
    echo "  \"size\": {"
    echo "    \"objects\": $(git count-objects -v | grep count | awk '{print $2}'),"
    echo "    \"pack_size\": \"$(git count-objects -v | grep 'size-pack' | awk '{print $2}')\","
    echo "    \"files\": $(git ls-files | wc -l)"
    echo "  },"
    echo "  \"commits\": {"
    echo "    \"total\": $(git log --oneline | wc -l),"
    echo "    \"merges\": $(git log --merges --oneline | wc -l)"
    echo "  },"
    echo "  \"branches\": {"
    echo "    \"local\": $(git branch | wc -l),"
    echo "    \"remote\": $(git branch -r | wc -l)"
    echo "  }"
    echo "}"
} > "$METRICS_FILE"

echo "指标已记录到 $METRICS_FILE"
```

### 告警规则

我们需要定义合理的告警规则，以便在性能问题发生时及时通知相关人员。以下是一些常见的告警规则：

- 仓库大小超过阈值
- 操作耗时超过阈值
- 网络传输失败率超过阈值
- 对象损坏或丢失

```bash
#!/bin/bash
# 告警检查脚本

REPO_NAME=$(basename $(pwd))
ALERT_THRESHOLD_STATUS=5  # git status 耗时阈值（秒）
ALERT_THRESHOLD_SIZE=1073741824  # 仓库大小阈值（1GB）

# 检查 git status 耗时
STATUS_TIME=$(TIMEFORMAT='%R'; time git status > /dev/null 2>&1)
if (( $(echo "$STATUS_TIME > $ALERT_THRESHOLD_STATUS" | bc -l) )); then
    echo "告警: git status 耗时 ${STATUS_TIME} 秒，超过阈值 ${ALERT_THRESHOLD_STATUS} 秒"
    # 发送告警通知
    # curl -X POST "https://hooks.slack.com/..." -d "{\"text\": \"Git 性能告警: git status 耗时 ${STATUS_TIME} 秒\"}"
fi

# 检查仓库大小
REPO_SIZE=$(git count-objects -v | grep 'size-pack' | awk '{print $2}')
if [ "$REPO_SIZE" -gt "$ALERT_THRESHOLD_SIZE" ]; then
    echo "告警: 仓库大小 ${REPO_SIZE} 字节，超过阈值 ${ALERT_THRESHOLD_SIZE} 字节"
    # 发送告警通知
fi
```

### 告警通知

当告警触发时，我们需要及时通知相关人员。这可以通过多种方式实现，包括邮件、即时通讯工具、短信等。

```bash
#!/bin/bash
# 告警通知脚本

ALERT_TYPE=$1
ALERT_MESSAGE=$2

case "$ALERT_TYPE" in
    "email")
        echo "$ALERT_MESSAGE" | mail -s "Git 性能告警" team@example.com
        ;;
    "slack")
        curl -X POST \
            -H 'Content-type: application/json' \
            --data "{\"text\": \"$ALERT_MESSAGE\"}" \
            https://hooks.slack.com/services/xxx/yyy/zzz
        ;;
    "dingtalk")
        curl -X POST \
            -H 'Content-Type: application/json' \
            -d "{\"msgtype\": \"text\", \"text\": {\"content\": \"$ALERT_MESSAGE\"}}" \
            https://oapi.dingtalk.com/robot/send?access_token=xxx
        ;;
    *)
        echo "未知的告警类型: $ALERT_TYPE"
        exit 1
        ;;
esac
```

---

## 附录七：性能优化的案例分析

通过分析实际的性能优化案例，我们可以更好地理解性能优化的方法和技巧。以下是一些典型的性能优化案例。

### 案例一：大型 Monorepo 的性能优化

某公司的 Monorepo 包含了 50 万个文件和 100 万次提交，git status 需要 30 秒才能完成。通过启用 fsmonitor 和 untracked cache，并使用稀疏检出，git status 的耗时降低到了 1 秒。

```bash
# 优化前
time git status  # 30 秒

# 启用 fsmonitor
git config core.fsmonitor true

# 启用 untracked cache
git config core.untrackedCache true

# 使用稀疏检出
git sparse-checkout init --cone
git sparse-checkout set src/team-a/

# 优化后
time git status  # 1 秒

# 性能提升: 30 倍
```

### 案例二：CI/CD 中的克隆优化

某团队的 CI/CD 流水线需要克隆一个 2GB 的仓库，每次克隆需要 10 分钟。通过使用浅克隆和部分克隆，克隆时间降低到了 30 秒。

```bash
# 优化前
time git clone URL  # 10 分钟

# 使用浅克隆
time git clone --depth=1 URL  # 3 分钟

# 使用部分克隆
time git clone --depth=1 --filter=blob:none URL  # 1 分钟

# 使用稀疏检出
time git clone --depth=1 --filter=blob:none --sparse URL  # 30 秒

# 性能提升: 20 倍
```

### 案例三：大文件存储优化

某游戏开发团队的仓库包含了大量纹理和模型文件，导致仓库大小超过 10GB。通过使用 Git LFS，仓库大小降低到了 500MB，克隆时间从 1 小时降低到了 5 分钟。

```bash
# 优化前
git count-objects -v --human-readable
# size-pack: 10GB

# 安装 Git LFS
git lfs install

# 迁移大文件到 LFS
git lfs migrate import --include="*.psd,*.fbx,*.png" --everything

# 优化后
git count-objects -v --human-readable
# size-pack: 500MB

# 性能提升: 20 倍
```

### 案例四：网络传输优化

某跨国团队在网络条件较差的环境下工作，git push 和 git pull 经常超时。通过配置代理和优化压缩参数，网络传输的稳定性得到了显著提升。

```bash
# 配置 HTTP 代理
git config --global http.proxy http://proxy.example.com:8080

# 配置压缩级别
git config --global core.compression 1

# 配置缓冲区大小
git config --global http.postBuffer 524288000

# 配置超时
git config --global http.lowSpeedLimit 1000
git config --global http.lowSpeedTime 60

# 验证优化效果
time git push origin main
time git pull origin main
```

---

## 附录八：性能优化的自动化测试

为了确保性能优化的效果，我们需要建立自动化的性能测试体系。这个体系可以定期运行性能测试，并生成测试报告。

### 性能测试脚本

```bash
#!/bin/bash
# Git 性能自动化测试脚本

TEST_DIR="/tmp/git-perf-test-$(date +%Y%m%d_%H%M%S)"
REPORT_FILE="$TEST_DIR/report.txt"

mkdir -p "$TEST_DIR"

echo "=== Git 性能自动化测试 ===" > "$REPORT_FILE"
echo "测试时间: $(date)" >> "$REPORT_FILE"
echo "仓库: $(pwd)" >> "$REPORT_FILE"
echo "" >> "$REPORT_FILE"

# 测试 git status
echo "测试 git status..." >> "$REPORT_FILE"
for i in {1..5}; do
    TIME=$(TIMEFORMAT='%R'; time git status > /dev/null 2>&1)
    echo "  第 $i 次: ${TIME} 秒" >> "$REPORT_FILE"
done

# 测试 git diff
echo "测试 git diff..." >> "$REPORT_FILE"
for i in {1..5}; do
    TIME=$(TIMEFORMAT='%R'; time git diff > /dev/null 2>&1)
    echo "  第 $i 次: ${TIME} 秒" >> "$REPORT_FILE"
done

# 测试 git log
echo "测试 git log -100..." >> "$REPORT_FILE"
for i in {1..5}; do
    TIME=$(TIMEFORMAT='%R'; time git log --oneline -100 > /dev/null 2>&1)
    echo "  第 $i 次: ${TIME} 秒" >> "$REPORT_FILE"
done

echo "" >> "$REPORT_FILE"
echo "测试完成" >> "$REPORT_FILE"

echo "测试报告已生成: $REPORT_FILE"
cat "$REPORT_FILE"
```

### 性能回归测试

```bash
#!/bin/bash
# Git 性能回归测试脚本

# 定义性能基线
BASELINE_STATUS=1.0  # git status 基线耗时（秒）
BASELINE_DIFF=0.5    # git diff 基线耗时（秒）
BASELINE_LOG=0.3     # git log 基线耗时（秒）

# 运行测试
STATUS_TIME=$(TIMEFORMAT='%R'; time git status > /dev/null 2>&1)
DIFF_TIME=$(TIMEFORMAT='%R'; time git diff > /dev/null 2>&1)
LOG_TIME=$(TIMEFORMAT='%R'; time git log --oneline -100 > /dev/null 2>&1)

# 检查是否超过基线
FAILED=0

if (( $(echo "$STATUS_TIME > $BASELINE_STATUS * 2" | bc -l) )); then
    echo "失败: git status 耗时 ${STATUS_TIME} 秒，超过基线 $BASELINE_STATUS 秒的 2 倍"
    FAILED=1
fi

if (( $(echo "$DIFF_TIME > $BASELINE_DIFF * 2" | bc -l) )); then
    echo "失败: git diff 耗时 ${DIFF_TIME} 秒，超过基线 $BASELINE_DIFF 秒的 2 倍"
    FAILED=1
fi

if (( $(echo "$LOG_TIME > $BASELINE_LOG * 2" | bc -l) )); then
    echo "失败: git log 耗时 ${LOG_TIME} 秒，超过基线 $BASELINE_LOG 秒的 2 倍"
    FAILED=1
fi

if [ "$FAILED" -eq 0 ]; then
    echo "通过: 所有测试都在基线范围内"
    exit 0
else
    echo "失败: 存在性能回归"
    exit 1
fi
```

---

## 附录九：性能优化的最佳实践总结

通过前面的讨论，我们可以总结出以下性能优化的最佳实践。

### 克隆优化的最佳实践

在克隆大型仓库时，我们应该根据实际需求选择合适的克隆策略。如果只需要最新代码，使用浅克隆；如果只需要部分目录，使用稀疏检出；如果仓库包含大文件，使用部分克隆。

```bash
# 最佳实践：组合使用多种优化策略
git clone \
    --depth=1 \
    --filter=blob:none \
    --sparse \
    --branch=main \
    https://github.com/user/repo.git

cd repo

git sparse-checkout init --cone
git sparse-checkout set src/ docs/ tests/
```

### 日常操作的优化最佳实践

在日常开发中，我们应该启用各种缓存和优化选项，以提高常用操作的性能。这包括启用 fsmonitor、untracked cache 和 commit-graph。

```bash
# 最佳实践：启用所有性能优化选项
git config core.fsmonitor true
git config core.untrackedCache true
git config feature.manyFiles true
git config gc.writeCommitGraph true
git config diff.algorithm histogram
git config protocol.version 2
git config fetch.parallel 4
git config push.parallel 4
```

### 仓库维护的最佳实践

为了保持仓库的最佳性能，我们应该定期进行仓库维护。这包括运行 gc、repack 和 prune，以及清理不可达对象。

```bash
# 最佳实践：定期维护仓库
git gc --auto
git repack -a -d --geometric=2
git commit-graph write --reachable
git prune --expire=30.days.ago
```

### 团队协作的最佳实践

在团队协作中，我们应该统一使用相同的优化配置，并建立共享的性能监控系统。这可以确保所有成员都能获得一致的性能体验，并及时发现和解决性能问题。

```bash
# 最佳实践：统一团队配置
# 在项目中提供配置脚本
./setup-git-performance.sh

# 在 CI/CD 中定期运行性能测试
# 在 GitHub Actions 中配置性能监控
```

### 大仓库管理的最佳实践

对于大型仓库，我们应该使用稀疏检出、部分克隆和 Git LFS 等技术来管理仓库的规模。这可以显著减少克隆时间和工作区大小。

```bash
# 最佳实践：大型仓库管理策略
# 使用稀疏检出
git sparse-checkout init --cone
git sparse-checkout set src/ docs/

# 使用部分克隆
git clone --filter=blob:none URL

# 使用 Git LFS
git lfs install
git lfs track "*.psd"
git lfs track "*.zip"
```

---

## 附录十：性能优化速查表

```bash
# ===== 快速优化配置 =====

# 1. 全局性能优化
git config --global core.fsmonitor true
git config --global core.untrackedCache true
git config --global feature.manyFiles true
git config --global protocol.version 2
git config --global fetch.parallel 4
git config --global push.parallel 4

# 2. 克隆优化
git clone --depth=1 --filter=blob:none --sparse URL

# 3. 状态优化
git config core.fsmonitor true
git config core.untrackedCache true

# 4. Diff 优化
git config diff.algorithm histogram

# 5. GC 优化
git config gc.auto 6700
git config gc.autoPackLimit 50
git config gc.writeCommitGraph true

# 6. LFS 优化
git config lfs.concurrenttransfers 8
git config lfs.storage ~/.git-lfs-cache

# 7. 网络优化
git config --global http.postBuffer 524288000
git config --global fetch.parallel 4

# 8. 监控诊断
GIT_TRACE=1 git status
git count-objects -v --human-readable
git fsck --no-reflogs
```

---

> **上一章**: [Git 内部机制深度解析](./X11-git-internals-deep-dive.md)

```bash
# ===== 快速优化配置 =====

# 1. 全局性能优化
git config --global core.fsmonitor true
git config --global core.untrackedCache true
git config --global feature.manyFiles true
git config --global protocol.version 2
git config --global fetch.parallel 4
git config --global push.parallel 4

# 2. 克隆优化
git clone --depth=1 --filter=blob:none --sparse URL

# 3. 状态优化
git config core.fsmonitor true
git config core.untrackedCache true

# 4. Diff 优化
git config diff.algorithm histogram

# 5. GC 优化
git config gc.auto 6700
git config gc.autoPackLimit 50
git config gc.writeCommitGraph true

# 6. LFS 优化
git config lfs.concurrenttransfers 8
git config lfs.storage ~/.git-lfs-cache

# 7. 网络优化
git config --global http.postBuffer 524288000
git config --global fetch.parallel 4

# 8. 监控诊断
GIT_TRACE=1 git status
git count-objects -v --human-readable
git fsck --no-reflogs
```

---

> **上一章**: [Git 内部机制深度解析](./X11-git-internals-deep-dive.md)
