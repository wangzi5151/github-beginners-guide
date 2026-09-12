# 练习 27：使用 GitHub Models 测试 AI 模型

## 学习目标

完成本练习后，你将能够：

- 理解 GitHub Models 的概念和用途
- 使用 GitHub Models API 调用各种 AI 模型
- 比较不同模型的性能和输出质量
- 构建基于 AI 模型的简单应用程序
- 了解模型评估和选择的最佳实践

## 前置条件

- 拥有 GitHub 账户
- 已安装 Python 3.10+ 或 Node.js 18+
- 基本的 API 调用知识
- 了解 JSON 数据格式

## 背景知识

### 什么是 GitHub Models

GitHub Models 是 GitHub 提供的 AI 模型测试平台，允许开发者：

1. **免费测试 AI 模型**：无需付费即可体验各种大语言模型
2. **比较不同模型**：在同一平台上测试多个模型的输出
3. **快速原型开发**：使用 API 快速构建 AI 应用原型
4. **集成到工作流**：将模型集成到 GitHub Actions 和应用程序中

### 支持的模型类型

GitHub Models 提供多种模型供测试：

- **语言模型**：GPT-4、Claude、Llama 等
- **嵌入模型**：用于文本向量化
- **图像模型**：用于图像生成和理解
- **多模态模型**：支持文本和图像输入

---

## 练习步骤

### 第一部分：设置 GitHub Models 访问

#### 步骤 1：获取 GitHub Personal Access Token

1. 访问 GitHub Settings → Developer settings → Personal access tokens → Tokens (classic)
2. 点击 "Generate new token"
3. 选择权限：`read:user` 和 `models:read`
4. 生成并保存令牌

```bash
# 设置环境变量
export GITHUB_TOKEN="your_github_token_here"
```

#### 步骤 2：安装必要的依赖

**Python 环境：**

```bash
# 创建虚拟环境
python -m venv venv
source venv/bin/activate  # Linux/Mac
# 或 venv\Scripts\activate  # Windows

# 安装依赖
pip install openai requests python-dotenv
```

**Node.js 环境：**

```bash
# 初始化项目
mkdir github-models-demo && cd github-models-demo
npm init -y

# 安装依赖
npm install openai dotenv
```

### 第二部分：使用 Python 调用 GitHub Models

#### 步骤 3：创建基础调用脚本

创建文件 `basic_call.py`：

```python
import os
from openai import OpenAI
from dotenv import load_dotenv

# 加载环境变量
load_dotenv()

# 配置 GitHub Models 客户端
client = OpenAI(
    base_url="https://models.inference.ai.azure.com",
    api_key=os.getenv("GITHUB_TOKEN"),
)

def call_model(model_name, messages, temperature=0.7, max_tokens=1000):
    """
    调用指定的 AI 模型
    
    Args:
        model_name: 模型名称
        messages: 消息列表
        temperature: 温度参数（控制随机性）
        max_tokens: 最大生成令牌数
    
    Returns:
        模型的响应文本
    """
    try:
        response = client.chat.completions.create(
            model=model_name,
            messages=messages,
            temperature=temperature,
            max_tokens=max_tokens,
            top_p=0.95,
        )
        return response.choices[0].message.content
    except Exception as e:
        return f"调用失败: {str(e)}"

# 测试调用
if __name__ == "__main__":
    messages = [
        {"role": "system", "content": "你是一个有用的助手，用中文回答问题。"},
        {"role": "user", "content": "请解释什么是 GitHub Actions？"}
    ]
    
    response = call_model("gpt-4o-mini", messages)
    print("模型响应：")
    print(response)
```

#### 步骤 4：创建环境配置文件

创建文件 `.env`：

```env
GITHUB_TOKEN=your_github_token_here
```

创建文件 `.gitignore`：

```
.env
venv/
node_modules/
__pycache__/
```

#### 步骤 5：运行基础调用

```bash
python basic_call.py
```

预期输出：

