# GitHub Models AI/ML 平台

## 什么是 GitHub Models？

GitHub Models 是 GitHub 提供的 AI 模型平台，让你可以直接在 GitHub 上使用和测试各种 AI 模型。

## 支持的模型

| 模型 | 提供商 | 用途 |
|------|--------|------|
| GPT-4o | OpenAI | 通用对话、代码生成 |
| GPT-4o-mini | OpenAI | 轻量级任务 |
| Claude 3.5 Sonnet | Anthropic | 代码分析、对话 |
| Llama 3.1 | Meta | 开源通用模型 |
| Mistral | Mistral AI | 欧洲开源模型 |
| Phi-3 | Microsoft | 轻量级模型 |
| GitLab Foundation | GitLab | 代码相关任务 |

## 快速开始

### 通过 API 使用

```bash
# 设置 API Key
export GITHUB_TOKEN="your-github-token"

# 调用 GPT-4o
curl -X POST "https://models.inference.ai.azure.com/chat/completions" \
  -H "Authorization: Bearer $GITHUB_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-4o",
    "messages": [
      {"role": "user", "content": "Hello, who are you?"}
    ]
  }'
```

### 使用 Python SDK

```python
from openai import OpenAI

client = OpenAI(
    base_url="https://models.inference.ai.azure.com",
    api_key=os.environ["GITHUB_TOKEN"],
)

response = client.chat.completions.create(
    model="gpt-4o",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "Write a Python function to sort a list"}
    ]
)

print(response.choices[0].message.content)
```

### 在 GitHub Actions 中使用

```yaml
# .github/workflows/ai-review.yml
name: AI Code Review

on:
  pull_request:

permissions:
  contents: read
  pull-requests: write

jobs:
  ai-review:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    
    - name: Get PR diff
      id: diff
      run: |
        DIFF=$(gh pr diff ${{ github.event.pull_request.number }})
        echo "diff<<EOF" >> $GITHUB_OUTPUT
        echo "$DIFF" >> $GITHUB_OUTPUT
        echo "EOF" >> $GITHUB_OUTPUT
    
    - name: AI Review
      run: |
        curl -X POST "https://models.inference.ai.azure.com/chat/completions" \
          -H "Authorization: Bearer ${{ secrets.GITHUB_TOKEN }}" \
          -H "Content-Type: application/json" \
          -d '{
            "model": "gpt-4o",
            "messages": [
              {"role": "system", "content": "Review this code change and provide feedback on bugs, improvements, and best practices."},
              {"role": "user", "content": "${{ steps.diff.outputs.diff }}"}
            ]
          }'
```

## 使用场景

### 1. 自动代码审查

```python
def ai_code_review(diff: str) -> str:
    """使用 AI 审查代码变更"""
    prompt = f"""
    审查以下代码变更，提供：
    1. 潜在的 bug
    2. 性能问题
    3. 安全漏洞
    4. 代码风格建议
    
    代码变更：
    {diff}
    """
    
    response = client.chat.completions.create(
        model="gpt-4o",
        messages=[{"role": "user", "content": prompt}]
    )
    
    return response.choices[0].message.content
```

### 2. 自动生成文档

```python
def generate_docs(code: str) -> str:
    """为代码生成文档"""
    prompt = f"""
    为以下代码生成详细的 API 文档，包括：
    1. 函数描述
    2. 参数说明
    3. 返回值
    4. 使用示例
    
    代码：
    {code}
    """
    
    response = client.chat.completions.create(
        model="gpt-4o",
        messages=[{"role": "user", "content": prompt}]
    )
    
    return response.choices[0].message.content
```

### 3. 测试用例生成

```python
def generate_tests(code: str) -> str:
    """为代码生成测试用例"""
    prompt = f"""
    为以下代码生成完整的测试用例，包括：
    1. 正常情况测试
    2. 边界情况测试
    3. 错误情况测试
    
    使用 pytest 框架。
    
    代码：
    {code}
    """
    
    response = client.chat.completions.create(
        model="gpt-4o",
        messages=[{"role": "user", "content": prompt}]
    )
    
    return response.choices[0].message.content
```

### 4. 自然语言转 SQL

```python
def nl_to_sql(question: str, schema: str) -> str:
    """将自然语言转换为 SQL"""
    prompt = f"""
    根据以下数据库 schema，将自然语言问题转换为 SQL 查询。
    
    Schema：
    {schema}
    
    问题：{question}
    """
    
    response = client.chat.completions.create(
        model="gpt-4o",
        messages=[{"role": "user", "content": prompt}]
    )
    
    return response.choices[0].message.content
```

## 定价

| 模型 | 每 1000 输入 token | 每 1000 输出 token |
|------|-------------------|-------------------|
| GPT-4o | $0.005 | $0.015 |
| GPT-4o-mini | $0.00015 | $0.0006 |
| Claude 3.5 Sonnet | $0.003 | $0.015 |
| Llama 3.1 | 免费 | 免费 |

## 最佳实践

1. **选择合适的模型**：根据任务复杂度选择模型
2. **优化提示词**：清晰的提示词获得更好的结果
3. **控制成本**：监控 API 使用量
4. **错误处理**：处理 API 调用失败的情况
5. **缓存结果**：缓存相同的请求以减少调用

## 常见问题

### Q: GitHub Models 和 GitHub Copilot 的区别？
A: GitHub Models 是 API 平台，可以直接调用模型；Copilot 是集成在 IDE 中的 AI 助手。

### Q: 如何提高 API 调用速度？
A: 使用流式响应、批量请求、缓存结果。

### Q: 模型生成的内容准确吗？
A: 模型生成的内容可能不准确，建议人工审查，特别是对于关键任务。

## 相关资源

- [GitHub Models 文档](https://docs.github.com/en/github-models)
- [模型列表](https://docs.github.com/en/github-models/available-models)
- [API 参考](https://docs.github.com/en/github-models/reference)

---

**上一篇：[GitHub Advanced Security](W10-advanced-security.md) | 下一篇：[Monorepo 管理](W12-monorepo.md)**
