# Git 暂存与提交：从入门到精通

在 Git 的工作流程中，暂存（staging）和提交（committing）是最核心的两个操作。理解这两个概念以及掌握相关命令的使用方法，是成为一名高效 Git 用户的基础。本章将深入讲解 Git 暂存区的工作原理、各种暂存和提交命令的用法，以及提交信息的最佳实践。

---

## 暂存区（Staging Area）概念详解

### Git 的三棵树模型

要理解暂存区，首先需要了解 Git 的三棵树（Three Trees）模型。Git 管理的项目中存在三个核心区域：

**工作目录（Working Directory）**
工作目录是你在电脑上实际看到和编辑的文件所在的目录。当你使用文本编辑器打开文件并进行修改时，这些修改首先存在于工作目录中。工作目录中的文件可能处于已跟踪状态（已被 Git 管理）或未跟踪状态（新创建的、尚未被 Git 管理的文件）。

**暂存区（Staging Area / Index）**
暂存区是一个中间区域，用于准备下一次提交的内容。它本质上是一个文件（通常位于 `.git/index`），记录了下一次提交将要包含的文件和更改。暂存区的存在是 Git 与许多其他版本控制系统的重要区别之一，它允许你精确控制每次提交的内容。

**仓库（Repository / HEAD）**
仓库是 Git 存储所有提交历史的地方。HEAD 指针指向当前分支的最新提交。当你执行 `git commit` 时，暂存区的内容会被永久记录为一个新的提交对象。

### 暂存区的工作流程

文件在 Git 中的状态转换遵循以下流程：

```
工作目录 --[git add]--> 暂存区 --[git commit]--> 仓库
   ^                                                |
   |_____________[git checkout / git reset]__________|
```

具体来说：

1. **修改文件**：在工作目录中编辑文件，文件变为"已修改"（modified）状态
2. **暂存更改**：使用 `git add` 将修改添加到暂存区，文件变为"已暂存"（staged）状态
3. **提交更改**：使用 `git commit` 将暂存区的内容提交到仓库，文件变为"已提交"（committed）状态
4. **查看状态**：使用 `git status` 可以随时查看文件在各个区域中的状态

### 为什么需要暂存区

暂存区的存在提供了以下优势：

**精确控制提交内容**：你可以选择性地暂存文件的部分更改，而不是将所有修改一次性提交。这对于将相关更改组织到同一提交中非常有用。

**提交前的审查机会**：暂存区作为一个缓冲区，让你有机会在提交前检查即将被提交的内容，避免意外提交不必要的更改。

**支持部分提交**：当一个文件中包含多个不相关的修改时，可以使用交互式暂存只提交其中的一部分。

**高效的性能**：暂存区的文件格式经过优化，Git 可以快速比较工作目录和暂存区的差异，提高状态检查的效率。

### 查看暂存区状态

使用 `git status` 命令可以查看当前暂存区的状态：

```bash
git status
```

输出示例：

```
On branch main
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   src/app.js
        new file:   src/utils.js

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
        modified:   README.md

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        docs/
```

输出中的三个部分分别表示：

- **Changes to be committed**：已暂存的更改，将在下次提交时被记录
- **Changes not staged for commit**：已修改但未暂存的更改
- **Untracked files**：未跟踪的新文件

---

## git add 详解（添加文件到暂存区）

### 基本用法

`git add` 命令用于将工作目录中的更改添加到暂存区。它的基本语法是：

```bash
git add <pathspec>
```

其中 `<pathspec>` 可以是文件路径、目录路径或通配符模式。

### 添加单个文件

```bash
# 添加指定文件
git add README.md

# 添加指定路径下的文件
git add src/app.js
```

### 添加多个文件

```bash
# 同时添加多个文件
git add file1.txt file2.txt file3.txt

# 添加多个路径下的文件
git add src/app.js src/utils.js README.md
```

### 添加目录

```bash
# 添加整个目录及其所有内容
git add src/

# 添加当前目录下的所有内容
git add .
```

### 使用通配符

```bash
# 添加所有 JavaScript 文件
git add *.js

# 添加 src 目录下所有 JavaScript 文件
git add src/*.js

# 使用 glob 模式匹配多级目录
git add "**/*.py"
```

### 添加已删除的文件

当文件被删除时，需要使用 `git add` 来暂存删除操作：

