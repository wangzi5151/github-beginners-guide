# Git 配置完全指南

Git 是一个高度可定制的版本控制系统。正确配置 Git 不仅能提升开发效率，还能避免许多常见的协作问题。很多初学者在安装完 Git 之后就直接开始使用，结果遇到各种奇怪的问题：提交记录显示的名字不对、换行符出错、编辑器打不开、每次推送都要输密码……这些都是因为没有做好基本配置导致的。

本章将从零开始，系统性地介绍 Git 配置的方方面面，帮助你打造一个高效、顺手的 Git 工作环境。无论你是刚入门的新手，还是有一定经验的开发者，都能在这里找到实用的配置技巧。

---

## 一、Git 配置层级详解

Git 的配置采用分层架构，由三个层级组成。理解这些层级对于管理不同场景下的配置至关重要。当同一个配置项在多个层级存在时，Git 会按照优先级从高到低覆盖：**仓库级 > 全局级 > 系统级**。

你可以把这三层配置想象成"继承关系"：系统级是基础配置，全局级覆盖系统级，仓库级覆盖全局级。这样设计的好处是，你可以有一个通用的全局配置，同时为特定项目设置特殊配置，而不会互相干扰。

### 1.1 系统级配置（System Level）

系统级配置影响该机器上的**所有用户**和**所有仓库**。配置文件位于 `/etc/gitconfig`（Linux/macOS）或 `C:\Program Files\Git\etc\gitconfig`（Windows）。在大多数个人开发场景中，你很少需要修改这一层配置。它通常由系统管理员维护，在企业统一开发环境、共享服务器、学校机房等场景中比较常见。

```bash
# 查看系统级配置
git config --list --system

# 编辑系统级配置（需要管理员权限）
sudo git config --system core.editor "vim"

# 直接编辑系统级配置文件
sudo nano /etc/gitconfig
```

系统级配置的典型用途包括：统一设置公司的代码审查规则、指定默认的差异对比工具、配置全局的行尾符处理策略等。如果你是个人开发者，通常可以跳过这一层。

### 1.2 全局级配置（Global Level）

全局级配置是**当前用户**级别的，影响该用户的所有仓库。配置文件位于用户主目录下的 `~/.gitconfig`（Linux/macOS）或 `C:\Users\你的用户名\.gitconfig`（Windows）。这是开发者最常用的配置层级，你大部分的 Git 配置都应该放在这里。

```bash
# 查看全局级配置
git config --list --global

# 编辑全局级配置
git config --global user.name "张三"
git config --global user.email "zhangsan@example.com"

# 直接编辑配置文件（效果等同于命令行配置）
git config --global -e

# 在 macOS 上用 Finder 打开配置文件
open ~/.gitconfig

# 在 Linux 上用默认编辑器打开
xdg-open ~/.gitconfig
```

全局级配置应该包含你个人的通用设置：用户名、邮箱、首选编辑器、常用的别名、默认的分支名等。这些设置会在你所有的项目中生效，除非被仓库级配置覆盖。

### 1.3 仓库级配置（Repository Level）

仓库级配置仅对**当前仓库**生效。配置文件位于仓库根目录下的 `.git/config`。当你需要为某个项目使用不同的配置时（例如公司项目和个人项目使用不同的邮箱、某个项目需要特殊的合作工具），这一层就非常有用。

```bash
# 查看仓库级配置
git config --list --local

# 编辑仓库级配置
git config --local user.name "San Zhang"
git config --local user.email "san.zhang@company.com"

# 直接编辑配置文件
git config --local -e

# 查看仓库级配置文件的完整路径
ls -la .git/config
```

仓库级配置的一个重要特点是，它可以覆盖全局配置的任何设置。比如你在全局配置中使用个人邮箱，但在公司项目中需要使用公司邮箱，只需要在该项目的仓库级配置中设置即可。这样你就不需要频繁切换全局配置了。

### 1.4 配置层级对比

| 特性 | 系统级 | 全局级 | 仓库级 |
|------|--------|--------|--------|
| 文件路径 | `/etc/gitconfig` | `~/.gitconfig` | `.git/config` |
| 影响范围 | 所有用户 | 当前用户 | 当前仓库 |
| 编辑命令 | `--system` | `--global` | `--local` |
| 优先级 | 最低 | 中等 | 最高 |
| 常见用途 | 企业统一配置 | 个人通用配置 | 项目特殊配置 |
| 需要权限 | 管理员权限 | 普通用户权限 | 仓库写入权限 |

### 1.5 查看所有配置及其来源

当你需要排查配置问题时，知道某个配置来自哪个层级非常有用。Git 提供了详细的命令来查看配置的来源：

```bash
# 查看所有配置及其来源文件
git config --list --show-origin

# 查看某个具体配置项的值及其来源
git config --show-origin user.name

# 查看生效的配置（合并后的最终结果）
git config --list

# 只查看全局配置
git config --list --global

# 只查看仓库级配置
git config --list --local

# 只查看系统级配置
git config --list --system
```

输出示例：
```
file:/home/zhangsan/.gitconfig    user.name=张三
file:/data/project/.git/config    user.email=zhangsan@company.com
file:/home/zhangsan/.gitconfig    core.editor=code --wait
file:/data/project/.git/config    core.autocrlf=input
```

### 1.6 工作目录配置（条件包含）

从 Git 2.x 开始，还支持一种基于目录的条件配置。当你在某个目录下工作时，Git 可以自动加载该目录下的 `.gitconfig` 文件。这在管理多个工作目录（比如工作项目和个人项目放在不同目录）时非常方便。

```bash
# 在全局配置中设置条件包含
git config --global includeIf "gitdir:~/work/" path "~/work/.gitconfig"
git config --global includeIf "gitdir:~/personal/" path "~/personal/.gitconfig"
git config --global includeIf "gitdir:~/opensource/" path "~/opensource/.gitconfig"
```

这样，当你在 `~/work/` 目录下的任何仓库中工作时，Git 会自动加载 `~/work/.gitconfig` 中的配置。你可以在这些子配置文件中设置不同的用户名、邮箱、代理等。这是管理多身份开发环境的最佳实践。

---

## 二、用户信息配置

用户信息是 Git 最基础也是最重要的配置。每次提交都会记录这些信息，它们是你在 Git 历史中的"身份标识"。如果配置不当，你的提交记录可能会显示错误的名字或邮箱，甚至无法与 GitHub 账户正确关联。

### 2.1 设置用户名和邮箱

```bash
# 全局配置（推荐，设置后所有仓库默认使用这个身份）
git config --global user.name "张三"
git config --global user.email "zhangsan@example.com"

# 验证配置是否成功
git config --global user.name
git config --global user.email

# 仓库级配置（用于覆盖全局配置，只对当前仓库生效）
cd /path/to/company-project
git config --local user.name "San Zhang"
git config --local user.email "san.zhang@company.com"
```

> **重要提示**：用户名和邮箱必须配置，否则 Git 会使用系统用户名和主机名作为默认值，这会导致提交记录混乱。而且在某些系统上，如果没有配置邮箱，`git commit` 命令会直接报错。

### 2.2 GitHub 用户名和邮箱匹配

