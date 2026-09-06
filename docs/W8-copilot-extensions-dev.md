# GitHub Copilot Extensions 开发

## 什么是 Copilot Extension？

Copilot Extension 让你可以构建自定义的 AI 助手，集成到 Copilot 的工作流中。

## 架构概览

```
用户 → GitHub Copilot → 你的 Extension API → 外部服务
```

## 创建 Extension

### 1. 注册 GitHub App

```bash
# 访问 https://github.com/settings/apps/new
# 配置：
# - GitHub Copilot: 发送回复 → 开启
# - Webhook URL: 你的 API 地址
# - 权限: 代码搜索、仓库读取
```

### 2. 创建 API 服务

```python
# app.py
from flask import Flask, request, jsonify
import hmac
import hashlib

app = Flask(__name__)

GITHUB_SECRET = "your-webhook-secret"

@app.route("/copilot", methods=["POST"])
def copilot_handler():
    # 验证签名
    signature = request.headers["X-Hub-Signature-256"]
    expected = "sha256=" + hmac.new(
        GITHUB_SECRET.encode(),
        request.data,
        hashlib.sha256
    ).hexdigest()
    
    if not hmac.compare_digest(signature, expected):
        return jsonify({"error": "Invalid signature"}), 401
    
    payload = request.json
    
    # 处理请求
    prompt = payload.get("prompt", "")
    context = payload.get("context", {})
    
    # 生成回复
    response = generate_response(prompt, context)
    
    return jsonify({
        "choices": [{
            "message": {
                "content": response
            }
        }]
    })

def generate_response(prompt, context):
    """你的 AI 逻辑"""
    # 这里可以调用任何外部 API
    # 例如：数据库查询、API 调用、文档搜索等
    return f"根据你的需求：{prompt}"

if __name__ == "__main__":
    app.run(port=8080)
```

### 3. 配置 Extension

```json
{
  "name": "my-extension",
  "description": "自定义 Copilot Extension",
  "hooks": {
    "copilot": "/copilot"
  },
  "permissions": {
    "repository": ["read"],
    "codespaces": ["read"]
  }
}
```

## 使用场景

### 1. 内部文档搜索

```python
@app.route("/copilot", methods=["POST"])
def search_docs():
    prompt = request.json["prompt"]
    
    # 搜索内部文档
    results = search_internal_docs(prompt)
    
    # 返回相关文档片段
    return jsonify({
        "choices": [{
            "message": {
                "content": format_doc_results(results)
            }
        }]
    })
```

### 2. 数据库查询

```python
@app.route("/copilot", methods=["POST"])
def query_database():
    prompt = request.json["prompt"]
    
    # 解析自然语言查询
    sql = natural_to_sql(prompt)
    
    # 执行查询
    results = db.execute(sql)
    
    return jsonify({
        "choices": [{
            "message": {
                "content": format_query_results(results)
            }
        }]
    })
```

### 3. API 集成

```python
@app.route("/copilot", methods=["POST"])
def integrate_api():
    prompt = request.json["prompt"]
    
    # 调用外部 API
    if "天气" in prompt:
        city = extract_city(prompt)
        weather = get_weather(city)
        return format_weather(weather)
    
    if "汇率" in prompt:
        currencies = extract_currencies(prompt)
        rate = get_exchange_rate(currencies)
        return format_rate(rate)
```

### 4. CI/CD 操作

```python
@app.route("/copilot", methods=["POST"])
def cicd_operations():
    prompt = request.json["prompt"]
    context = request.json["context"]
    
    if "部署" in prompt:
        # 触发部署
        deployment = trigger_deployment(context["repo"])
        return f"已触发部署：{deployment.url}"
    
    if "回滚" in prompt:
        # 执行回滚
        rollback = execute_rollback(context["repo"])
        return f"已回滚到版本：{rollback.version}"
```

## 部署

### 使用 Vercel

```json
// vercel.json
{
  "version": 2,
  "builds": [
    {
      "src": "app.py",
      "use": "@vercel/python"
    }
  ],
  "routes": [
    {
      "src": "/(.*)",
      "dest": "app.py"
    }
  ]
}
```

### 使用 Docker

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .

EXPOSE 8080

CMD ["python", "app.py"]
```

## 最佳实践

1. **快速响应**：保持响应时间 < 3 秒
2. **清晰输出**：使用 Markdown 格式化输出
3. **错误处理**：提供有用的错误信息
4. **日志记录**：记录请求和响应以便调试
5. **速率限制**：防止 API 滥用

## 相关资源

- [Copilot Extensions 文档](https://docs.github.com/en/copilot/extensions)
- [GitHub App 创建](https://docs.github.com/en/apps/creating-github-apps)
- [Webhook 开发](https://docs.github.com/en/webhooks)

---

**上一篇：[GitHub Copilot 进阶功能](W7-copilot-advanced.md) | 下一篇：[GitHub Actions 高级用法](W9-actions-advanced.md)**
