# 查看历史与差异：Git 日志与差异对比完全指南

> **本章目标**：掌握 Git 中查看提交历史、对比代码差异、追溯代码来源的各种命令和技巧，成为代码考古专家。

## 为什么需要查看历史？

在实际开发中，我们经常需要回答以下问题：

- 这个 bug 是什么时候引入的？
- 谁修改了这行代码？为什么要修改？
- 这个分支和主分支有什么差异？
- 最近一周有哪些提交？
- 某个文件的修改历史是什么？

Git 提供了强大的历史查看和差异对比工具，帮助我们回答这些问题。

---

## 一、git log 详解

`git log` 是查看提交历史的核心命令，它显示从最新提交到最早提交的所有记录。

### 1.1 基本用法

```bash
# 查看完整提交历史（按时间倒序）
git log

# 查看最近 N 次提交
git log -5
git log -n 10

# 查看某个文件的历史
git log -- path/to/file.txt
git log -- src/main.py
```

默认的 `git log` 输出包含以下信息：

```
commit 7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e4f5a6b
Author: 张三 <zhangsan@example.com>
Date:   Mon Sep 11 10:30:00 2026 +0800

    修复用户登录验证的 bug

    详细描述：当用户密码包含特殊字符时，验证逻辑会出现错误。
    修复方法：对密码进行 URL 编码处理。

commit 1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b
Author: 李四 <lisi@example.com>
Date:   Sun Sep 10 15:20:00 2026 +0800

    添加用户注册功能
```

### 1.2 单行简洁模式

```bash
# 单行显示，每个提交一行
git log --oneline

# 输出示例：
# 7a8b9c0 修复用户登录验证的 bug
# 1a2b3c4 添加用户注册功能
# 5e6f7a8 初始提交
```

`--oneline` 将每个提交压缩为一行，只显示提交哈希的前几位和提交信息，非常适合快速浏览历史。

### 1.3 图形化分支显示

```bash
# 图形化显示分支合并历史
git log --oneline --graph

# 图形化显示所有分支
git log --oneline --graph --all

# 输出示例：
# * 7a8b9c0 (HEAD -> main) 合并 feature 分支
# |\
# | * 3c4d5e6 (feature) 添加新功能
# | * 1a2b3c4 初始化功能模块
# |/
# * 5e6f7a8 初始提交
```

`--graph` 选项用 ASCII 字符绘制分支拓扑图，`--all` 显示所有分支的提交历史。这对于理解分支合并关系非常有帮助。

### 1.4 显示每次提交的详细差异

```bash
# 显示每次提交的具体修改内容
git log -p
git log --patch

# 显示最近 3 次提交的差异
git log -p -3

# 只显示某个文件的修改历史及差异
git log -p -- src/app.js
```

`-p`（patch）选项会在每个提交后显示具体的代码变更，包括修改前后的对比。

### 1.5 显示文件变更统计

```bash
# 显示每次提交修改了哪些文件及统计信息
git log --stat

# 输出示例：
# commit 7a8b9c0
# Author: 张三 <zhangsan@example.com>
# Date:   Mon Sep 11 10:30:00 2026 +0800
#
#     修复用户登录验证的 bug
#
#  src/auth/login.js    | 15 +++++++---
#  src/auth/validate.js |  8 ++++--
#  tests/auth.test.js   | 12 ++++++++
#  3 files changed, 28 insertions(+), 7 deletions(-)
```

### 1.6 常用选项速查表

| 选项 | 说明 | 示例 |
|------|------|------|
| `-n <数量>` | 限制显示的提交数量 | `git log -5` |
| `--oneline` | 单行简洁显示 | `git log --oneline` |
| `--graph` | 图形化显示分支 | `git log --graph` |
| `--all` | 显示所有分支 | `git log --all` |
| `-p` | 显示具体差异 | `git log -p` |
| `--stat` | 显示文件统计 | `git log --stat` |
| `--name-only` | 只显示文件名 | `git log --name-only` |
| `--name-status` | 显示文件名和状态 | `git log --name-status` |
| `--shortstat` | 显示简短统计 | `git log --shortstat` |
| `--no-merges` | 排除合并提交 | `git log --no-merges` |
| `--first-parent` | 只跟随第一个父提交 | `git log --first-parent` |

