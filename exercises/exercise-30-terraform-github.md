# 练习 30：Terraform + GitHub Actions 基础设施自动化

## 学习目标

完成本练习后，你将能够：

- 理解基础设施即代码（IaC）的概念和优势
- 编写基础的 Terraform 配置文件
- 使用 GitHub Actions 自动化基础设施部署
- 管理多个环境（开发、测试、生产）的基础设施
- 实现基础设施的版本控制和审计

## 前置条件

- 拥有 GitHub 仓库
- 拥有云服务账户（AWS、Azure 或 GCP）
- 了解基本的云服务概念
- 已完成 GitHub Actions 基础练习

## 背景知识

### 什么是基础设施即代码

基础设施即代码（Infrastructure as Code，IaC）是一种通过代码来管理和配置基础设施的方法：

1. **声明式配置**：定义期望的基础设施状态
2. **版本控制**：配置代码存放在版本控制系统中
3. **自动化部署**：通过工具自动创建和更新基础设施
4. **可重复性**：确保环境的一致性

### Terraform 简介

Terraform 是 HashiCorp 开发的基础设施即代码工具：

- **多云支持**：支持 AWS、Azure、GCP 等主流云平台
- **状态管理**：跟踪基础设施的当前状态
- **计划和应用**：先预览变更，再执行
- **模块化**：支持代码复用和组织

### GitHub Actions 集成

GitHub Actions 可以与 Terraform 结合实现：

- 自动化基础设施部署
- Pull Request 时自动计划
- 多环境管理
- 安全扫描和合规检查

---

## 练习步骤

### 第一部分：Terraform 基础配置

#### 步骤 1：安装 Terraform

**macOS：**

```bash
# 使用 Homebrew
brew install terraform

# 或使用 tfenv 版本管理器
brew install tfenv
tfenv install 1.7.0
tfenv use 1.7.0
```

**Linux：**

```bash
# 下载 Terraform
wget https://releases.hashicorp.com/terraform/1.7.0/terraform_1.7.0_linux_amd64.zip

# 解压
unzip terraform_1.7.0_linux_amd64.zip

# 移动到 PATH
sudo mv terraform /usr/local/bin/

# 验证安装
terraform version
```

**Windows：**

```bash
# 使用 Chocolatey
choco install terraform

# 或使用 Scoop
scoop install terraform

# 或使用 tfenv
scoop install tfenv
tfenv install 1.7.0
tfenv use 1.7.0
```

#### 步骤 2：创建项目目录结构

```bash
# 创建项目结构
mkdir -p terraform-github-demo/{environments/{dev,staging,prod},modules/network,scripts}

# 查看目录结构
tree terraform-github-demo/
```

预期输出：

```
terraform-github-demo/
├── environments/
│   ├── dev/
│   ├── staging/
│   └── prod/
├── modules/
│   └── network/
└── scripts/
```

#### 步骤 3：创建基础 Terraform 配置

创建文件 `terraform-github-demo/main.tf`：

```hcl
# 配置 Terraform 提供商
terraform {
  required_version = ">= 1.7.0"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
  
  # 远程状态存储（推荐）
  backend "s3" {
    bucket         = "my-terraform-state-bucket"
    key            = "terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-locks"
  }
}

# 配置 AWS 提供商
provider "aws" {
  region = var.aws_region
  
  default_tags {
    tags = {
      Environment = var.environment
      Project     = var.project_name
      ManagedBy   = "Terraform"
      Repository  = var.repository
    }
  }
}

# 定义变量
variable "aws_region" {
  description = "AWS 区域"
  type        = string
  default     = "us-east-1"
}

variable "environment" {
  description = "环境名称"
  type        = string
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "环境必须是 dev、staging 或 prod。"
  }
}

variable "project_name" {
  description = "项目名称"
  type        = string
  default     = "my-project"
}

variable "repository" {
  description = "GitHub 仓库地址"
  type        = string
  default     = "github.com/username/repo"
}

# 输出值
output "environment" {
  description = "当前环境"
  value       = var.environment
}

output "region" {
  description = "AWS 区域"
  value       = var.aws_region
}
```

#### 步骤 4：创建网络模块

创建文件 `terraform-github-demo/modules/network/main.tf`：

```hcl
# 网络模块 - 创建 VPC 和子网

variable "environment" {
  description = "环境名称"
  type        = string
}

variable "vpc_cidr" {
  description = "VPC CIDR 块"
  type        = string
  default     = "10.0.0.0/16"
}

variable "availability_zones" {
  description = "可用区列表"
  type        = list(string)
  default     = ["us-east-1a", "us-east-1b", "us-east-1c"]
}

variable "private_subnet_cidrs" {
  description = "私有子网 CIDR 列表"
  type        = list(string)
  default     = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
}

variable "public_subnet_cidrs" {
  description = "公有子网 CIDR 列表"
  type        = list(string)
  default     = ["10.0.101.0/24", "10.0.102.0/24", "10.0.103.0/24"]
}

# 创建 VPC
resource "aws_vpc" "main" {
  cidr_block           = var.vpc_cidr
  enable_dns_hostnames = true
  enable_dns_support   = true
  
  tags = {
    Name = "${var.environment}-vpc"
  }
}

# 创建互联网网关
resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id
  
  tags = {
    Name = "${var.environment}-igw"
  }
}

# 创建公有子网
resource "aws_subnet" "public" {
  count                   = length(var.public_subnet_cidrs)
  vpc_id                  = aws_vpc.main.id
  cidr_block              = var.public_subnet_cidrs[count.index]
  availability_zone       = var.availability_zones[count.index]
  map_public_ip_on_launch = true
  
  tags = {
    Name = "${var.environment}-public-subnet-${count.index + 1}"
    Tier = "Public"
  }
}

# 创建私有子网
resource "aws_subnet" "private" {
  count             = length(var.private_subnet_cidrs)
  vpc_id            = aws_vpc.main.id
  cidr_block        = var.private_subnet_cidrs[count.index]
  availability_zone = var.availability_zones[count.index]
  
  tags = {
    Name = "${var.environment}-private-subnet-${count.index + 1}"
    Tier = "Private"
  }
}

# 创建公有子网路由表
resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id
  
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }
  
  tags = {
    Name = "${var.environment}-public-rt"
  }
}

# 关联公有子网和路由表
resource "aws_route_table_association" "public" {
  count          = length(aws_subnet.public)
  subnet_id      = aws_subnet.public[count.index].id
  route_table_id = aws_route_table.public.id
}

# 输出值
output "vpc_id" {
  description = "VPC ID"
  value       = aws_vpc.main.id
}

output "public_subnet_ids" {
  description = "公有子网 ID 列表"
  value       = aws_subnet.public[*].id
}

output "private_subnet_ids" {
  description = "私有子网 ID 列表"
  value       = aws_subnet.private[*].id
}
```