```bash
# 删除文件后暂存删除操作
git add deleted-file.txt

# 或者使用 git rm 直接删除并暂存
git rm file-to-delete.txt
```

---

## git add 的各种用法

### git add . （添加当前目录及子目录）

```bash
git add .
```

这是最常用的命令之一。它会添加当前目录及其所有子目录中的更改，包括新文件和已修改的文件。在 Git 2.x 中，`git add .` 的行为与 `git add -A` 在当前目录下基本一致。

**适用场景**：当你想要暂存当前项目中的所有更改时。

### git add -A 或 git add --all （添加所有更改）

```bash
git add -A
git add --all
```

该命令会添加整个工作目录中的所有更改，无论你当前在哪个子目录中执行命令。包括：
- 新文件（未跟踪的文件）
- 已修改的文件
- 已删除的文件

**与 `git add .` 的区别**：在仓库根目录下执行时，两者效果相同。但在子目录中执行时：
- `git add .` 只暂存当前目录及其子目录的更改
- `git add -A` 暂存整个仓库的所有更改

### git add -u 或 git add --update （只添加已跟踪文件的更改）

```bash
git add -u
git add --update
```

该命令只暂存已经被 Git 跟踪的文件的更改，不会添加新文件。包括：
- 已修改的已跟踪文件
- 已删除的已跟踪文件

**适用场景**：当你只想暂存已跟踪文件的更改，而不希望将新文件包含在提交中时。

### git add -p 或 git add --patch （交互式部分暂存）

```bash
git add -p
git add --patch
```

这是最强大的暂存选项之一。它允许你交互式地选择要暂存文件的哪些部分（称为"hunk"），实现精确的部分暂存。

**交互选项说明**：

```
Stage this hunk [y,n,q,a,d,s,e,?]?
```

- `y`：暂存当前 hunk
- `n`：跳过当前 hunk
- `q`：退出，不再暂存任何内容
- `a`：暂存当前 hunk 及后续所有 hunk
- `d`：跳过当前 hunk 及后续所有 hunk
- `s`：将当前 hunk 分割为更小的部分
- `e`：手动编辑当前 hunk
- `?`：显示帮助信息

**使用示例**：

假设文件中有以下修改：

```diff
@@ -1,5 +1,6 @@
 function calculateTotal(price, tax) {
-  return price + tax;
+  const total = price + tax;
+  console.log('Total:', total);
+  return total;
 }

 function formatCurrency(amount) {
```

使用 `git add -p` 时，Git 会询问你是否要暂存这个 hunk。如果你只想暂存 `const total = price + tax;` 和 `return total;` 两行，而不暂存 `console.log`，可以使用 `s` 选项分割 hunk，然后选择性地暂存。

### git add -i 或 git add --interactive （交互式暂存）

```bash
git add -i
git add --interactive
```

进入交互式暂存模式，提供一个菜单界面让你选择各种操作：

```
*** Commands ***
  1: status       2: update       3: revert       4: add untracked
  5: patch        6: diff         7: quit         8: help
```

各选项说明：

- **status**：显示当前暂存状态
- **update**：选择要暂存的已跟踪文件
- **revert**：取消暂存已暂存的文件
- **add untracked**：添加未跟踪的文件
- **patch**：交互式选择要暂存的代码块
- **diff**：查看暂存区与工作目录的差异
- **quit**：退出交互模式
- **help**：显示帮助信息

### git add -n 或 git add --dry-run （模拟运行）

```bash
git add -n
git add --dry-run
```

该命令不会实际暂存任何文件，只是显示哪些文件会被暂存。适用于在执行实际操作前进行确认。

### git add -v 或 git add --verbose （详细输出）

```bash
git add -v
git add --verbose
```

显示被添加的每个文件的详细信息，包括文件名和操作类型。

---

## 交互式暂存（Interactive Staging）

### 部分暂存文件内容

交互式暂存是 Git 最强大的功能之一，它允许你暂存文件的部分修改。这在以下场景中特别有用：

**场景一：一个文件中包含多个不相关的修改**

假设你在一个文件中同时修复了一个 bug 和添加了一个新功能：

```javascript
// 文件: src/app.js

// Bug 修复：修正计算逻辑
function calculateTotal(price, tax) {
-  return price + tax;
+  return price * (1 + tax);
 }

// 新功能：添加格式化功能
+function formatCurrency(amount) {
+  return `$${amount.toFixed(2)}`;
+}
```