---

## 二、git log 高级过滤

当项目历史很长时，我们需要使用过滤选项来缩小搜索范围。

### 2.1 按作者过滤

```bash
# 查看某个作者的提交
git log --author="张三"

# 支持正则表达式
git log --author="张\|李"
git log --author="zhangsan"

# 查看邮箱包含特定字符串的作者
git log --author="@example.com"
```

### 2.2 按日期过滤

```bash
# 查看某个日期之后的提交
git log --after="2026-01-01"
git log --since="2026-01-01"

# 查看某个日期之前的提交
git log --before="2026-12-31"
git log --until="2026-12-31"

# 组合使用：查看某个时间段的提交
git log --after="2026-01-01" --before="2026-06-30"

# 支持相对日期
git log --since="2 weeks ago"
git log --since="1 month ago"
git log --since="3 days ago"
git log --since="yesterday"

# 查看今天的提交
git log --since="today"
```

### 2.3 按提交信息过滤

```bash
# 搜索提交信息中包含特定关键词的提交
git log --grep="fix"
git log --grep="修复"
git log --grep="feature"

# 使用正则表达式
git log --grep="fix\|bug"
git log --grep="^feat"

# 不区分大小写
git log --grep="fix" -i

# 排除特定提交信息
git log --grep="merge" --invert-grep
```

### 2.4 按代码变更内容搜索

```bash
# 查找添加或删除了特定字符串的提交
git log -S "functionName"
git log -S "某个变量名"

# 使用正则表达式搜索代码变更
git log -G "pattern"
git log -G "def\s+\w+"

# 区别：-S 搜索字符串的出现次数变化，-G 搜索匹配正则的行
```

`-S` 和 `-G` 的区别：

| 选项 | 说明 | 适用场景 |
|------|------|----------|
| `-S "字符串"` | 搜索字符串出现次数变化的提交 | 查找某个函数/变量何时被引入或删除 |
| `-G "正则"` | 搜索匹配正则表达式的代码变更 | 复杂模式匹配 |

```bash
# 实用示例：查找引入了安全漏洞的提交
git log -S "eval(" --oneline

# 查找修改了数据库连接的提交
git log -G "mysql_connect\|PDO" --oneline

# 查找添加了 TODO 注释的提交
git log -S "TODO" --oneline
```

### 2.5 按路径过滤

```bash
# 查看特定文件的提交历史
git log -- path/to/file.txt

# 查看特定目录的提交历史
git log -- src/components/

# 查看特定类型文件的历史
git log -- "*.js"
git log -- "*.py"
```

### 2.6 过滤合并提交

```bash
# 排除合并提交
git log --no-merges

# 只显示合并提交
git log --merges

# 只跟随第一个父提交（通常是主分支）
git log --first-parent
```

### 2.7 组合过滤示例

```bash
# 查看张三在最近一个月内修复 bug 的提交
git log --author="张三" --since="1 month ago" --grep="fix" --oneline

# 查看最近一周修改了 src/ 目录的提交
git log --since="1 week ago" -- src/

# 查看某个文件的所有修改历史（不包括合并提交）
git log --no-merges -- src/app.js

# 查看引入了特定函数的提交及差异
git log -S "handleSubmit" -p --oneline
```

---

## 三、git log 输出格式定制

### 3.1 使用 --format 自定义输出

`git log --format` 允许你完全控制输出格式：

```bash
# 使用预定义的简短格式
git log --format=short
git log --format=medium
git log --format=full
git log --format=fuller

# 使用自定义格式
git log --format="%h - %an, %ar : %s"
```

### 3.2 常用格式占位符

| 占位符 | 说明 |
|--------|------|
| `%H` | 完整提交哈希 |
| `%h` | 简短提交哈希 |
| `%T` | 完整树哈希 |
| `%t` | 简短树哈希 |
| `%P` | 完整父提交哈希 |
| `%p` | 简短父提交哈希 |
| `%an` | 作者名字 |
| `%ae` | 作者邮箱 |
| `%ad` | 作者日期 |
| `%ar` | 作者日期（相对格式） |
| `%cn` | 提交者名字 |
| `%ce` | 提交者邮箱 |
| `%cd` | 提交者日期 |
| `%cr` | 提交者日期（相对格式） |
| `%s` | 提交信息标题 |
| `%b` | 提交信息正文 |
| `%d` | 引用名称（分支、标签） |