#### 步骤 5：创建环境配置

创建文件 `terraform-github-demo/environments/dev/main.tf`：

```hcl
# 开发环境配置

terraform {
  required_version = ">= 1.7.0"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
  
  # 开发环境状态文件
  backend "s3" {
    bucket         = "my-terraform-state-bucket"
    key            = "dev/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-locks"
  }
}

provider "aws" {
  region = "us-east-1"
  
  default_tags {
    tags = {
      Environment = "dev"
      Project     = "my-project"
      ManagedBy   = "Terraform"
    }
  }
}

# 调用网络模块
module "network" {
  source = "../../modules/network"
  
  environment = "dev"
  vpc_cidr    = "10.0.0.0/16"
  
  availability_zones    = ["us-east-1a", "us-east-1b"]
  private_subnet_cidrs = ["10.0.1.0/24", "10.0.2.0/24"]
  public_subnet_cidrs  = ["10.0.101.0/24", "10.0.102.0/24"]
}

# 输出值
output "vpc_id" {
  value = module.network.vpc_id
}

output "public_subnet_ids" {
  value = module.network.public_subnet_ids
}

output "private_subnet_ids" {
  value = module.network.private_subnet_ids
}
```

创建文件 `terraform-github-demo/environments/dev/terraform.tfvars`：

```hcl
# 开发环境变量
aws_region   = "us-east-1"
environment  = "dev"
project_name = "my-project"
```

#### 步骤 6：创建生产环境配置

创建文件 `terraform-github-demo/environments/prod/main.tf`：

```hcl
# 生产环境配置

terraform {
  required_version = ">= 1.7.0"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
  
  # 生产环境状态文件
  backend "s3" {
    bucket         = "my-terraform-state-bucket"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-locks"
  }
}

provider "aws" {
  region = "us-east-1"
  
  default_tags {
    tags = {
      Environment = "prod"
      Project     = "my-project"
      ManagedBy   = "Terraform"
    }
  }
}

# 调用网络模块
module "network" {
  source = "../../modules/network"
  
  environment = "prod"
  vpc_cidr    = "10.1.0.0/16"
  
  availability_zones    = ["us-east-1a", "us-east-1b", "us-east-1c"]
  private_subnet_cidrs = ["10.1.1.0/24", "10.1.2.0/24", "10.1.3.0/24"]
  public_subnet_cidrs  = ["10.1.101.0/24", "10.1.102.0/24", "10.1.103.0/24"]
}

# 输出值
output "vpc_id" {
  value = module.network.vpc_id
}

output "public_subnet_ids" {
  value = module.network.public_subnet_ids
}

output "private_subnet_ids" {
  value = module.network.private_subnet_ids
}
```

### 第二部分：GitHub Actions 集成

#### 步骤 7：创建 Terraform 计划工作流

创建文件 `.github/workflows/terraform-plan.yml`：

```yaml
name: Terraform Plan

on:
  pull_request:
    branches: [main]
    paths:
      - 'terraform-github-demo/**'
      - '.github/workflows/terraform-*'

permissions:
  contents: read
  pull-requests: write
  id-token: write

env:
  TF_VERSION: '1.7.0'
  AWS_REGION: 'us-east-1'

jobs:
  terraform-plan:
    name: Terraform Plan
    runs-on: ubuntu-latest
    strategy:
      matrix:
        environment: [dev, staging, prod]
    
    steps:
      - name: 检出代码
        uses: actions/checkout@v4
      
      - name: 配置 AWS 凭证
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}
      
      - name: 设置 Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: ${{ env.TF_VERSION }}
      
      - name: Terraform 格式检查
        run: terraform fmt -check -recursive
        working-directory: terraform-github-demo
      
      - name: Terraform 初始化
        run: terraform init
        working-directory: terraform-github-demo/environments/${{ matrix.environment }}
      
      - name: Terraform 验证
        run: terraform validate
        working-directory: terraform-github-demo/environments/${{ matrix.environment }}
      
      - name: Terraform 计划
        id: plan
        run: |
          terraform plan -no-color -out=tfplan
        working-directory: terraform-github-demo/environments/${{ matrix.environment }}
        continue-on-error: true
      
      - name: 生成计划输出
        run: |
          terraform show -no-color tfplan > plan_output.txt
        working-directory: terraform-github-demo/environments/${{ matrix.environment }}
      
      - name: 添加 PR 评论
        uses: actions/github-script@v7
        with:
          script: |
            const fs = require('fs');
            const planOutput = fs.readFileSync(
              `terraform-github-demo/environments/${{ matrix.environment }}/plan_output.txt`,
              'utf8'
            );
            
            const environment = '${{ matrix.environment }}';
            const planStatus = '${{ steps.plan.outcome }}';
            const emoji = planStatus === 'success' ? '✅' : '❌';
            
            const body = `## ${emoji} Terraform Plan - ${environment}
            
            \`\`\`
            ${planOutput.substring(0, 60000)}
            \`\`\`
            
            **Status:** ${planStatus}
            `;
            
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: body
            });
      
      - name: 检查计划结果
        if: steps.plan.outcome == 'failure'
        run: exit 1
