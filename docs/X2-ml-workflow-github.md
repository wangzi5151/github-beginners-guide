# GitHub 上的机器学习工作流

> 本文档面向中国开发者，全面介绍如何在 GitHub 上构建高效的机器学习开发工作流。

---

## 目录

1. [GitHub + Jupyter Notebook 协作](#1-github--jupyter-notebook-协作)
2. [GitHub Models 使用指南](#2-github-models-使用指南)
3. [ML 项目结构最佳实践](#3-ml-项目结构最佳实践)
4. [DVC 数据版本控制](#4-dvc-数据版本控制)
5. [ML Pipeline 与 GitHub Actions 集成](#5-ml-pipeline-与-github-actions-集成)
6. [模型注册与部署](#6-模型注册与部署)
7. [MLflow 与 GitHub 集成](#7-mlflow-与-github-集成)
8. [Hugging Face + GitHub 工作流](#8-hugging-face--github-工作流)
9. [GPU Runner 与自托管 Runner](#9-gpu-runner-与自托管-runner)
10. [ML 项目的 CI/CD 最佳实践](#10-ml-项目的-cicd-最佳实践)
11. [大模型项目管理](#11-大模型项目管理)
12. [AI 安全与负责任的 AI 开发](#12-ai-安全与负责任的-ai-开发)
13. [国内 ML 开发者工具链](#13-国内-ml-开发者工具链)

---

## 1. GitHub + Jupyter Notebook 协作

### 1.1 Jupyter Notebook 的版本控制挑战

Jupyter Notebook（.ipynb 文件）本质上是 JSON 格式的文件，包含代码、输出、元数据等信息。直接使用 Git 进行版本控制会遇到一些问题：

```text
常见问题：
1. 输出结果（如图片、大段文本）导致文件过大
2. 执行顺序混乱导致 diff 难以阅读
3. 元数据频繁变化产生噪音
4. 合并冲突难以解决
```

### 1.2 配置 Git 优化 Notebook 版本控制

```bash
# 项目根目录创建 .gitattributes 文件
*.ipynb filter=strip-notebook-output

# 配置 Git filter
git config --global filter.strip-notebook-output.clean 'jupyter nbconvert --ClearOutputPreprocessor.enabled=True --to=notebook --stdin --stdout --log-level=ERROR'
git config --global filter.strip-notebook-output.smudge cat
git config --global filter.strip-notebook-output.required true
```

```bash
# 使用 nbstripout 工具（推荐）
pip install nbstripout

# 在项目中启用
nbstripout --install

# .gitattributes 自动生成：
*.ipynb filter=strip-notebook-output
*.ipynb diff=ipynb
```

### 1.3 Notebook 友好的项目结构

```
ml-project/
├── notebooks/
│   ├── 01_data_exploration.ipynb
│   ├── 02_feature_engineering.ipynb
│   ├── 03_model_training.ipynb
│   ├── 04_evaluation.ipynb
│   └── 05_inference.ipynb
├── src/
│   ├── data/
│   │   ├── __init__.py
│   │   ├── dataset.py
│   │   └── preprocessing.py
│   ├── models/
│   │   ├── __init__.py
│   │   ├── base_model.py
│   │   └── custom_model.py
│   ├── training/
│   │   ├── __init__.py
│   │   ├── trainer.py
│   │   └── callbacks.py
│   └── utils/
│       ├── __init__.py
│       ├── metrics.py
│       └── visualization.py
├── configs/
│   ├── train_config.yaml
│   └── model_config.yaml
├── data/
│   ├── raw/
│   ├── processed/
│   └── features/
├── models/
│   └── checkpoints/
├── tests/
├── requirements.txt
├── pyproject.toml
└── README.md
```

### 1.4 使用 ReviewNB 进行 Notebook Review

```text
ReviewNB 是 GitHub 的 Notebook 代码审查工具：

功能：
- 可视化 Notebook diff
- 支持行级评论
- 渲染图表和输出
- 与 GitHub PR 集成

安装：
1. 访问 reviewnb.com
2. 安装 GitHub App
3. 在仓库设置中启用

使用：
- 创建 PR 时自动显示 Notebook diff
- 可以在特定 cell 上添加评论
- 支持查看历史版本
```

### 1.5 JupyterLab Git 扩展

```bash
# 安装 JupyterLab Git 扩展
pip install jupyterlab-git

# 配置 JupyterLab
jupyter labextension install @jupyterlab/git

# 在 JupyterLab 中使用
# 左侧边栏会出现 Git 图标
# 支持 commit、push、pull 等操作
# 支持查看 diff 和历史
```

---

## 2. GitHub Models 使用指南

### 2.1 什么是 GitHub Models

GitHub Models 是 GitHub 提供的 AI 模型测试平台，允许开发者直接在 GitHub 上测试和评估各种 AI 模型。

```text
支持的模型类别：
1. 语言模型（LLM）
   - GPT-4o、GPT-4o mini
   - Claude 3.5 Sonnet、Claude 3 Haiku
   - Llama 3.1、Mistral Large

2. 嵌入模型（Embedding）
   - OpenAI text-embedding-3-small
   - Cohere embed-v3

3. 图像模型
   - DALL-E 3
   - Stable Diffusion

4. 语音模型
   - Whisper（语音识别）
   - TTS（文本转语音）
```

### 2.2 使用 GitHub Models 测试模型

```text
访问方式：
1. 访问 github.com/marketplace/models
2. 选择想要测试的模型
3. 在 Playground 中输入提示
4. 查看模型响应

Playground 功能：
- 调整模型参数（temperature、top_p 等）
- 对比多个模型的响应
- 保存和分享提示模板
- 查看 API 调用示例
```

### 2.3 通过 API 使用 GitHub Models

```python
# 使用 OpenAI SDK 调用 GitHub Models
from openai import OpenAI

# 初始化客户端（使用 GitHub Token）
client = OpenAI(
    base_url="https://models.inference.ai.azure.com",
    api_key="your-github-token"
)

# 调用 GPT-4o 模型
response = client.chat.completions.create(
    model="gpt-4o",
    messages=[
        {"role": "system", "content": "你是一个有帮助的AI助手。"},
        {"role": "user", "content": "请解释什么是机器学习？"}
    ],
    temperature=0.7,
    max_tokens=1000
)

print(response.choices[0].message.content)
```

```python
# 使用 Azure AI SDK
from azure.ai.inference import ChatCompletionsClient
from azure.ai.inference.models import SystemMessage, UserMessage
from azure.core.credentials import AzureKeyCredential

client = ChatCompletionsClient(
    endpoint="https://models.inference.ai.azure.com",
    credential=AzureKeyCredential("your-github-token")
)

response = client.complete(
    messages=[
        SystemMessage(content="你是一个有帮助的AI助手。"),
        UserMessage(content="请解释什么是深度学习？")
    ],
    model="gpt-4o"
)

print(response.choices[0].message.content)
```

### 2.4 模型评估与对比

```python
# 模型评估脚本
import json
from openai import OpenAI

def evaluate_model(client, model_name, test_cases):
    """评估模型在测试用例上的表现"""
    results = []
    
    for case in test_cases:
        response = client.chat.completions.create(
            model=model_name,
            messages=[
                {"role": "system", "content": case["system_prompt"]},
                {"role": "user", "content": case["input"]}
            ],
            temperature=0.0  # 使用确定性输出
        )
        
        output = response.choices[0].message.content
        results.append({
            "input": case["input"],
            "expected": case["expected"],
            "actual": output,
            "correct": case["expected"].lower() in output.lower()
        })
    
    accuracy = sum(1 for r in results if r["correct"]) / len(results)
    return {
        "model": model_name,
        "accuracy": accuracy,
        "results": results
    }

# 测试用例
test_cases = [
    {
        "input": "什么是梯度下降？",
        "expected": "优化算法",
        "system_prompt": "用一句话回答"
    },
    {
        "input": "什么是过拟合？",
        "expected": "泛化能力",
        "system_prompt": "用一句话回答"
    }
]

# 对比多个模型
models = ["gpt-4o", "gpt-4o-mini", "claude-3-5-sonnet"]
client = OpenAI(
    base_url="https://models.inference.ai.azure.com",
    api_key="your-github-token"
)

for model in models:
    result = evaluate_model(client, model, test_cases)
    print(f"{model}: {result['accuracy']:.2%} 准确率")
```

---

## 3. ML 项目结构最佳实践

### 3.1 标准项目结构

```text
ml-project/
├── .github/
│   ├── workflows/
│   │   ├── train.yml         # 训练流水线
│   │   ├── evaluate.yml      # 评估流水线
│   │   └── deploy.yml        # 部署流水线
│   ├── ISSUE_TEMPLATE/
│   │   ├── bug_report.md
│   │   └── feature_request.md
│   └── CODEOWNERS
├── configs/
│   ├── data_config.yaml      # 数据配置
│   ├── model_config.yaml     # 模型配置
│   ├── train_config.yaml     # 训练配置
│   └── eval_config.yaml      # 评估配置
├── data/
│   ├── raw/                  # 原始数据（不提交到 Git）
│   ├── processed/            # 处理后的数据
│   └── features/             # 特征数据
├── docs/
│   ├── data_dictionary.md    # 数据字典
│   ├── model_card.md         # 模型卡
│   └── experiments.md        # 实验记录
├── models/
│   ├── checkpoints/          # 模型检查点
│   └── final/                # 最终模型
├── notebooks/
│   ├── exploration/          # 探索性分析
│   ├── training/             # 训练实验
│   └── evaluation/           # 评估分析
├── scripts/
│   ├── data_download.py      # 数据下载脚本
│   ├── preprocess.py         # 数据预处理
│   ├── train.py              # 训练脚本
│   ├── evaluate.py           # 评估脚本
│   └── predict.py            # 推理脚本
├── src/
│   ├── __init__.py
│   ├── data/
│   │   ├── __init__.py
│   │   ├── dataset.py        # 数据集类
│   │   ├── dataloader.py     # 数据加载器
│   │   └── transforms.py     # 数据变换
│   ├── models/
│   │   ├── __init__.py
│   │   ├── base.py           # 基础模型类
│   │   ├── layers.py         # 自定义层
│   │   └── architectures.py  # 模型架构
│   ├── training/
│   │   ├── __init__.py
│   │   ├── trainer.py        # 训练器
│   │   ├── losses.py         # 损失函数
│   │   └── optimizers.py     # 优化器
│   ├── evaluation/
│   │   ├── __init__.py
│   │   ├── metrics.py        # 评估指标
│   │   └── evaluator.py      # 评估器
│   └── utils/
│       ├── __init__.py
│       ├── io.py             # 输入输出工具
│       ├── logging.py        # 日志工具
│       └── visualization.py  # 可视化工具
├── tests/
│   ├── unit/
│   │   ├── test_data.py
│   │   ├── test_models.py
│   │   └── test_training.py
│   └── integration/
│       └── test_pipeline.py
├── .dvc/                     # DVC 配置
├── .env.example              # 环境变量模板
├── .gitignore
├── .pre-commit-config.yaml
├── dvc.yaml                  # DVC 流水线
├── Makefile                  # 常用命令
├── pyproject.toml            # 项目配置
├── requirements.txt          # 依赖列表
├── requirements-dev.txt      # 开发依赖
└── README.md                 # 项目文档
```

### 3.2 配置管理

```yaml
# configs/train_config.yaml
project:
  name: "my-ml-project"
  version: "1.0.0"
  description: "项目描述"

data:
  train_path: "data/processed/train.parquet"
  val_path: "data/processed/val.parquet"
  test_path: "data/processed/test.parquet"
  batch_size: 32
  num_workers: 4

model:
  architecture: "transformer"
  hidden_size: 768
  num_layers: 12
  num_heads: 12
  dropout: 0.1

training:
  epochs: 100
  learning_rate: 0.0001
  weight_decay: 0.01
  warmup_steps: 1000
  gradient_clip: 1.0
  save_every: 10
  eval_every: 5

logging:
  level: "INFO"
  wandb_project: "my-ml-project"
  log_dir: "logs/"
```

```python
# src/utils/config.py
from dataclasses import dataclass
from typing import Optional
import yaml
from pathlib import Path

@dataclass
class DataConfig:
    train_path: str
    val_path: str
    test_path: str
    batch_size: int = 32
    num_workers: int = 4

@dataclass
class ModelConfig:
    architecture: str
    hidden_size: int = 768
    num_layers: int = 12
    num_heads: int = 12
    dropout: float = 0.1

@dataclass
class TrainingConfig:
    epochs: int = 100
    learning_rate: float = 1e-4
    weight_decay: float = 0.01
    warmup_steps: int = 1000
    gradient_clip: float = 1.0

@dataclass
class Config:
    project: dict
    data: DataConfig
    model: ModelConfig
    training: TrainingConfig
    
    @classmethod
    def from_yaml(cls, path: str) -> "Config":
        """从 YAML 文件加载配置"""
        with open(path) as f:
            config_dict = yaml.safe_load(f)
        
        return cls(
            project=config_dict["project"],
            data=DataConfig(**config_dict["data"]),
            model=ModelConfig(**config_dict["model"]),
            training=TrainingConfig(**config_dict["training"])
        )
    
    def save(self, path: str):
        """保存配置到 YAML 文件"""
        with open(path, "w") as f:
            yaml.dump(self.__dict__, f, default_flow_style=False)
```

### 3.3 Makefile 管理常用命令

```makefile
# Makefile
.PHONY: setup data train evaluate clean

# 安装依赖
setup:
	pip install -r requirements.txt
	pre-commit install

# 下载和处理数据
data:
	python scripts/data_download.py
	python scripts/preprocess.py

# 训练模型
train:
	python scripts/train.py --config configs/train_config.yaml

# 评估模型
evaluate:
	python scripts/evaluate.py --config configs/eval_config.yaml

# 运行测试
test:
	pytest tests/ -v

# 代码质量检查
lint:
	ruff check src/ scripts/ tests/
	mypy src/

# 清理生成的文件
clean:
	rm -rf data/processed/
	rm -rf models/checkpoints/*
	rm -rf logs/*

# 完整流水线
all: setup data train evaluate

# DVC 命令
dvc-repro:
	dvc repro

dvc-push:
	dvc push

dvc-pull:
	dvc pull
```

---

## 4. DVC 数据版本控制

### 4.1 什么是 DVC

DVC（Data Version Control）是一个开源的数据版本控制工具，专门用于机器学习项目。它解决了 ML 项目中数据和模型文件版本控制的问题。

```text
DVC 的核心功能：
1. 数据版本控制：像 Git 管理代码一样管理数据
2. 流水线管理：定义和重现 ML 流水线
3. 实验管理：跟踪和对比实验结果
4. 远程存储：支持多种云存储后端
5. 模型注册：管理模型版本和元数据
```

### 4.2 DVC 安装与配置

```bash
# 安装 DVC
pip install dvc

# 安装特定存储后端
pip install dvc-s3      # AWS S3
pip install dvc-gs      # Google Cloud Storage
pip install dvc-azure   # Azure Blob Storage
pip install dvc-oss     # 阿里云 OSS

# 初始化 DVC
cd my-ml-project
dvc init

# 配置远程存储（以 S3 为例）
dvc remote add -d storage s3://my-bucket/dvc-store
dvc remote modify storage access_key_id YOUR_ACCESS_KEY
dvc remote modify storage secret_access_key YOUR_SECRET_KEY

# 配置阿里云 OSS
dvc remote add -d oss-storage oss://my-bucket/dvc-store
dvc remote modify oss-storage oss_key_id YOUR_KEY_ID
dvc remote modify oss-storage oss_key_secret YOUR_KEY_SECRET
dvc remote modify oss-storage oss_endpoint oss-cn-hangzhou.aliyuncs.com
```

### 4.3 数据版本控制实践

```bash
# 添加数据文件到 DVC 跟踪
dvc add data/raw/train.csv
dvc add data/raw/test.csv

# 这会创建 .dvc 文件
# data/raw/train.csv.dvc
# data/raw/test.csv.dvc

# 提交 DVC 文件到 Git
git add data/raw/*.dvc .gitignore
git commit -m "添加训练和测试数据"

# 推送数据到远程存储
dvc push

# 拉取数据
dvc pull

# 查看数据版本历史
dvc dag
git log --oneline

# 切换到特定版本
git checkout v1.0
dvc checkout
```

### 4.4 DVC 流水线定义

```yaml
# dvc.yaml
stages:
  prepare:
    cmd: python scripts/preprocess.py --config configs/data_config.yaml
    deps:
      - data/raw/
      - scripts/preprocess.py
      - configs/data_config.yaml
    outs:
      - data/processed/train.parquet
      - data/processed/val.parquet
      - data/processed/test.parquet
    metrics:
      - data_stats.json:
          cache: false

  train:
    cmd: python scripts/train.py --config configs/train_config.yaml
    deps:
      - data/processed/
      - src/models/
      - src/training/
      - configs/train_config.yaml
    outs:
      - models/checkpoints/best_model.pt
    metrics:
      - train_metrics.json:
          cache: false
    plots:
      - logs/train_loss.csv:
          x: step
          y: loss

  evaluate:
    cmd: python scripts/evaluate.py --config configs/eval_config.yaml
    deps:
      - models/checkpoints/best_model.pt
      - data/processed/test.parquet
      - scripts/evaluate.py
    metrics:
      - eval_metrics.json:
          cache: false
    plots:
      - logs/confusion_matrix.png
      - logs/roc_curve.png
```

### 4.5 实验管理

```bash
# 运行实验
dvc exp run

# 查看实验历史
dvc exp show

# 对比实验
dvc exp diff

# 创建新实验（修改参数）
dvc exp run -S train_config.yaml:training.learning_rate=0.001
dvc exp run -S train_config.yaml:training.batch_size=64

# 命名实验
dvc exp run --name "lr-0.001-bs-64"

# 应用实验结果
dvc exp apply exp-name

# 删除实验
dvc exp remove exp-name
```

---

## 5. ML Pipeline 与 GitHub Actions 集成

### 5.1 训练流水线自动化

```yaml
# .github/workflows/train.yml
name: ML Training Pipeline

on:
  push:
    branches: [main]
    paths:
      - 'src/**'
      - 'configs/**'
      - 'scripts/**'
      - 'data/**/*.dvc'
  workflow_dispatch:
    inputs:
      epochs:
        description: 'Number of epochs'
        default: '100'
      learning_rate:
        description: 'Learning rate'
        default: '0.0001'

jobs:
  prepare-data:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'
          
      - name: Install dependencies
        run: |
          pip install -r requirements.txt
          
      - name: Setup DVC
        uses: iterative/setup-dvc@v1
        with:
          version: '3.x'
          
      - name: Pull data
        run: dvc pull
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          
      - name: Prepare data
        run: python scripts/preprocess.py --config configs/data_config.yaml
        
      - name: Upload processed data
        uses: actions/upload-artifact@v4
        with:
          name: processed-data
          path: data/processed/

  train-model:
    needs: prepare-data
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Download processed data
        uses: actions/download-artifact@v4
        with:
          name: processed-data
          path: data/processed/
          
      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'
          
      - name: Install dependencies
        run: pip install -r requirements.txt
        
      - name: Train model
        run: |
          python scripts/train.py \
            --config configs/train_config.yaml \
            --epochs ${{ github.event.inputs.epochs || '100' }} \
            --learning-rate ${{ github.event.inputs.learning_rate || '0.0001' }}
            
      - name: Upload model
        uses: actions/upload-artifact@v4
        with:
          name: trained-model
          path: models/checkpoints/best_model.pt

  evaluate-model:
    needs: train-model
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Download model
        uses: actions/download-artifact@v4
        with:
          name: trained-model
          path: models/checkpoints/
          
      - name: Download processed data
        uses: actions/download-artifact@v4
        with:
          name: processed-data
          path: data/processed/
          
      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'
          
      - name: Install dependencies
        run: pip install -r requirements.txt
        
      - name: Evaluate model
        run: python scripts/evaluate.py --config configs/eval_config.yaml
        
      - name: Upload evaluation results
        uses: actions/upload-artifact@v4
        with:
          name: evaluation-results
          path: eval_metrics.json

  register-model:
    needs: evaluate-model
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v4
      
      - name: Download model
        uses: actions/download-artifact@v4
        with:
          name: trained-model
          path: models/checkpoints/
          
      - name: Download evaluation results
        uses: actions/download-artifact@v4
        with:
          name: evaluation-results
          
      - name: Register model
        run: |
          python scripts/register_model.py \
            --model-path models/checkpoints/best_model.pt \
            --metrics eval_metrics.json \
            --version ${{ github.sha }}
```

### 5.2 模型评估流水线

```yaml
# .github/workflows/evaluate.yml
name: Model Evaluation

on:
  pull_request:
    paths:
      - 'src/models/**'
      - 'configs/model_config.yaml'
  schedule:
    - cron: '0 0 * * 0'  # 每周日运行

jobs:
  evaluate:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        dataset: ['test', 'validation', 'holdout']
        
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'
          
      - name: Install dependencies
        run: pip install -r requirements.txt
        
      - name: Pull data and model
        run: |
          dvc pull data/processed/${{ matrix.dataset }}.parquet
          dvc pull models/checkpoints/best_model.pt
          
      - name: Run evaluation
        run: |
          python scripts/evaluate.py \
            --dataset ${{ matrix.dataset }} \
            --output results/
            
      - name: Compare with baseline
        run: |
          python scripts/compare_results.py \
            --current results/metrics.json \
            --baseline baseline_metrics.json
            
      - name: Comment on PR
        if: github.event_name == 'pull_request'
        uses: actions/github-script@v7
        with:
          script: |
            const fs = require('fs');
            const metrics = JSON.parse(fs.readFileSync('results/metrics.json', 'utf8'));
            const body = `## 模型评估结果 (${{ matrix.dataset }})
            
            | 指标 | 值 | 基线 | 变化 |
            |------|-----|------|------|
            | Accuracy | ${metrics.accuracy} | - | - |
            | F1 Score | ${metrics.f1} | - | - |
            | AUC-ROC | ${metrics.auc_roc} | - | - |
            `;
            
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: body
            });
```

---

## 6. 模型注册与部署

### 6.1 GitHub Container Registry 模型注册

```python
# scripts/register_model.py
import argparse
import json
import datetime
from pathlib import Path
import hashlib

class ModelRegistry:
    def __init__(self, registry_path: str = "models/registry"):
        self.registry_path = Path(registry_path)
        self.registry_path.mkdir(parents=True, exist_ok=True)
        
    def register_model(
        self,
        model_path: str,
        metrics: dict,
        version: str,
        description: str = "",
        tags: list[str] = None
    ) -> dict:
        """注册模型到模型仓库"""
        model_path = Path(model_path)
        
        # 计算模型文件的哈希值
        model_hash = self._compute_hash(model_path)
        
        # 创建模型元数据
        metadata = {
            "version": version,
            "registered_at": datetime.datetime.now().isoformat(),
            "model_path": str(model_path),
            "model_hash": model_hash,
            "metrics": metrics,
            "description": description,
            "tags": tags or [],
            "framework": "pytorch",  # 或 tensorflow, onnx 等
            "input_format": "tensor",
            "output_format": "class_probabilities"
        }
        
        # 保存元数据
        version_dir = self.registry_path / version
        version_dir.mkdir(parents=True, exist_ok=True)
        
        with open(version_dir / "metadata.json", "w") as f:
            json.dump(metadata, f, indent=2)
            
        # 创建 latest 软链接
        latest_link = self.registry_path / "latest"
        if latest_link.exists():
            latest_link.unlink()
        latest_link.symlink_to(version_dir)
        
        print(f"模型已注册: {version}")
        return metadata
        
    def get_model_info(self, version: str) -> dict:
        """获取模型信息"""
        metadata_path = self.registry_path / version / "metadata.json"
        with open(metadata_path) as f:
            return json.load(f)
            
    def list_models(self) -> list[dict]:
        """列出所有注册的模型"""
        models = []
        for version_dir in self.registry_path.iterdir():
            if version_dir.is_dir() and version_dir.name != "latest":
                metadata_path = version_dir / "metadata.json"
                if metadata_path.exists():
                    with open(metadata_path) as f:
                        models.append(json.load(f))
        return sorted(models, key=lambda x: x["registered_at"], reverse=True)
        
    def _compute_hash(self, file_path: Path) -> str:
        """计算文件的 SHA256 哈希值"""
        sha256 = hashlib.sha256()
        with open(file_path, "rb") as f:
            for chunk in iter(lambda: f.read(8192), b""):
                sha256.update(chunk)
        return sha256.hexdigest()

if __name__ == "__main__":
    parser = argparse.ArgumentParser(description="注册模型到模型仓库")
    parser.add_argument("--model-path", required=True, help="模型文件路径")
    parser.add_argument("--metrics", required=True, help="评估指标 JSON 文件")
    parser.add_argument("--version", required=True, help="模型版本")
    parser.add_argument("--description", default="", help="模型描述")
    
    args = parser.parse_args()
    
    # 加载评估指标
    with open(args.metrics) as f:
        metrics = json.load(f)
        
    # 注册模型
    registry = ModelRegistry()
    metadata = registry.register_model(
        model_path=args.model_path,
        metrics=metrics,
        version=args.version,
        description=args.description
    )
    
    print(json.dumps(metadata, indent=2))
```

### 6.2 模型部署到 GitHub Packages

```yaml
# .github/workflows/deploy-model.yml
name: Deploy Model to GitHub Packages

on:
  push:
    tags:
      - 'v*'

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
      
    steps:
      - uses: actions/checkout@v4
      
      - name: Log in to GitHub Container Registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
          
      - name: Build and push model image
        run: |
          # 创建包含模型的 Docker 镜像
          docker build \
            --tag ghcr.io/${{ github.repository }}/model:${{ github.ref_name }} \
            --tag ghcr.io/${{ github.repository }}/model:latest \
            --build-arg MODEL_VERSION=${{ github.ref_name }} \
            .
          docker push ghcr.io/${{ github.repository }}/model:${{ github.ref_name }}
          docker push ghcr.io/${{ github.repository }}/model:latest
```

```dockerfile
# Dockerfile
FROM python:3.11-slim

WORKDIR /app

# 安装依赖
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# 复制模型代码
COPY src/ src/
COPY models/ models/
COPY scripts/ scripts/

# 复制模型文件
ARG MODEL_VERSION
COPY models/checkpoints/best_model.pt /app/models/best_model.pt

# 复制推理脚本
COPY scripts/serve.py /app/serve.py

# 暴露端口
EXPOSE 8000

# 启动推理服务
CMD ["python", "serve.py", "--model-path", "/app/models/best_model.pt"]
```

---

## 7. MLflow 与 GitHub 集成

### 7.1 MLflow 配置

```python
# src/training/trainer_with_mlflow.py
import mlflow
import mlflow.pytorch
from mlflow.tracking import MlflowClient
import torch
from pathlib import Path

class MLflowTrainer:
    def __init__(self, config):
        self.config = config
        
        # 配置 MLflow
        mlflow.set_tracking_uri(config.mlflow_tracking_uri)
        mlflow.set_experiment(config.experiment_name)
        
        self.client = MlflowClient()
        
    def train(self, model, train_loader, val_loader):
        """训练模型并记录到 MLflow"""
        with mlflow.start_run(run_name=self.config.run_name) as run:
            # 记录参数
            mlflow.log_params({
                "learning_rate": self.config.learning_rate,
                "batch_size": self.config.batch_size,
                "epochs": self.config.epochs,
                "optimizer": self.config.optimizer,
                "model_architecture": self.config.model_architecture
            })
            
            # 训练循环
            for epoch in range(self.config.epochs):
                train_loss = self._train_epoch(model, train_loader)
                val_loss, val_metrics = self._validate(model, val_loader)
                
                # 记录指标
                mlflow.log_metrics({
                    "train_loss": train_loss,
                    "val_loss": val_loss,
                    **val_metrics
                }, step=epoch)
                
                # 保存检查点
                if epoch % self.config.save_every == 0:
                    checkpoint_path = f"checkpoints/model_epoch_{epoch}.pt"
                    torch.save(model.state_dict(), checkpoint_path)
                    mlflow.log_artifact(checkpoint_path)
                    
            # 保存最终模型
            mlflow.pytorch.log_model(
                model,
                "model",
                registered_model_name=self.config.model_name
            )
            
            # 记录模型卡
            mlflow.log_artifact("docs/model_card.md")
            
            return run.info.run_id
            
    def _train_epoch(self, model, train_loader):
        """训练一个 epoch"""
        model.train()
        total_loss = 0
        
        for batch in train_loader:
            loss = self._compute_loss(model, batch)
            loss.backward()
            self._optimizer.step()
            self._optimizer.zero_grad()
            total_loss += loss.item()
            
        return total_loss / len(train_loader)
        
    def _validate(self, model, val_loader):
        """验证模型"""
        model.eval()
        total_loss = 0
        all_predictions = []
        all_targets = []
        
        with torch.no_grad():
            for batch in val_loader:
                loss, predictions, targets = self._compute_validation(model, batch)
                total_loss += loss.item()
                all_predictions.extend(predictions)
                all_targets.extend(targets)
                
        # 计算评估指标
        metrics = self._compute_metrics(all_predictions, all_targets)
        
        return total_loss / len(val_loader), metrics
```

### 7.2 MLflow 与 GitHub Actions 集成

```yaml
# .github/workflows/train-with-mlflow.yml
name: Train with MLflow Tracking

on:
  push:
    branches: [main]

jobs:
  train:
    runs-on: ubuntu-latest
    
    services:
      mlflow:
        image: ghcr.io/mlflow/mlflow:v2.x
        ports:
          - 5000:5000
        options: >-
          --health-cmd "curl -f http://localhost:5000/health || exit 1"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
          
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'
          
      - name: Install dependencies
        run: |
          pip install -r requirements.txt
          pip install mlflow boto3
          
      - name: Configure MLflow
        run: |
          mlflow server \
            --backend-store-uri sqlite:///mlflow.db \
            --default-artifact-root ./mlruns \
            --host 0.0.0.0 \
            --port 5000 &
          sleep 5
          
      - name: Train model
        env:
          MLFLOW_TRACKING_URI: http://localhost:5000
        run: |
          python scripts/train.py \
            --config configs/train_config.yaml \
            --mlflow-tracking-uri http://localhost:5000
            
      - name: Upload MLflow artifacts
        uses: actions/upload-artifact@v4
        with:
          name: mlflow-runs
          path: mlruns/
          
      - name: Deploy model to MLflow Registry
        if: github.ref == 'refs/heads/main'
        env:
          MLFLOW_TRACKING_URI: http://localhost:5000
        run: |
          python scripts/register_to_mlflow.py \
            --model-name "production-model" \
            --stage "Production"
```

---

## 8. Hugging Face + GitHub 工作流

### 8.1 Hugging Face Hub 集成

```python
# src/models/huggingface_integration.py
from transformers import AutoModel, AutoTokenizer
from huggingface_hub import HfApi, HfFolder, Repository
import torch

class HuggingFaceIntegration:
    def __init__(self, repo_name: str, token: str = None):
        self.repo_name = repo_name
        self.api = HfApi()
        self.token = token or HfFolder.get_token()
        
    def push_model(self, model, tokenizer, commit_message: str = "Update model"):
        """推送模型到 Hugging Face Hub"""
        # 保存模型和分词器
        model.save_pretrained("temp_model")
        tokenizer.save_pretrained("temp_model")
        
        # 推送到 Hub
        self.api.upload_folder(
            folder_path="temp_model",
            repo_id=self.repo_name,
            token=self.token,
            commit_message=commit_message
        )
        
    def load_model(self):
        """从 Hugging Face Hub 加载模型"""
        model = AutoModel.from_pretrained(self.repo_name)
        tokenizer = AutoTokenizer.from_pretrained(self.repo_name)
        return model, tokenizer
        
    def create_model_card(self, model_card_content: str):
        """创建模型卡"""
        with open("README.md", "w") as f:
            f.write(model_card_content)
            
        self.api.upload_file(
            path_or_fileobj="README.md",
            path_in_repo="README.md",
            repo_id=self.repo_name,
            token=self.token,
            commit_message="Update model card"
        )
```

### 8.2 GitHub Actions 自动推送到 Hugging Face

```yaml
# .github/workflows/push-to-huggingface.yml
name: Push Model to Hugging Face

on:
  push:
    tags:
      - 'model-v*'

jobs:
  push-model:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'
          
      - name: Install dependencies
        run: |
          pip install -r requirements.txt
          pip install transformers huggingface_hub
          
      - name: Pull trained model
        run: dvc pull models/checkpoints/
        
      - name: Push to Hugging Face
        env:
          HUGGING_FACE_HUB_TOKEN: ${{ secrets.HUGGING_FACE_HUB_TOKEN }}
        run: |
          python scripts/push_to_huggingface.py \
            --model-path models/checkpoints/best_model.pt \
            --repo-name ${{ secrets.HF_REPO_NAME }} \
            --version ${{ github.ref_name }}
```

### 8.3 使用 Hugging Face Spaces 展示模型

```yaml
# .github/workflows/deploy-to-spaces.yml
name: Deploy to Hugging Face Spaces

on:
  push:
    branches: [main]
    paths:
      - 'app/**'

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'
          
      - name: Install dependencies
        run: pip install -r app/requirements.txt
        
      - name: Deploy to Spaces
        env:
          HF_TOKEN: ${{ secrets.HF_TOKEN }}
        run: |
          huggingface-cli login --token $HF_TOKEN
          git clone https://huggingface.co/spaces/${{ secrets.HF_SPACE_NAME }} space_repo
          cp -r app/* space_repo/
          cd space_repo
          git add .
          git commit -m "Deploy from GitHub Actions"
          git push
```

---

## 9. GPU Runner 与自托管 Runner

### 9.1 GitHub GPU Runner 配置

```yaml
# 使用 GitHub 提供的 GPU Runner
jobs:
  train-gpu:
    runs-on: ubuntu-latest-gpu  # GitHub 提供的 GPU Runner
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'
          
      - name: Install CUDA dependencies
        run: |
          pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118
          
      - name: Verify GPU
        run: |
          python -c "import torch; print(f'CUDA available: {torch.cuda.is_available()}'); print(f'GPU count: {torch.cuda.device_count()}')"
          
      - name: Train model
        run: |
          python scripts/train.py --config configs/train_config.yaml --device cuda
```

### 9.2 自托管 GPU Runner 设置

```yaml
# .github/workflows/self-hosted-gpu.yml
name: Train on Self-Hosted GPU

on:
  push:
    branches: [main]

jobs:
  train:
    runs-on: [self-hosted, gpu, linux]
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup environment
        run: |
          source /opt/conda/etc/profile.d/conda.sh
          conda activate ml-project
          
      - name: Pull data
        run: dvc pull
        
      - name: Train with GPU
        env:
          CUDA_VISIBLE_DEVICES: "0,1"
        run: |
          python scripts/train.py \
            --config configs/train_config.yaml \
            --device cuda \
            --num-gpus 2
            
      - name: Upload results
        uses: actions/upload-artifact@v4
        with:
          name: training-results
          path: |
            models/checkpoints/
            logs/
```

### 9.3 自托管 Runner 安装脚本

```bash
#!/bin/bash
# setup-gpu-runner.sh

# 安装必要的软件
sudo apt-get update
sudo apt-get install -y \
    curl \
    git \
    jq \
    build-essential \
    libssl-dev \
    libffi-dev \
    python3-dev

# 安装 NVIDIA 驱动（如果没有）
if ! command -v nvidia-smi &> /dev/null; then
    sudo apt-get install -y nvidia-driver-535
    sudo reboot
fi

# 安装 Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER

# 安装 NVIDIA Container Toolkit
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg
curl -s -L https://nvidia.github.io/libnvidia-container/$distribution/libnvidia-container.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
sudo apt-get update
sudo apt-get install -y nvidia-container-toolkit
sudo systemctl restart docker

# 下载 GitHub Actions Runner
mkdir actions-runner && cd actions-runner
curl -o actions-runner-linux-x64.tar.gz -L https://github.com/actions/runner/releases/latest/download/actions-runner-linux-x64.tar.gz
tar xzf actions-runner-linux-x64.tar.gz

# 配置 Runner
./config.sh \
    --url https://github.com/YOUR_ORG/YOUR_REPO \
    --token YOUR_TOKEN \
    --labels gpu,self-hosted,linux \
    --name gpu-runner-01

# 安装为服务
sudo ./svc.sh install
sudo ./svc.sh start
```

---

## 10. ML 项目的 CI/CD 最佳实践

### 10.1 代码质量检查

```yaml
# .github/workflows/code-quality.yml
name: Code Quality

on:
  pull_request:
    branches: [main]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'
          
      - name: Install linters
        run: |
          pip install ruff mypy pytest pytest-cov
          
      - name: Run Ruff
        run: ruff check src/ scripts/ tests/
        
      - name: Run MyPy
        run: mypy src/ --ignore-missing-imports
        
      - name: Run tests
        run: pytest tests/ -v --cov=src --cov-report=xml
        
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          file: ./coverage.xml
```

### 10.2 数据验证

```yaml
# .github/workflows/data-validation.yml
name: Data Validation

on:
  push:
    paths:
      - 'data/**/*.dvc'
      - 'scripts/preprocess.py'

jobs:
  validate-data:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'
          
      - name: Install dependencies
        run: pip install -r requirements.txt great_expectations
        
      - name: Pull data
        run: dvc pull
        
      - name: Validate data quality
        run: |
          python scripts/validate_data.py \
            --data-path data/raw/ \
            --expectations configs/data_expectations.json
            
      - name: Generate data report
        run: |
          python scripts/generate_data_report.py \
            --output data_report.html
            
      - name: Upload data report
        uses: actions/upload-artifact@v4
        with:
          name: data-report
          path: data_report.html
```

### 10.3 模型性能测试

```python
# tests/test_model_performance.py
import pytest
import torch
from src.models import load_model
from src.evaluation import ModelEvaluator

class TestModelPerformance:
    """模型性能测试"""
    
    @pytest.fixture
    def model(self):
        """加载测试模型"""
        return load_model("models/checkpoints/best_model.pt")
        
    @pytest.fixture
    def test_data(self):
        """加载测试数据"""
        return load_test_data("data/processed/test.parquet")
        
    def test_accuracy_threshold(self, model, test_data):
        """测试准确率是否达到阈值"""
        evaluator = ModelEvaluator(model)
        metrics = evaluator.evaluate(test_data)
        
        assert metrics["accuracy"] >= 0.95, \
            f"准确率 {metrics['accuracy']:.4f} 低于阈值 0.95"
            
    def test_inference_latency(self, model, test_data):
        """测试推理延迟"""
        sample_input = test_data[0]["input"]
        
        # 预热
        for _ in range(10):
            model(sample_input)
            
        # 测量延迟
        import time
        latencies = []
        for _ in range(100):
            start = time.time()
            model(sample_input)
            latencies.append(time.time() - start)
            
        avg_latency = sum(latencies) / len(latencies)
        p99_latency = sorted(latencies)[98]
        
        assert avg_latency < 0.05, \
            f"平均延迟 {avg_latency:.4f}s 超过阈值 0.05s"
        assert p99_latency < 0.1, \
            f"P99延迟 {p99_latency:.4f}s 超过阈值 0.1s"
            
    def test_model_size(self, model):
        """测试模型大小"""
        model_size = sum(p.numel() for p in model.parameters()) * 4 / 1024 / 1024  # MB
        
        assert model_size < 500, \
            f"模型大小 {model_size:.2f}MB 超过阈值 500MB"
            
    def test_memory_usage(self, model, test_data):
        """测试内存使用"""
        import psutil
        import os
        
        process = psutil.Process(os.getpid())
        initial_memory = process.memory_info().rss / 1024 / 1024  # MB
        
        # 批量推理
        for batch in test_data.batches(batch_size=32):
            model(batch["input"])
            
        peak_memory = process.memory_info().rss / 1024 / 1024  # MB
        memory_increase = peak_memory - initial_memory
        
        assert memory_increase < 2000, \
            f"内存使用增加 {memory_increase:.2f}MB 超过阈值 2000MB"
```

### 10.4 完整的 CI/CD 流程

```yaml
# .github/workflows/ml-cicd.yml
name: ML CI/CD Pipeline

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  # 阶段 1: 代码质量检查
  code-quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Lint and test
        run: |
          pip install ruff pytest
          ruff check .
          pytest tests/unit/ -v

  # 阶段 2: 数据验证
  data-validation:
    runs-on: ubuntu-latest
    needs: code-quality
    steps:
      - uses: actions/checkout@v4
      - name: Validate data
        run: python scripts/validate_data.py

  # 阶段 3: 训练
  training:
    runs-on: ubuntu-latest-gpu
    needs: data-validation
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v4
      - name: Train model
        run: python scripts/train.py --config configs/train_config.yaml
      - name: Upload model
        uses: actions/upload-artifact@v4
        with:
          name: trained-model
          path: models/checkpoints/

  # 阶段 4: 评估
  evaluation:
    runs-on: ubuntu-latest
    needs: training
    steps:
      - uses: actions/checkout@v4
      - name: Download model
        uses: actions/download-artifact@v4
        with:
          name: trained-model
      - name: Evaluate model
        run: python scripts/evaluate.py
      - name: Performance tests
        run: pytest tests/performance/ -v

  # 阶段 5: 部署
  deploy:
    runs-on: ubuntu-latest
    needs: evaluation
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v4
      - name: Deploy model
        run: python scripts/deploy.py
```

---

## 11. 大模型项目管理

### 11.1 大模型项目的特殊挑战

```text
大模型项目（如 LLM 微调）的特殊性：

1. 数据规模大
   - 训练数据可能达到 TB 级别
   - 需要高效的数据处理流水线
   - 数据版本管理更加复杂

2. 计算资源需求高
   - 需要多 GPU 训练
   - 训练时间可能长达数天
   - 成本控制很重要

3. 模型版本管理
   - 模型文件可能达到数十 GB
   - 需要专门的存储方案
   - 版本追溯很重要

4. 实验管理
   - 需要跟踪大量超参数
   - 实验对比分析复杂
   - 需要自动化工具支持
```

### 11.2 大模型项目结构

```text
llm-project/
├── data/
│   ├── raw/                  # 原始数据
│   ├── processed/            # 处理后的数据
│   ├── tokenized/            # 分词后的数据
│   └── cache/                # 缓存数据
├── models/
│   ├── base/                 # 基础模型
│   ├── finetuned/            # 微调后的模型
│   └── merged/               # 合并后的模型
├── configs/
│   ├── lora_config.yaml      # LoRA 配置
│   ├── training_config.yaml  # 训练配置
│   └── inference_config.yaml # 推理配置
├── scripts/
│   ├── prepare_dataset.py    # 数据准备
│   ├── finetune.py           # 微调脚本
│   ├── merge_adapters.py     # 合并 LoRA 权重
│   ├── evaluate.py           # 评估脚本
│   └── serve.py              # 推理服务
├── src/
│   ├── data/
│   │   ├── dataset.py        # 数据集类
│   │   └── collator.py       # 数据整理器
│   ├── models/
│   │   ├── lora.py           # LoRA 实现
│   │   └── quantization.py   # 量化工具
│   └── training/
│       ├── trainer.py        # 训练器
│       └── callbacks.py      # 回调函数
├── docker/
│   ├── Dockerfile.train      # 训练容器
│   └── Dockerfile.serve      # 推理容器
└── README.md
```

### 11.3 使用 LoRA 微调大模型

```python
# scripts/finetune.py
from transformers import AutoModelForCausalLM, AutoTokenizer, TrainingArguments
from peft import LoraConfig, get_peft_model, prepare_model_for_kbit_training
from trl import SFTTrainer
import torch

def finetune_with_lora(config):
    """使用 LoRA 微调大模型"""
    
    # 加载基础模型
    model = AutoModelForCausalLM.from_pretrained(
        config.base_model,
        torch_dtype=torch.float16,
        device_map="auto",
        load_in_4bit=True  # 4-bit 量化
    )
    
    tokenizer = AutoTokenizer.from_pretrained(config.base_model)
    tokenizer.pad_token = tokenizer.eos_token
    
    # 准备模型
    model = prepare_model_for_kbit_training(model)
    
    # 配置 LoRA
    lora_config = LoraConfig(
        r=config.lora_r,  # 秩
        lora_alpha=config.lora_alpha,
        target_modules=["q_proj", "v_proj", "k_proj", "o_proj"],
        lora_dropout=0.05,
        bias="none",
        task_type="CAUSAL_LM"
    )
    
    # 应用 LoRA
    model = get_peft_model(model, lora_config)
    model.print_trainable_parameters()
    
    # 训练参数
    training_args = TrainingArguments(
        output_dir=config.output_dir,
        num_train_epochs=config.epochs,
        per_device_train_batch_size=config.batch_size,
        gradient_accumulation_steps=config.gradient_accumulation,
        learning_rate=config.learning_rate,
        weight_decay=0.01,
        warmup_steps=100,
        logging_steps=10,
        save_steps=500,
        evaluation_strategy="steps",
        eval_steps=500,
        fp16=True,
        optim="paged_adamw_8bit",
        lr_scheduler_type="cosine",
        max_grad_norm=0.3
    )
    
    # 创建训练器
    trainer = SFTTrainer(
        model=model,
        args=training_args,
        train_dataset=train_dataset,
        eval_dataset=val_dataset,
        tokenizer=tokenizer,
        dataset_text_field="text",
        max_seq_length=config.max_seq_length,
        packing=True
    )
    
    # 开始训练
    trainer.train()
    
    # 保存 LoRA 权重
    model.save_pretrained(config.output_dir)
    
    return model

if __name__ == "__main__":
    from src.utils.config import Config
    config = Config.from_yaml("configs/training_config.yaml")
    finetune_with_lora(config)
```

### 11.4 GitHub Actions 大模型训练流水线

```yaml
# .github/workflows/llm-finetune.yml
name: LLM Fine-tuning

on:
  workflow_dispatch:
    inputs:
      base_model:
        description: 'Base model name'
        default: 'meta-llama/Llama-2-7b-hf'
      lora_r:
        description: 'LoRA rank'
        default: '16'
      epochs:
        description: 'Number of epochs'
        default: '3'

jobs:
  finetune:
    runs-on: [self-hosted, gpu, a100]
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup environment
        run: |
          source /opt/conda/etc/profile.d/conda.sh
          conda activate llm-project
          
      - name: Pull data
        run: dvc pull data/tokenized/
        
      - name: Fine-tune model
        env:
          WANDB_API_KEY: ${{ secrets.WANDB_API_KEY }}
          HUGGING_FACE_HUB_TOKEN: ${{ secrets.HF_TOKEN }}
        run: |
          python scripts/finetune.py \
            --base-model ${{ github.event.inputs.base_model }} \
            --lora-r ${{ github.event.inputs.lora_r }} \
            --epochs ${{ github.event.inputs.epochs }} \
            --output-dir models/finetuned/
            
      - name: Merge LoRA weights
        run: |
          python scripts/merge_adapters.py \
            --base-model ${{ github.event.inputs.base_model }} \
            --adapter-path models/finetuned/ \
            --output-path models/merged/
            
      - name: Evaluate model
        run: |
          python scripts/evaluate.py \
            --model-path models/merged/ \
            --output eval_results.json
            
      - name: Push to Hugging Face
        if: success()
        env:
          HF_TOKEN: ${{ secrets.HF_TOKEN }}
        run: |
          python scripts/push_to_huggingface.py \
            --model-path models/merged/ \
            --repo-name ${{ secrets.HF_REPO_NAME }}
```

---

## 12. AI 安全与负责任的 AI 开发

### 12.1 AI 安全检查清单

```text
AI 安全检查清单：

1. 数据安全
   □ 数据来源是否合法合规
   □ 是否包含个人隐私信息
   □ 数据是否经过脱敏处理
   □ 数据存储是否加密

2. 模型安全
   □ 模型是否经过对抗性测试
   □ 是否存在后门攻击风险
   □ 模型输出是否经过过滤
   □ 是否有模型水印机制

3. 部署安全
   □ API 是否有访问控制
   □ 是否有请求频率限制
   □ 是否有输入验证
   □ 是否有监控和告警

4. 合规性
   □ 是否符合相关法规（如《生成式人工智能服务管理暂行办法》）
   □ 是否有用户协议和隐私政策
   □ 是否有内容审核机制
   □ 是否有投诉处理流程
```

### 12.2 模型公平性检测

```python
# src/evaluation/fairness.py
import numpy as np
from collections import defaultdict

class FairnessEvaluator:
    """模型公平性评估器"""
    
    def __init__(self, model, protected_attributes: list[str]):
        self.model = model
        self.protected_attributes = protected_attributes
        
    def evaluate(self, dataset) -> dict:
        """评估模型公平性"""
        results = {}
        
        for attr in self.protected_attributes:
            # 按保护属性分组
            groups = self._group_by_attribute(dataset, attr)
            
            # 计算各组的性能指标
            group_metrics = {}
            for group_name, group_data in groups.items():
                predictions = self.model.predict(group_data["features"])
                metrics = self._compute_metrics(predictions, group_data["labels"])
                group_metrics[group_name] = metrics
                
            # 计算公平性指标
            results[attr] = {
                "group_metrics": group_metrics,
                "demographic_parity": self._demographic_parity(group_metrics),
                "equalized_odds": self._equalized_odds(group_metrics),
                "disparate_impact": self._disparate_impact(group_metrics)
            }
            
        return results
        
    def _demographic_parity(self, group_metrics: dict) -> float:
        """人口统计平等性"""
        positive_rates = {}
        for group, metrics in group_metrics.items():
            positive_rates[group] = metrics["positive_rate"]
            
        max_rate = max(positive_rates.values())
        min_rate = min(positive_rates.values())
        
        return min_rate / max_rate if max_rate > 0 else 1.0
        
    def _equalized_odds(self, group_metrics: dict) -> float:
        """机会平等性"""
        tpr_values = []
        fpr_values = []
        
        for group, metrics in group_metrics.items():
            tpr_values.append(metrics["true_positive_rate"])
            fpr_values.append(metrics["false_positive_rate"])
            
        tpr_diff = max(tpr_values) - min(tpr_values)
        fpr_diff = max(fpr_values) - min(fpr_values)
        
        return 1.0 - (tpr_diff + fpr_diff) / 2
        
    def _disparate_impact(self, group_metrics: dict) -> float:
        """差异影响比"""
        positive_rates = {}
        for group, metrics in group_metrics.items():
            positive_rates[group] = metrics["positive_rate"]
            
        rates = list(positive_rates.values())
        return min(rates) / max(rates) if max(rates) > 0 else 1.0
        
    def _group_by_attribute(self, dataset, attribute: str) -> dict:
        """按属性分组数据"""
        groups = defaultdict(lambda: {"features": [], "labels": []})
        
        for sample in dataset:
            group = sample[attribute]
            groups[group]["features"].append(sample["features"])
            groups[group]["labels"].append(sample["label"])
            
        return dict(groups)
        
    def _compute_metrics(self, predictions, labels) -> dict:
        """计算评估指标"""
        predictions = np.array(predictions)
        labels = np.array(labels)
        
        positive_rate = np.mean(predictions == 1)
        true_positive_rate = np.mean(predictions[labels == 1] == 1)
        false_positive_rate = np.mean(predictions[labels == 0] == 1)
        accuracy = np.mean(predictions == labels)
        
        return {
            "positive_rate": positive_rate,
            "true_positive_rate": true_positive_rate,
            "false_positive_rate": false_positive_rate,
            "accuracy": accuracy
        }
```

### 12.3 内容安全过滤

```python
# src/safety/content_filter.py
import re
from typing import Optional

class ContentSafetyFilter:
    """内容安全过滤器"""
    
    def __init__(self):
        # 敏感词库（示例，实际应该使用更完整的词库）
        self.blocked_patterns = [
            r"(暴力|血腥|色情)",
            r"(歧视|仇恨|侮辱)",
            r"(违法|犯罪|恐怖)",
            r"(政治敏感词1|政治敏感词2)"
        ]
        
        # 编译正则表达式
        self.compiled_patterns = [re.compile(p) for p in self.blocked_patterns]
        
    def check(self, text: str) -> tuple[bool, Optional[str]]:
        """检查文本是否安全
        
        Returns:
            (is_safe, reason): 是否安全，不安全的原因
        """
        for pattern in self.compiled_patterns:
            match = pattern.search(text)
            if match:
                return False, f"包含敏感内容: {match.group()}"
                
        return True, None
        
    def filter(self, text: str) -> str:
        """过滤文本中的敏感内容"""
        filtered_text = text
        
        for pattern in self.compiled_patterns:
            filtered_text = pattern.sub("***", filtered_text)
            
        return filtered_text
        
    def check_model_output(self, output: str, input_text: str = "") -> dict:
        """检查模型输出是否安全"""
        is_safe, reason = self.check(output)
        
        result = {
            "is_safe": is_safe,
            "original_output": output,
            "filtered_output": self.filter(output) if not is_safe else output
        }
        
        if reason:
            result["reason"] = reason
            
        return result
```

### 12.4 GitHub Actions 安全检查

```yaml
# .github/workflows/ai-safety.yml
name: AI Safety Check

on:
  pull_request:
    branches: [main]

jobs:
  safety-check:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'
          
      - name: Install dependencies
        run: |
          pip install -r requirements.txt
          pip install safety bandit
          
      - name: Check dependencies for vulnerabilities
        run: safety check
        
      - name: Run security linter
        run: bandit -r src/ -f json -o security_report.json || true
        
      - name: Run fairness tests
        run: pytest tests/fairness/ -v
        
      - name: Run safety tests
        run: pytest tests/safety/ -v
        
      - name: Upload security report
        uses: actions/upload-artifact@v4
        with:
          name: security-report
          path: security_report.json
```

---

## 13. 国内 ML 开发者工具链

### 13.1 国内云服务商 ML 平台

```text
主要的国内 ML 平台：

1. 阿里云 - PAI (Platform for AI)
   - 优势：功能全面，集成阿里云生态
   - 适用场景：企业级 ML 项目
   - 网址：https://pai.console.aliyun.com/

2. 腾讯云 - TI 平台
   - 优势：与腾讯生态集成好
   - 适用场景：游戏、社交领域的 AI 应用
   - 网址：https://cloud.tencent.com/product/ti

3. 华为云 - ModelArts
   - 优势：支持昇腾芯片，国产化
   - 适用场景：政企客户、国产化需求
   - 网址：https://www.huaweicloud.com/product/modelarts.html

4. 百度智能云 - BML
   - 优势：预置百度 AI 能力
   - 适用场景：NLP、CV 领域应用
   - 网址：https://cloud.baidu.com/product/bml

5. 字节跳动 - 火山引擎
   - 优势：字节内部 ML 经验沉淀
   - 适用场景：推荐系统、内容理解
   - 网址：https://www.volcengine.com/product/ml-platform
```

### 13.2 国内 MLOps 工具

```text
MLOps 工具对比：

1. MLflow（开源，国际）
   - 优势：社区活跃，功能完善
   - 劣势：中文文档少，需要自部署
   - 适用：技术能力强的团队

2. Kubeflow（开源，国际）
   - 优势：Kubernetes 原生，扩展性强
   - 劣势：部署复杂，学习曲线陡
   - 适用：大规模 ML 平台

3. AutoDL（国内）
   - 优势：GPU 租用便宜，操作简单
   - 劣势：功能相对简单
   - 适用：个人开发者、小团队

4. 趋动云（国内）
   - 优势：GPU 资源丰富，性价比高
   - 劣势：功能还在完善中
   - 适用：需要 GPU 资源的团队
```

### 13.3 国内数据存储方案

```python
# 数据存储配置示例

# 阿里云 OSS
import oss2

def setup_aliyun_oss():
    """配置阿里云 OSS"""
    auth = oss2.Auth('your-access-key-id', 'your-access-key-secret')
    bucket = oss2.Bucket(auth, 'https://oss-cn-hangzhou.aliyuncs.com', 'your-bucket-name')
    
    # 上传文件
    bucket.put_object('data/train.csv', open('data/train.csv', 'rb'))
    
    # 下载文件
    bucket.get_object_to_file('data/train.csv', 'downloaded_train.csv')

# 腾讯云 COS
from qcloud_cos import CosConfig, CosS3Client

def setup_tencent_cos():
    """配置腾讯云 COS"""
    config = CosConfig(
        Region='ap-guangzhou',
        SecretId='your-secret-id',
        SecretKey='your-secret-key'
    )
    client = CosS3Client(config)
    
    # 上传文件
    client.upload_file(
        Bucket='your-bucket-name',
        Key='data/train.csv',
        LocalFilePath='data/train.csv'
    )

# 华为云 OBS
from obs import ObsClient

def setup_huawei_obs():
    """配置华为云 OBS"""
    client = ObsClient(
        access_key_id='your-access-key-id',
        secret_access_key='your-secret-access-key',
        server='https://obs.cn-hangzhou.myhuaweicloud.com'
    )
    
    # 上传文件
    client.putFile(
        bucketName='your-bucket-name',
        objectKey='data/train.csv',
        file_path='data/train.csv'
    )
```

### 13.4 DVC 国内存储配置

```bash
# 配置阿里云 OSS 作为 DVC 远程存储
dvc remote add -d myremote oss://my-bucket/dvc-store
dvc remote modify myremote oss_key_id YOUR_KEY_ID
dvc remote modify myremote oss_key_secret YOUR_KEY_SECRET
dvc remote modify myremote oss_endpoint oss-cn-hangzhou.aliyuncs.com

# 配置腾讯云 COS
pip install dvc-cos
dvc remote add -d myremote cos://my-bucket/dvc-store
dvc remote modify myremote cos_secret_id YOUR_SECRET_ID
dvc remote modify myremote cos_secret_key YOUR_SECRET_KEY
dvc remote modify myremote cos_region ap-guangzhou

# 配置华为云 OBS
pip install dvc-obs
dvc remote add -d myremote obs://my-bucket/dvc-store
dvc remote modify myremote obs_access_key_id YOUR_KEY_ID
dvc remote modify myremote obs_secret_access_key YOUR_SECRET_KEY
dvc remote modify myremote obs_endpoint obs.cn-hangzhou.myhuaweicloud.com
```

### 13.5 国内 ML 开发最佳实践

```text
国内 ML 开发建议：

1. 网络优化
   - 使用国内镜像源（清华、阿里云）
   - 模型文件优先放在国内存储
   - 使用 CDN 加速数据传输

2. 合规性
   - 遵守《数据安全法》
   - 遵守《个人信息保护法》
   - 遵守《生成式人工智能服务管理暂行办法》
   - 做好数据出境安全评估

3. 成本控制
   - 选择合适的 GPU 实例
   - 使用 Spot 实例降低成本
   - 合理规划训练任务

4. 团队协作
   - 使用中文文档
   - 建立内部知识库
   - 定期技术分享
```

```python
# 配置国内 pip 镜像源
# ~/.pip/pip.conf
"""
[global]
index-url = https://mirrors.aliyun.com/pypi/simple/
trusted-host = mirrors.aliyun.com
"""

# 配置国内 Hugging Face 镜像
import os
os.environ["HF_ENDPOINT"] = "https://hf-mirror.com"

# 使用镜像下载模型
from transformers import AutoModel

model = AutoModel.from_pretrained("bert-base-chinese")
```

### 13.6 Makefile 国内优化配置

```makefile
# Makefile - 国内优化版本

# 配置镜像源
.PHONY: setup-mirrors
setup-mirrors:
	pip config set global.index-url https://mirrors.aliyun.com/pypi/simple/
	pip config set install.trusted-host mirrors.aliyun.com
	echo "export HF_ENDPOINT=https://hf-mirror.com" >> ~/.bashrc

# 安装依赖（使用国内镜像）
.PHONY: install
install:
	pip install -r requirements.txt -i https://mirrors.aliyun.com/pypi/simple/

# 下载模型（使用镜像）
.PHONY: download-model
download-model:
	HF_ENDPOINT=https://hf-mirror.com python scripts/download_model.py

# 配置 DVC 远程存储（阿里云 OSS）
.PHONY: setup-dvc
setup-dvc:
	dvc remote add -d myremote oss://my-bucket/dvc-store
	dvc remote modify myremote oss_key_id $(OSS_KEY_ID)
	dvc remote modify myremote oss_key_secret $(OSS_KEY_SECRET)
	dvc remote modify myremote oss_endpoint oss-cn-hangzhou.aliyuncs.com

# 训练（使用国内 W&B 替代方案）
.PHONY: train
train:
	python scripts/train.py --config configs/train_config.yaml --tracker none
```

---

## 总结

GitHub 上的机器学习工作流已经形成了一个完整的生态系统，从数据版本控制到模型部署，从实验管理到 CI/CD 自动化，都有成熟的工具和最佳实践可供参考。

**关键要点：**

1. **版本控制**：使用 Git + DVC 管理代码、数据和模型
2. **实验管理**：使用 MLflow 或 Weights & Biases 跟踪实验
3. **自动化**：使用 GitHub Actions 构建 ML 流水线
4. **协作**：使用 GitHub 的协作功能（Issues、PR、Code Review）
5. **部署**：使用 GitHub Packages 和 Container Registry 部署模型
6. **安全**：遵循 AI 安全最佳实践

**下一步行动：**

- 选择一个适合你项目的 ML 工具栈
- 建立标准化的项目结构
- 配置 CI/CD 流水线
- 建立实验管理流程
- 关注 AI 安全和合规性

---

> **文档版本：** v1.0  
> **最后更新：** 2025 年  
> **作者：** GitHub 中文开发者社区