### 3.3 自定义格式示例

```bash
# 一行显示：哈希 | 作者 | 日期 | 信息
git log --format="%h | %an | %ad | %s" --date=short

# 输出示例：
# 7a8b9c0 | 张三 | 2026-09-11 | 修复用户登录验证的 bug
# 1a2b3c4 | 李四 | 2026-09-10 | 添加用户注册功能

# 带颜色输出
git log --format="%C(yellow)%h%C(reset) %C(blue)%an%C(reset) %C(green)%ad%C(reset) %s" --date=short

# 显示分支和标签信息
git log --format="%h %d %s"

# 类似 changelog 的格式
git log --format="* %s (%an, %ad)" --date=short

# 适合导出的格式
git log --format="%H,%an,%ae,%ad,%s" --date=iso > commits.csv
```

### 3.4 预定义的 Pretty Formats

```bash
# oneline - 每个提交一行
git log --pretty=oneline

# short - 简短格式
git log --pretty=short

# medium - 中等格式（默认）
git log --pretty=medium

# full - 完整格式
git log --pretty=full

# fuller - 更完整的格式（显示作者和提交者日期）
git log --pretty=fuller

# email - 邮件格式
git log --pretty=email

# raw - 原始格式
git log --pretty=raw

# reference - 引用格式
git log --pretty=reference
```

### 3.5 配置默认格式

```bash
# 设置全局默认格式
git config --global format.pretty "%h %s (%an, %ar)"

# 设置特定仓库的默认格式
git config format.pretty "%h %s (%an, %ar)"

# 使用自定义别名
git config --global alias.lg "log --format='%h | %an | %ar | %s' --graph"
```

---

## 四、git diff 详解

`git diff` 是查看文件差异的核心命令，它比较不同版本之间的文件内容。

### 4.1 工作区 vs 暂存区

```bash
# 查看工作区和暂存区的差异（未暂存的修改）
git diff

# 查看特定文件的差异
git diff -- path/to/file.txt

# 查看所有未暂存的修改
git diff -- .
```

### 4.2 暂存区 vs 仓库

```bash
# 查看暂存区和最新提交的差异（已暂存但未提交的修改）
git diff --staged
git diff --cached

# 查看特定文件的暂存差异
git diff --staged -- path/to/file.txt
```

### 4.3 工作区 vs 仓库

```bash
# 查看工作区和最新提交的差异（所有修改）
git diff HEAD

# 查看工作区和某个特定提交的差异
git diff commit-hash
git diff 7a8b9c0
```

### 4.4 两个提交之间的差异

```bash
# 查看两个提交之间的差异
git diff commit1 commit2
git diff 7a8b9c0 1a2b3c4

# 查看某个提交和其父提交的差异
git diff commit^ commit
git diff 7a8b9c0^ 7a8b9c0

# 查看某个提交引入的变更
git diff commit~1 commit
```

### 4.5 理解差异输出格式

```bash
# 差异输出示例：
diff --git a/src/app.js b/src/app.js
index 1234567..abcdefg 100644
--- a/src/app.js
+++ b/src/app.js
@@ -10,7 +10,9 @@ function App() {
   const [count, setCount] = useState(0);
 
-  const handleClick = () => {
+  const handleClick = (event) => {
+    event.preventDefault();
     setCount(count + 1);
   };
```

输出格式说明：

| 部分 | 说明 |
|------|------|
| `diff --git a/... b/...` | 表示正在比较的文件 |
| `--- a/...` | 旧版本（通常是暂存区或基准） |
| `+++ b/...` | 新版本（通常是工作区或目标） |
| `@@ -10,7 +10,9 @@` | 变更位置：从第10行开始的7行变为9行 |
| `-` 开头的行 | 被删除的行 |
| `+` 开头的行 | 新增的行 |
| 无前缀的行 | 未变更的上下文行 |