使用交互式暂存，你可以只暂存 bug 修复部分：

```bash
git add -p src/app.js
```

Git 会显示每个 hunk 并询问你是否要暂存。选择暂存第一个 hunk（bug 修复），跳过第二个 hunk（新功能）。

**场景二：调试代码混入正式代码**

如果在开发过程中添加了调试代码（如 `console.log`），可以使用交互式暂存只提交正式代码，将调试代码留在工作目录中。

**场景三：重构过程中的部分提交**

在重构过程中，你可能想要分步骤提交更改。交互式暂存允许你将大的重构分解为多个小的、有意义的提交。

### 使用 git add -e 手动编辑

`git add -p` 的 `e` 选项允许你手动编辑要暂存的代码块。这在自动分割不够精确时非常有用：

```bash
git add -p
# 当提示 Stage this hunk [y,n,q,a,d,s,e,?]?
# 输入 e 进入手动编辑模式
```

Git 会打开你的默认编辑器，显示当前 hunk 的 diff 格式内容。你可以直接编辑这个 diff，删除不想暂存的行，然后保存退出。

编辑时需要注意：
- 以 `+` 开头的行表示新增的代码
- 以 `-` 开头的行表示删除的代码
- 以空格开头的行表示上下文
- 删除某行（包括行首的 `+` 或 `-`）会将其排除在暂存之外

### 交互式暂存的实际操作示例

让我们通过一个完整的示例来演示交互式暂存的工作流程：

```bash
# 1. 创建一个测试文件
echo "Line 1" > test.txt
git add test.txt
git commit -m "Initial commit"

# 2. 修改文件，添加多处更改
cat > test.txt << 'EOF'
Line 1 - Modified for feature A
Line 2 - New line for feature A
Line 3 - Modified for bug fix
Line 4 - New line for bug fix
EOF

# 3. 使用交互式暂存
git add -p test.txt
```

Git 会显示 diff 并询问：

```
diff --git a/test.txt b/test.txt
index 1234567..abcdefg 100644
--- a/test.txt
+++ b/test.txt
@@ -1 +1,4 @@
-Line 1
+Line 1 - Modified for feature A
+Line 2 - New line for feature A
+Line 3 - Modified for bug fix
+Line 4 - New line for bug fix
Stage this hunk [y,n,q,a,d,s,e,?]?
```

此时你可以输入 `s` 将其分割为更小的 hunk，然后分别选择暂存。

---

## git commit 详解（提交更改）

### 基本提交

`git commit` 命令用于将暂存区的内容记录为一个新的提交：

```bash
git commit
```

执行后，Git 会打开你的默认文本编辑器，让你输入提交信息。编辑器中会显示当前提交的摘要信息（包括暂存的文件列表和统计）。

### 直接指定提交信息

使用 `-m` 选项可以直接在命令行中指定提交信息：

```bash
git commit -m "Add user authentication feature"
```

### 多行提交信息

对于较复杂的提交，可以提供多行提交信息：

```bash
git commit -m "Add user authentication feature" -m "Implement JWT-based authentication with refresh token support"
```

或者使用 Here Document 语法：

```bash
git commit -m "Add user authentication feature

- Implement JWT-based authentication
- Add refresh token support
- Add login/logout endpoints
- Update user model with password hashing"
```

### 暂存并提交

使用 `-a` 选项可以跳过 `git add` 步骤，直接将所有已跟踪文件的更改暂存并提交：

```bash
git commit -a -m "Fix typo in README"
```

**注意**：`-a` 选项只会暂存已跟踪文件的更改，不会自动添加新文件。如果有新文件需要提交，仍然需要先使用 `git add`。

### 提交统计信息

使用 `--stat` 选项可以在提交信息后显示更改的统计信息：

```bash
git commit --stat -m "Update documentation"
```

输出示例：

```
 docs/README.md | 10 ++++++----
 docs/guide.md  |  5 +++++
 2 files changed, 11 insertions(+), 4 deletions(-)
```

### 空提交

使用 `--allow-empty` 选项可以创建一个没有任何更改的提交：

```bash
git commit --allow-empty -m "Trigger CI build"
```