```

#### 步骤 8：创建 Terraform 应用工作流

创建文件 `.github/workflows/terraform-apply.yml`：

```yaml
name: Terraform Apply

on:
  push:
    branches: [main]
    paths:
      - 'terraform-github-demo/**'
  workflow_dispatch:
    inputs:
      environment:
        description: '部署环境'
        required: true
        type: choice
        options:
          - dev
          - staging
          - prod
      action:
        description: '执行动作'
        required: true
        type: choice
        options:
          - plan
          - apply
          - destroy

permissions:
  contents: read
  id-token: write

env:
  TF_VERSION: '1.7.0'
  AWS_REGION: 'us-east-1'

jobs:
  terraform-apply:
    name: Terraform Apply
    runs-on: ubuntu-latest
    environment: ${{ github.event.inputs.environment || 'dev' }}
    
    steps:
      - name: 检出代码
        uses: actions/checkout@v4
      
      - name: 配置 AWS 凭证
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}
      
      - name: 设置 Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: ${{ env.TF_VERSION }}
      
      - name: 确定环境
        id: env
        run: |
          if [ "${{ github.event_name }}" = "workflow_dispatch" ]; then
            echo "environment=${{ github.event.inputs.environment }}" >> $GITHUB_OUTPUT
          else
            echo "environment=dev" >> $GITHUB_OUTPUT
          fi
      
      - name: Terraform 初始化
        run: terraform init
        working-directory: terraform-github-demo/environments/${{ steps.env.outputs.environment }}
      
      - name: Terraform 计划
        run: terraform plan -out=tfplan
        working-directory: terraform-github-demo/environments/${{ steps.env.outputs.environment }}
      
      - name: Terraform 应用
        if: |
          (github.event_name == 'push' && github.ref == 'refs/heads/main') ||
          (github.event_name == 'workflow_dispatch' && github.event.inputs.action == 'apply')
        run: terraform apply -auto-approve tfplan
        working-directory: terraform-github-demo/environments/${{ steps.env.outputs.environment }}
      
      - name: Terraform 销毁
        if: github.event_name == 'workflow_dispatch' && github.event.inputs.action == 'destroy'
        run: terraform destroy -auto-approve
        working-directory: terraform-github-demo/environments/${{ steps.env.outputs.environment }}
      
      - name: 输出结果
        run: |
          echo "环境: ${{ steps.env.outputs.environment }}"
          echo "操作完成"
```

#### 步骤 9：创建多环境部署工作流

创建文件 `.github/workflows/terraform-multi-env.yml`：

```yaml
name: Terraform Multi-Environment

on:
  workflow_dispatch:
    inputs:
      environment:
        description: '选择环境'
        required: true
        type: choice
        options:
          - dev
          - staging
          - prod
      action:
        description: '执行动作'
        required: true
        type: choice
        options:
          - plan
          - apply
          - destroy

permissions:
  contents: read
  id-token: write

env:
  TF_VERSION: '1.7.0'

jobs:
  # 审批作业（生产环境需要）
  approval:
    name: Approval Required
    runs-on: ubuntu-latest
    if: github.event.inputs.environment == 'prod'
    environment: production-approval
    
    steps:
      - name: 等待审批
        run: echo "生产环境部署已获得批准"

  # Terraform 作业
  terraform:
    name: Terraform ${{ github.event.inputs.action }}
    runs-on: ubuntu-latest
    needs: [approval]
    if: always() && (needs.approval.result == 'success' || github.event.inputs.environment != 'prod')
    
    steps:
      - name: 检出代码
        uses: actions/checkout@v4
      
      - name: 配置 AWS 凭证
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: 'us-east-1'
      
      - name: 设置 Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: ${{ env.TF_VERSION }}
      
      - name: Terraform 初始化
        run: terraform init
        working-directory: terraform-github-demo/environments/${{ github.event.inputs.environment }}
      
      - name: Terraform 计划
        id: plan
        run: terraform plan -no-color -out=tfplan
        working-directory: terraform-github-demo/environments/${{ github.event.inputs.environment }}
      
      - name: Terraform 应用
        if: github.event.inputs.action == 'apply'
        run: terraform apply -auto-approve tfplan
        working-directory: terraform-github-demo/environments/${{ github.event.inputs.environment }}
      
      - name: Terraform 销毁
        if: github.event.inputs.action == 'destroy'
        run: terraform destroy -auto-approve
        working-directory: terraform-github-demo/environments/${{ github.event.inputs.environment }}
      
      - name: 生成报告
        if: always()
        run: |
          echo "# Terraform 部署报告" > report.md
          echo "" >> report.md
          echo "## 环境: ${{ github.event.inputs.environment }}" >> report.md
          echo "## 操作: ${{ github.event.inputs.action }}" >> report.md
          echo "## 状态: ${{ job.status }}" >> report.md
          echo "## 时间: $(date)" >> report.md
      
      - name: 上传报告
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: terraform-report-${{ github.event.inputs.environment }}
          path: report.md