如果你想让 GitHub 自动关联你的提交到你的账户，你需要使用与 GitHub 账户关联的邮箱。你可以在 GitHub 的 **Settings > Emails** 页面查看和管理关联的邮箱。如果提交中使用的邮箱不在你的 GitHub 账户关联列表中，这些提交在 GitHub 上会显示为灰色头像，无法点击跳转到你的个人主页。

```bash
# 使用 GitHub 提供的 noreply 邮箱（保护隐私的最佳方式）
git config --global user.email "123456789+your-username@users.noreply.github.com"
```

> **隐私提示**：如果你不想公开真实邮箱，可以在 GitHub 设置中勾选 "Keep my email addresses private"，然后使用 GitHub 提供的 noreply 邮箱。这样你的真实邮箱就不会出现在公开的提交记录中。很多开源项目贡献者都使用这种方式来保护自己的隐私。

### 2.3 多账户配置

许多开发者同时拥有公司账户和个人账户。使用条件包含（includeIf）可以轻松管理多个身份，让你在不同目录下自动切换身份：

```bash
# ~/.gitconfig（全局默认配置 - 个人项目）
[user]
    name = 张三
    email = zhangsan@personal.com

# ~/work/.gitconfig（工作配置）
[user]
    name = San Zhang
    email = san.zhang@company.com

# ~/opensource/.gitconfig（开源项目配置）
[user]
    name = zhangsan
    email = 123456789+zhangsan@users.noreply.github.com
```

在全局配置中添加条件包含：

```bash
git config --global includeIf "gitdir:~/work/" path "~/work/.gitconfig"
git config --global includeIf "gitdir:~/opensource/" path "~/opensource/.gitconfig"
```

这样配置后，你在 `~/work/` 目录下提交时会使用公司身份，在 `~/opensource/` 目录下提交时会使用开源身份，其他目录则使用默认的个人身份。完全自动化切换，不需要手动操作。

### 2.4 使用 GPG 签名提交

为了证明提交确实由你本人创建，而不是被他人冒充，可以使用 GPG 密钥进行签名。签名后的提交在 GitHub 上会显示为 "Verified" 标记，这在开源项目中非常常见。

```bash
# 生成 GPG 密钥（按照提示操作）
gpg --full-generate-key

# 列出密钥，找到你要使用的密钥 ID
gpg --list-secret-keys --keyid-format=long

# 输出示例：
# sec   rsa4096/ABCDEF1234567890 2024-01-01 [SC]
#       1234567890ABCDEF1234567890ABCDEF12345678
# uid                 [ultimate] 张三 <zhangsan@example.com>

# 设置 Git 使用指定的 GPG 密钥
git config --global user.signingkey ABCDEF1234567890

# 自动签名所有提交（推荐，避免每次都要手动加 -S 参数）
git config --global commit.gpgsign true

# 签名单个提交（不设置自动签名时使用）
git commit -S -m "Signed commit"

# 将 GPG 公钥添加到 GitHub（让 GitHub 能验证你的签名）
gpg --armor --export ABCDEF1234567890
# 复制输出内容，粘贴到 GitHub Settings > SSH and GPG keys > New GPG key
```

### 2.5 配置默认分支名

Git 的默认分支名从 2020 年开始逐步从 `master` 改为 `main`。GitHub 在 2020 年 10 月之后创建的仓库默认使用 `main` 作为主分支名。建议你统一配置：

```bash
# 设置新仓库的默认分支名为 main
git config --global init.defaultBranch main
```

这个配置只影响新创建的仓库（`git init` 或 `git clone` 时），不会改变已有仓库的分支名。如果你需要重命名已有仓库的分支，可以使用 `git branch -m master main` 命令。

---

## 三、编辑器配置

Git 在某些操作（如编写提交信息、解决合并冲突、编写变基脚本）时会调用文本编辑器。配置一个你熟悉的编辑器能大幅提高效率。很多新手在执行 `git commit` 时发现打开了一个陌生的编辑器（通常是 Vim），不知道怎么退出，就是因为没有配置编辑器。

### 3.1 常见编辑器配置

```bash
# VS Code（推荐，功能最丰富，支持语法高亮和多文件编辑）
git config --global core.editor "code --wait"

# VS Code（新窗口打开，避免复用已有窗口）
git config --global core.editor "code --new-window --wait"

# Vim（终端编辑器，功能强大但学习曲线陡峭）
git config --global core.editor "vim"

# Neovim（Vim 的现代化分支）
git config --global core.editor "nvim"

# Nano（对新手最友好的终端编辑器，底部有快捷键提示）
git config --global core.editor "nano"

# Sublime Text
git config --global core.editor "subl -n -w"

# JetBrains IDE（如 IntelliJ IDEA、WebStorm、PyCharm 等）
git config --global core.editor "idea --wait"

# Notepad++（Windows 用户）
git config --global core.editor "'C:/Program Files/Notepad++/notepad++.exe' -multiInst -notabbar -nosession -noPlugin"

# Emacs
git config --global core.editor "emacs"
```

> **`--wait` 参数说明**：这个参数告诉 Git 等待编辑器关闭后再继续执行。对于 GUI 编辑器（如 VS Code、Sublime Text），这是必须的，否则 Git 会在编辑器打开的瞬间就认为编辑完成，导致提交信息为空。对于终端编辑器（如 Vim、Nano），通常不需要这个参数，因为终端编辑器本身就阻塞了命令行。

### 3.2 差异和合并工具配置

如果你使用 VS Code，可以将它同时配置为差异对比工具和合并工具。这样在查看差异或解决冲突时，VS Code 会自动打开并提供可视化的对比界面：

```bash
# 将 VS Code 设置为 Git 的差异工具
git config --global diff.tool vscode
git config --global difftool.vscode.cmd 'code --wait --diff $LOCAL $REMOTE'

# 将 VS Code 设置为 Git 的合并工具
git config --global merge.tool vscode
git config --global mergetool.vscode.cmd 'code --wait $MERGED'

# 合并时是否提示确认（false 表示直接打开，不弹确认框）
git config --global mergetool.prompt false

# 是否保留合并工具创建的备份文件（.orig 文件）
git config --global mergetool.keepBackup false
```

### 3.3 Git 默认编辑器的确定顺序

如果你没有显式配置 `core.editor`，Git 会按照以下顺序查找可用的编辑器。了解这个顺序有助于你排查"为什么 Git 使用了我不想要的编辑器"这类问题：

1. `$GIT_EDITOR` 环境变量（最高优先级）
2. `core.editor` 配置项
3. `$VISUAL` 环境变量
4. `$EDITOR` 环境变量
5. 系统默认（通常是 `vi`）

```bash
# 通过环境变量临时指定编辑器（仅当前终端会话生效，不影响全局配置）
export GIT_EDITOR="nano"
git commit

# 或者直接在命令中指定（一次性生效）
GIT_EDITOR="nano" git commit
```

> **新手提示**：如果你不小心进入了 Vim 编辑器，按 `Esc` 键，然后输入 `:q!` 并回车即可退出（不保存）。如果你是新手，强烈建议将编辑器配置为 Nano 或 VS Code，避免在 Vim 中迷失。