```
模型响应：
GitHub Actions 是 GitHub 提供的持续集成和持续部署（CI/CD）平台。它允许开发者自动化软件开发工作流，包括构建、测试和部署等任务。

主要特点：
1. 基于事件触发：可以在 push、pull request、issue 等事件发生时自动运行
2. YAML 配置：使用 YAML 文件定义工作流
3. 矩阵构建：支持在多个操作系统和语言版本上并行测试
4. 市场共享：可以通过 GitHub Marketplace 共享和复用 Actions
5. 免费额度：公开仓库免费使用，私有仓库有免费额度
```

### 第三部分：比较不同模型

#### 步骤 6：创建模型比较脚本

创建文件 `compare_models.py`：

```python
import os
import time
import json
from openai import OpenAI
from dotenv import load_dotenv
from typing import Dict, List, Any

load_dotenv()

client = OpenAI(
    base_url="https://models.inference.ai.azure.com",
    api_key=os.getenv("GITHUB_TOKEN"),
)

# 可用模型列表
AVAILABLE_MODELS = [
    "gpt-4o-mini",
    "gpt-4o",
    "Meta-Llama-3.1-405B-Instruct",
    "Meta-Llama-3.1-70B-Instruct",
    "Mistral-large",
    "Phi-3-medium-4k-instruct",
]

def compare_models(
    prompt: str,
    models: List[str] = None,
    system_prompt: str = "你是一个有用的助手，用中文回答问题。",
) -> Dict[str, Any]:
    """
    比较多个模型对同一提示的响应
    
    Args:
        prompt: 用户提示
        models: 要比较的模型列表
        system_prompt: 系统提示
    
    Returns:
        包含各模型响应的字典
    """
    if models is None:
        models = AVAILABLE_MODELS[:3]  # 默认比较前3个模型
    
    results = {}
    messages = [
        {"role": "system", "content": system_prompt},
        {"role": "user", "content": prompt},
    ]
    
    for model in models:
        print(f"\n正在测试模型: {model}")
        print("-" * 50)
        
        start_time = time.time()
        
        try:
            response = client.chat.completions.create(
                model=model,
                messages=messages,
                temperature=0.7,
                max_tokens=500,
            )
            
            elapsed_time = time.time() - start_time
            content = response.choices[0].message.content
            
            results[model] = {
                "response": content,
                "time": round(elapsed_time, 2),
                "tokens": {
                    "prompt": response.usage.prompt_tokens,
                    "completion": response.usage.completion_tokens,
                    "total": response.usage.total_tokens,
                },
                "success": True,
            }
            
            print(f"响应时间: {elapsed_time:.2f} 秒")
            print(f"令牌使用: {response.usage.total_tokens}")
            print(f"响应预览: {content[:200]}...")
            
        except Exception as e:
            elapsed_time = time.time() - start_time
            results[model] = {
                "response": None,
                "time": round(elapsed_time, 2),
                "error": str(e),
                "success": False,
            }
            print(f"错误: {str(e)}")
    
    return results

def format_comparison_report(results: Dict[str, Any]) -> str:
    """生成比较报告"""
    report = "# 模型比较报告\n\n"
    
    # 性能摘要
    report += "## 性能摘要\n\n"
    report += "| 模型 | 响应时间 | 令牌数 | 状态 |\n"
    report += "|------|---------|--------|------|\n"
    
    for model, data in results.items():
        status = "✅ 成功" if data["success"] else "❌ 失败"
        time_str = f"{data['time']}s"
        tokens = data.get("tokens", {}).get("total", "N/A")
        report += f"| {model} | {time_str} | {tokens} | {status} |\n"
    
    # 详细响应
    report += "\n## 详细响应\n\n"
    for model, data in results.items():
        report += f"### {model}\n\n"
        if data["success"]:
            report += f"{data['response']}\n\n"
        else:
            report += f"错误: {data.get('error', '未知错误')}\n\n"
        report += "---\n\n"
    
    return report

# 主程序
if __name__ == "__main__":
    test_prompt = "请用简洁的语言解释机器学习、深度学习和人工智能之间的关系。"
    
    print("=" * 60)
    print("GitHub Models 比较测试")
    print("=" * 60)
    print(f"\n测试提示: {test_prompt}\n")
    
    results = compare_models(test_prompt)
    
    # 生成报告
    report = format_comparison_report(results)
    
    # 保存报告
    with open("model_comparison_report.md", "w", encoding="utf-8") as f:
        f.write(report)
    
    print("\n" + "=" * 60)
    print("比较报告已保存到 model_comparison_report.md")
    print("=" * 60)
    
    # 保存原始数据
    with open("model_comparison_data.json", "w", encoding="utf-8") as f:
        json.dump(results, f, ensure_ascii=False, indent=2)
    
    print("原始数据已保存到 model_comparison_data.json")
```