### 4.6 diff 常用选项速查表

| 选项 | 说明 |
|------|------|
| `--staged` / `--cached` | 比较暂存区与仓库 |
| `--stat` | 只显示统计信息 |
| `--name-only` | 只显示文件名 |
| `--name-status` | 显示文件名和状态（A/M/D） |
| `--shortstat` | 显示简短统计 |
| `--diff-filter=[ACDMRTUXB]` | 过滤特定类型的变更 |
| `--no-index` | 比较两个不在仓库中的文件 |
| `-w` / `--ignore-all-space` | 忽略所有空白字符 |
| `-b` / `--ignore-space-change` | 忽略空白字符数量变化 |
| `--ignore-blank-lines` | 忽略空白行 |
| `--unified=<N>` | 显示 N 行上下文 |

---

## 五、git diff 分支对比

### 5.1 两个分支之间的差异

```bash
# 查看两个分支之间的差异
git diff main..feature
git diff main feature

# 查看分支和主分支的差异
git diff main..HEAD

# 输出示例：
# diff --git a/src/feature.js b/src/feature.js
# new file mode 100644
# index 0000000..1234567
# --- /dev/null
# +++ b/src/feature.js
# @@ -0,0 +1,15 @@
# +export function newFeature() {
# +  // 新功能实现
# +}
```

### 5.2 分支差异统计

```bash
# 查看分支差异统计
git diff --stat main..feature

# 输出示例：
#  src/feature.js  | 15 +++++++++++++++
#  src/app.js      |  5 +++--
#  tests/feature.test.js | 20 ++++++++++++++++++++
#  3 files changed, 38 insertions(+), 2 deletions(-)
```

### 5.3 双点语法与三点语法

```bash
# 双点语法：比较两个分支的最新提交
git diff main..feature

# 三点语法：比较 feature 分支相对于共同祖先的变更
git diff main...feature

# 区别：
# main..feature - main 有而 feature 没有的，加上 feature 有而 main 没有的
# main...feature - 只显示 feature 分支相对于共同祖先的变更
```

### 5.4 对比特定提交

```bash
# 对比当前分支和某个提交
git diff HEAD 7a8b9c0

# 对比某个标签和当前分支
git diff v1.0.0..main

# 对比远程分支和本地分支
git diff origin/main..main
```

---

## 六、git diff 高级用法

### 6.1 使用 --stat 查看变更统计

```bash
# 只显示统计信息，不显示具体差异
git diff --stat

# 输出示例：
#  src/app.js       | 12 ++++++------
#  src/utils.js     |  5 ++++-
#  package.json     |  2 +-
#  3 files changed, 11 insertions(+), 8 deletions(-)

# 显示更详细的统计
git diff --numstat

# 输出示例（添加行数 | 删除行数 | 文件名）：
# 6    6    src/app.js
# 4    1    src/utils.js
# 1    1    package.json
```

### 6.2 使用 --word-diff 按词对比

```bash
# 按单词显示差异（适合文本文件）
git diff --word-diff

# 输出示例：
# 这是一个 [-旧的-]{+新的+} 示例文本

# 只显示变更的单词
git diff --word-diff=porcelain

# 按颜色高亮显示单词差异
git diff --color-words
```

### 6.3 使用 --color-words 高亮单词差异

```bash
# 按颜色高亮显示单词级别的差异
git diff --color-words

# 这比默认的行级差异更精细，适合：
# - 文档文件
# - 配置文件
# - 少量修改的代码行
```

### 6.4 比较二进制文件

```bash
# 显示二进制文件差异
git diff --binary

# 显示二进制文件统计
git diff --stat --binary
```

### 6.5 忽略空白字符差异

```bash
# 忽略所有空白字符差异
git diff -w
git diff --ignore-all-space

# 忽略空白字符数量变化
git diff -b
git diff --ignore-space-change

# 忽略空白行
git diff --ignore-blank-lines

# 忽略特定行的空白（使用正则）
git diff --ignore-matching-lines=<正则>
```

### 6.6 使用外部 diff 工具