---

## 四、差异对比工具配置

当代码出现冲突或你需要查看修改差异时，一个好的差异对比工具能让你事半功倍。Git 自带的 `git diff` 命令输出的是纯文本格式的差异，对于复杂的修改不太直观。使用图形化的差异工具可以更清晰地看到每一行的变更。

### 4.1 常见差异工具配置

```bash
# VS Code（推荐，免费且功能强大）
git config --global diff.tool vscode
git config --global difftool.vscode.cmd 'code --wait --diff $LOCAL $REMOTE'

# Vimdiff（终端工具，适合喜欢在终端中工作的开发者）
git config --global diff.tool vimdiff

# Beyond Compare（付费工具，功能非常强大）
git config --global diff.tool bc
git config --global difftool.bc.cmd '/usr/local/bin/bcomp "$LOCAL" "$REMOTE"'

# Meld（Linux 上常用的免费可视化 diff 工具）
git config --global diff.tool meld
git config --global difftool.meld.cmd 'meld "$LOCAL" "$REMOTE"'

# KDiff3（免费的三路合并工具）
git config --global diff.tool kdiff3
git config --global difftool.kdiff3.cmd '/usr/local/bin/kdiff3 "$LOCAL" "$REMOTE"'

# WinMerge（Windows 用户的免费选择）
git config --global diff.tool winmerge
git config --global difftool.winmerge.cmd "'C:/Program Files/WinMerge/WinMergeU.exe' -e -u -dl 'Original' -dr 'Modified' \"$LOCAL\" \"$REMOTE\""
```

### 4.2 使用差异工具

```bash
# 查看工作区与暂存区的差异（最常用）
git diff

# 查看暂存区与最新提交的差异
git diff --staged

# 使用可视化工具查看差异（打开配置的 diff 工具）
git difftool

# 使用可视化工具查看暂存区差异
git difftool --staged

# 对比两个分支的所有差异
git difftool branch1..branch2

# 对比两个提交之间的差异
git difftool abc1234..def5678

# 对比某个文件在两个版本之间的差异
git difftool HEAD~3..HEAD -- src/main.js

# 对比工作区与某个分支的差异
git difftool main
```

### 4.3 差异对比的颜色和格式配置

```bash
# 启用颜色输出（让差异更易读）
git config --global color.diff auto
git config --global color.status auto
git config --global color.branch auto
git config --global color.interactive auto

# 设置差异算法（patience 算法通常产生更可读的差异，尤其适合有大量格式化代码的项目）
git config --global diff.algorithm patience

# 设置差异的上下文行数（默认是3行）
git config --global diff.context 5

# 启用单词级别差异（显示行内具体哪些单词被修改了）
git config --global diff.wordRegex '[a-z]+|[A-Z][a-z]+|[A-Z]+|[0-9]+|[^[:space:]]'
```

---

## 五、Git 别名配置

别名（Alias）是提高 Git 使用效率的最佳方式之一。通过为常用命令设置简短的别名，你可以大幅减少打字量，同时避免记忆复杂的命令参数。Git 别名的强大之处在于，它不仅仅是简单的命令替换，还可以组合多个命令，甚至调用外部脚本。

### 5.1 基础别名

这些是最常用的别名，为最频繁使用的命令提供简短的替代：

```bash
# 基本操作（这些是最经典的 Git 别名，几乎是所有资深开发者必备的）
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.cp cherry-pick
git config --global alias.rb rebase
git config --global alias.sw switch

# 查看状态（简洁模式，只显示变更的文件名）
git config --global alias.s "status -s"

# 查看状态（带分支信息的简洁模式）
git config --global alias.ss "status -sb"
```

使用这些别名后，你可以用 `git st` 代替 `git status`，用 `git co main` 代替 `git checkout main`，用 `git br` 代替 `git branch`。虽然每次只省了几个字母，但日积月累下来效率提升非常可观。

### 5.2 日志别名

Git 的日志功能非常强大，但默认输出格式比较冗长。通过别名可以快速查看更美观、更有用的日志：

```bash
# 单行日志（带分支图，最推荐的日志别名）
git config --global alias.lg "log --oneline --graph --all --decorate"

# 详细但美观的日志输出（显示日期、作者、提交信息）
git config --global alias.ll "log --pretty=format:'%C(yellow)%h%Creset %ad %C(green)%an%Creset: %s' --date=short"

# 查看最后一次提交的详细信息（包括修改了哪些文件）
git config --global alias.last "log -1 HEAD --stat"

# 搜索提交信息（在所有分支中搜索包含关键词的提交）
git config --global alias.search "log --all --grep"

# 查看某个文件的修改历史（包括重命名追踪）
git config --global alias.flog "log --follow -p --"

# 查看某个人的所有提交
git config --global alias.who "log --author"

# 查看最近 5 次提交
git config --global alias.recent "log -5 --oneline --decorate"

# 查看今天的提交
git config --global alias.today "log --since=midnight --oneline --all"
```

### 5.3 高级别名

这些别名组合了多个操作，可以一步完成复杂的任务：

```bash
# 撤销最后一次提交（保留修改在暂存区，非常实用）
git config --global alias.undo "reset --soft HEAD~1"

# 取消暂存文件（把文件从暂存区移回工作区）
git config --global alias.unstage "reset HEAD --"

# 暂存所有修改并创建一个 WIP 提交（用于临时保存工作进度）
git config --global alias.save "!git add -A && git commit -m 'WIP: savepoint'"

# 恢复到最后一次提交的状态（丢弃所有未提交的修改，慎用！）
git config --global alias.discard "!git checkout -- . && git clean -fd"

# 查看当前分支名（在脚本中很有用）
git config --global alias.current "rev-parse --abbrev-ref HEAD"

# 快速切换到主分支（自动判断是 main 还是 master）
git config --global alias.main "!git checkout main || git checkout master"

# 删除所有已合并的本地分支（清理已完成的功能分支）
git config --global alias.cleanup "!git branch --merged | grep -v '\\*\\|main\\|master\\|develop' | xargs -n 1 git branch -d"

# 提交并推送到远程（一步完成提交和推送）
git config --global alias.publish "!git push origin \$(git rev-parse --abbrev-ref HEAD)"

# 从远程拉取并变基（保持线性历史）
git config --global alias.sync "!git fetch origin && git rebase origin/\$(git rev-parse --abbrev-ref HEAD)"

# 查看文件的每一行是谁在什么时候修改的（代码考古利器）
git config --global alias.blame-line "blame -L"
```

### 5.4 带参数的别名

```bash
# 快速修改最后一次提交信息（不改变提交内容）
git config --global alias.amend "commit --amend --no-edit"

# 交互式暂存（逐块选择要暂存的修改，精确控制提交内容）
git config --global alias.stage "add -p"

# 交互式变基（压缩、重排、编辑提交历史）
git config --global alias.ri "rebase -i"

# 压缩提交（将最近 N 次提交合并为一次，参数 N 和提交信息）
git config --global alias.squash "!f(){ git reset --soft HEAD~\$1 && git commit -m \"\$2\"; }; f"

# 使用示例：git squash 3 "合并三个提交"
```