#### 步骤 7：运行模型比较

```bash
python compare_models.py
```

### 第四部分：构建简单的 AI 应用

#### 步骤 8：创建代码审查助手

创建文件 `code_reviewer.py`：

```python
import os
from openai import OpenAI
from dotenv import load_dotenv
from typing import Optional

load_dotenv()

client = OpenAI(
    base_url="https://models.inference.ai.azure.com",
    api_key=os.getenv("GITHUB_TOKEN"),
)

class CodeReviewer:
    """AI 代码审查助手"""
    
    def __init__(self, model: str = "gpt-4o-mini"):
        self.model = model
        self.system_prompt = """你是一个专业的代码审查专家。你的任务是：
1. 检查代码中的潜在问题和bug
2. 评估代码的可读性和可维护性
3. 提供改进建议
4. 检查安全漏洞
5. 优化性能建议

请用中文回答，并按照以下格式输出：
- 严重问题（必须修复）
- 建议改进（推荐修复）
- 代码风格（可选优化）
- 总体评价
"""
    
    def review_code(
        self,
        code: str,
        language: str = "python",
        context: Optional[str] = None,
    ) -> str:
        """
        审查代码
        
        Args:
            code: 要审查的代码
            language: 编程语言
            context: 额外上下文信息
        
        Returns:
            审查结果
        """
        user_message = f"请审查以下 {language} 代码：\n\n```{language}\n{code}\n```"
        
        if context:
            user_message += f"\n\n上下文信息：{context}"
        
        messages = [
            {"role": "system", "content": self.system_prompt},
            {"role": "user", "content": user_message},
        ]
        
        try:
            response = client.chat.completions.create(
                model=self.model,
                messages=messages,
                temperature=0.3,  # 低温度以获得更一致的输出
                max_tokens=2000,
            )
            return response.choices[0].message.content
        except Exception as e:
            return f"审查失败: {str(e)}"
    
    def suggest_fixes(self, code: str, issues: str) -> str:
        """
        根据发现的问题提供修复建议
        
        Args:
            code: 原始代码
            issues: 发现的问题描述
        
        Returns:
            修复后的代码
        """
        messages = [
            {
                "role": "system",
                "content": "你是一个代码修复专家。根据提供的问题描述，给出修复后的完整代码。",
            },
            {
                "role": "user",
                "content": f"原始代码：\n```python\n{code}\n```\n\n发现的问题：\n{issues}\n\n请提供修复后的完整代码。",
            },
        ]
        
        try:
            response = client.chat.completions.create(
                model=self.model,
                messages=messages,
                temperature=0.2,
                max_tokens=2000,
            )
            return response.choices[0].message.content
        except Exception as e:
            return f"修复建议失败: {str(e)}"


# 示例使用
if __name__ == "__main__":
    reviewer = CodeReviewer()
    
    # 示例代码
    sample_code = '''
def get_user_data(user_id):
    import sqlite3
    conn = sqlite3.connect('users.db')
    cursor = conn.cursor()
    query = f"SELECT * FROM users WHERE id = {user_id}"
    cursor.execute(query)
    result = cursor.fetchone()
    conn.close()
    return result

def process_data(data):
    if data != None:
        result = data[0] + data[1]
        return result
    else:
        return 0

class UserManager:
    def __init__(self):
        self.users = []
    
    def add_user(self, user):
        self.users.append(user)
        print("User added: " + str(user))
    
    def get_all_users(self):
        return self.users
'''
    
    print("=" * 60)
    print("AI 代码审查助手")
    print("=" * 60)
    
    # 审查代码
    print("\n正在审查代码...\n")
    review_result = reviewer.review_code(sample_code, "python", "这是一个用户管理模块")
    print(review_result)
    
    print("\n" + "=" * 60)
    print("获取修复建议...")
    print("=" * 60)
    
    # 获取修复建议
    fix_suggestion = reviewer.suggest_fixes(sample_code, review_result)
    print(f"\n{fix_suggestion}")
```