```

### 第三部分：高级配置

#### 步骤 10：创建 Terraform 后端配置

创建文件 `terraform-github-demo/backend/main.tf`：

```hcl
# 后端资源创建（需要先手动创建）

provider "aws" {
  region = "us-east-1"
}

# S3 存储桶用于存储状态
resource "aws_s3_bucket" "terraform_state" {
  bucket = "my-terraform-state-bucket"
  
  lifecycle {
    prevent_destroy = true
  }
  
  tags = {
    Name        = "Terraform State Bucket"
    Environment = "shared"
    ManagedBy   = "Terraform"
  }
}

# 启用版本控制
resource "aws_s3_bucket_versioning" "terraform_state" {
  bucket = aws_s3_bucket.terraform_state.id
  
  versioning_configuration {
    status = "Enabled"
  }
}

# 启用加密
resource "aws_s3_bucket_server_side_encryption_configuration" "terraform_state" {
  bucket = aws_s3_bucket.terraform_state.id
  
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "aws:kms"
    }
  }
}

# 阻止公共访问
resource "aws_s3_bucket_public_access_block" "terraform_state" {
  bucket = aws_s3_bucket.terraform_state.id
  
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

# DynamoDB 表用于状态锁定
resource "aws_dynamodb_table" "terraform_locks" {
  name         = "terraform-locks"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "LockID"
  
  attribute {
    name = "LockID"
    type = "S"
  }
  
  tags = {
    Name        = "Terraform Lock Table"
    Environment = "shared"
    ManagedBy   = "Terraform"
  }
}

# 输出值
output "s3_bucket_name" {
  description = "状态存储桶名称"
  value       = aws_s3_bucket.terraform_state.id
}

output "dynamodb_table_name" {
  description = "锁定表名称"
  value       = aws_dynamodb_table.terraform_locks.name
}
```

#### 步骤 11：创建安全扫描工作流

创建文件 `.github/workflows/terraform-security.yml`：

```yaml
name: Terraform Security Scan

on:
  pull_request:
    branches: [main]
    paths:
      - 'terraform-github-demo/**'

permissions:
  contents: read
  pull-requests: write

jobs:
  tfsec:
    name: tfsec Security Scan
    runs-on: ubuntu-latest
    
    steps:
      - name: 检出代码
        uses: actions/checkout@v4
      
      - name: 运行 tfsec
        uses: aquasecurity/tfsec-action@v1.0.3
        with:
          working_directory: terraform-github-demo
          soft_fail: true
      
      - name: 上传 tfsec SARIF 报告
        uses: github/codeql-action/upload-sarif@v3
        if: always()
        with:
          sarif_file: tfsec.sarif
  
  checkov:
    name: Checkov Security Scan
    runs-on: ubuntu-latest
    
    steps:
      - name: 检出代码
        uses: actions/checkout@v4
      
      - name: 运行 Checkov
        uses: bridgecrewio/checkov-action@v12
        with:
          directory: terraform-github-demo
          soft_fail: true
          output_format: sarif
          output_file_path: checkov.sarif
      
      - name: 上传 Checkov SARIF 报告
        uses: github/codeql-action/upload-sarif@v3
        if: always()
        with:
          sarif_file: checkov.sarif
  
  infracost:
    name: Infracost Estimate
    runs-on: ubuntu-latest
    
    steps:
      - name: 检出代码
        uses: actions/checkout@v4
      
      - name: 设置 Infracost
        uses: infracost/actions/setup@v2
        with:
          api-key: ${{ secrets.INFRACOST_API_KEY }}
      
      - name: 生成成本估算
        run: |
          infracost breakdown --path terraform-github-demo/environments/dev \
            --format json \
            --out-file infracost.json
      
      - name: 添加 PR 评论
        uses: infracost/actions/comment@v2
        with:
          path: infracost.json
          behavior: update
```

#### 步骤 12：创建状态管理工作流

创建文件 `.github/workflows/terraform-state.yml`：

```yaml
name: Terraform State Management

on:
  workflow_dispatch:
    inputs:
      environment:
        description: '环境'
        required: true
        type: choice
        options:
          - dev
          - staging
          - prod
      action:
        description: '操作'
        required: true
        type: choice
        options:
          - list
          - show
          - mv
          - rm
          - import
      resource:
        description: '资源地址（用于 mv/rm/import）'
        required: false
        type: string
      target:
        description: '目标地址（用于 mv）'
        required: false
        type: string

permissions:
  contents: read
  id-token: write

jobs:
  state-management:
    name: State Management
    runs-on: ubuntu-latest
    environment: ${{ github.event.inputs.environment }}
    
    steps:
      - name: 检出代码
        uses: actions/checkout@v4
      
      - name: 配置 AWS 凭证
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: 'us-east-1'
      
      - name: 设置 Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: '1.7.0'
      
      - name: Terraform 初始化
        run: terraform init
        working-directory: terraform-github-demo/environments/${{ github.event.inputs.environment }}
      
      - name: 列出资源
        if: github.event.inputs.action == 'list'
        run: terraform state list
        working-directory: terraform-github-demo/environments/${{ github.event.inputs.environment }}
      
      - name: 显示资源
        if: github.event.inputs.action == 'show'
        run: terraform state show "${{ github.event.inputs.resource }}"
        working-directory: terraform-github-demo/environments/${{ github.event.inputs.environment }}
      
      - name: 移动资源
        if: github.event.inputs.action == 'mv'
        run: |
          terraform state mv \
            "${{ github.event.inputs.resource }}" \
            "${{ github.event.inputs.target }}"
        working-directory: terraform-github-demo/environments/${{ github.event.inputs.environment }}
      
      - name: 删除资源
        if: github.event.inputs.action == 'rm'
        run: terraform state rm "${{ github.event.inputs.resource }}"
        working-directory: terraform-github-demo/environments/${{ github.event.inputs.environment }}
      
      - name: 导入资源
        if: github.event.inputs.action == 'import'
        run: |
          terraform import \
            "${{ github.event.inputs.resource }}" \
            "${{ github.event.inputs.target }}"
        working-directory: terraform-github-demo/environments/${{ github.event.inputs.environment }}
```

### 第四部分：实用脚本

#### 步骤 13：创建初始化脚本

创建文件 `terraform-github-demo/scripts/init.sh`：

```bash
#!/bin/bash

# Terraform 初始化脚本

set -e

# 颜色定义
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

# 打印帮助信息
show_help() {
    echo "Terraform 初始化工具"
    echo ""
    echo "用法: $0 [选项] <环境>"
    echo ""
    echo "环境:"
    echo "  dev      - 开发环境"
    echo "  staging  - 测试环境"
    echo "  prod     - 生产环境"
    echo ""
    echo "选项:"
    echo "  -h, --help   显示帮助信息"
    echo "  -f, --force  强制重新初始化"
}

# 初始化环境
init_environment() {
    local env=$1
    local force=$2
    
    echo -e "${GREEN}=== 初始化 ${env} 环境 ===${NC}"
    echo ""
    
    # 检查环境目录
    if [ ! -d "environments/${env}" ]; then
        echo -e "${RED}错误: 环境目录不存在: environments/${env}${NC}"
        exit 1
    fi
    
    # 进入环境目录
    cd "environments/${env}"
    
    # 检查是否需要强制初始化
    if [ "$force" = "true" ] && [ -d ".terraform" ]; then
        echo -e "${YELLOW}清除现有 Terraform 状态...${NC}"
        rm -rf .terraform .terraform.lock.hcl
    fi
    
    # 初始化 Terraform
    echo "正在初始化 Terraform..."
    terraform init
    
    # 验证配置
    echo ""
    echo "正在验证配置..."
    terraform validate
    
    # 格式检查
    echo ""
    echo "正在检查代码格式..."
    terraform fmt -check
    
    echo ""
    echo -e "${GREEN}初始化完成！${NC}"
    
    # 显示工作区信息
    echo ""
    echo "当前工作区:"
    terraform workspace list
    
    # 返回原目录
    cd ../..
}

# 主逻辑
FORCE=false
ENV=""

while [[ $# -gt 0 ]]; do
    case $1 in
        -h|--help)
            show_help
            exit 0
            ;;
        -f|--force)
            FORCE=true
            shift
            ;;
        dev|staging|prod)
            ENV=$1
            shift
            ;;
        *)
            echo -e "${RED}未知选项: $1${NC}"
            show_help
            exit 1
            ;;
    esac