### 5.5 直接编辑配置文件添加别名

你也可以直接编辑 `~/.gitconfig` 文件来管理别名，这样可以看到所有别名一目了然，也方便批量修改：

```ini
[alias]
    st = status
    s = status -s
    ss = status -sb
    co = checkout
    br = branch
    ci = commit
    cp = cherry-pick
    rb = rebase
    sw = switch
    lg = log --oneline --graph --all --decorate
    ll = log --pretty=format:'%C(yellow)%h%Creset %ad %C(green)%an%Creset: %s' --date=short
    last = log -1 HEAD --stat
    search = log --all --grep
    who = log --author
    undo = reset --soft HEAD~1
    unstage = reset HEAD --
    stage = add -p
    amend = commit --amend --no-edit
    cleanup = !git branch --merged | grep -v '\\*\\|main\\|master\\|develop' | xargs -n 1 git branch -d
    publish = !git push origin $(git rev-parse --abbrev-ref HEAD)
    sync = !git fetch origin && git rebase origin/$(git rev-parse --abbrev-ref HEAD)
```

---

## 六、Git 钳子（Smudge/Clean Filter）

Git 过滤器（Filter）可以在文件被检出或暂存时自动对文件内容进行转换。这对于处理行尾符、敏感信息过滤、关键字替换等场景非常有用。虽然这个功能相对高级，但了解它的原理能帮助你更好地理解 Git 的工作机制。

### 6.1 工作原理

Git 过滤器分为两种类型：

- **clean filter**：在 `git add` 时执行，将工作区文件内容转换后存入暂存区。你可以把它想象成一个"净化器"，在数据进入 Git 仓库之前对其进行处理。例如：删除敏感信息、统一行尾符、压缩代码。
- **smudge filter**：在 `git checkout` 时执行，将暂存区内容转换后写入工作区。它是 clean filter 的逆操作，在数据从 Git 仓库出来时进行处理。例如：恢复关键字、展开模板变量、解密文件。

```
工作区 --[clean filter]--> 暂存区 --[提交]--> 仓库
仓库   --[smudge filter]--> 暂存区 --[检出]--> 工作区
```

### 6.2 基本配置

```bash
# 配置 clean 和 smudge 过滤器
git config --global filter.<name>.clean <command>
git config --global filter.<name>.smudge <command>
```

### 6.3 实例：关键字展开

假设你想在文件中使用 `$Date$` 这样的关键字，让它在检出时自动替换为提交日期：

```bash
# 配置过滤器
git config --global filter.dater.smudge 'sed "s/\\$Date[^$]*\\$/\\$Date: $(date)\\$/g"'
git config --global filter.dater.clean 'sed "s/\\$Date[^$]*\\$/\\$Date\\$/g"'
```

然后在 `.gitattributes` 文件中指定哪些文件使用该过滤器：

```
*.txt filter=dater
```

### 6.4 实例：清理敏感信息

防止意外提交包含 API 密钥或密码的文件：

```bash
# 自动替换敏感信息（clean 时脱敏，smudge 时原样输出）
git config --global filter.cleanse.clean 'sed -E "s/(api_key|password|secret)\\s*=\\s*\"[^\"]+\"/\\1 = \"REDACTED\"/g"'
git config --global filter.cleanse.smudge cat
```

在 `.gitattributes` 中指定：

```
*.config filter=cleanse
```

> **注意**：过滤器的密钥不应硬编码在 `.gitconfig` 中。建议使用密钥管理工具或环境变量来管理加密密钥，避免将密钥提交到仓库中。

### 6.5 在 .gitattributes 中配置

过滤器需要在 `.gitattributes` 文件中关联到特定文件模式才能生效：

```gitattributes
# 对所有 .txt 文件应用 dater 过滤器
*.txt filter=dater

# 对 secrets 目录下的文件应用 crypt 过滤器
secrets/** filter=crypt

# 对所有 Python 文件应用 cleanse 过滤器
*.py filter=cleanse

# 对 JSON 配置文件应用过滤器
config/*.json filter=config-template
```

---

## 七、Git Hooks 配置

Git Hooks 是在特定事件发生时自动执行的脚本。它们位于 `.git/hooks/` 目录下，是实现自动化工作流的强大工具。通过 hooks，你可以在提交前自动运行代码检查、在推送前自动运行测试、在合并后自动更新依赖等。

### 7.1 常用 Hooks 列表

| Hook 名称 | 触发时机 | 典型用途 |
|-----------|---------|---------|
| `pre-commit` | 执行 `git commit` 之前 | 代码检查、格式化、运行测试 |
| `prepare-commit-msg` | 编辑器打开之前 | 自动生成提交信息模板 |
| `commit-msg` | 提交信息写入之后 | 验证提交信息格式是否规范 |
| `post-commit` | 提交完成后 | 发送通知、记录日志 |
| `pre-push` | 执行 `git push` 之前 | 运行完整测试、检查分支保护规则 |
| `pre-rebase` | 执行 `git rebase` 之前 | 防止对特定分支进行变基 |
| `post-merge` | 合并完成后 | 更新依赖包、重置配置文件 |
| `post-checkout` | 切换分支后 | 清理缓存、更新开发环境 |
| `pre-auto-gc` | 自动垃圾回收前 | 自定义清理逻辑 |

### 7.2 创建 Hook 脚本

Hook 脚本必须是可执行的，并且文件名不能有扩展名。以下是一个实用的 `pre-commit` hook 示例：

```bash
# 创建一个 pre-commit hook
cat > .git/hooks/pre-commit << 'EOF'
#!/bin/bash
# 检查是否有未解决的冲突标记
if git diff --cached | grep -E '^[<>=]{7}'; then
    echo "错误：文件中存在未解决的冲突标记！"
    echo "请先解决冲突再提交。"
    exit 1
fi

# 检查是否有调试语句（如 console.log、print、debugger）
if git diff --cached --name-only | grep -qE '\.(js|ts|py)$'; then
    if git diff --cached | grep -qE '(console\.log|debugger|print\()'; then
        echo "警告：提交中可能包含调试语句！"
        echo "请检查是否需要移除。"
        read -p "是否继续提交？(y/N) " -n 1 -r
        echo
        if [[ ! $REPLY =~ ^[Yy]$ ]]; then
            exit 1
        fi
    fi
fi

echo "pre-commit 检查通过"
exit 0
EOF

# 赋予执行权限
chmod +x .git/hooks/pre-commit
```

### 7.3 使用 Husky 管理 Hooks（Node.js 项目）

对于 Node.js 项目，推荐使用 `husky` 来管理 Git Hooks。因为 `.git/hooks/` 目录下的文件不会被 Git 跟踪，使用 husky 可以将 hooks 纳入版本控制，让团队成员共享相同的 hooks 配置：

```bash
# 安装 husky
npm install husky --save-dev

# 初始化 husky（会在项目根目录创建 .husky 目录）
npx husky init

# 添加 pre-commit hook（在提交前运行代码检查）
echo "npm run lint" > .husky/pre-commit

# 添加 commit-msg hook（验证提交信息格式）
echo 'npx commitlint --edit $1' > .husky/commit-msg

# 在 package.json 中添加 prepare 脚本（让其他开发者安装依赖时自动配置 husky）
npm pkg set scripts.prepare="husky"
```