#### 步骤 9：创建文本摘要工具

创建文件 `text_summarizer.py`：

```python
import os
from openai import OpenAI
from dotenv import load_dotenv

load_dotenv()

client = OpenAI(
    base_url="https://models.inference.ai.azure.com",
    api_key=os.getenv("GITHUB_TOKEN"),
)

class TextSummarizer:
    """AI 文本摘要工具"""
    
    def __init__(self, model: str = "gpt-4o-mini"):
        self.model = model
    
    def summarize(
        self,
        text: str,
        max_length: int = 200,
        style: str = "concise",
    ) -> str:
        """
        生成文本摘要
        
        Args:
            text: 原始文本
            max_length: 摘要最大长度（字数）
            style: 摘要风格 ('concise', 'detailed', 'bullet_points')
        
        Returns:
            生成的摘要
        """
        style_prompts = {
            "concise": f"请用简洁的语言总结以下内容，不超过{max_length}字：",
            "detailed": f"请详细总结以下内容，包含关键信息，不超过{max_length}字：",
            "bullet_points": f"请用要点列表的形式总结以下内容，每个要点简洁明了：",
        }
        
        system_prompt = style_prompts.get(style, style_prompts["concise"])
        
        messages = [
            {"role": "system", "content": system_prompt},
            {"role": "user", "content": text},
        ]
        
        try:
            response = client.chat.completions.create(
                model=self.model,
                messages=messages,
                temperature=0.5,
                max_tokens=500,
            )
            return response.choices[0].message.content
        except Exception as e:
            return f"摘要生成失败: {str(e)}"
    
    def extract_key_points(self, text: str) -> list:
        """
        提取文本关键点
        
        Args:
            text: 原始文本
        
        Returns:
            关键点列表
        """
        messages = [
            {
                "role": "system",
                "content": "请从文本中提取关键点，每行一个，用 '-' 开头。",
            },
            {"role": "user", "content": text},
        ]
        
        try:
            response = client.chat.completions.create(
                model=self.model,
                messages=messages,
                temperature=0.3,
                max_tokens=500,
            )
            points = response.choices[0].message.content.strip().split("\n")
            return [p.strip("- ").strip() for p in points if p.strip()]
        except Exception as e:
            return [f"提取失败: {str(e)}"]


# 示例使用
if __name__ == "__main__":
    summarizer = TextSummarizer()
    
    sample_text = """
    GitHub Actions 是 GitHub 提供的持续集成和持续部署（CI/CD）服务。
    它允许开发者自动化软件开发工作流程，包括构建、测试、打包、发布和部署等任务。
    
    GitHub Actions 使用 YAML 语法定义工作流文件，这些文件存放在仓库的
    .github/workflows 目录下。每个工作流由一个或多个作业（jobs）组成，
    每个作业包含一系列步骤（steps）。
    
    工作流可以在特定事件发生时自动触发，例如代码推送到仓库、创建拉取请求、
    定时任务等。开发者也可以手动触发工作流。
    
    GitHub Actions 提供了丰富的内置 Actions，可以通过 GitHub Marketplace
    获取社区共享的 Actions。这大大简化了 CI/CD 流程的配置。
    
    此外，GitHub Actions 支持矩阵构建，可以在多个操作系统和语言版本上
    并行运行测试，确保代码的兼容性。
    """
    
    print("=" * 60)
    print("AI 文本摘要工具")
    print("=" * 60)
    
    # 生成不同风格的摘要
    print("\n1. 简洁摘要：")
    print("-" * 40)
    print(summarizer.summarize(sample_text, max_length=100, style="concise"))
    
    print("\n2. 详细摘要：")
    print("-" * 40)
    print(summarizer.summarize(sample_text, max_length=200, style="detailed"))
    
    print("\n3. 要点列表：")
    print("-" * 40)
    print(summarizer.summarize(sample_text, style="bullet_points"))
    
    print("\n4. 关键点提取：")
    print("-" * 40)
    key_points = summarizer.extract_key_points(sample_text)
    for i, point in enumerate(key_points, 1):
        print(f"{i}. {point}")
```