```bash
# 使用外部 diff 工具（如 vimdiff）
git diff --tool=vimdiff

# 配置默认 diff 工具
git config --global diff.tool vimdiff

# 使用 meld、kdiff3、beyond compare 等
git config --global diff.tool meld
```

---

## 七、git show 详解

`git show` 用于查看特定提交的详细信息，包括提交元数据和代码变更。

### 7.1 查看提交详情

```bash
# 查看最新提交
git show

# 查看特定提交
git show commit-hash
git show 7a8b9c0

# 输出包含：
# - 提交哈希
# - 作者信息
# - 提交日期
# - 提交信息
# - 具体代码变更
```

### 7.2 查看提交的文件列表

```bash
# 查看提交修改了哪些文件
git show --stat commit-hash

# 只显示文件名
git show --name-only commit-hash

# 显示文件名和状态
git show --name-status commit-hash

# 输出示例：
# commit 7a8b9c0
# Author: 张三 <zhangsan@example.com>
# Date:   Mon Sep 11 10:30:00 2026 +0800
#
#     修复用户登录验证的 bug
#
# M       src/auth/login.js
# M       src/auth/validate.js
# A       tests/auth.test.js
```

### 7.3 查看特定提交中的特定文件

```bash
# 查看某个提交中特定文件的变更
git show commit-hash: path/to/file.txt
git show 7a8b9c0: src/app.js

# 只查看文件内容（不显示差异）
git show commit-hash: path/to/file.txt
```

### 7.4 查看标签信息

```bash
# 查看标签的详细信息
git show v1.0.0

# 轻量标签：显示关联的提交
# 附注标签：显示标签信息和关联的提交
```

### 7.5 查看树对象

```bash
# 查看某个提交的目录树
git show commit-hash^{tree}

# 查看根目录树
git show HEAD^{tree}
```

---

## 八、git blame 详解（逐行追溯）

`git blame` 用于查看文件每一行的最后修改者和修改时间，是追溯代码来源的利器。

### 8.1 基本用法

```bash
# 查看文件每一行的修改信息
git blame path/to/file.txt

# 输出示例：
# 7a8b9c0 (张三 2026-09-11 10:30:00 +0800  1) import React from 'react';
# 7a8b9c0 (张三 2026-09-11 10:30:00 +0800  2) import { useState } from 'react';
# 1a2b3c4 (李四 2026-09-10 15:20:00 +0800  3) 
# 1a2b3c4 (李四 2026-09-10 15:20:00 +0800  4) function App() {
# 3c4d5e6 (王五 2026-09-09 09:15:00 +0800  5)   const [count, setCount] = useState(0);
# 3c4d5e6 (王五 2026-09-09 09:15:00 +0800  6) 
# 1a2b3c4 (李四 2026-09-10 15:20:00 +0800  7)   return (
# 1a2b3c4 (李四 2026-09-10 15:20:00 +0800  8)     <div>{count}</div>
# 1a2b3c4 (李四 2026-09-10 15:20:00 +0800  9)   );
# 1a2b3c4 (李四 2026-09-10 15:20:00 +0800 10) }
```

### 8.2 常用选项

```bash
# 显示简短的提交哈希
git blame -s path/to/file.txt

# 显示邮箱而不是名字
git blame -e path/to/file.txt

# 显示行号
git blame -n path/to/file.txt

# 显示日期
git blame -d path/to/file.txt

# 组合使用
git blame -s -n -d path/to/file.txt
```

### 8.3 追溯特定行范围

```bash
# 只查看第 10-20 行的修改信息
git blame -L 10,20 path/to/file.txt

# 从第 10 行开始显示 5 行
git blame -L 10,+5 path/to/file.txt

# 查看包含特定函数的行
git blame -L :functionName path/to/file.txt
```

### 8.4 忽略特定提交

```bash
# 忽略某个提交的变更（通常是格式化提交）
git blame --ignore-rev 7a8b9c0 path/to/file.txt

# 忽略多个提交
git blame --ignore-rev 7a8b9c0 --ignore-rev 1a2b3c4 path/to/file.txt

# 从文件读取要忽略的提交列表
git blame --ignore-revs-file .git-blame-ignore-revs path/to/file.txt
```

### 8.5 跨文件追踪移动的代码