### 7.4 使用 pre-commit 框架（Python 项目）

```bash
# 安装 pre-commit
pip install pre-commit

# 创建 .pre-commit-config.yaml 文件
cat > .pre-commit-config.yaml << 'EOF'
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.5.0
    hooks:
      - id: trailing-whitespace        # 移除行尾空格
      - id: end-of-file-fixer          # 确保文件以换行符结尾
      - id: check-yaml                 # 检查 YAML 文件语法
      - id: check-added-large-files    # 防止提交大文件
      - id: check-json                 # 检查 JSON 文件语法
  - repo: https://github.com/psf/black
    rev: 24.3.0
    hooks:
      - id: black                      # Python 代码格式化
  - repo: https://github.com/pycqa/isort
    rev: 5.13.2
    hooks:
      - id: isort                      # Python import 排序
EOF

# 安装 hooks（会在 .git/hooks/ 中创建 pre-commit 脚本）
pre-commit install

# 手动对所有文件运行一次（首次设置时推荐）
pre-commit run --all-files
```

### 7.5 提交信息规范 Hook

使用 `commitlint` 强制执行规范的提交信息格式，确保团队的提交历史清晰可读：

```bash
# 安装 commitlint
npm install --save-dev @commitlint/cli @commitlint/config-conventional

# 创建 commitlint 配置文件
echo "export default { extends: ['@commitlint/config-conventional'] };" > commitlint.config.js

# 配置 commit-msg hook
npx husky add .husky/commit-msg 'npx commitlint --edit $1'
```

配置后，提交信息必须遵循 Conventional Commits 格式，否则会被拒绝：

```
feat: 添加用户登录功能
fix: 修复密码验证逻辑中的边界条件
docs: 更新 README 文档的安装说明
style: 格式化代码，修复缩进问题
refactor: 重构用户模块，提取公共方法
test: 添加用户注册功能的单元测试
chore: 更新依赖包到最新版本
```

---

## 八、.gitignore 详解与模板

`.gitignore` 文件用于告诉 Git 哪些文件或目录应该被忽略，不纳入版本控制。正确配置 `.gitignore` 可以避免将临时文件、编译产物、敏感信息等不该提交的内容推送到仓库中。

### 8.1 基本语法

```gitignore
# 注释行（以 # 开头，不会被 Git 解析）

# 忽略所有 .log 文件
*.log

# 但保留 important.log（使用 ! 取反，这是"白名单"机制）
!important.log

# 忽略 build 目录下的所有文件（包括子目录）
build/

# 只忽略根目录下的 TODO 文件，不忽略子目录中的同名文件
/TODO

# 忽略 doc 目录下所有子目录中的 .pdf 文件
doc/**/*.pdf

# 忽略所有 .pyc 文件（Python 编译字节码）
*.pyc

# 忽略所有 __pycache__ 目录（Python 缓存目录）
__pycache__/

# 忽略 .env 文件（通常包含数据库密码、API 密钥等敏感信息）
.env

# 忽略 .env.local 和 .env.*.local 文件
.env.local
.env.*.local

# 忽略 node_modules 目录（Node.js 依赖包，体积通常很大）
node_modules/

# 忽略 IDE 配置文件（这些是个人偏好，不应该纳入版本控制）
.idea/
.vscode/
*.swp
*.swo
*~
.project
.settings/
```

### 8.2 模式匹配规则

| 模式 | 说明 | 示例 |
|------|------|------|
| `*` | 匹配任意字符（不含 `/`） | `*.log` 匹配 `app.log`、`error.log` |
| `**` | 匹配任意层级目录 | `**/temp` 匹配 `temp`、`a/temp`、`a/b/temp` |
| `?` | 匹配单个字符 | `file?.txt` 匹配 `file1.txt`、`fileA.txt` |
| `[abc]` | 匹配括号内的任意一个字符 | `file[12].txt` 匹配 `file1.txt`、`file2.txt` |
| `[0-9]` | 匹配范围内的字符 | `file[0-9].txt` 匹配 `file0.txt` 到 `file9.txt` |
| `!` | 取反（不忽略，用于白名单） | `!important.log` |

### 8.3 常用 .gitignore 模板

**Node.js 项目：**
```gitignore
# 依赖包
node_modules/

# 构建产物
dist/
build/
out/

# 环境变量文件（包含敏感信息）
.env
.env.local
.env.development.local
.env.test.local
.env.production.local

# 调试日志
npm-debug.log*
yarn-debug.log*
yarn-error.log*
.pnpm-debug.log*

# 缓存
.cache/
.eslintcache
.stylelintcache

# 操作系统文件
.DS_Store
Thumbs.db
```

**Python 项目：**
```gitignore
# Python 编译文件
__pycache__/
*.py[cod]
*$py.class

# 分发和打包
*.egg-info/
dist/
build/
.eggs/
*.egg

# 虚拟环境
.venv/
venv/
env/
ENV/

# 环境变量
.env

# 类型检查和测试缓存
.mypy_cache/
.pytest_cache/
.tox/
.coverage
htmlcov/

# IDE
.idea/
.vscode/
*.swp
```

**Java 项目：**
```gitignore
# 编译产物
*.class
*.jar
*.war
*.ear

# 构建工具
target/
build/
.gradle/
.mvn/

# IDE
.settings/
.classpath
.project
.idea/
*.iml

# 日志
*.log
```

**Go 项目：**
```gitignore
# 编译产物
*.exe
*.exe~
*.dll
*.so
*.dylib

# 测试输出
*.test
*.out

# 依赖目录
vendor/

# Go workspace 文件
go.work
```

### 8.4 全局 .gitignore

你可以设置一个全局的 `.gitignore` 文件，对所有仓库生效。这对于忽略操作系统文件和 IDE 配置文件特别有用：

```bash
# 创建全局 gitignore 文件
git config --global core.excludesFile ~/.gitignore_global

# 编辑该文件
cat > ~/.gitignore_global << 'EOF'
# 操作系统文件
.DS_Store
.DS_Store?
Thumbs.db
ehthumbs.db
Desktop.ini

# IDE 和编辑器文件
.idea/
.vscode/
*.swp
*.swo
*~
.project
.settings/
.classpath
*.iml
.nbproject/

# 环境变量文件
.env
.env.local

# 临时文件
*.tmp
*.temp
*.bak
*.orig
EOF
```

### 8.5 已跟踪文件的忽略

如果文件已经被 Git 跟踪（之前已经提交过），仅仅添加到 `.gitignore` 不会自动取消跟踪。你需要手动取消跟踪：

```bash
# 取消跟踪单个文件（但保留本地文件不被删除）
git rm --cached <file>

# 取消跟踪整个目录（递归）
git rm -r --cached <directory>

# 示例：取消跟踪 .env 文件
git rm --cached .env
echo ".env" >> .gitignore
git commit -m "chore: stop tracking .env file"

# 批量取消跟踪所有 .pyc 文件
git rm -r --cached **/*.pyc
```