done

if [ -z "$ENV" ]; then
    echo -e "${RED}错误: 请指定环境${NC}"
    show_help
    exit 1
fi

init_environment "$ENV" "$FORCE"
```

#### 步骤 14：创建部署脚本

创建文件 `terraform-github-demo/scripts/deploy.sh`：

```bash
#!/bin/bash

# Terraform 部署脚本

set -e

# 颜色定义
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m'

# 打印帮助信息
show_help() {
    echo "Terraform 部署工具"
    echo ""
    echo "用法: $0 [选项] <环境> <动作>"
    echo ""
    echo "环境:"
    echo "  dev      - 开发环境"
    echo "  staging  - 测试环境"
    echo "  prod     - 生产环境"
    echo ""
    echo "动作:"
    echo "  plan     - 查看变更计划"
    echo "  apply    - 应用变更"
    echo "  destroy  - 销毁资源"
    echo "  output   - 显示输出"
    echo ""
    echo "选项:"
    echo "  -h, --help       显示帮助信息"
    echo "  -y, --auto-approve 自动确认"
    echo "  -var-file=FILE   指定变量文件"
}

# 部署函数
deploy() {
    local env=$1
    local action=$2
    local auto_approve=$3
    local var_file=$4
    
    echo -e "${GREEN}=== ${env} 环境 - ${action} ===${NC}"
    echo ""
    
    # 检查环境目录
    if [ ! -d "environments/${env}" ]; then
        echo -e "${RED}错误: 环境目录不存在: environments/${env}${NC}"
        exit 1
    fi
    
    # 进入环境目录
    cd "environments/${env}"
    
    # 构建命令参数
    local cmd_args=""
    if [ -n "$var_file" ]; then
        cmd_args="-var-file=${var_file}"
    fi
    
    # 执行动作
    case $action in
        plan)
            echo "正在生成变更计划..."
            terraform plan $cmd_args
            ;;
        apply)
            echo "正在应用变更..."
            if [ "$auto_approve" = "true" ]; then
                terraform apply -auto-approve $cmd_args
            else
                terraform apply $cmd_args
            fi
            ;;
        destroy)
            echo -e "${YELLOW}警告: 即将销毁所有资源！${NC}"
            if [ "$auto_approve" = "true" ]; then
                terraform destroy -auto-approve $cmd_args
            else
                terraform destroy $cmd_args
            fi
            ;;
        output)
            echo "输出值:"
            terraform output
            ;;
        *)
            echo -e "${RED}未知动作: $action${NC}"
            exit 1
            ;;
    esac
    
    echo ""
    echo -e "${GREEN}操作完成！${NC}"
    
    # 返回原目录
    cd ../..
}