**适用场景**：
- 触发持续集成（CI）构建
- 标记重要的里程碑
- 创建用于测试的提交

### 详细提交

使用 `-v` 或 `--verbose` 选项可以在编辑器中显示完整的 diff：

```bash
git commit -v
```

这在编写提交信息时特别有用，因为它让你能够同时查看具体的更改内容。

---

## 提交信息规范（Conventional Commits）

### 为什么需要提交信息规范

规范化的提交信息有以下好处：

1. **自动生成变更日志**：通过解析提交信息可以自动生成 CHANGELOG
2. **自动确定版本号**：根据提交类型可以自动判断应该发布哪个版本
3. **简化贡献流程**：统一的格式让团队成员更容易理解和遵循
4. **提高代码审查效率**：清晰的提交信息帮助审查者快速理解更改意图

### Conventional Commits 规范

Conventional Commits 是一个被广泛采用的提交信息规范。其基本格式如下：

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

**type（类型）**：说明提交的性质，常用类型包括：

- `feat`：新功能
- `fix`：修复 bug
- `docs`：文档更新
- `style`：代码格式调整（不影响代码逻辑）
- `refactor`：代码重构（既不是修复 bug 也不是添加功能）
- `perf`：性能优化
- `test`：添加或修改测试
- `build`：构建系统或外部依赖的更改
- `ci`：CI 配置和脚本的更改
- `chore`：其他不修改源代码或测试的更改
- `revert`：回滚之前的提交

**scope（范围）**：可选，说明更改影响的范围：

```
feat(auth): add JWT token refresh
fix(parser): handle null input correctly
docs(readme): update installation instructions
```

**description（描述）**：简短描述更改内容，使用祈使语气，首字母小写，不加句号。

**body（正文）**：可选，提供更详细的更改说明：

```
feat(auth): add JWT token refresh

Implement automatic token refresh mechanism to improve user experience.
The refresh token is stored securely in httpOnly cookies.

- Add refresh token endpoint
- Implement token rotation strategy
- Add automatic retry logic for expired tokens
```

**footer（页脚）**：可选，用于引用相关 issue 或标记破坏性更改：

```
feat(api): change user endpoint response format

BREAKING CHANGE: The user endpoint now returns data in a different format.
Migration guide: https://example.com/migration

Closes #123
Fixes #456
```

### 破坏性更改的表示

破坏性更改（Breaking Changes）有两种表示方式：

1. 在 type 后面加 `!`：

```
feat!: change authentication method
```

2. 在 footer 中使用 `BREAKING CHANGE:`：

```
feat: change authentication method

BREAKING CHANGE: OAuth 1.0 is no longer supported
```

### 提交信息示例

```
feat: add user registration endpoint

fix(auth): prevent token reuse after logout

docs: update API documentation for v2

refactor(database): optimize query performance

test: add unit tests for user service

chore(deps): update dependencies to latest versions

ci: add GitHub Actions workflow for automated testing

perf(api): implement response caching

feat(ui): add dark mode support

fix(ui): resolve button alignment issue on mobile
```

---

## 修改最后一次提交（--amend）

### 基本用法

`--amend` 选项允许你修改最近一次提交：

```bash
git commit --amend
```

这会打开编辑器，让你修改提交信息。如果暂存区有新的更改，它们也会被包含在修改后的提交中。

### 修改提交信息

如果你只是想修改提交信息，可以这样操作：

```bash
# 打开编辑器修改提交信息
git commit --amend

# 直接指定新的提交信息
git commit --amend -m "New commit message"
```

### 添加遗漏的文件

如果你在提交后发现遗漏了某些文件：

```bash
# 添加遗漏的文件
git add forgotten-file.js

# 将其添加到最后一次提交
git commit --amend --no-edit
```

`--no-edit` 选项表示保留原来的提交信息不变。

### 修改提交的文件

如果你想从最后一次提交中移除某个文件：

```bash
# 取消暂存某个文件
git restore --staged file-to-remove.js

# 修改提交
git commit --amend --no-edit
```

### 注意事项

**不要修改已推送的提交**：如果提交已经被推送到远程仓库，修改提交会改变提交的历史哈希值，导致与远程仓库不一致。在这种情况下，你需要使用 `git push --force` 来强制推送，但这可能会影响其他协作者。