### 第五部分：在 GitHub Actions 中使用 GitHub Models

#### 步骤 10：创建使用 GitHub Models 的工作流

创建文件 `.github/workflows/ai-review.yml`：

```yaml
name: AI Code Review

on:
  pull_request:
    types: [opened, synchronize]

permissions:
  contents: read
  pull-requests: write

jobs:
  ai-review:
    runs-on: ubuntu-latest
    
    steps:
      - name: 检出代码
        uses: actions/checkout@v4
      
      - name: 设置 Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'
      
      - name: 安装依赖
        run: |
          pip install openai requests PyGithub
      
      - name: 获取 PR 变更
        id: changes
        uses: actions/github-script@v7
        with:
          script: |
            const { data: files } = await github.rest.pulls.listFiles({
              owner: context.repo.owner,
              repo: context.repo.repo,
              pull_number: context.issue.number,
            });
            
            const changes = files
              .filter(f => f.status !== 'removed')
              .map(f => ({
                filename: f.filename,
                patch: f.patch || '',
              }))
              .slice(0, 10);  // 限制最多10个文件
            
            core.setOutput('changes', JSON.stringify(changes));
      
      - name: AI 代码审查
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          python << 'EOF'
          import os
          import json
          from openai import OpenAI
          
          client = OpenAI(
              base_url="https://models.inference.ai.azure.com",
              api_key=os.getenv("GITHUB_TOKEN"),
          )
          
          changes = json.loads('${{ steps.changes.outputs.changes }}')
          
          reviews = []
          for change in changes:
              if not change['patch']:
                  continue
              
              messages = [
                  {
                      "role": "system",
                      "content": "你是代码审查专家。请用中文简要指出代码中的问题，每条不超过50字。如果没有问题，回复'代码看起来不错'。"
                  },
                  {
                      "role": "user",
                      "content": f"文件: {change['filename']}\n\n变更:\n{change['patch']}"
                  }
              ]
              
              try:
                  response = client.chat.completions.create(
                      model="gpt-4o-mini",
                      messages=messages,
                      temperature=0.3,
                      max_tokens=500,
                  )
                  review = response.choices[0].message.content
                  reviews.append(f"**{change['filename']}**:\n{review}")
              except Exception as e:
                  reviews.append(f"**{change['filename']}**: 审查失败 - {str(e)}")
          
          # 输出结果
          with open('review_result.md', 'w') as f:
              f.write("## AI 代码审查结果\n\n")
              f.write("\n\n---\n\n".join(reviews))
          
          EOF
      
      - name: 发布审查评论
        uses: actions/github-script@v7
        with:
          script: |
            const fs = require('fs');
            const review = fs.readFileSync('review_result.md', 'utf8');
            
            await github.rest.issues.createComment({
              owner: context.repo.owner,
              repo: context.repo.repo,
              issue_number: context.issue.number,
              body: review,
            });
```

---

## 验证练习结果

### 检查清单

完成练习后，请验证以下内容：

- [ ] 成功配置 GitHub Token
- [ ] 能够调用 GitHub Models API
- [ ] 模型比较脚本正常运行并生成报告
- [ ] 代码审查助手能够分析代码
- [ ] 文本摘要工具能够生成摘要
- [ ] GitHub Actions 工作流能够自动审查 PR