> **警告**：`git rm --cached` 只是从 Git 的暂存区移除文件，不会删除工作区的文件。但是，其他人拉取你这个提交后，这些文件会从他们的工作区中删除。所以在操作前最好通知团队成员。

### 8.6 检查文件是否被忽略

```bash
# 检查某个文件是否被忽略（以及被哪个 .gitignore 规则忽略）
git check-ignore -v <file>

# 示例
git check-ignore -v .env
# 输出: .gitignore:5:.env    .env

# 查看所有被忽略的文件
git status --ignored

# 调试为什么某个文件没有被忽略（检查是否有 ! 取反规则）
git check-ignore -v --no-index <file>
```

---

## 九、.gitattributes 详解

`.gitattributes` 文件用于定义文件的属性，控制 Git 如何处理特定类型的文件。它比 `.gitignore` 更细粒度，可以控制行尾符、差异比较方式、合并策略等行为。

### 9.1 行尾符处理

跨平台协作时，行尾符是最常见的问题之一。Windows 使用 CRLF（`\r\n`），而 macOS/Linux 使用 LF（`\n`）。如果团队成员使用不同的操作系统，行尾符不一致会导致大量的无意义差异。

```gitattributes
# 设置默认行为，对所有文本文件自动处理行尾符
* text=auto

# 明确指定某些文件类型为文本文件（自动处理行尾符）
*.js text
*.ts text
*.css text
*.html text
*.json text
*.md text
*.yml text
*.yaml text
*.xml text
*.sql text

# 明确指定某些文件类型为二进制文件（不处理行尾符）
*.png binary
*.jpg binary
*.jpeg binary
*.gif binary
*.ico binary
*.pdf binary
*.zip binary
*.tar.gz binary
*.woff binary
*.woff2 binary

# 特定文件强制使用 LF（避免 shell 脚本在 Windows 上出错）
*.sh text eol=lf
*.bash text eol=lf
*.zsh text eol=lf
*.fish text eol=lf
Dockerfile text eol=lf
docker-compose*.yml text eol=lf

# 特定文件强制使用 CRLF（Windows 批处理文件）
*.bat text eol=crlf
*.cmd text eol=crlf
```

### 9.2 差异对比配置

```gitattributes
# 对特定文件类型使用自定义差异驱动（Git 内置支持）
*.json diff=json
*.html diff=html
*.css diff=css
*.java diff=java
*.cpp diff=cpp
*.py diff=python

# 忽略特定文件的差异（不显示 diff，适合压缩文件和生成文件）
*.min.js -diff
*.min.css -diff
vendor/** -diff
dist/** -diff
package-lock.json -diff
yarn.lock -diff

# 对二进制文件使用自定义差异比较方式（需要配置 diff 驱动）
*.png diff=image
```

### 9.3 合并策略配置

```gitattributes
# 对特定文件使用自定义合并策略（避免合并冲突）
database.xml merge=ours
package-lock.json merge=ours

# 对特定文件始终使用二进制合并策略（保留当前版本，不尝试合并）
*.pbxproj merge=union
*.lock merge=union

# 配置自定义合并驱动
# 在 .git/config 中定义：
# [merge "ours"]
#     driver = true
```

### 9.4 LFS（Large File Storage）配置

```gitattributes
# 使用 Git LFS 管理大文件（这些文件不会直接存储在 Git 仓库中）
*.psd filter=lfs diff=lfs merge=lfs -text
*.ai filter=lfs diff=lfs merge=lfs -text
*.zip filter=lfs diff=lfs merge=lfs -text
*.tar.gz filter=lfs diff=lfs merge=lfs -text
*.mp4 filter=lfs diff=lfs merge=lfs -text
*.mp3 filter=lfs diff=lfs merge=lfs -text
*.wav filter=lfs diff=lfs merge=lfs -text
*.blend filter=lfs diff=lfs merge=lfs -text
*.unitypackage filter=lfs diff=lfs merge=lfs -text
```

### 9.5 导出归档排除

```gitattributes
# 从 git archive 导出时排除特定文件（这些文件不会出现在打包的源码中）
tests/ export-ignore
test/ export-ignore
.github/ export-ignore
.gitignore export-ignore
.gitattributes export-ignore
.editorconfig export-ignore
phpunit.xml export-ignore
phpunit.xml.dist export-ignore
.travis.yml export-ignore
Makefile export-ignore
```

---

## 十、Git Credential 管理

当你通过 HTTPS 与远程仓库交互时，Git 需要你的身份凭证。默认情况下，每次推送或拉取都会要求你输入用户名和密码，这非常繁琐。Credential Helper 可以安全地存储和管理这些凭证，避免重复输入。

### 10.1 Credential Helper 模式

Git 支持多种凭证存储方式，每种方式有不同的安全性和持久性特点：

| 模式 | 说明 | 安全性 | 持久性 | 适用场景 |
|------|------|--------|--------|---------|
| `cache` | 内存缓存 | 高 | 临时（默认15分钟） | 临时开发环境 |
| `store` | 明文文件存储 | 低 | 永久 | 个人电脑 |
| `manager` | Windows 凭据管理器 | 高 | 永久 | Windows 用户 |
| `manager-core` | Git Credential Manager（跨平台） | 高 | 永久 | 推荐使用 |

### 10.2 使用 cache 模式（临时存储）

```bash
# 启用 cache 模式（默认缓存15分钟）
git config --global credential.helper cache

# 设置更长的缓存时间（单位：秒）
git config --global credential.helper 'cache --timeout=3600'  # 1小时

# 设置缓存时间为1天
git config --global credential.helper 'cache --timeout=86400'

# 设置缓存时间为1周
git config --global credential.helper 'cache --timeout=604800'
```

### 10.3 使用 store 模式（永久存储）

```bash
# 启用 store 模式（凭证保存在 ~/.git-credentials 文件中）
git config --global credential.helper store

# 指定自定义存储文件路径
git config --global credential.helper 'store --file ~/.my-git-credentials'
```

> **安全提示**：`store` 模式将密码以明文形式存储在磁盘上，安全性较低。在生产环境、共享机器或服务器上不推荐使用。如果你使用这种模式，建议设置合适的文件权限（`chmod 600 ~/.git-credentials`）。

### 10.4 使用 Git Credential Manager（推荐）

Git Credential Manager（GCM）是官方推荐的跨平台凭证管理工具，支持 OAuth 认证，安全性更高：

```bash
# 安装 GCM（以 Linux 为例）
# Ubuntu/Debian
sudo apt-get install git-credential-manager

# 配置使用 GCM
git config --global credential.helper manager

# 使用 GCM Core（新版推荐）
git config --global credential.helper manager-core

# macOS 用户也可以使用 Keychain
git config --global credential.helper osxkeychain
```

### 10.5 GitHub Personal Access Token（PAT）

由于 GitHub 已经不支持密码认证，你需要使用 Personal Access Token（PAT）来代替密码：

```bash
# 1. 在 GitHub 上创建 PAT：
#    Settings > Developer settings > Personal access tokens > Generate new token
#    选择需要的权限范围（repo、workflow 等）

# 2. 使用 PAT 进行认证
# 当 Git 要求输入密码时，粘贴你的 PAT（不是你的 GitHub 密码）

# 3. 配置 credential helper 存储 PAT
git config --global credential.helper store

# 4. 测试连接
git fetch origin
# 输入用户名和 PAT，之后就不需要再输入了
```