**修改提交会创建新的提交对象**：`--amend` 实际上是创建了一个新的提交对象，原来的提交对象仍然存在于 Git 的对象数据库中，只是不再被引用。

---

## 空提交（--allow-empty）

### 创建空提交

```bash
git commit --allow-empty -m "Empty commit for testing"
```

空提交不包含任何文件更改，但它仍然会创建一个提交对象，包含提交信息、作者信息、时间戳和父提交引用。

### 使用场景

**触发 CI/CD 流水线**：

```bash
git commit --allow-empty -m "ci: trigger deployment pipeline"
```

当 CI 系统通过提交事件触发时，有时需要在不修改代码的情况下触发新的构建。

**标记重要里程碑**：

```bash
git commit --allow-empty -m "milestone: project kickoff"
git commit --allow-empty -m "milestone: alpha release complete"
```

**测试提交钩子**：

```bash
git commit --allow-empty -m "test: verify commit hook works"
```

### 与 --allow-empty-message 的区别

`--allow-empty` 允许没有更改的提交，而 `--allow-empty-message` 允许没有提交信息的提交：

```bash
# 允许空更改
git commit --allow-empty -m "Message"

# 允许空信息
git commit --allow-empty-message
```

通常不建议使用 `--allow-empty-message`，因为没有提交信息的提交会降低代码历史的可读性。

---

## 签名提交（GPG/SSH signing）

### 为什么要签名提交

签名提交可以验证提交确实是由你创建的，而不是由其他人伪造的。这在开源项目和企业环境中都非常重要：

1. **身份验证**：证明提交确实来自你
2. **完整性**：确保提交内容没有被篡改
3. **信任链**：建立可验证的提交历史

### 使用 GPG 签名

**第一步：安装 GPG**

```bash
# macOS
brew install gnupg

# Ubuntu/Debian
sudo apt-get install gnupg

# Windows（通过 Git Bash）
# GPG 通常已随 Git for Windows 安装
```

**第二步：生成 GPG 密钥**

```bash
gpg --full-generate-key
```

按照提示选择：
- 密钥类型：RSA and RSA
- 密钥大小：4096
- 有效期：根据需要选择（建议设置过期时间）
- 输入姓名和邮箱（应与 Git 配置的邮箱一致）

**第三步：获取密钥 ID**

```bash
gpg --list-secret-keys --keyid-format=long
```

输出示例：

```
sec   rsa4096/ABC123DEF456 2024-01-01 [SC]
      1234567890ABCDEF1234567890ABCDEF12345678
uid                 [ultimate] Your Name <your.email@example.com>
```

密钥 ID 是 `ABC123DEF456`（`/` 后面的部分）。

**第四步：配置 Git 使用 GPG 签名**

```bash
# 设置 GPG 签名密钥
git config --global user.signingkey ABC123DEF456

# 启用自动签名所有提交
git config --global commit.gpgsign true
```

**第五步：签名提交**

```bash
# 单次签名提交
git commit -S -m "Signed commit"

# 如果已启用自动签名
git commit -m "Regular commit with auto-signing"
```

### 使用 SSH 签名

Git 2.34+ 支持使用 SSH 密钥进行签名，这比 GPG 更简单：

**第一步：配置 Git 使用 SSH 签名**

```bash
git config --global gpg.format ssh
git config --global user.signingkey ~/.ssh/id_ed25519.pub
```

**第二步：签名提交**

```bash
git commit -S -m "SSH signed commit"
```

**第三步：在 GitHub 上添加 SSH 签名密钥**

1. 访问 GitHub Settings -> SSH and GPG keys
2. 点击 "New SSH key"
3. 选择类型为 "Signing Key"
4. 粘贴你的公钥

### 验证签名

```bash
# 验证最后一次提交的签名
git verify-commit HEAD

# 查看提交的签名信息
git log --show-signature -1
```

---

## 提交模板配置（commit.template）

### 创建提交模板

提交模板可以帮助团队成员遵循统一的提交信息格式。创建一个模板文件：

```bash
# 创建模板文件
cat > ~/.gitmessage << 'EOF'
# <type>[optional scope]: <description>
# 
# [optional body]
# 
# [optional footer(s)]
# 
# Type 标签:
#   feat:     新功能
#   fix:      修复 bug
#   docs:     文档更新
#   style:    代码格式（不影响功能）
#   refactor: 代码重构
#   perf:     性能优化
#   test:     测试相关
#   build:    构建系统
#   ci:       CI 配置
#   chore:    其他更改
#
# 示例:
# feat(auth): add login functionality
#
# 关联 Issue:
# Closes #123
EOF
```