### 验证命令

```bash
# 测试 API 连接
python -c "
from openai import OpenAI
import os
client = OpenAI(base_url='https://models.inference.ai.azure.com', api_key=os.getenv('GITHUB_TOKEN'))
response = client.chat.completions.create(model='gpt-4o-mini', messages=[{'role': 'user', 'content': 'Hello'}], max_tokens=10)
print('API 连接成功:', response.choices[0].message.content)
"

# 运行模型比较
python compare_models.py

# 检查生成的报告
cat model_comparison_report.md
```

---

## 进阶挑战

### 挑战 1：构建多语言翻译助手

创建支持多种语言互译的工具：

```python
class TranslationAssistant:
    def translate(self, text, source_lang, target_lang):
        # 实现翻译功能
        pass
    
    def detect_language(self, text):
        # 实现语言检测
        pass
```

### 挑战 2：实现对话式 AI 助手

创建支持多轮对话的助手：

```python
class ConversationalAssistant:
    def __init__(self):
        self.conversation_history = []
    
    def chat(self, message):
        # 维护对话历史
        # 调用模型生成响应
        # 返回响应并更新历史
        pass
    
    def clear_history(self):
        self.conversation_history = []
```

### 挑战 3：创建文档生成器

使用 AI 自动生成 API 文档：

```python
def generate_api_docs(code):
    """
    从代码自动生成 API 文档
    """
    # 解析代码结构
    # 生成文档
    # 格式化输出
    pass
```

### 挑战 4：实现情感分析工具

创建分析文本情感的工具：

```python
class SentimentAnalyzer:
    def analyze(self, text):
        """
        分析文本情感
        返回: {
            'sentiment': 'positive' | 'negative' | 'neutral',
            'confidence': 0.95,
            'keywords': ['关键词1', '关键词2']
        }
        """
        pass
```

---

## 提示词工程最佳实践

### 提示词的基本结构

提示词工程是使用大语言模型的核心技能。一个好的提示词通常包含以下几个部分：系统提示词用于设定模型的角色和行为规范，例如指定模型扮演代码审查专家或技术文档翻译员。用户提示词包含具体的任务描述和输入数据。上下文信息为模型提供必要的背景知识，帮助模型更好地理解任务。输出格式要求明确指定期望的输出结构，例如使用表格、列表或特定的标记语言。

### 提示词优化技巧

提升模型输出质量有几个关键技巧。首先是具体化指令，避免模糊的描述，例如将"写一篇文章"改为"用300字介绍GitHub Actions的核心概念，面向初学者"。其次是提供示例，在提示词中给出输入输出的示例对，让模型学习期望的格式和风格。第三是分步思考，对于复杂任务，引导模型逐步推理，而不是直接给出答案。第四是角色设定，为模型指定一个专业角色，可以显著提升输出的专业性和一致性。第五是约束条件，明确指定输出的长度、语言、风格等限制条件。

### 模型选择指南

不同的模型适合不同的任务场景。对于简单的文本分类和信息提取任务，小型模型如 GPT-4o-mini 就足够，它的响应速度快且成本低。对于复杂的推理和创意写作任务，建议使用大型模型如 GPT-4o 或 Claude。对于代码生成和代码审查任务，需要选择在代码数据上训练过的模型。对于多语言任务，需要选择支持目标语言的模型。在实际应用中，建议先用小型模型进行原型验证，在确认效果后再考虑是否需要升级到大型模型。

### 令牌管理与成本控制

使用大语言模型时，令牌消耗直接影响成本。输入令牌包括系统提示词、用户提示词和上下文信息的总长度。输出令牌是模型生成的响应长度。为了优化成本，可以采取以下措施：精简系统提示词，去除冗余的描述。控制输出长度，使用 `max_tokens` 参数限制生成长度。使用上下文窗口管理，只保留最近几轮的对话历史。对于结构化输出任务，使用 JSON 模式可以减少无效的格式化文本。对于批量处理任务，合并请求减少调用次数。