### 10.6 SSH vs HTTPS

| 特性 | SSH | HTTPS |
|------|-----|-------|
| 认证方式 | SSH 密钥对（无需密码） | 用户名 + Token |
| 配置复杂度 | 中等（需要生成和上传密钥） | 简单 |
| 安全性 | 高（密钥加密） | 取决于实现 |
| 企业防火墙 | 可能被阻止（使用 22 端口） | 通常可用（使用 443 端口） |
| GitHub 推荐 | ✓ | ✓ |

```bash
# 使用 SSH 地址
git clone git@github.com:user/repo.git

# 使用 HTTPS 地址
git clone https://github.com/user/repo.git

# 将已有的 HTTPS 远程仓库改为 SSH
git remote set-url origin git@github.com:user/repo.git

# 将已有的 SSH 远程仓库改为 HTTPS
git remote set-url origin https://github.com/user/repo.git
```

---

## 十一、国内开发者特殊配置

由于网络环境的原因，国内开发者在使用 GitHub 时经常会遇到访问缓慢或连接超时的问题。以下是一些经过验证的解决方案，帮助你更顺畅地使用 GitHub。

### 11.1 配置 Git 代理

如果你有可用的代理服务器（如 Clash、V2Ray 等），可以为 Git 配置代理：

```bash
# 配置 HTTP 代理（适用于大多数代理软件）
git config --global http.proxy http://127.0.0.1:7890
git config --global https.proxy http://127.0.0.1:7890

# 配置 SOCKS5 代理（安全性更高）
git config --global http.proxy socks5://127.0.0.1:7891
git config --global https.proxy socks5://127.0.0.1:7891

# 取消代理设置
git config --global --unset http.proxy
git config --global --unset https.proxy
```

**仅对 GitHub 配置代理（不影响其他仓库，推荐）：**

```bash
# 只对 github.com 配置代理
git config --global http.https://github.com.proxy http://127.0.0.1:7890

# 取消 github.com 的代理
git config --global --unset http.https://github.com.proxy
```

### 11.2 使用国内镜像加速

**GitHub 镜像站点（用于克隆和下载）：**

```bash
# 使用 ghproxy.com 镜像克隆（免费加速）
git clone https://mirror.ghproxy.com/https://github.com/user/repo.git

# 使用 gitclone.com 镜像
git clone https://gitclone.com/github.com/user/repo.git

# 使用 Gitee 镜像（需要先在 Gitee 上导入仓库）
git clone https://gitee.com/mirrors/repo.git
```

**配置包管理器镜像：**

```bash
# npm 镜像（Node.js 项目）
npm config set registry https://registry.npmmirror.com

# pip 镜像（Python 项目）
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple

# Cargo 镜像（Rust 项目）
# 在 ~/.cargo/config.toml 中添加：
# [source.crates-io]
# replace-with = 'tuna'
# [source.tuna]
# registry = "https://mirrors.tuna.tsinghua.edu.cn/git/crates.io-index.git"
```

### 11.3 配置 Git 全局加速

```bash
# 增大 Git 缓冲区大小（有助于克隆大仓库，避免中途断开）
git config --global http.postBuffer 524288000

# 设置较低的超时速度阈值（避免在慢速网络下过早放弃）
git config --global http.lowSpeedLimit 1000
git config --global http.lowSpeedTime 60

# 使用浅克隆减少下载量（只下载最近的历史）
git clone --depth 1 https://github.com/user/repo.git

# 需要完整历史时再补全
git fetch --unshallow

# 增加压缩级别（减少传输数据量，但会增加 CPU 使用）
git config --global core.compression 9
```

### 11.4 配置 SSH 连接优化

```bash
# 编辑 SSH 配置文件
cat >> ~/.ssh/config << 'EOF'
# GitHub SSH 配置
Host github.com
    HostName github.com
    User git
    # 如果你有代理，取消下面的注释并修改端口
    # ProxyCommand nc -v -x 127.0.0.1:7891 %h %p
    # 增加连接保持时间（防止连接断开）
    ServerAliveInterval 60
    ServerAliveCountMax 30
    # 使用更短的连接超时（避免长时间等待）
    ConnectTimeout 30
    # 复用已有连接（加速多次操作）
    ControlMaster auto
    ControlPath ~/.ssh/sockets/%r@%h-%p
    ControlPersist 600
EOF

# 创建 socket 目录
mkdir -p ~/.ssh/sockets
```

### 11.5 Gitee 与 GitHub 双远程仓库

```bash
# 添加两个远程仓库
git remote add origin https://github.com/user/repo.git
git remote add gitee https://gitee.com/user/repo.git

# 推送到 GitHub
git push origin main

# 推送到 Gitee
git push gitee main

# 查看所有远程仓库
git remote -v

# 设置 push-url 同时推送到两个仓库（一推两送）
git remote set-url --add --push origin https://github.com/user/repo.git
git remote set-url --add --push origin https://gitee.com/user/repo.git

# 现在 git push origin 会同时推送到 GitHub 和 Gitee
```

### 11.6 完整的国内开发者推荐配置

以下是针对国内开发者的完整配置方案，复制粘贴即可使用：

```bash
# ====================
# 基本信息配置
# ====================
git config --global user.name "你的名字"
git config --global user.email "your-email@example.com"
git config --global init.defaultBranch main

# ====================
# 编辑器配置
# ====================
git config --global core.editor "code --wait"

# ====================
# 差异和合并工具
# ====================
git config --global diff.tool vscode
git config --global difftool.vscode.cmd 'code --wait --diff $LOCAL $REMOTE'
git config --global merge.tool vscode
git config --global mergetool.vscode.cmd 'code --wait $MERGED'
git config --global mergetool.prompt false

# ====================
# 行尾符配置
# ====================
# Windows 用户取消注释下一行：
# git config --global core.autocrlf true
# macOS/Linux 用户取消注释下一行：
git config --global core.autocrlf input

# ====================
# 推送和拉取行为
# ====================
git config --global push.autoSetupRemote true
git config --global push.default current
git config --global pull.rebase true

# ====================
# 性能优化
# ====================
git config --global http.postBuffer 524288000
git config --global core.compression 9

# ====================
# 常用别名
# ====================
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.lg "log --oneline --graph --all --decorate"
git config --global alias.ll "log --pretty=format:'%C(yellow)%h%Creset %ad %C(green)%an%Creset: %s' --date=short"
git config --global alias.undo "reset --soft HEAD~1"
git config --global alias.unstage "reset HEAD --"
git config --global alias.amend "commit --amend --no-edit"
git config --global alias.cleanup "!git branch --merged | grep -v '\\*\\|main\\|master\\|develop' | xargs -n 1 git branch -d"

# ====================
# 凭证管理
# ====================
git config --global credential.helper store

# ====================
# 代理配置（如果有代理，取消注释并修改端口）
# ====================
# git config --global http.https://github.com.proxy http://127.0.0.1:7890
```

---

## 十二、常见配置问题排查