### 配置 Git 使用模板

```bash
# 全局配置
git config --global commit.template ~/.gitmessage

# 仅对当前仓库配置
git config commit.template .gitmessage
```

### 使用模板

配置模板后，每次运行 `git commit`（不带 `-m` 选项）时，编辑器会自动显示模板内容。你只需要填写相应的信息即可。

### 项目级模板

你也可以为特定项目创建模板：

```bash
# 在项目根目录创建模板
cat > .gitmessage << 'EOF'
# 项目名称: MyProject
# 
# <type>(<scope>): <subject>
#
# 类型:
#   feat:     新功能
#   fix:      Bug 修复
#   docs:     文档
#   style:    格式
#   refactor: 重构
#   test:     测试
#   chore:    构建/工具
#
EOF

# 配置项目使用此模板
git config commit.template .gitmessage
```

建议将 `.gitmessage` 文件提交到仓库中，以便团队成员共享使用。

---

## git stash 暂存工作

### 基本概念

`git stash` 用于临时保存当前工作目录和暂存区的更改，将工作区恢复到干净的状态。这在以下场景中非常有用：

- 需要临时切换到其他分支处理紧急任务
- 想要拉取远程最新代码但本地有未完成的更改
- 需要清理工作目录进行测试

### 基本用法

```bash
# 保存当前更改
git stash

# 保存并添加描述信息
git stash push -m "Work in progress: user authentication"

# 保存未跟踪的文件
git stash -u
git stash --include-untracked

# 保存所有文件（包括被 .gitignore 忽略的文件）
git stash -a
git stash --all
```

### 查看暂存列表

```bash
# 查看所有暂存
git stash list

# 输出示例：
# stash@{0}: WIP on main: abc1234 Last commit message
# stash@{1}: On feature-branch: Work in progress on new feature
```

### 恢复暂存

```bash
# 恢复最近的暂存并从列表中删除
git stash pop

# 恢复指定的暂存但不删除
git stash apply stash@{1}

# 恢复最近的暂存但不删除
git stash apply
```

### 删除暂存

```bash
# 删除最近的暂存
git stash drop

# 删除指定的暂存
git stash drop stash@{1}

# 清除所有暂存
git stash clear
```

### 查看暂存内容

```bash
# 查看最近暂存的更改摘要
git stash show

# 查看完整的 diff
git stash show -p

# 查看指定暂存的更改
git stash show -p stash@{1}
```

### 从暂存创建分支

如果暂存的更改与当前分支不兼容，可以从暂存创建新分支：

```bash
git stash branch new-branch-name stash@{0}
```

这会基于创建暂存时的提交创建一个新分支，应用暂存的更改，并删除该暂存。

### 部分暂存

使用 `-p` 选项可以选择性地暂存部分更改：

```bash
git stash push -p -m "Partial stash of changes"
```

这会进入交互式模式，让你选择要暂存的代码块。

### 暂存特定文件

```bash
# 暂存指定文件
git stash push -m "Stash specific files" -- file1.txt file2.txt

# 暂存匹配模式的文件
git stash push -m "Stash JS files" -- "*.js"
```

---

## 提交最佳实践

### 原子性提交

每个提交应该是一个原子性的、独立的更改。一个提交应该：

- 只做一件事
- 可以独立应用而不会破坏代码
- 包含所有相关的更改（如代码、测试、文档）

**不好的做法**：

```bash
# 一个提交包含多个不相关的更改
git add .
git commit -m "Fix bug, add feature, update docs, refactor code"
```

**好的做法**：

```bash
# 每个更改单独提交
git add src/auth.js test/auth.test.js
git commit -m "fix(auth): resolve token validation issue"

git add src/api/users.js
git commit -m "feat(api): add user search endpoint"

git add docs/api.md
git commit -m "docs(api): update user search documentation"

git add src/utils.js
git commit -m "refactor(utils): extract common validation logic"
```

### 提交信息要清晰

**使用祈使语气**：提交信息应该描述提交"做什么"，而不是"做了什么"。