# 主逻辑
AUTO_APPROVE=false
VAR_FILE=""
ENV=""
ACTION=""

while [[ $# -gt 0 ]]; do
    case $1 in
        -h|--help)
            show_help
            exit 0
            ;;
        -y|--auto-approve)
            AUTO_APPROVE=true
            shift
            ;;
        -var-file=*)
            VAR_FILE="${1#*=}"
            shift
            ;;
        dev|staging|prod)
            ENV=$1
            shift
            ;;
        plan|apply|destroy|output)
            ACTION=$1
            shift
            ;;
        *)
            echo -e "${RED}未知选项: $1${NC}"
            show_help
            exit 1
            ;;
    esac
done

if [ -z "$ENV" ] || [ -z "$ACTION" ]; then
    echo -e "${RED}错误: 请指定环境和动作${NC}"
    show_help
    exit 1
fi

deploy "$ENV" "$ACTION" "$AUTO_APPROVE" "$VAR_FILE"
```

#### 步骤 15：创建 Terraform 文档生成脚本

创建文件 `terraform-github-demo/scripts/generate-docs.sh`：

```bash
#!/bin/bash

# Terraform 文档生成脚本

set -e

# 颜色定义
GREEN='\033[0;32m'
NC='\033[0m'

echo -e "${GREEN}=== 生成 Terraform 文档 ===${NC}"
echo ""

# 检查 terraform-docs 是否安装
if ! command -v terraform-docs &> /dev/null; then
    echo "安装 terraform-docs..."
    
    # 检测操作系统
    if [[ "$OSTYPE" == "darwin"* ]]; then
        brew install terraform-docs
    elif [[ "$OSTYPE" == "linux-gnu"* ]]; then
        curl -sSLo terraform-docs.tar.gz https://github.com/terraform-docs/terraform-docs/releases/download/v0.16.0/terraform-docs-v0.16.0-linux-amd64.tar.gz
        tar -xzf terraform-docs.tar.gz
        sudo mv terraform-docs /usr/local/bin/
        rm terraform-docs.tar.gz
    fi
fi

# 生成模块文档
echo "生成模块文档..."
for module_dir in modules/*/; do
    if [ -d "$module_dir" ]; then
        module_name=$(basename "$module_dir")
        echo "  - ${module_name}"
        
        # 生成 Markdown 文档
        terraform-docs markdown table "${module_dir}" > "${module_dir}/README.md"
    fi
done

# 生成环境文档
echo ""
echo "生成环境文档..."
for env_dir in environments/*/; do
    if [ -d "$env_dir" ]; then
        env_name=$(basename "$env_dir")
        echo "  - ${env_name}"
        
        # 生成 Markdown 文档
        terraform-docs markdown table "${env_dir}" > "${env_dir}/README.md"
    fi
done

# 生成主 README
echo ""
echo "生成主 README..."
cat > README.md << 'EOF'
# Terraform GitHub Demo

本项目演示如何使用 Terraform 和 GitHub Actions 管理云基础设施。

## 项目结构

```
.
├── environments/          # 环境配置
│   ├── dev/              # 开发环境
│   ├── staging/          # 测试环境
│   └── prod/             # 生产环境
├── modules/              # 可复用模块
│   └── network/          # 网络模块
├── scripts/              # 实用脚本
└── .github/workflows/    # GitHub Actions 工作流
```

## 快速开始

### 初始化环境

```bash
# 初始化开发环境
./scripts/init.sh dev

# 初始化测试环境
./scripts/init.sh staging

# 初始化生产环境
./scripts/init.sh prod
```

### 部署资源

```bash
# 查看变更计划
./scripts/deploy.sh dev plan

# 应用变更
./scripts/deploy.sh dev apply

# 销毁资源
./scripts/deploy.sh dev destroy
```

### 生成文档

```bash
./scripts/generate-docs.sh
```

## GitHub Actions 工作流

- `terraform-plan.yml` - PR 时自动计划
- `terraform-apply.yml` - 合并后自动部署
- `terraform-multi-env.yml` - 多环境部署
- `terraform-security.yml` - 安全扫描
- `terraform-state.yml` - 状态管理

## 最佳实践

1. 使用远程状态存储
2. 启用状态锁定
3. 使用变量文件管理环境差异
4. 在 PR 中审查变更计划
5. 使用 GitHub Environments 保护生产环境
6. 定期运行安全扫描

## 安全注意事项

- 不要在代码中硬编码凭证
- 使用 GitHub Secrets 存储敏感信息
- 启用状态加密
- 限制 IAM 权限
- 定期审计基础设施变更
EOF

