# 练习 6：参与开源项目

## 目标

学习如何 Fork 项目、创建 PR 和参与开源贡献。

## 步骤

### 1. 找到适合的项目

使用以下标签搜索：
- `good first issue`
- `help wanted`
- `beginner friendly`

### 2. Fork 项目

1. 找到目标仓库
2. 点击右上角 **Fork** 按钮
3. 等待 Fork 完成

### 3. 克隆你的 Fork

```bash
git clone git@github.com:你的用户名/项目名.git
cd 项目名
```

### 4. 添加上游仓库

```bash
git remote add upstream git@github.com:原作者/项目名.git

# 验证
git remote -v
```

### 5. 创建功能分支

```bash
# 同步上游最新代码
git fetch upstream
git checkout main
git merge upstream/main

# 创建功能分支
git checkout -b fix/typo-in-readme
```

### 6. 进行修改

例如修复 README 中的拼写错误：

```bash
# 修改文件
vim README.md

# 提交
git add README.md
git commit -m "docs: 修复 README 中的拼写错误"
```

### 7. 推送到你的 Fork

```bash
git push origin fix/typo-in-readme
```

### 8. 创建 Pull Request

1. 打开你的 Fork 页面
2. 点击 **Compare & pull request**
3. 确认：
   - base repository: 原作者的仓库
   - head repository: 你的 Fork
4. 填写 PR 描述
5. 点击 **Create pull request**

### 9. 撰写好的 PR 描述

```markdown
## 变更说明
修复了 README 中 "installtion" 拼写错误，改为 "installation"。

## 变更类型
- [x] 文档更新 (docs)

## 关联 Issue
无

## 截图
无
```

### 10. 等待审查

- 回复审查者的评论
- 根据反馈修改代码
- 保持耐心和友好

## 最佳实践

### 提交前
1. 阅读 CONTRIBUTING.md
2. 了解代码规范
3. 检查是否已有相关 Issue/PR

### 提交时
1. 使用清晰的提交信息
2. 一个 PR 只做一件事
3. 写好 PR 描述

### 提交后
1. 及时响应反馈
2. 保持礼貌和专业
3. 接受可能被拒绝

## 常见问题

### Q: PR 被拒绝了怎么办？
**A:** 这很正常，不要灰心。询问原因，学习改进，或者寻找其他贡献机会。

### Q: 如何同步上游更新？
**A:**
```bash
git fetch upstream
git checkout main
git merge upstream/main
git push origin main
```

### Q: 如何撤销 PR 中的提交？
**A:**
```bash
git revert commit-hash
git push
```

## 开源贡献类型

1. **修复 Bug**：最简单的贡献方式
2. **改进文档**：添加说明、修复错误
3. **添加测试**：提高代码质量
4. **添加功能**：较大的改动
5. **翻译内容**：帮助更多人

## 知识点

- Fork 的概念和使用
- 同步上游仓库
- 创建高质量的 PR
- 开源贡献礼仪

## 恭喜你！

完成这个练习后，你已经掌握了 GitHub 的基本使用方法。继续参与开源项目，你会变得越来越熟练！

## 推荐资源

- [First Timers Only](https://www.firsttimersonly.com/)
- [Good First Issues](https://goodfirstissue.dev/)
- [Up For Grabs](https://up-for-grabs.net/)
