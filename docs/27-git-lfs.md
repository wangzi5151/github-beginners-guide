# Git LFS 大文件管理

## 什么是 Git LFS？

Git LFS (Large File Storage) 用于管理大型文件，避免仓库体积过大。

## 安装

### macOS
```bash
brew install git-lfs
```

### Windows
```powershell
winget install Git.GitLFS
```

### Linux
```bash
sudo apt install git-lfs
```

## 基本使用

### 初始化

```bash
git lfs install
```

### 追踪文件

```bash
# 追踪特定文件
git lfs track "*.psd"
git lfs track "*.zip"

# 追踪特定目录
git lfs track "assets/**"
```

### 查看追踪规则

```bash
cat .gitattributes
```

### 取消追踪

```bash
git lfs untrack "*.psd"
```

## 工作流程

```bash
# 1. 初始化 LFS
git lfs install

# 2. 追踪大文件
git lfs track "*.zip"

# 3. 添加并提交
git add .gitattributes
git add large-file.zip
git commit -m "添加大文件"

# 4. 推送
git push
```

## .gitattributes 文件

```
*.psd filter=lfs diff=lfs merge=lfs -text
*.zip filter=lfs diff=lfs merge=lfs -text
*.mp4 filter=lfs diff=lfs merge=lfs -text
```

## 常用命令

```bash
# 查看 LFS 状态
git lfs status

# 查看追踪的文件
git lfs ls-files

# 迁移到 LFS
git lfs migrate import --include="*.psd" --everything

# 从 LFS 移除
git lfs migrate export --include="*.psd" --everything
```

## 配额限制

| 计划 | 存储 | 带宽 |
|------|------|------|
| Free | 1 GB | 1 GB/月 |
| Pro | 2 GB | 2 GB/月 |
| Team | 10 GB | 10 GB/月 |

## 使用场景

- 设计文件（PSD, AI, Sketch）
- 视频文件（MP4, MOV）
- 音频文件（WAV, MP3）
- 数据集（CSV, JSON 大文件）
- 二进制文件（EXE, DLL）

## 注意事项

1. **小文件不需要 LFS**：一般小于 100MB 的文件不必使用
2. **团队都需要安装 LFS**：否则无法正常克隆
3. **免费额度有限**：注意存储和带宽限制
4. **迁移要谨慎**：修改历史可能影响协作

## 下一步

[安全与权限管理 →](28-security-permissions.md)