### 12.1 查看当前配置

```bash
# 查看所有配置（包括来源文件路径，排查问题时最有用）
git config --list --show-origin

# 查看某个配置项的值
git config user.name
git config core.editor

# 查看所有生效的配置（合并后的最终结果）
git config --list

# 只查看全局配置
git config --list --global

# 只查看仓库级配置
git config --list --local

# 只查看系统级配置
git config --list --system
```

### 12.2 修改配置

```bash
# 修改已有的配置项
git config --global user.name "新名字"

# 添加新的配置项
git config --global alias.new-alias "status -s"

# 删除配置项
git config --global --unset alias.old-alias

# 删除整个配置段（慎用！）
git config --global --remove-section alias
```

### 12.3 常见错误及解决

**错误：`Author identity unknown`**

```bash
# 原因：没有配置用户名和邮箱，Git 无法记录提交者身份
# 解决：
git config --global user.name "你的名字"
git config --global user.email "your-email@example.com"

# 如果只想为当前仓库配置
git config --local user.name "你的名字"
git config --local user.email "your-email@example.com"
```

**错误：`fatal: not in a git directory`**

```bash
# 原因：当前目录不是 Git 仓库，且没有使用 --global 修饰符
# 解决：确保在 Git 仓库目录中执行命令，或使用 --global 修饰符
git config --global user.name "你的名字"
```

**错误：`error: could not lock config file ~/.gitconfig: Permission denied`**

```bash
# 原因：没有写入权限（可能是文件被其他进程占用，或权限设置不当）
# 解决：
# Linux/macOS
chmod 644 ~/.gitconfig
# 或者修复文件所有者
sudo chown $(whoami) ~/.gitconfig

# Windows（以管理员身份运行 Git Bash）
icacls "%USERPROFILE%\.gitconfig" /grant "%USERNAME%:F"
```

**错误：`warning: LF will be replaced by CRLF`**

```bash
# 原因：行尾符自动转换警告（通常出现在 Windows 上）
# 解决：根据操作系统设置正确的 autocrlf 值
# Windows
git config --global core.autocrlf true
# macOS/Linux
git config --global core.autocrlf input
# 关闭自动转换（不推荐，除非你清楚自己在做什么）
git config --global core.autocrlf false
```

**错误：`fatal: refusing to merge unrelated histories`**

```bash
# 原因：两个分支没有共同的祖先提交（通常是两个独立创建的仓库）
# 解决：
git merge main --allow-unrelated-histories
```

**错误：`fatal: The remote end hung up unexpectedly`**

```bash
# 原因：网络问题或仓库太大，连接中断
# 解决：增大缓冲区和设置超时
git config --global http.postBuffer 524288000
git config --global http.lowSpeedLimit 0
git config --global http.lowSpeedTime 999999
```

**错误：`error: Your local changes to the following files would be overwritten by checkout`**

```bash
# 原因：工作区有未提交的修改，切换分支会覆盖这些修改
# 解决：
# 方案1：提交修改
git add . && git commit -m "保存当前修改"

# 方案2：暂存修改
git stash
git checkout <branch>
# 切换回来后恢复
git stash pop
```

### 12.4 重置配置

```bash
# 删除全局配置文件（慎用！会丢失所有全局配置）
rm ~/.gitconfig

# 重置特定配置项
git config --global --unset user.name
git config --global --unset user.email

# 查看并手动编辑配置文件（最精确的修改方式）
git config --global -e

# 备份当前配置
cp ~/.gitconfig ~/.gitconfig.backup

# 恢复备份
cp ~/.gitconfig.backup ~/.gitconfig
```

### 12.5 配置文件格式问题

如果你手动编辑 `.gitconfig` 文件时出现格式错误，Git 会报错。正确的格式如下：

```ini
# ~/.gitconfig 示例（注意缩进使用 Tab 或空格）
[user]
    name = 张三
    email = zhangsan@example.com
[core]
    editor = code --wait
    autocrlf = input
    excludesfile = ~/.gitignore_global
[alias]
    st = status
    co = checkout
    br = branch
    ci = commit
    lg = log --oneline --graph --all
[push]
    default = current
    autoSetupRemote = true
[pull]
    rebase = true
[init]
    defaultBranch = main
[diff]
    tool = vscode
    algorithm = patience
[difftool "vscode"]
    cmd = code --wait --diff $LOCAL $REMOTE
[merge]
    tool = vscode
[mergetool "vscode"]
    cmd = code --wait $MERGED
[http]
    postBuffer = 524288000
    https://github.com.proxy = http://127.0.0.1:7890
[credential]
    helper = store
```

### 12.6 调试 Git 配置

当遇到难以排查的问题时，可以使用 Git 的跟踪日志功能：

```bash
# 开启 Git 的跟踪日志（显示详细的执行过程）
GIT_TRACE=1 git status

# 查看 Git 如何解析配置文件
GIT_TRACE=1 git config --list

# 查看凭证管理的调试信息
GIT_TRACE=1 git credential-manager

# 查看网络请求的详细信息（排查网络问题）
GIT_CURL_VERBOSE=1 git fetch origin

# 查看 Git 内部命令的执行过程
GIT_TRACE=1 GIT_TRACE_SETUP=1 git status
```

---

## 附录：常用配置速查表

| 配置项 | 命令 | 说明 |
|--------|------|------|
| 用户名 | `git config --global user.name "名字"` | 必须配置 |
| 邮箱 | `git config --global user.email "邮箱"` | 必须配置 |
| 默认分支 | `git config --global init.defaultBranch main` | 推荐使用 main |
| 编辑器 | `git config --global core.editor "code --wait"` | VS Code |
| 差异工具 | `git config --global diff.tool vscode` | VS Code |
| 合并工具 | `git config --global merge.tool vscode` | VS Code |
| 行尾符 | `git config --global core.autocrlf input` | macOS/Linux |
| 推送 | `git config --global push.autoSetupRemote true` | 简化推送 |
| 拉取 | `git config --global pull.rebase true` | 使用 rebase |
| 凭证 | `git config --global credential.helper store` | 永久存储 |
| 代理 | `git config --global http.proxy "地址:端口"` | 网络代理 |
| 缓冲区 | `git config --global http.postBuffer 524288000` | 大仓库加速 |
| 全局忽略 | `git config --global core.excludesFile ~/.gitignore_global` | 全局 gitignore |
| GPG 签名 | `git config --global commit.gpgsign true` | 自动签名提交 |

---

## 下一步

掌握了 Git 配置之后，你可以继续学习：

- [Git 工作原理 →](06-how-git-works.md) — 深入理解 Git 的内部机制，了解 blob、tree、commit 等对象
- [初始化与克隆仓库 →](07-init-clone.md) — 创建新仓库和获取已有仓库
- [暂存与提交 →](08-add-commit.md) — Git 的基本操作流程
- [查看日志与差异 →](09-log-diff.md) — 查看提交历史和代码变更
- [SSH 密钥配置 →](04-ssh-keys.md) — 安全地连接 GitHub

[← 上一章：SSH 密钥配置](04-ssh-keys.md) | [下一章：Git 工作原理 →](06-how-git-works.md)