echo ""
echo -e "${GREEN}文档生成完成！${NC}"
```

```bash
# 使脚本可执行
chmod +x terraform-github-demo/scripts/*.sh
```

---

## 验证练习结果

### 检查清单

完成练习后，请验证以下内容：

- [ ] Terraform 已正确安装
- [ ] 项目目录结构已创建
- [ ] Terraform 配置文件语法正确
- [ ] GitHub Actions 工作流已创建
- [ ] 脚本文件可执行

### 验证命令

```bash
# 检查 Terraform 安装
terraform version

# 验证配置文件
cd terraform-github-demo/environments/dev
terraform init -backend=false
terraform validate
terraform fmt -check

# 检查工作流语法
actionlint .github/workflows/terraform-*.yml

# 检查脚本
bash -n terraform-github-demo/scripts/init.sh
bash -n terraform-github-demo/scripts/deploy.sh
```

---

## 进阶挑战

### 挑战 1：实现 Terraform Cloud 集成

使用 Terraform Cloud 替代 S3 后端：

```hcl
terraform {
  cloud {
    organization = "your-org"
    
    workspaces {
      name = "my-project-dev"
    }
  }
}
```

### 挑战 2：创建自定义 Terraform Provider

开发一个简单的 Terraform Provider：

```go
package main

import (
    "github.com/hashicorp/terraform-plugin-sdk/v2/helper/schema"
    "github.com/hashicorp/terraform-plugin-sdk/v2/plugin"
)

func main() {
    plugin.Serve(&plugin.ServeOpts{
        ProviderFunc: func() *schema.Provider {
            return Provider()
        },
    })
}
```

### 挑战 3：实现基础设施漂移检测

创建定期检测基础设施漂移的工作流：

```yaml
name: Drift Detection

on:
  schedule:
    - cron: '0 8 * * 1'  # 每周一早上8点

jobs:
  detect-drift:
    runs-on: ubuntu-latest
    
    steps:
      - name: 检出代码
        uses: actions/checkout@v4
      
      - name: 检测漂移
        run: |
          terraform plan -detailed-exitcode
          if [ $? -eq 2 ]; then
            echo "检测到基础设施漂移！"
            exit 1
          fi
```

### 挑战 4：实现成本优化建议

使用 Infracost 分析并提供成本优化建议：

```yaml
- name: 成本优化分析
  run: |
    infracost diff --path terraform-github-demo/environments/dev \
      --format json \
      --out-file infracost-diff.json
    
    # 分析成本变化
    python scripts/analyze-cost.py infracost-diff.json
```

---

## 常见问题

### Q1：如何处理 Terraform 状态冲突？

```bash
# 查看当前锁
terraform force-unlock <LOCK_ID>

# 或使用 DynamoDB 自动解锁
aws dynamodb delete-item \
  --table-name terraform-locks \
  --key '{"LockID":{"S":"<LOCK_ID>"}}'
```

### Q2：如何安全地删除资源？

```bash
# 先查看将要删除的资源
terraform plan -destroy

# 使用 -target 选择性删除
terraform destroy -target=aws_instance.example

# 或在代码中移除资源后执行
terraform apply
```

### Q3：如何回滚 Terraform 变更？

```bash
# 使用 Git 回滚代码
git revert <commit-hash>

# 重新应用
terraform apply

# 或手动修改状态
terraform state mv aws_instance.old aws_instance.new
```

### Q4：如何优化 Terraform 执行速度？

1. 使用 `-parallelism` 参数增加并行度
2. 使用 `-refresh=false` 跳过状态刷新
3. 使用 `-target` 只处理特定资源
4. 使用 Terraform Cloud 的远程执行
5. 使用模块缓存

---

## Terraform 核心概念深入解析

### 状态管理的重要性

Terraform 的状态文件是整个基础设施管理的核心。状态文件记录了 Terraform 管理的所有资源的当前状态，包括资源的属性、依赖关系和唯一标识符。当执行 `terraform plan` 命令时，Terraform 会对比配置文件中定义的期望状态和状态文件中记录的当前状态，计算出需要执行的变更操作。如果状态文件丢失或损坏，Terraform 将无法正确管理基础设施，可能导致资源重复创建或意外删除。

远程状态存储是团队协作的基础。将状态文件存储在 S3、Azure Blob Storage 或 Terraform Cloud 等远程服务中，可以确保团队所有成员使用同一份状态文件。状态锁定机制防止多人同时修改状态导致冲突，DynamoDB 表用于实现 AWS 环境下的状态锁定。状态加密确保敏感信息不会泄露，S3 支持使用 KMS 进行服务端加密。状态版本控制允许回滚到之前的状态版本，在出现问题时可以快速恢复。

### 工作区策略

Terraform 工作区是在同一配置中管理多个环境的机制。每个工作区有独立的状态文件，使用相同配置创建不同的环境实例。工作区适合管理环境差异较小的场景，例如开发、测试和生产环境使用相同的资源配置，只是规模和参数不同。

然而，对于环境差异较大的场景，建议使用目录隔离方式。每个环境使用独立的配置目录，可以有完全不同的资源配置和参数。目录隔离方式的灵活性更高，可以针对不同环境进行独立的优化和定制。在本练习中，我们采用了目录隔离方式，每个环境有独立的配置目录和状态文件。

### 模块化设计原则

良好的模块化设计可以显著提升 Terraform 代码的可维护性和复用性。模块应该遵循单一职责原则，每个模块只负责一类资源的管理。模块的接口应该简洁明了，输入变量和输出值应该有清晰的描述和类型定义。模块内部应该包含合理的默认值，减少调用方的配置负担。模块应该支持参数化，通过变量控制资源的规模、名称和其他属性。

模块的版本管理也很重要。建议使用 Git 标签管理模块版本，语义化版本号表示变更的性质。在模块仓库中维护变更日志，记录每个版本的变更内容。模块的使用者可以通过版本约束指定使用的模块版本，避免意外的破坏性变更。

### 计划与应用的工作流

Terraform 的计划和应用流程是确保基础设施安全变更的关键。在执行 `terraform apply` 之前，应该先执行 `terraform plan` 查看变更计划。变更计划会列出所有将要执行的操作，包括资源的创建、修改和删除。仔细审查变更计划，确认所有变更都是预期的。特别注意标记为销毁的操作，确保不会意外删除重要资源。

在 CI/CD 流程中，可以在 Pull Request 阶段自动执行 `terraform plan`，将变更计划作为评论添加到 PR 中。审查者可以查看变更计划，确认变更是否合理。合并 PR 后，自动执行 `terraform apply` 应用变更。这种工作流确保了所有基础设施变更都经过审查，降低了误操作的风险。

### 变量管理最佳实践

Terraform 的变量管理直接影响配置的灵活性和可维护性。使用变量文件可以将环境特定的参数与通用配置分离。每个环境可以有独立的变量文件，例如 `dev.tfvars`、`staging.tfvars` 和 `prod.tfvars`。变量文件不应该包含敏感信息，敏感信息应该通过环境变量或密钥管理服务传递。

变量的验证规则可以在定义时指定，确保输入值符合预期。例如，可以限制环境变量只能是 `dev`、`staging` 或 `prod`。可以限制实例类型只能是特定的值列表。可以限制端口范围在有效范围内。变量验证可以在计划阶段捕获配置错误，避免在应用阶段才发现问题。

### 输出值的设计

Terraform 的输出值用于将资源属性暴露给外部使用。输出值可以用于模块之间的信息传递，例如网络模块输出子网 ID，计算模块使用这些子网 ID 创建实例。输出值也可以用于 CI/CD 流程，例如输出负载均衡器的 DNS 名称用于后续的健康检查。输出值应该有清晰的描述，说明输出的含义和用途。敏感的输出值应该标记为 `sensitive = true`，防止在日志中泄露。

### 提供商配置

Terraform 提供商是与云服务 API 交互的插件。提供商的版本约束应该明确指定，避免使用未经测试的版本。默认标签可以在提供商级别配置，确保所有资源都有统一的标签。多提供商配置允许在同一配置中管理多个区域或多账户的资源。提供商的认证信息应该通过环境变量或配置文件传递，不应该硬编码在配置中。

### 生命周期管理

Terraform 的生命周期规则可以控制资源的创建、更新和删除行为。`create_before_destroy` 规则在替换资源时先创建新资源再删除旧资源，避免服务中断。`prevent_destroy` 规则防止重要资源被意外删除，例如数据库和状态存储桶。`ignore_changes` 规则忽略特定属性的变更，避免外部修改触发不必要的更新。合理使用生命周期规则可以提升基础设施的稳定性和安全性。

### 秘密管理

在 Terraform 中管理秘密信息需要特别谨慎。不要在配置文件或变量文件中硬编码密码、密钥等敏感信息。使用 AWS Secrets Manager、Azure Key Vault 或 HashiCorp Vault 等秘密管理服务存储敏感信息。通过数据源在运行时获取秘密值，而不是将其存储在状态文件中。如果必须在状态文件中存储秘密，确保状态文件的存储位置有适当的访问控制和加密保护。定期轮换秘密值，降低泄露风险。

### 灾难恢复计划

基础设施的灾难恢复计划是生产环境的重要保障。定期备份状态文件，确保可以在状态文件损坏时恢复。记录所有手动配置的资源，这些资源不在 Terraform 管理范围内，需要单独备份。建立基础设施重建流程，在灾难发生时可以快速重建整个环境。定期进行灾难恢复演练，验证恢复流程的有效性。使用基础设施版本控制，可以快速回滚到之前已知正常的状态。

### 性能优化

大型基础设施的 Terraform 执行可能需要较长时间。使用 `-parallelism` 参数增加资源创建的并行度，默认值是 10，可以根据云服务的限制适当增加。使用 `-refresh=false` 跳过状态刷新，在状态文件已经是最新时可以节省时间。使用 `-target` 参数只处理特定资源，在调试时可以避免执行完整的计划和应用。将大型配置拆分为多个独立的状态文件，减少每个状态文件管理的资源数量。使用 Terraform Cloud 的远程执行功能，利用其缓存和并行能力提升执行效率。

### 代码审查清单

建立 Terraform 配置的代码审查清单可以提升代码质量。审查内容包括安全性检查，确认没有硬编码的凭证和过于宽松的权限。资源命名检查，确认资源名称符合命名规范且具有描述性。标签检查，确认所有资源都有必要的标签。变量检查，确认变量有清晰的描述和合理的默认值。输出检查，确认输出值有清晰的描述且敏感值已标记。生命周期检查，确认生命周期规则的使用合理。依赖检查，确认资源之间的依赖关系正确。格式检查，确认代码格式符合 Terraform 规范。

---

## 延伸阅读

- [Terraform 官方文档](https://developer.hashicorp.com/terraform/docs)
- [GitHub Actions 集成 Terraform](https://learn.hashicorp.com/tutorials/terraform/github-actions)
- [Terraform 最佳实践](https://www.terraform-best-practices.com/)
- [Infracost 文档](https://www.infracost.io/docs/)

---

## 练习总结

通过本练习，你已经学会了：

1. ✅ 理解基础设施即代码的概念
2. ✅ 编写 Terraform 配置文件
3. ✅ 创建可复用的 Terraform 模块
4. ✅ 使用 GitHub Actions 自动化部署
5. ✅ 管理多环境基础设施
6. ✅ 实现安全扫描和成本估算

Terraform 与 GitHub Actions 的结合是现代 DevOps 的重要实践，建议在实际项目中逐步采用，从开发环境开始，逐步扩展到生产环境。
