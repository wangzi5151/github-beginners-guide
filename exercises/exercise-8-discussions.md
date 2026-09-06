# 练习 8：设置 GitHub Discussions

## 目标

学习如何设置和管理 GitHub Discussions。

## 步骤

### 1. 创建练习仓库

```bash
mkdir discussions-practice
cd discussions-practice
git init

echo "# Discussions Practice" > README.md
git add README.md
git commit -m "Initial commit"

git remote add origin git@github.com:你的用户名/discussions-practice.git
git push -u origin main
```

### 2. 启用 Discussions

1. 进入仓库 **Settings**
2. 在 **Features** 部分勾选 **Discussions**
3. 点击 **Set up discussions**

### 3. 配置分类

1. 进入 **Discussions** 标签
2. 点击 **Categories**
3. 查看默认分类：

| 分类 | 图标 | 用途 |
|------|------|------|
| Announcements | 📣 | 官方公告 |
| General | 💬 | 一般讨论 |
| Ideas | 💡 | 想法和建议 |
| Q&A | 🙋 | 问答 |
| Show and tell | 📝 | 展示和分享 |

### 4. 创建自定义分类

1. 点击 **New category**
2. 填写信息：
   - **Name**: Resources
   - **Description**: 分享有用的资源和链接
   - **Emoji**: 📚
   - **Discussion format**: Open-ended discussion
3. 点击 **Create**

### 5. 创建第一个 Discussion

#### 创建公告

1. 点击 **Announcements** 分类
2. 点击 **New discussion**
3. 填写标题：`欢迎来到我们的社区！`
4. 填写内容：

```markdown
# 欢迎！

欢迎来到我们的 Discussions 社区。

## 社区规则

1. 尊重他人
2. 保持友善
3. 分享有价值的内容

## 如何参与

- 在 **Q&A** 中提问
- 在 **Ideas** 中分享想法
- 在 **Show and tell** 中展示你的作品

期待你的参与！ 🎉
```

5. 点击 **Start discussion**

### 6. 创建问答

1. 点击 **Q&A** 分类
2. 点击 **New discussion**
3. 填写标题：`如何配置项目？`
4. 填写内容：

```markdown
# 问题

我在配置项目时遇到以下问题...

## 已尝试的方法

1. 方法1
2. 方法2

## 期望结果

希望得到关于配置的指导。
```

### 7. 回答问题

1. 打开刚才创建的 Q&A
2. 在评论框中回答：

```markdown
# 解决方案

你可以按照以下步骤配置：

1. 步骤一
2. 步骤二
3. 步骤三

希望这对你有帮助！
```

3. 点击 **Comment**

### 8. 标记最佳答案

1. 在回答上点击 **...** 菜单
2. 选择 **Mark as answer**

最佳答案会突出显示。

### 9. 创建 Ideas 讨论

1. 点击 **Ideas** 分类
2. 点击 **New discussion**
3. 填写标题：`建议：添加暗黑模式`
4. 填写内容：

```markdown
# 功能建议

## 描述

建议添加暗黑模式支持，提升用户体验。

## 动机

- 保护用户眼睛
- 符合现代 UI 趋势
- 提升可访问性

## 实现方案

使用 CSS 变量和 JavaScript 切换主题。

## 截图

（如有设计稿可在此添加）

大家觉得怎么样？ 👍 或 👎
```

### 10. 使用标签

1. 在 Discussion 右侧点击 **Labels**
2. 添加标签：`enhancement`、`community`

### 11. 转换为 Issue

如果讨论发现是 bug：
1. 点击 **Convert to issue**
2. 填写 Issue 信息
3. 点击 **Create issue**

### 12. 锁定讨论

如果讨论已完成：
1. 点击 **Lock discussion**
2. 选择锁定原因
3. 点击 **Lock this discussion**

## 管理最佳实践

### 分类管理

1. **定期清理**：删除无意义的内容
2. **合并相似讨论**：避免重复
3. **更新分类**：根据需要调整

### 社区管理

1. **及时回复**：保持社区活跃
2. **标记答案**：帮助后来者
3. **鼓励参与**：感谢贡献者
4. **设置规则**：维护讨论秩序

## 知识点

- 启用 Discussions
- 配置分类
- 创建和管理讨论
- Q&A 最佳答案
- 标签使用
- 讨论转换
- 锁定讨论

## 相关资源

- [GitHub Discussions 官方文档](https://docs.github.com/en/discussions)
- [使用讨论进行社区管理](https://docs.github.com/en/discussions/collaborating-with-your-community-using-discussions)