```bash
# 追踪从其他文件移动过来的代码
git blame -C path/to/file.txt

# 更强的追踪（追踪跨文件的代码移动）
git blame -CC path/to/file.txt

# 最强的追踪
git blame -CCC path/to/file.txt
```

### 8.6 实用技巧

```bash
# 使用 GUI 工具查看 blame
git gui blame path/to/file.txt

# 在 VS Code 中查看 blame（需要 GitLens 扩展）

# 查看某个提交的所有变更
git show $(git blame -L 10,10 -s path/to/file.txt | cut -d' ' -f1)
```

---

## 九、git shortlog 统计

`git shortlog` 用于生成提交统计摘要，常用于生成贡献者列表。

### 9.1 基本用法

```bash
# 按作者统计提交数量
git shortlog

# 输出示例：
# 张三 (5):
#       修复用户登录验证的 bug
#       添加用户注册功能
#       更新文档
#       修复样式问题
#       初始提交
#
# 李四 (3):
#       添加新功能
#       优化性能
#       修复配置问题
```

### 9.2 常用选项

```bash
# 只显示提交数量和作者名字
git shortlog -sn

# 输出示例：
#     5 张三
#     3 李四

# 按提交数量排序
git shortlog -n

# 显示邮箱
git shortlog -se

# 显示百分比
git shortlog -sn --no-merges

# 统计特定时间范围
git shortlog --since="2026-01-01" --until="2026-12-31"

# 统计特定分支
git shortlog main

# 统计特定文件的贡献者
git shortlog -- src/
```

### 9.3 生成贡献者列表

```bash
# 生成不包含合并提交的贡献者列表
git shortlog -sn --no-merges

# 输出适合添加到 README 中的格式
git shortlog -sn --no-merges | while read count name; do
  echo "- $name ($count commits)"
done

# 生成 changelog 格式
git shortlog --format="%s" --no-merges v1.0.0..v2.0.0
```

---

## 十、git reflog 引用日志

`git reflog` 记录了 HEAD 和分支引用的所有变更历史，包括已经被删除的提交。

### 10.1 基本用法

```bash
# 查看 HEAD 的引用日志
git reflog

# 输出示例：
# 7a8b9c0 HEAD@{0}: commit: 修复用户登录验证的 bug
# 1a2b3c4 HEAD@{1}: commit: 添加用户注册功能
# 5e6f7a8 HEAD@{2}: checkout: moving from feature to main
# 3c4d5e6 HEAD@{3}: commit: 添加新功能
# 9a0b1c2 HEAD@{4}: reset: moving to HEAD~1
# 7a8b9c0 HEAD@{5}: commit: 初始提交

# 查看特定分支的引用日志
git reflog main
git reflog feature

# 查看简短格式
git reflog --oneline
```

### 10.2 引用日志的用途

```bash
# 恢复误删的提交
git reset --hard HEAD@{2}

# 恢复误删的分支
git checkout -b recovered-branch HEAD@{3}

# 恢复误执行的 reset
git reset --hard HEAD@{1}

# 查看某个引用的历史位置
git reflog show main

# 查看特定时间的引用状态
git show main@{2026-09-10}
git show main@{yesterday}
git show main@{2.weeks.ago}
```

### 10.3 引用日志的限制

```bash
# 引用日志只在本地有效
# 引用日志有时间限制（默认90天）
# 引用日志不包含其他仓库的操作

# 配置引用日志保留时间
git config gc.reflogExpire "90 days"
git config gc.reflogExpireUnreachable "30 days"
```

### 10.4 使用 reflog 恢复数据

```bash
# 场景1：误删了分支
git branch -D important-feature
# 恢复：
git checkout -b important-feature HEAD@{1}

# 场景2：误执行了 reset --hard
git reset --hard HEAD~3
# 恢复：
git reset --hard HEAD@{1}

# 场景3：误执行了 rebase
git reflog
# 找到 rebase 前的提交，然后 reset
git reset --hard HEAD@{5}
```

---

## 十一、git bisect 二分查找

`git bisect` 使用二分查找算法帮助你找到引入 bug 的提交，当项目有大量提交时特别有用。

### 11.1 基本用法