```
# 好的
feat(auth): add user login functionality

# 不好的
feat(auth): added user login functionality
```

**保持简洁**：第一行（标题）应该简洁明了，通常不超过 50 个字符。

**提供上下文**：如果需要，在正文中解释为什么做这个更改，而不仅仅是做了什么。

### 提交频率

**频繁提交**：小步快跑，经常提交。这比攒一大堆更改然后一次性提交要好。

- 更容易定位问题
- 更容易理解历史
- 更容易回滚更改
- 减少冲突风险

**不要提交半成品**：虽然要频繁提交，但也要确保每次提交都是有意义的、完整的更改。

### 测试后再提交

在提交之前，确保：

```bash
# 运行测试
npm test

# 检查代码风格
npm run lint

# 检查类型（TypeScript 项目）
npm run typecheck

# 确认更改内容
git diff --staged
git status
```

### 编写有意义的提交信息

**好的提交信息示例**：

```
fix(auth): prevent session fixation attack

The previous implementation did not regenerate the session ID after
successful authentication, making it vulnerable to session fixation
attacks.

- Regenerate session ID on login
- Invalidate old session
- Add CSRF token to login form

Fixes #234
Security: CVE-2024-XXXXX
```

**不好的提交信息示例**：

```
fix bug
update
changes
wip
asdfasdf
临时提交
```

---

## 常见提交问题排查

### 问题一：提交了错误的文件

**场景**：不小心将不应该提交的文件（如 `.env`、`node_modules`）提交了。

**解决方案**：

```bash
# 如果是最后一次提交，使用 --amend
git rm --cached .env
git commit --amend --no-edit

# 如果不是最后一次提交，使用 reset
git reset HEAD~1
# 取消暂存错误的文件
git restore --staged .env
# 重新提交
git commit -m "Your commit message"

# 添加到 .gitignore
echo ".env" >> .gitignore
git add .gitignore
git commit -m "chore: add .env to gitignore"
```

### 问题二：提交信息写错了

**解决方案**：

```bash
# 修改最后一次提交的信息
git commit --amend -m "Correct commit message"

# 如果已经推送到远程
git commit --amend -m "Correct commit message"
git push --force-with-lease
```

### 问题三：遗漏了文件

**解决方案**：

```bash
# 添加遗漏的文件到最后一次提交
git add forgotten-file.js
git commit --amend --no-edit

# 如果已经推送
git add forgotten-file.js
git commit --amend --no-edit
git push --force-with-lease
```

### 问题四：提交了错误的分支

**场景**：在 main 分支上提交了应该在功能分支上的更改。

**解决方案**：

```bash
# 创建新分支并保留当前提交
git branch feature/new-branch

# 回滚 main 分支上的提交
git reset HEAD~1

# 切换到新分支
git checkout feature/new-branch
```

### 问题五：提交包含敏感信息

**场景**：提交了密码、API 密钥等敏感信息。

**解决方案**：

```bash
# 1. 立即撤销提交
git reset HEAD~1

# 2. 从文件中移除敏感信息

# 3. 添加到 .gitignore
echo "config/secrets.yml" >> .gitignore

# 4. 重新提交
git add .
git commit -m "Remove sensitive information"

# 5. 如果已经推送，需要重写历史
# 使用 git filter-branch 或 BFG Repo-Cleaner
# 注意：这需要强制推送，会影响所有协作者

# 使用 BFG Repo-Cleaner（推荐）
bfg --replace-text passwords.txt
git reflog expire --expire=now --all
git gc --prune=now --aggressive
git push --force
```

### 问题六：提交太大

**场景**：提交包含大量文件或大文件，导致提交缓慢。

**解决方案**：

```bash
# 查看提交大小
git count-objects -vH

# 如果需要提交大文件，考虑使用 Git LFS
git lfs install
git lfs track "*.psd"
git add .gitattributes
git add large-file.psd
git commit -m "Add design file with LFS"
```

### 问题七：合并冲突导致的提交问题

**场景**：合并分支时遇到冲突，解决后想要提交。

**解决方案**：

```bash
# 解决冲突后
git add resolved-file.js
git commit -m "merge: resolve conflicts between feature and main"
```

---

## 国内开发者的提交规范建议

### 中英文结合的提交信息

在国内团队中，建议采用以下策略：

**方案一：英文类型 + 中文描述**