### 错误处理与重试策略

在生产环境中调用 AI 模型需要健壮的错误处理机制。常见的错误类型包括速率限制错误、服务不可用错误、上下文长度超限错误和内容过滤错误。对于速率限制错误，建议使用指数退避策略进行重试。对于服务不可用错误，可以配置多个模型作为备选方案。对于上下文长度超限，需要实现自动截断或摘要机制。对于内容过滤错误，需要调整输入内容或修改提示词。建议实现统一的错误处理中间件，集中管理所有模型调用的错误处理逻辑。

### 评估指标体系

建立科学的模型评估体系对于选择和优化模型至关重要。常见的评估指标包括准确性指标，衡量模型输出的正确性。相关性指标，衡量模型输出与用户需求的匹配程度。流畅性指标，衡量模型输出的语言质量。安全性指标，衡量模型输出是否存在有害或不当内容。效率指标，包括响应时间、令牌消耗和并发处理能力。建议构建一个包含多种类型测试用例的评估数据集，定期对模型进行基准测试。

### 数据隐私与安全

在使用 AI 模型时，数据隐私和安全是需要特别关注的问题。不要在提示词中包含敏感的个人信息、商业机密或凭证数据。了解模型提供商的数据使用政策，确认输入数据是否会被用于模型训练。对于企业应用场景，建议使用企业级 API 服务，通常提供更严格的数据保护承诺。实现数据脱敏机制，在发送给模型之前对敏感信息进行匿名化处理。定期审查模型的使用日志，确保没有敏感数据泄露。

### 本地模型与云端模型的权衡

选择本地部署模型还是使用云端 API 需要综合考虑多个因素。本地部署的优势包括数据不出本地网络、无网络延迟、无调用费用和完全可控。云端 API 的优势包括无需管理基础设施、持续获得模型更新、支持更大规模的并发和无需投入 GPU 硬件成本。对于开发和测试阶段，建议使用云端 API 快速验证。对于生产环境中的敏感数据处理，可以考虑本地部署。混合架构也是一种选择，将非敏感任务路由到云端 API，敏感任务使用本地模型。

---

## 常见问题

### Q1：GitHub Models 有使用限制吗？

是的，GitHub Models 有速率限制：
- 每分钟请求数限制
- 每天请求数限制
- 具体限制取决于你的 GitHub 计划

### Q2：如何选择合适的模型？

选择模型时考虑：
- **任务类型**：简单任务用小模型，复杂任务用大模型
- **响应速度**：小模型通常更快
- **成本**：大模型消耗更多令牌
- **质量**：在测试数据上比较不同模型

### Q3：如何处理 API 调用失败？

```python
import time
from tenacity import retry, stop_after_attempt, wait_exponential

@retry(stop=stop_after_attempt(3), wait=wait_exponential(multiplier=1, min=4, max=10))
def call_model_with_retry(client, model, messages):
    return client.chat.completions.create(model=model, messages=messages)
```

### Q4：如何优化模型调用成本？

1. 使用缓存避免重复调用
2. 选择合适的模型大小
3. 限制输出令牌数
4. 批量处理请求

---

## 延伸阅读

- [GitHub Models 官方文档](https://docs.github.com/en/github-models)
- [OpenAI API 文档](https://platform.openai.com/docs)
- [Prompt Engineering 指南](https://docs.github.com/en/github-models/prototyping-with-ai-models)

---

## 练习总结

通过本练习，你已经学会了：

1. ✅ 配置 GitHub Models 访问权限
2. ✅ 使用 Python 调用 GitHub Models API
3. ✅ 比较不同模型的性能和输出质量
4. ✅ 构建代码审查助手和文本摘要工具
5. ✅ 在 GitHub Actions 中集成 AI 模型

GitHub Models 为开发者提供了一个便捷的 AI 模型测试平台，建议多尝试不同的模型和提示词，找到最适合你用例的解决方案。