```bash
# 开始二分查找
git bisect start

# 标记当前版本有 bug
git bisect bad

# 标记某个版本没有 bug
git bisect good v1.0.0

# Git 会自动切换到中间的提交，让你测试
# 测试后，标记结果：
git bisect good  # 这个版本没有 bug
# 或
git bisect bad   # 这个版本有 bug

# 重复上述过程，直到找到引入 bug 的提交
# 最终输出：
# 7a8b9c0 is the first bad commit

# 结束二分查找，回到原始状态
git bisect reset
```

### 11.2 自动化二分查找

```bash
# 使用脚本自动标记（返回0表示good，非0表示bad）
git bisect start HEAD v1.0.0
git bisect run ./test-script.sh

# 脚本示例 (test-script.sh)：
# #!/bin/bash
# npm test
# 如果测试通过，返回0（good）
# 如果测试失败，返回1（bad）

# 使用 make 命令
git bisect run make test

# 使用特定测试命令
git bisect run npm run test:unit
```

### 11.3 跳过特定提交

```bash
# 如果某个提交无法测试（如编译错误），可以跳过
git bisect skip

# 跳过多个提交
git bisect skip HEAD~3..HEAD
```

### 11.4 实用示例

```bash
# 完整的二分查找示例：

# 1. 开始查找
git bisect start

# 2. 标记当前版本有 bug
git bisect bad

# 3. 标记一周前的版本没有 bug
git bisect good HEAD@{1.week.ago}

# 4. Git 切换到中间提交，运行测试
npm test

# 5. 根据测试结果标记
git bisect good  # 或 git bisect bad

# 6. 重复步骤 4-5，直到找到 bug

# 7. 修复 bug
git bisect reset

# 8. 应用修复
git cherry-pick <bug-fix-commit>
```

### 11.5 bisect 的优势

| 特性 | 说明 |
|------|------|
| 效率 | 对数时间复杂度，1000个提交只需约10次测试 |
| 精确 | 定位到具体的提交 |
| 可自动化 | 可以用脚本自动测试 |
| 安全 | 不会修改代码历史 |

---

## 十二、实用别名推荐

配置 Git 别名可以大大提高工作效率。以下是一些推荐的别名配置：

### 12.1 日志相关别名

```bash
# 美化的日志
git config --global alias.lg "log --format='%h | %an | %ar | %s' --graph --all"

# 简洁的日志
git config --global alias.lgo "log --oneline --graph --all"

# 带统计的日志
git config --global alias.ls "log --stat --oneline"

# 最近的提交
git config --global alias.last "log -1 HEAD --stat"

# 今天的提交
git config --global alias.today "log --since=today --oneline"

# 本周的提交
git config --global alias.week "log --since=1.week.ago --oneline"
```

### 12.2 差异相关别名

```bash
# 简洁的 diff
git config --global alias.d "diff"

# 暂存区差异
git config --global alias.ds "diff --staged"

# 统计差异
git config --global alias.dst "diff --stat"

# 单词差异
git config --global alias.dw "diff --color-words"
```

### 12.3 状态相关别名

```bash
# 简洁的状态
git config --global alias.s "status -sb"

# 详细的状态
git config --global alias.st "status"
```

### 12.4 提交相关别名

```bash
# 快速提交
git config --global alias.cm "commit -m"

# 修改上一次提交
git config --global alias.amend "commit --amend --no-edit"

# 暂存所有并提交
git config --global alias.ac "!git add -A && git commit -m"
```

### 12.5 分支相关别名

```bash
# 列出所有分支
git config --global alias.br "branch -a"

# 切换分支
git config --global alias.co "checkout"

# 创建并切换分支
git config --global alias.cb "checkout -b"

# 删除分支
git config --global alias.bd "branch -d"
```

### 12.6 高级别名

```bash
# 显示仓库信息
git config --global alias.info "!echo 'Remote:' && git remote -v && echo && echo 'Branch:' && git branch -vv && echo && echo 'Status:' && git status -sb"

# 清理合并的分支
git config --global alias.cleanup "!git branch --merged | grep -v '\*\|main\|master' | xargs -n 1 git branch -d"

# 显示某个文件的提交历史
git config --global alias.filelog "log --oneline --follow --"

# 搜索提交
git config --global alias.search "log --grep"

# 搜索代码
git config --global alias.grep "log -S"
```