```
feat(auth): 添加用户登录功能
fix(api): 修复用户搜索返回空结果的问题
docs(readme): 更新安装说明文档
refactor(db): 优化数据库查询性能
```

这种方式结合了国际规范的类型标签和中文的可读性，是目前国内团队最常用的方式。

**方案二：纯英文（适合开源项目）**

```
feat(auth): add user login functionality
fix(api): fix empty result in user search
```

如果你的项目是开源项目或需要与国际团队协作，建议使用纯英文提交信息。

**方案三：纯中文（适合内部项目）**

```
功能：添加用户登录功能
修复：修复用户搜索返回空结果的问题
文档：更新安装说明文档
```

这种方式只适合纯中文内部团队，不推荐用于开源项目。

### 推荐的国内提交规范

```bash
# 格式
<type>(<scope>): <中文描述>

# 示例
feat(用户模块): 添加微信登录功能
fix(支付系统): 修复支付宝回调验签失败的问题
docs(README): 添加快速开始指南
refactor(订单服务): 重构订单状态机逻辑
test(用户接口): 添加用户注册接口的单元测试
chore(依赖): 升级 Spring Boot 到 3.2.0
ci(GitHub Actions): 添加自动化部署流程
```

### 中文提交信息的注意事项

1. **描述要简洁明了**：中文描述应该简短清晰，避免冗长
2. **使用专业术语**：使用行业通用的中文术语
3. **保持一致性**：团队内统一使用相同的格式
4. **避免口语化**：使用书面语，避免网络用语

### 与代码审查工具集成

许多代码审查工具和 CI/CD 系统支持基于提交信息的自动化处理：

```yaml
# .github/workflows/release.yml
name: Release
on:
  push:
    branches: [main]
jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: conventional-changelog/commitlint-action@v1
        with:
          config: .commitlintrc.yml
```

### commitlint 配置示例

```javascript
// commitlint.config.js
module.exports = {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'type-enum': [
      2,
      'always',
      [
        'feat',     // 新功能
        'fix',      // 修复
        'docs',     // 文档
        'style',    // 格式
        'refactor', // 重构
        'perf',     // 性能
        'test',     // 测试
        'build',    // 构建
        'ci',       // CI
        'chore',    // 其他
        'revert',   // 回滚
      ],
    ],
    'type-case': [2, 'always', 'lower-case'],
    'type-empty': [2, 'never'],
    'subject-empty': [2, 'never'],
    'subject-full-stop': [2, 'never', '.'],
    'header-max-length': [2, 'always', 100],
  },
};
```

### 使用 husky 配置 Git 钩子

```bash
# 安装依赖
npm install -D husky @commitlint/cli @commitlint/config-conventional

# 初始化 husky
npx husky install

# 添加 commit-msg 钩子
npx husky add .husky/commit-msg 'npx --no -- commitlint --edit ${1}'
```

### 国内团队的分支策略与提交规范结合

```
main (生产分支)
  ├── develop (开发分支)
  │   ├── feature/user-login (功能分支)
  │   │   └── feat(用户): 添加登录功能
  │   ├── feature/payment (功能分支)
  │   │   └── feat(支付): 接入微信支付
  │   └── fix/search-bug (修复分支)
  │       └── fix(搜索): 修复分页参数错误
  └── release/v1.0.0 (发布分支)
      └── docs(更新日志): 添加 v1.0.0 更新说明
```

---

## 总结

Git 的暂存和提交是版本控制的核心操作。掌握这些概念和命令能够帮助你：

1. **精确控制提交内容**：使用暂存区选择性地提交更改
2. **编写清晰的提交信息**：遵循 Conventional Commits 规范
3. **维护干净的提交历史**：使用原子性提交和最佳实践
4. **处理各种提交场景**：解决常见的提交问题

记住以下关键点：

- 暂存区是 Git 的独特优势，善用它可以提高代码管理的精确度
- `git add -p` 是最强大的暂存工具，支持部分暂存
- 提交信息应该清晰、简洁、有意义
- 原子性提交有助于代码审查和问题定位
- 签名提交可以提高代码的可信度
- 使用提交模板和钩子可以强制执行团队规范

通过遵循本章介绍的最佳实践，你将能够更好地管理代码版本，与团队成员高效协作，并维护一个清晰、可追溯的项目历史。