### 12.7 配置文件示例

在 `~/.gitconfig` 文件中添加：

```ini
[alias]
    lg = log --format='%h | %an | %ar | %s' --graph --all
    lgo = log --oneline --graph --all
    ls = log --stat --oneline
    last = log -1 HEAD --stat
    today = log --since=today --oneline
    d = diff
    ds = diff --staged
    dst = diff --stat
    s = status -sb
    cm = commit -m
    amend = commit --amend --no-edit
    ac = !git add -A && git commit -m
    br = branch -a
    co = checkout
    cb = checkout -b
    bd = branch -d
```

---

## 十三、GUI 工具推荐

对于不习惯命令行的开发者，有许多优秀的 Git GUI 工具可供选择。

### 13.1 gitk（Git 自带）

```bash
# 启动 gitk
gitk

# 查看特定文件的历史
gitk path/to/file.txt

# 查看所有分支
gitk --all

# 查看特定提交
gitk commit-hash
```

**特点**：
- Git 自带，无需安装
- 轻量级，启动快速
- 显示提交历史和分支图
- 支持搜索和过滤

### 13.2 SourceTree

**特点**：
- 免费，由 Atlassian 开发
- 界面美观，功能强大
- 支持 Git 和 Mercurial
- 可视化分支管理
- 支持交互式 rebase
- 内置 merge tool

**下载地址**：https://www.sourcetreeapp.com/

**主要功能**：
- 提交历史可视化
- 分支管理
- 暂存区管理
- 冲突解决
- 子模块管理
- Git Flow 支持

### 13.3 VS Code GitLens

**安装**：
1. 打开 VS Code
2. 搜索扩展 "GitLens"
3. 安装 GitLens — Git supercharged

**主要功能**：

| 功能 | 说明 |
|------|------|
| Line Blame | 在代码行末尾显示最后修改者和时间 |
| File Blame | 在文件顶部显示整个文件的 blame 信息 |
| Commit Search | 搜索提交历史 |
| Side by Side Diff | 并排显示文件差异 |
| Inline Diff | 在编辑器内显示差异 |
| Git Graph | 可视化分支图 |
| Revision Navigation | 在文件的不同版本间导航 |

**常用快捷键**：
- `Ctrl+Shift+G`：打开 Git 面板
- `Ctrl+Shift+P` → "GitLens"：打开 GitLens 命令面板
- 悬停在代码行上查看 blame 信息
- 点击状态栏查看当前分支信息

### 13.4 其他推荐工具

| 工具 | 平台 | 特点 |
|------|------|------|
| **GitKraken** | 跨平台 | 界面美观，功能强大，有免费版 |
| **Tower** | macOS/Windows | 专业级 Git 客户端 |
| **Fork** | macOS/Windows | 快速，支持交互式 rebase |
| **GitHub Desktop** | 跨平台 | GitHub 官方客户端，简单易用 |
| **Sublime Merge** | 跨平台 | Sublime Text 团队开发，速度快 |
| **Lazygit** | 终端 | 终端 UI，适合命令行爱好者 |

### 13.5 选择建议

- **初学者**：GitHub Desktop 或 SourceTree
- **VS Code 用户**：GitLens 扩展
- **命令行爱好者**：Lazygit
- **专业开发者**：GitKraken 或 Tower
- **轻量需求**：gitk（Git 自带）

---

## 总结

本章介绍了 Git 中查看历史和差异的核心命令：

| 命令 | 用途 |
|------|------|
| `git log` | 查看提交历史 |
| `git diff` | 查看文件差异 |
| `git show` | 查看特定提交详情 |
| `git blame` | 逐行追溯代码来源 |
| `git shortlog` | 生成提交统计 |
| `git reflog` | 查看引用日志 |
| `git bisect` | 二分查找引入 bug 的提交 |

掌握这些命令，你就能成为代码考古专家，轻松回答"谁、什么时候、为什么修改了代码"这类问题。

---

## 下一步

[分支操作 →](10-branching.md)
