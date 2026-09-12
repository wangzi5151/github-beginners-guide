# Terraform IaC + GitHub 实践指南

> 本教程面向中国开发者，系统讲解如何使用 Terraform 实现基础设施即代码（IaC），并与 GitHub Actions 深度集成，实现基础设施的自动化管理。

---

## 目录

1. [Infrastructure as Code 概念](#1-infrastructure-as-code-概念)
2. [Terraform 基础语法与 Provider](#2-terraform-基础语法与-provider)
3. [Terraform + GitHub Actions CI/CD](#3-terraform--github-actions-cicd)
4. [Terraform State 管理（远程 Backend）](#4-terraform-state-管理远程-backend)
5. [Terraform 模块化开发](#5-terraform-模块化开发)
6. [Terraform 与 GitHub Provider](#6-terraform-与-github-provider)
7. [GitHub Actions Runner 部署到云厂商](#7-github-actions-runner-部署到云厂商)
8. [多环境 Terraform 管理](#8-多环境-terraform-管理)
9. [Terraform 安全扫描](#9-terraform-安全扫描)
10. [Terratest 测试框架](#10-terratest-测试框架)
11. [OpenTofu vs Terraform](#11-opentofu-vs-terraform)
12. [国内云厂商 Terraform Provider](#12-国内云厂商-terraform-provider)
13. [IaC 最佳实践](#13-iac-最佳实践)

---

## 1. Infrastructure as Code 概念

### 1.1 什么是 IaC

Infrastructure as Code（基础设施即代码，IaC）是一种使用代码来定义和管理基础设施的方法。通过 IaC，开发者可以用声明式的方式描述云资源（如服务器、数据库、网络等），并通过版本控制系统管理这些配置。

### 1.2 IaC 的核心优势

| 优势 | 说明 |
|------|------|
| **版本控制** | 基础设施变更有完整的 Git 历史记录 |
| **可重复性** | 相同的代码产生相同的基础设施 |
| **自动化** | 减少手动操作，降低人为错误 |
| **文档化** | 代码本身就是最好的文档 |
| **协作** | 团队成员可以通过 PR 审查基础设施变更 |
| **成本管理** | 清晰追踪资源创建和销毁 |

### 1.3 声明式 vs 命令式

```hcl
// 声明式（Terraform）- 描述期望状态
resource "alicloud_instance" "web" {
  instance_name = "web-server"
  image_id      = "ubuntu_22_04_x64_20G_alibase_20240101.vhd"
  instance_type = "ecs.g6.large"
}
```

```bash
# 命令式（CLI）- 描述操作步骤
aliyun ecs CreateInstance --RegionId cn-hangzhou --InstanceName web-server
aliyun ecs StartInstance --InstanceId i-xxx
```

### 1.4 主流 IaC 工具对比

| 工具 | 类型 | 语言 | 状态管理 | 适用场景 |
|------|------|------|----------|----------|
| **Terraform** | 通用 | HCL | 远程 | 多云环境 |
| **OpenTofu** | 通用 | HCL | 远程 | Terraform 开源替代 |
| **Pulumi** | 通用 | 多语言 | 远程 | 偏好编程语言 |
| **AWS CDK** | AWS 专用 | 多语言 | CloudFormation | AWS 深度用户 |
| **CloudFormation** | AWS 专用 | JSON/YAML | AWS 管理 | AWS 深度用户 |
| **Ansible** | 配置管理 | YAML | 无 | 配置管理+简单 IaC |

### 1.5 Terraform 生态系统

```
Terraform 生态
├── Terraform Core     # 核心引擎
├── Providers          # 云厂商插件（阿里云、腾讯云、AWS 等）
├── Modules            # 可复用的配置模块
├── Registry           # 公共模块和 Provider 仓库
├── Cloud              # Terraform Cloud/Enterprise（远程状态管理）
├── Sentinel           # 策略即代码
└── CDKTF              # 使用编程语言写 Terraform
```

---

## 2. Terraform 基础语法与 Provider

### 2.1 安装 Terraform

```bash
# Linux
wget https://releases.hashicorp.com/terraform/1.7.0/terraform_1.7.0_linux_amd64.zip
unzip terraform_1.7.0_linux_amd64.zip
sudo mv terraform /usr/local/bin/
terraform version

# macOS
brew tap hashicorp/tap
brew install hashicorp/tap/terraform

# 使用国内镜像（推荐）
# 设置环境变量
export TF_RELEASES_MIRROR=https://mirrors.tencent.com/terraform/
# 或
export TF_RELEASES_MIRROR=https://releases.hashicorp.mirrors.ustc.edu.cn/
```

### 2.2 HCL 基础语法

```hcl
# 变量定义
variable "region" {
  description = "云厂商区域"
  type        = string
  default     = "cn-hangzhou"
}

variable "instance_count" {
  description = "实例数量"
  type        = number
  default     = 2
}

variable "tags" {
  description = "资源标签"
  type        = map(string)
  default = {
    Environment = "production"
    Project     = "my-app"
  }
}

# 局部变量
locals {
  common_tags = merge(var.tags, {
    ManagedBy = "terraform"
    Repository = "my-org/my-infra"
  })
  
  name_prefix = "${var.project}-${var.environment}"
}

# 输出
output "instance_ids" {
  description = "实例 ID 列表"
  value       = alicloud_instance.web[*].id
}

output "load_balancer_ip" {
  description = "负载均衡 IP"
  value       = alicloud_slb.this.address
}
```

### 2.3 阿里云 Provider 配置

```hcl
# main.tf
terraform {
  required_version = ">= 1.5.0"
  
  required_providers {
    alicloud = {
      source  = "aliyun/alicloud"
      version = "~> 1.220"
    }
  }
}

provider "alicloud" {
  region     = var.region
  access_key = var.access_key
  secret_key = var.secret_key
  
  # 或使用环境变量
  # ALICLOUD_ACCESS_KEY
  # ALICLOUD_SECRET_KEY
  # ALICLOUD_REGION
}

# 创建 VPC
resource "alicloud_vpc" "main" {
  vpc_name   = "${local.name_prefix}-vpc"
  cidr_block = "172.16.0.0/12"
  
  tags = local.common_tags
}

# 创建交换机
resource "alicloud_vswitch" "main" {
  count        = 2
  vpc_id       = alicloud_vpc.main.id
  cidr_block   = cidrsubnet(alicloud_vpc.main.cidr_block, 8, count.index)
  zone_id      = data.alicloud_zones.available.zones[count.index].id
  vswitch_name = "${local.name_prefix}-vsw-${count.index}"
  
  tags = local.common_tags
}

# 创建安全组
resource "alicloud_security_group" "web" {
  name   = "${local.name_prefix}-web-sg"
  vpc_id = alicloud_vpc.main.id
  
  tags = local.common_tags
}

resource "alicloud_security_group_rule" "allow_http" {
  type              = "ingress"
  ip_protocol       = "tcp"
  nic_type          = "intranet"
  policy            = "accept"
  port_range        = "80/80"
  security_group_id = alicloud_security_group.web.id
  cidr_ip           = "0.0.0.0/0"
}

# 创建 ECS 实例
resource "alicloud_instance" "web" {
  count                = var.instance_count
  instance_name        = "${local.name_prefix}-web-${count.index}"
  host_name            = "web-${count.index}"
  image_id             = data.alicloud_images.ubuntu.images[0].id
  instance_type        = "ecs.g6.large"
  security_groups      = [alicloud_security_group.web.id]
  vswitch_id           = alicloud_vswitch.main[count.index % 2].id
  system_disk_category = "cloud_essd"
  system_disk_size     = 40
  
  tags = local.common_tags
}

# 数据源
data "alicloud_zones" "available" {
  available_resource_creation = "VSwitch"
}

data "alicloud_images" "ubuntu" {
  most_recent = true
  owners      = "system"
  name_regex  = "^ubuntu_22"
}
```

### 2.4 腾讯云 Provider 配置

```hcl
terraform {
  required_providers {
    tencentcloud = {
      source  = "tencentcloudstack/tencentcloud"
      version = "~> 1.81"
    }
  }
}

provider "tencentcloud" {
  region     = "ap-guangzhou"
  secret_id  = var.secret_id
  secret_key = var.secret_key
}

# 创建 VPC
resource "tencentcloud_vpc" "main" {
  name       = "${local.name_prefix}-vpc"
  cidr_block = "172.16.0.0/12"
}

# 创建子网
resource "tencentcloud_subnet" "main" {
  name              = "${local.name_prefix}-subnet"
  vpc_id            = tencentcloud_vpc.main.id
  cidr_block        = "172.16.0.0/24"
  availability_zone = "ap-guangzhou-3"
}

# 创建 CVM 实例
resource "tencentcloud_instance" "web" {
  instance_name     = "${local.name_prefix}-web"
  availability_zone = "ap-guangzhou-3"
  image_id          = "img-2lr9q49h"
  instance_type     = "SA2.MEDIUM4"
  system_disk_type  = "CLOUD_SSD"
  system_disk_size  = 50
  vpc_id            = tencentcloud_vpc.main.id
  subnet_id         = tencentcloud_subnet.main.id
}
```

### 2.5 AWS Provider 配置（中国区）

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "cn-northwest-1"  # 宁夏区域
  # 或 "cn-north-1"  # 北京区域
  
  # 使用环境变量或 shared credentials
  # AWS_ACCESS_KEY_ID
  # AWS_SECRET_ACCESS_KEY
}

# 创建 VPC
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support   = true
  
  tags = {
    Name = "${local.name_prefix}-vpc"
  }
}

# 创建子网
resource "aws_subnet" "public" {
  count             = 2
  vpc_id            = aws_vpc.main.id
  cidr_block        = cidrsubnet(aws_vpc.main.cidr_block, 8, count.index)
  availability_zone = data.aws_availability_zones.available.names[count.index]
  
  tags = {
    Name = "${local.name_prefix}-public-${count.index}"
  }
}
```

---

## 3. Terraform + GitHub Actions CI/CD

### 3.1 基本工作流

```yaml
name: Terraform CI/CD

on:
  push:
    branches: [main]
    paths:
    - 'terraform/**'
  pull_request:
    branches: [main]
    paths:
    - 'terraform/**'

env:
  TF_DIR: terraform
  TF_VERSION: 1.7.0

jobs:
  terraform-plan:
    runs-on: ubuntu-latest
    if: github.event_name == 'pull_request'
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Setup Terraform
      uses: hashicorp/setup-terraform@v3
      with:
        terraform_version: ${{ env.TF_VERSION }}
    
    - name: Configure credentials
      run: |
        cat > backend.tf << 'EOF'
        terraform {
          backend "oss" {
            bucket = "my-terraform-state"
            prefix = "my-project"
            region = "cn-hangzhou"
          }
        }
        EOF
    
    - name: Terraform Init
      run: terraform init
      working-directory: ${{ env.TF_DIR }}
    
    - name: Terraform Format Check
      run: terraform fmt -check -recursive
      working-directory: ${{ env.TF_DIR }}
    
    - name: Terraform Validate
      run: terraform validate
      working-directory: ${{ env.TF_DIR }}
    
    - name: Terraform Plan
      id: plan
      run: |
        terraform plan -no-color -out=tfplan 2>&1 | tee plan_output.txt
      working-directory: ${{ env.TF_DIR }}
      env:
        ALICLOUD_ACCESS_KEY: ${{ secrets.ALICLOUD_ACCESS_KEY }}
        ALICLOUD_SECRET_KEY: ${{ secrets.ALICLOUD_SECRET_KEY }}
        ALICLOUD_REGION: cn-hangzhou
    
    - name: Comment PR with Plan
      uses: actions/github-script@v7
      with:
        script: |
          const fs = require('fs');
          const plan = fs.readFileSync('${{ env.TF_DIR }}/plan_output.txt', 'utf8');
          const truncated = plan.length > 60000 ? plan.substring(0, 60000) + '\n... (truncated)' : plan;
          
          github.rest.issues.createComment({
            issue_number: context.issue.number,
            owner: context.repo.owner,
            repo: context.repo.repo,
            body: `## Terraform Plan 输出
          
          \`\`\`hcl
          ${truncated}
          \`\`\`
          
          *触发者: @${{ github.actor }}, Commit: \`${{ github.sha }}\`*`
          });

  terraform-apply:
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    environment: production
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Setup Terraform
      uses: hashicorp/setup-terraform@v3
      with:
        terraform_version: ${{ env.TF_VERSION }}
    
    - name: Terraform Init
      run: terraform init
      working-directory: ${{ env.TF_DIR }}
      env:
        ALICLOUD_ACCESS_KEY: ${{ secrets.ALICLOUD_ACCESS_KEY }}
        ALICLOUD_SECRET_KEY: ${{ secrets.ALICLOUD_SECRET_KEY }}
    
    - name: Terraform Apply
      run: |
        terraform apply -auto-approve
      working-directory: ${{ env.TF_DIR }}
      env:
        ALICLOUD_ACCESS_KEY: ${{ secrets.ALICLOUD_ACCESS_KEY }}
        ALICLOUD_SECRET_KEY: ${{ secrets.ALICLOUD_SECRET_KEY }}
        ALICLOUD_REGION: cn-hangzhou
```

### 3.2 使用 OIDC 认证（推荐）

```yaml
name: Terraform with OIDC

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

permissions:
  id-token: write
  contents: read
  pull-requests: write

jobs:
  terraform:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Setup Terraform
      uses: hashicorp/setup-terraform@v3
    
    - name: Configure Alibaba Cloud OIDC
      run: |
        # 获取 GitHub OIDC Token
        OIDC_TOKEN=$(curl -H "Authorization: bearer $ACTIONS_ID_TOKEN_REQUEST_TOKEN" \
          "$ACTIONS_ID_TOKEN_REQUEST_URL&audience=sts.aliyuncs.com" | jq -r '.value')
        
        # 使用 STS Assume Role
        STS_RESPONSE=$(aliyun sts AssumeRoleWithOIDC \
          --RoleArn "acs:ram::123456789:role/terraform-role" \
          --OIDCProviderArn "acs:ram::123456789:oidc-provider/github" \
          --OIDCToken "$OIDC_TOKEN" \
          --RoleSessionName "github-actions")
        
        # 设置环境变量
        echo "ALICLOUD_ACCESS_KEY=$(echo $STS_RESPONSE | jq -r '.Credentials.AccessKeyId')" >> $GITHUB_ENV
        echo "ALICLOUD_SECRET_KEY=$(echo $STS_RESPONSE | jq -r '.Credentials.AccessKeySecret')" >> $GITHUB_ENV
        echo "ALICLOUD_SECURITY_TOKEN=$(echo $STS_RESPONSE | jq -r '.Credentials.SecurityToken')" >> $GITHUB_ENV
    
    - name: Terraform Init & Apply
      run: |
        terraform init
        terraform plan -out=tfplan
        terraform apply tfplan
      working-directory: terraform
```

### 3.3 完整 CI/CD 流水线

```yaml
name: Terraform Full Pipeline

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  # 阶段1：代码质量检查
  quality:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    
    - name: Terraform Format Check
      uses: hashicorp/setup-terraform@v3
    
    - name: Check formatting
      run: terraform fmt -check -recursive
      working-directory: terraform
    
    - name: TFLint
      uses: terraform-linters/setup-tflint@v4
      run: |
        tflint --init
        tflint --recursive
      working-directory: terraform
    
    - name: tfsec Security Scan
      uses: aquasecurity/tfsec-action@v1.0.3
      with:
        working_directory: terraform
    
    - name: Checkov Scan
      uses: bridgecrewio/checkov-action@v12
      with:
        directory: terraform
        framework: terraform

  # 阶段2：Plan
  plan:
    needs: quality
    runs-on: ubuntu-latest
    outputs:
      has_changes: ${{ steps.plan.outputs.has_changes }}
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Setup Terraform
      uses: hashicorp/setup-terraform@v3
    
    - name: Terraform Init
      run: terraform init
      working-directory: terraform
    
    - name: Terraform Plan
      id: plan
      run: |
        terraform plan -no-color -detailed-exitcode -out=tfplan 2>&1 | tee plan.txt
        if [ $? -eq 2 ]; then
          echo "has_changes=true" >> $GITHUB_OUTPUT
        else
          echo "has_changes=false" >> $GITHUB_OUTPUT
        fi
      working-directory: terraform
    
    - name: Upload Plan Artifact
      if: steps.plan.outputs.has_changes == 'true'
      uses: actions/upload-artifact@v4
      with:
        name: tfplan
        path: terraform/tfplan
        retention-days: 5

  # 阶段3：Apply（仅 main 分支）
  apply:
    needs: plan
    if: github.ref == 'refs/heads/main' && needs.plan.outputs.has_changes == 'true'
    runs-on: ubuntu-latest
    environment: production
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Download Plan Artifact
      uses: actions/download-artifact@v4
      with:
        name: tfplan
        path: terraform
    
    - name: Setup Terraform
      uses: hashicorp/setup-terraform@v3
    
    - name: Terraform Init
      run: terraform init
      working-directory: terraform
    
    - name: Terraform Apply
      run: terraform apply -auto-approve tfplan
      working-directory: terraform
```

---

## 4. Terraform State 管理（远程 Backend）

### 4.1 为什么需要远程 State

本地 state 文件的问题：
- 无法多人协作
- 容易丢失
- 无法实现状态锁定

### 4.2 阿里云 OSS Backend

```hcl
# backend.tf
terraform {
  backend "oss" {
    bucket              = "my-terraform-state"
    prefix              = "my-project/production"
    region              = "cn-hangzhou"
    encrypt             = true
    tablestore_endpoint = "https://tf-state-lock.cn-hangzhou.ots.aliyuncs.com"
    tablestore_table    = "terraform_lock"
  }
}
```

**创建 OSS Bucket 和 TableStore：**

```hcl
# 先手动创建存储后端资源
provider "alicloud" {
  region = "cn-hangzhou"
}

# OSS Bucket
resource "alicloud_oss_bucket" "terraform_state" {
  bucket = "my-terraform-state"
  acl    = "private"
  
  versioning {
    status = "Enabled"
  }
  
  server_side_encryption_rule {
    sse_algorithm = "AES256"
  }
  
  lifecycle {
    prevent_destroy = true
  }
}

# TableStore（用于状态锁定）
resource "alicloud_ots_table" "terraform_lock" {
  instance_name = "terraform-state-lock"
  table_name    = "terraform_lock"
  
  primary_key {
    name = "LockID"
    type = "String"
  }
  
  time_to_live = -1
  max_version  = 1
}
```

### 4.3 腾讯云 COS Backend

```hcl
terraform {
  backend "cos" {
    region = "ap-guangzhou"
    bucket = "my-terraform-state-1250000000"
    prefix = "my-project/production"
    
    encrypt = true
  }
}
```

### 4.4 AWS S3 Backend（中国区）

```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "my-project/production/terraform.tfstate"
    region         = "cn-northwest-1"
    encrypt        = true
    dynamodb_table = "terraform-lock"
    
    # 中国区需要特殊配置
    endpoints {
      s3 = "https://s3.cn-northwest-1.amazonaws.com.cn"
    }
  }
}
```

### 4.5 使用 GitHub Actions 管理 State

```yaml
# 在 GitHub Actions 中安全地管理 state
name: Terraform State Management

on:
  workflow_dispatch:
    inputs:
      action:
        description: 'Action to perform'
        required: true
        type: choice
        options:
        - list
        - show
        - import
        - remove

jobs:
  state:
    runs-on: ubuntu-latest
    environment: production
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Setup Terraform
      uses: hashicorp/setup-terraform@v3
    
    - name: Terraform Init
      run: terraform init
      working-directory: terraform
    
    - name: List Resources
      if: inputs.action == 'list'
      run: terraform state list
      working-directory: terraform
    
    - name: Show Resource
      if: inputs.action == 'show'
      run: terraform state show ${{ github.event.inputs.resource }}
      working-directory: terraform
```

### 4.6 State 迁移

```bash
# 从本地迁移到远程 Backend
# 1. 配置远程 Backend
# 2. 运行 terraform init
# 3. Terraform 会询问是否迁移 state

terraform init
# 输出：
# Initializing the backend...
# Do you want to copy existing state to the new backend?
#   Pre-existing state was found while migrating the previous backend to the
#   newly configured backend. Do you want to copy this state to the new
#   backend? Enter "yes" to copy and "no" to start with an empty state.
#   Enter a value: yes
```

---

## 5. Terraform 模块化开发

### 5.1 模块结构

```
modules/
├── vpc/
│   ├── main.tf
│   ├── variables.tf
│   ├── outputs.tf
│   └── README.md
├── ecs/
│   ├── main.tf
│   ├── variables.tf
│   ├── outputs.tf
│   └── README.md
└── slb/
    ├── main.tf
    ├── variables.tf
    ├── outputs.tf
    └── README.md
```

### 5.2 VPC 模块示例

```hcl
# modules/vpc/main.tf
resource "alicloud_vpc" "this" {
  vpc_name   = var.vpc_name
  cidr_block = var.cidr_block
  
  tags = merge(var.tags, {
    Name = var.vpc_name
  })
}

resource "alicloud_vswitch" "this" {
  count        = length(var.availability_zones)
  vpc_id       = alicloud_vpc.this.id
  cidr_block   = cidrsubnet(var.cidr_block, 8, count.index)
  zone_id      = var.availability_zones[count.index]
  vswitch_name = "${var.vpc_name}-vsw-${count.index}"
  
  tags = merge(var.tags, {
    Name = "${var.vpc_name}-vsw-${count.index}"
  })
}

resource "alicloud_nat_gateway" "this" {
  count              = var.enable_nat_gateway ? 1 : 0
  vpc_id             = alicloud_vpc.this.id
  nat_gateway_name   = "${var.vpc_name}-nat"
  payment_type       = "PayAsYouGo"
  vswitch_id         = alicloud_vswitch.this[0].id
  nat_type           = "Enhanced"
}
```

```hcl
# modules/vpc/variables.tf
variable "vpc_name" {
  description = "VPC 名称"
  type        = string
}

variable "cidr_block" {
  description = "VPC CIDR"
  type        = string
  default     = "172.16.0.0/12"
}

variable "availability_zones" {
  description = "可用区列表"
  type        = list(string)
}

variable "enable_nat_gateway" {
  description = "是否创建 NAT 网关"
  type        = bool
  default     = false
}

variable "tags" {
  description = "资源标签"
  type        = map(string)
  default     = {}
}
```

```hcl
# modules/vpc/outputs.tf
output "vpc_id" {
  description = "VPC ID"
  value       = alicloud_vpc.this.id
}

output "vswitch_ids" {
  description = "交换机 ID 列表"
  value       = alicloud_vswitch.this[*].id
}

output "nat_gateway_id" {
  description = "NAT 网关 ID"
  value       = var.enable_nat_gateway ? alicloud_nat_gateway.this[0].id : null
}
```

### 5.3 使用模块

```hcl
# environments/production/main.tf
module "vpc" {
  source = "../../modules/vpc"
  
  vpc_name           = "production-vpc"
  cidr_block         = "172.16.0.0/12"
  availability_zones = ["cn-hangzhou-h", "cn-hangzhou-i"]
  enable_nat_gateway = true
  
  tags = local.common_tags
}

module "ecs_cluster" {
  source = "../../modules/ecs"
  
  cluster_name       = "production-cluster"
  vpc_id             = module.vpc.vpc_id
  vswitch_ids        = module.vpc.vswitch_ids
  instance_type      = "ecs.g6.large"
  instance_count     = 3
  
  tags = local.common_tags
}

module "load_balancer" {
  source = "../../modules/slb"
  
  slb_name           = "production-slb"
  vpc_id             = module.vpc.vpc_id
  vswitch_id         = module.vpc.vswitch_ids[0]
  backend_server_ids = module.ecs_cluster.instance_ids
  
  tags = local.common_tags
}
```

### 5.4 模块版本管理

```hcl
# 使用 Git 仓库作为模块源
module "vpc" {
  source = "git::https://github.com/my-org/terraform-modules.git//vpc?ref=v1.2.0"
}

# 使用 Terraform Registry
module "vpc" {
  source  = "terraform-alicloud-modules/vpc/alicloud"
  version = "~> 1.0"
}

# 使用本地路径（开发阶段）
module "vpc" {
  source = "../../modules/vpc"
}
```

### 5.5 模块测试与文档

```hcl
# modules/vpc/tests/vpc_test.go
package test

import (
	"testing"
	"github.com/gruntwork-io/terratest/modules/terraform"
	"github.com/stretchr/testify/assert"
)

func TestVpcModule(t *testing.T) {
	terraformOptions := terraform.WithDefaultRetryableErrors(t, &terraform.Options{
		TerraformDir: "../",
		Vars: map[string]interface{}{
			"vpc_name":           "test-vpc",
			"cidr_block":         "10.0.0.0/16",
			"availability_zones": []string{"cn-hangzhou-h"},
		},
	})

	defer terraform.Destroy(t, terraformOptions)
	terraform.InitAndApply(t, terraformOptions)

	vpcId := terraform.Output(t, terraformOptions, "vpc_id")
	assert.NotEmpty(t, vpcId)
}
```

---

## 6. Terraform 与 GitHub Provider

### 6.1 配置 GitHub Provider

```hcl
terraform {
  required_providers {
    github = {
      source  = "integrations/github"
      version = "~> 6.0"
    }
  }
}

provider "github" {
  owner = "my-org"
  token = var.github_token
}
```

### 6.2 管理 GitHub 仓库

```hcl
# 创建仓库
resource "github_repository" "app" {
  name        = "my-app"
  description = "My application repository"
  visibility  = "private"
  
  has_issues   = true
  has_projects = true
  has_wiki     = false
  
  auto_init          = true
  gitignore_template = "Go"
  license_template   = "mit"
  
  # 分支保护
  # 注意：需要单独使用 github_branch_protection 资源
}

# 分支保护规则
resource "github_branch_protection" "main" {
  repository_id = github_repository.app.node_id
  pattern       = "main"
  
  enforce_admins = true
  
  required_pull_request_reviews {
    required_approving_review_count = 2
    dismiss_stale_reviews          = true
    require_code_owner_reviews     = true
  }
  
  required_status_checks {
    strict   = true
    contexts = ["ci/build", "ci/test"]
  }
  
  restrict_pushes {
    blocks_creations = true
    push_allowances  = ["my-org/admins"]
  }
}
```

### 6.3 管理 GitHub Secrets

```hcl
# 仓库 Secrets
resource "github_actions_secret" "alicloud_access_key" {
  repository      = github_repository.app.name
  secret_name     = "ALICLOUD_ACCESS_KEY"
  plaintext_value = var.alicloud_access_key
}

resource "github_actions_secret" "alicloud_secret_key" {
  repository      = github_repository.app.name
  secret_name     = "ALICLOUD_SECRET_KEY"
  plaintext_value = var.alicloud_secret_key
}

# 环境 Secrets
resource "github_repository_environment" "production" {
  repository  = github_repository.app.name
  environment = "production"
  
  reviewers {
    users = [data.github_user.admin.id]
  }
  
  deployment_branch_policy {
    protected_branches     = true
    custom_branch_policies = false
  }
}

resource "github_actions_environment_secret" "kubeconfig" {
  repository      = github_repository.app.name
  environment     = github_repository_environment.production.environment
  secret_name     = "KUBECONFIG"
  plaintext_value = base64encode(var.kubeconfig)
}
```

### 6.4 管理 GitHub Teams

```hcl
# 创建团队
resource "github_team" "developers" {
  name        = "developers"
  description = "开发团队"
  privacy     = "closed"
}

resource "github_team" "devops" {
  name        = "devops"
  description = "DevOps 团队"
  privacy     = "closed"
}

# 团队成员
resource "github_team_members" "developers" {
  team_id = github_team.developers.id
  
  members {
    username = "developer1"
    role     = "member"
  }
  
  members {
    username = "developer2"
    role     = "member"
  }
}

# 团队仓库权限
resource "github_team_repository" "developers" {
  team_id    = github_team.developers.id
  repository = github_repository.app.name
  permission = "push"
}

resource "github_team_repository" "devops" {
  team_id    = github_team.devops.id
  repository = github_repository.app.name
  permission = "admin"
}
```

### 6.5 管理 GitHub Actions Workflows

```hcl
# 使用 GitHub Actions 变量
resource "github_actions_variable" "environment" {
  repository    = github_repository.app.name
  variable_name = "DEPLOY_ENVIRONMENT"
  value         = "production"
}

# 组织级 Secrets
resource "github_actions_organization_secret" "shared_token" {
  secret_name     = "SHARED_TOKEN"
  visibility      = "selected"
  plaintext_value = var.shared_token
  
  selected_repository_ids = [
    github_repository.app.repo_id
  ]
}
```

---

## 7. GitHub Actions Runner 部署到云厂商

### 7.1 自托管 Runner 概述

自托管 Runner 的优势：
- 访问内网资源
- 自定义运行环境
- 无 GitHub 托管限制
- 成本可控

### 7.2 阿里云 ECS Runner

```hcl
# modules/github-runner/main.tf
resource "alicloud_instance" "runner" {
  count             = var.runner_count
  instance_name     = "${var.runner_name}-${count.index}"
  image_id          = data.alicloud_images.ubuntu.images[0].id
  instance_type     = var.instance_type
  security_groups   = [alicloud_security_group.runner.id]
  vswitch_id        = var.vswitch_id
  system_disk_category = "cloud_essd"
  system_disk_size  = 100
  
  user_data = base64encode(templatefile("${path.module}/userdata.sh", {
    github_token = var.github_token
    runner_name  = "${var.runner_name}-${count.index}"
    repo_url     = var.repo_url
    labels       = var.runner_labels
  }))
  
  tags = merge(var.tags, {
    Name = "${var.runner_name}-${count.index}"
    Role = "github-runner"
  })
}

resource "alicloud_security_group" "runner" {
  name   = "${var.runner_name}-sg"
  vpc_id = var.vpc_id
}

# 允许 HTTPS 出站（与 GitHub 通信）
resource "alicloud_security_group_rule" "allow_https_out" {
  type              = "egress"
  ip_protocol       = "tcp"
  nic_type          = "intranet"
  policy            = "accept"
  port_range        = "443/443"
  security_group_id = alicloud_security_group.runner.id
  cidr_ip           = "0.0.0.0/0"
}
```

```bash
#!/bin/bash
# modules/github-runner/userdata.sh
set -e

# 安装依赖
apt-get update
apt-get install -y curl jq

# 创建 runner 用户
useradd -m -s /bin/bash runner
usermod -aG sudo runner

# 下载 GitHub Actions Runner
RUNNER_VERSION="2.311.0"
cd /home/runner
curl -o actions-runner-linux-x64-${RUNNER_VERSION}.tar.gz -L \
  https://github.com/actions/runner/releases/download/v${RUNNER_VERSION}/actions-runner-linux-x64-${RUNNER_VERSION}.tar.gz
tar xzf actions-runner-linux-x64-${RUNNER_VERSION}.tar.gz
rm actions-runner-linux-x64-${RUNNER_VERSION}.tar.gz

# 配置 Runner
chown -R runner:runner /home/runner
su - runner -c "./config.sh \
  --url ${repo_url} \
  --token ${github_token} \
  --name ${runner_name} \
  --labels ${labels} \
  --work _work \
  --unattended"

# 安装为服务
./svc.sh install runner
./svc.sh start
```

### 7.3 阿里云 ACK Runner（K8s 部署）

```hcl
# 使用 Actions Runner Controller (ARC)
resource "helm_release" "arc" {
  name       = "actions-runner-controller"
  repository = "https://actions-runner-controller.github.io/actions-runner-controller"
  chart      = "actions-runner-controller"
  namespace  = "arc-systems"
  create_namespace = true
  
  set {
    name  = "authSecret.create"
    value = "true"
  }
  
  set {
    name  = "authSecret.github_token"
    value = var.github_token
  }
}
```

```yaml
# Runner 部署配置
apiVersion: actions.summerwind.dev/v1alpha1
kind: RunnerDeployment
metadata:
  name: github-runner
  namespace: arc-systems
spec:
  replicas: 3
  template:
    spec:
      repository: my-org/my-repo
      labels:
        - self-hosted
        - linux
        - x64
      resources:
        requests:
          cpu: "500m"
          memory: "1Gi"
        limits:
          cpu: "2"
          memory: "4Gi"
```

### 7.4 腾讯云 CVM Runner

```hcl
resource "tencentcloud_instance" "runner" {
  count             = var.runner_count
  instance_name     = "github-runner-${count.index}"
  availability_zone = "ap-guangzhou-3"
  image_id          = "img-2lr9q49h"
  instance_type     = "SA2.MEDIUM4"
  system_disk_type  = "CLOUD_SSD"
  system_disk_size  = 100
  
  user_data = base64encode(templatefile("${path.module}/userdata.sh", {
    github_token = var.github_token
    runner_name  = "github-runner-${count.index}"
    repo_url     = var.repo_url
  }))
}
```

### 7.5 Runner 自动伸缩

```hcl
# 使用 ARC 的 RunnerSet 实现自动伸缩
resource "kubernetes_manifest" "runner_set" {
  manifest = {
    apiVersion = "actions.summerwind.dev/v1alpha1"
    kind       = "RunnerSet"
    metadata = {
      name      = "github-runner"
      namespace = "arc-systems"
    }
    spec = {
      replicas = 2
      repository = "my-org/my-repo"
      labels = ["self-hosted", "linux"]
      
      # 基于队列长度自动伸缩
      # 需要配置 HorizontalRunnerAutoscaler
    }
  }
}

resource "kubernetes_manifest" "autoscaler" {
  manifest = {
    apiVersion = "actions.summerwind.dev/v1alpha1"
    kind       = "HorizontalRunnerAutoscaler"
    metadata = {
      name      = "github-runner-autoscaler"
      namespace = "arc-systems"
    }
    spec = {
      scaleTargetRef = {
        kind = "RunnerSet"
        name = "github-runner"
      }
      minReplicas = 1
      maxReplicas = 10
      scaleMetrics = [
        {
          type = "TotalNumberOfQueuedAndInProgressWorkflowRuns"
          repositoryNames = ["my-repo"]
        }
      ]
    }
  }
}
```

---

## 8. 多环境 Terraform 管理

### 8.1 目录结构方式

```
terraform/
├── modules/                  # 共享模块
│   ├── vpc/
│   ├── ecs/
│   └── slb/
├── environments/
│   ├── dev/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── outputs.tf
│   │   ├── terraform.tfvars
│   │   └── backend.tf
│   ├── staging/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── outputs.tf
│   │   ├── terraform.tfvars
│   │   └── backend.tf
│   └── production/
│       ├── main.tf
│       ├── variables.tf
│       ├── outputs.tf
│       ├── terraform.tfvars
│       └── backend.tf
└── README.md
```

**环境配置示例：**

```hcl
# environments/dev/terraform.tfvars
environment    = "development"
region         = "cn-hangzhou"
instance_type  = "ecs.g6.large"
instance_count = 1
enable_monitoring = false

# environments/production/terraform.tfvars
environment    = "production"
region         = "cn-hangzhou"
instance_type  = "ecs.g6.2xlarge"
instance_count = 5
enable_monitoring = true
```

### 8.2 Terraform Workspace 方式

```bash
# 创建工作空间
terraform workspace new dev
terraform workspace new staging
terraform workspace new production

# 切换工作空间
terraform workspace select production

# 列出工作空间
terraform workspace list

# 显示当前工作空间
terraform workspace show
```

```hcl
# 使用 workspace 区分环境
locals {
  env = terraform.workspace
  
  instance_type = {
    dev      = "ecs.g6.large"
    staging  = "ecs.g6.xlarge"
    production = "ecs.g6.2xlarge"
  }
  
  instance_count = {
    dev      = 1
    staging  = 2
    production = 5
  }
}

resource "alicloud_instance" "web" {
  count         = local.instance_count[local.env]
  instance_type = local.instance_type[local.env]
  # ...
}
```

### 8.3 Terragrunt 方式

```hcl
# terragrunt.hcl（根配置）
remote_state {
  backend = "oss"
  generate = {
    path      = "backend.tf"
    if_exists = "overwrite"
  }
  config = {
    bucket = "my-terraform-state"
    prefix = "${path_relative_to_include()}/terraform.tfstate"
    region = "cn-hangzhou"
  }
}

generate "provider" {
  path      = "provider.tf"
  if_exists = "overwrite"
  contents = <<EOF
provider "alicloud" {
  region = var.region
}
EOF
}
```

```hcl
# environments/production/vpc/terragrunt.hcl
include "root" {
  path = find_in_parent_folders()
}

terraform {
  source = "../../../modules/vpc"
}

inputs = {
  vpc_name           = "production-vpc"
  cidr_block         = "172.16.0.0/12"
  availability_zones = ["cn-hangzhou-h", "cn-hangzhou-i"]
  enable_nat_gateway = true
}
```

### 8.4 GitHub Actions 多环境部署

```yaml
name: Terraform Multi-Environment

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        environment: [dev, staging, production]
        include:
        - environment: dev
          auto_approve: true
        - environment: staging
          auto_approve: true
        - environment: production
          auto_approve: false
    
    environment: ${{ matrix.environment }}
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Setup Terraform
      uses: hashicorp/setup-terraform@v3
    
    - name: Terraform Init
      run: terraform init
      working-directory: terraform/environments/${{ matrix.environment }}
    
    - name: Terraform Plan
      run: terraform plan -out=tfplan
      working-directory: terraform/environments/${{ matrix.environment }}
    
    - name: Terraform Apply
      if: matrix.auto_approve && github.ref == 'refs/heads/main'
      run: terraform apply -auto-approve tfplan
      working-directory: terraform/environments/${{ matrix.environment }}
    
    - name: Manual Approval Required
      if: !matrix.auto_approve && github.ref == 'refs/heads/main'
      uses: trstringer/manual-approval@v1
      with:
        secret: ${{ secrets.GITHUB_TOKEN }}
        approvers: my-org/admins
```

---

## 9. Terraform 安全扫描

### 9.1 tfsec

```yaml
# 在 GitHub Actions 中使用 tfsec
- name: Run tfsec
  uses: aquasecurity/tfsec-action@v1.0.3
  with:
    working_directory: terraform
    soft_fail: false
    format: json
    output: tfsec-results.json

- name: Upload tfsec results
  uses: github/codeql-action/upload-sarif@v3
  if: always()
  with:
    sarif_file: tfsec-results.sarif
```

**tfsec 配置文件：**

```yaml
# .tfsec/config.yml
minimum_severity: MEDIUM
exclude:
  - aws-vpc-no-public-ingress-sgr  # 排除特定规则
  - alicloud-ecs-no-public-ingress-sgr
```

### 9.2 Checkov

```yaml
# 在 GitHub Actions 中使用 Checkov
- name: Run Checkov
  uses: bridgecrewio/checkov-action@v12
  with:
    directory: terraform
    framework: terraform
    output_format: json
    output_file_path: checkov-results.json
    soft_fail: false
    skip_check: CKV_AWS_18  # 跳过特定检查
```

**Checkov 配置文件：**

```yaml
# .checkov.yml
framework:
- terraform
directory:
- terraform
skip-check:
- CKV_AWS_18  # S3 访问日志
- CKV_ALI_1   # 阿里云特定检查
```

### 9.3 TFLint

```yaml
# 安装和运行 TFLint
- name: Setup TFLint
  uses: terraform-linters/setup-tflint@v4

- name: Run TFLint
  run: |
    tflint --init
    tflint --recursive --format json > tflint-results.json
  working-directory: terraform
```

**TFLint 配置：**

```hcl
# .tflint.hcl
plugin "alicloud" {
  enabled = true
  version = "0.25.0"
  source  = "github.com/terraform-linters/tflint-ruleset-alicloud"
}

rule "terraform_naming_convention" {
  enabled = true
  format  = "snake_case"
}

rule "terraform_documented_variables" {
  enabled = true
}

rule "terraform_documented_outputs" {
  enabled = true
}
```

### 9.4 综合安全扫描工作流

```yaml
name: Terraform Security Scan

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  security:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Run tfsec
      uses: aquasecurity/tfsec-action@v1.0.3
      with:
        working_directory: terraform
    
    - name: Run Checkov
      uses: bridgecrewio/checkov-action@v12
      with:
        directory: terraform
    
    - name: Setup TFLint
      uses: terraform-linters/setup-tflint@v4
    
    - name: Run TFLint
      run: |
        tflint --init
        tflint --recursive
      working-directory: terraform
    
    - name: Terraform Validate
      run: |
        terraform init -backend=false
        terraform validate
      working-directory: terraform
```

---

## 10. Terratest 测试框架

### 10.1 Terratest 简介

Terratest 是 Gruntwork 开发的 Go 测试框架，用于编写基础设施的自动化测试。

### 10.2 安装

```bash
# 初始化 Go 模块
cd test
go mod init github.com/my-org/terraform-tests
go mod tidy

# 安装依赖
go get github.com/gruntwork-io/terratest
go get github.com/stretchr/testify
```

### 10.3 VPC 模块测试

```go
// test/vpc_test.go
package test

import (
	"testing"
	"github.com/gruntwork-io/terratest/modules/terraform"
	"github.com/stretchr/testify/assert"
	"github.com/stretchr/testify/require"
)

func TestVpcModule(t *testing.T) {
	t.Parallel()

	terraformOptions := terraform.WithDefaultRetryableErrors(t, &terraform.Options{
		TerraformDir: "../modules/vpc",
		Vars: map[string]interface{}{
			"vpc_name":           "test-vpc",
			"cidr_block":         "10.0.0.0/16",
			"availability_zones": []string{"cn-hangzhou-h"},
		},
		PlanFilePath: "tfplan",
	})

	defer terraform.Destroy(t, terraformOptions)

	terraform.InitAndApply(t, terraformOptions)

	// 验证输出
	vpcId := terraform.Output(t, terraformOptions, "vpc_id")
	assert.NotEmpty(t, vpcId)

	vswitchIds := terraform.OutputList(t, terraformOptions, "vswitch_ids")
	require.Len(t, vswitchIds, 1)
	assert.NotEmpty(t, vswitchIds[0])
}

func TestVpcWithNatGateway(t *testing.T) {
	t.Parallel()

	terraformOptions := terraform.WithDefaultRetryableErrors(t, &terraform.Options{
		TerraformDir: "../modules/vpc",
		Vars: map[string]interface{}{
			"vpc_name":           "test-vpc-nat",
			"cidr_block":         "10.1.0.0/16",
			"availability_zones": []string{"cn-hangzhou-h", "cn-hangzhou-i"},
			"enable_nat_gateway": true,
		},
	})

	defer terraform.Destroy(t, terraformOptions)
	terraform.InitAndApply(t, terraformOptions)

	natGatewayId := terraform.Output(t, terraformOptions, "nat_gateway_id")
	assert.NotEmpty(t, natGatewayId)
}
```

### 10.4 ECS 模块测试

```go
// test/ecs_test.go
package test

import (
	"testing"
	"fmt"
	http_helper "github.com/gruntwork-io/terratest/modules/http-helper"
	"github.com/gruntwork-io/terratest/modules/terraform"
	"github.com/stretchr/testify/assert"
)

func TestEcsModule(t *testing.T) {
	t.Parallel()

	terraformOptions := terraform.WithDefaultRetryableErrors(t, &terraform.Options{
		TerraformDir: "../modules/ecs",
		Vars: map[string]interface{}{
			"cluster_name":   "test-cluster",
			"vpc_id":         "vpc-xxx",       // 使用实际的 VPC ID
			"vswitch_ids":    []string{"vsw-xxx"},
			"instance_type":  "ecs.g6.large",
			"instance_count": 2,
		},
	})

	defer terraform.Destroy(t, terraformOptions)
	terraform.InitAndApply(t, terraformOptions)

	// 验证实例数量
	instanceIds := terraform.OutputList(t, terraformOptions, "instance_ids")
	assert.Len(t, instanceIds, 2)

	// 验证实例可访问
	publicIp := terraform.Output(t, terraformOptions, "public_ip")
	url := fmt.Sprintf("http://%s", publicIp)
	http_helper.HttpGet(t, url, nil, 200, "Welcome")
}
```

### 10.5 集成测试工作流

```yaml
name: Terraform Tests

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Setup Go
      uses: actions/setup-go@v5
      with:
        go-version: '1.22'
    
    - name: Setup Terraform
      uses: hashicorp/setup-terraform@v3
    
    - name: Run Tests
      run: |
        cd test
        go test -v -timeout 30m ./...
      env:
        ALICLOUD_ACCESS_KEY: ${{ secrets.ALICLOUD_ACCESS_KEY }}
        ALICLOUD_SECRET_KEY: ${{ secrets.ALICLOUD_SECRET_KEY }}
        ALICLOUD_REGION: cn-hangzhou
```

---

## 11. OpenTofu vs Terraform

### 11.1 背景

2023 年，HashiCorp 将 Terraform 的许可证从 MPL 2.0 更改为 BSL 1.1，引发社区创建了 OpenTofu 作为开源替代。

### 11.2 对比

| 特性 | Terraform | OpenTofu |
|------|-----------|----------|
| 许可证 | BSL 1.1 | MPL 2.0 |
| 维护者 | HashiCorp | Linux Foundation |
| 兼容性 | 原版 | 高度兼容 |
| 模块注册表 | Terraform Registry | OpenTofu Registry |
| 状态加密 | 不支持 | 支持 |
| 社区 | 较大 | 增长中 |

### 11.3 迁移到 OpenTofu

```bash
# 安装 OpenTofu
curl -fsSL https://get.opentofu.org/install-opentofu.sh | bash

# 或使用 Homebrew
brew install opentofu

# 迁移步骤
# 1. 备份当前 state
cp terraform.tfstate terraform.tfstate.backup

# 2. 初始化 OpenTofu（兼容现有配置）
tofu init

# 3. 验证状态
tofu plan

# 4. 确认无误后使用 tofu 命令替代 terraform
tofu apply
```

### 11.4 GitHub Actions 中使用 OpenTofu

```yaml
name: OpenTofu CI/CD

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  tofu:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Setup OpenTofu
      uses: opentofu/setup-opentofu@v1
      with:
        tofu_version: '1.6.0'
    
    - name: Tofu Init
      run: tofu init
      working-directory: terraform
    
    - name: Tofu Plan
      run: tofu plan
      working-directory: terraform
    
    - name: Tofu Apply
      if: github.ref == 'refs/heads/main'
      run: tofu apply -auto-approve
      working-directory: terraform
```

### 11.5 OpenTofu 特有功能

```hcl
# OpenTofu 支持 state 加密
terraform {
  backend "s3" {
    bucket = "my-terraform-state"
    key    = "terraform.tfstate"
    region = "cn-hangzhou"
    
    # OpenTofu 独有：state 加密
    encryption {
      key_provider "pbkdf2" "my_key" {
        passphrase = var.encryption_passphrase
      }
      
      state {
        method = method.aes_gcm.my_key
      }
    }
  }
}
```

---

## 12. 国内云厂商 Terraform Provider

### 12.1 阿里云 Provider

```hcl
# 阿里云 Provider
terraform {
  required_providers {
    alicloud = {
      source  = "aliyun/alicloud"
      version = "~> 1.220"
    }
  }
}

# 常用资源
resource "alicloud_vpc" "main" { }
resource "alicloud_vswitch" "main" { }
resource "alicloud_instance" "main" { }
resource "alicloud_slb_load_balancer" "main" { }
resource "alicloud_db_instance" "main" { }
resource "alicloud_redis_instance" "main" { }
resource "alicloud_oss_bucket" "main" { }
resource "alicloud_cs_managed_kubernetes" "main" { }
resource "alicloud_cdn_domain_new" "main" { }
resource "alicloud_dns_record" "main" { }
```

### 12.2 腾讯云 Provider

```hcl
# 腾讯云 Provider
terraform {
  required_providers {
    tencentcloud = {
      source  = "tencentcloudstack/tencentcloud"
      version = "~> 1.81"
    }
  }
}

# 常用资源
resource "tencentcloud_vpc" "main" { }
resource "tencentcloud_subnet" "main" { }
resource "tencentcloud_instance" "main" { }
resource "tencentcloud_clb_instance" "main" { }
resource "tencentcloud_mysql_instance" "main" { }
resource "tencentcloud_redis_instance" "main" { }
resource "tencentcloud_cos_bucket" "main" { }
resource "tencentcloud_kubernetes_cluster" "main" { }
resource "tencentcloud_cdn_domain" "main" { }
resource "tencentcloud_dns_record" "main" { }
```

### 12.3 华为云 Provider

```hcl
# 华为云 Provider
terraform {
  required_providers {
    huaweicloud = {
      source  = "huaweicloud/huaweicloud"
      version = "~> 1.60"
    }
  }
}

# 常用资源
resource "huaweicloud_vpc_v1" "main" { }
resource "huaweicloud_vpc_subnet_v1" "main" { }
resource "huaweicloud_compute_instance_v2" "main" { }
resource "huaweicloud_lb_loadbalancer_v2" "main" { }
resource "huaweicloud_rds_instance_v3" "main" { }
resource "huaweicloud_redis_instance" "main" { }
resource "huaweicloud_obs_bucket" "main" { }
resource "huaweicloud_cce_cluster" "main" { }
resource "huaweicloud_cdn_domain" "main" { }
resource "huaweicloud_dns_recordset_v2" "main" { }
```

### 12.4 多云 Provider 配置

```hcl
# 同时管理多个云厂商
provider "alicloud" {
  alias  = "hangzhou"
  region = "cn-hangzhou"
}

provider "alicloud" {
  alias  = "shanghai"
  region = "cn-shanghai"
}

provider "tencentcloud" {
  alias  = "guangzhou"
  region = "ap-guangzhou"
}

provider "huaweicloud" {
  alias  = "beijing"
  region = "cn-north-4"
}

# 阿里云杭州资源
resource "alicloud_vpc" "hangzhou" {
  provider   = alicloud.hangzhou
  vpc_name   = "hangzhou-vpc"
  cidr_block = "172.16.0.0/12"
}

# 腾讯云广州资源
resource "tencentcloud_vpc" "guangzhou" {
  provider   = tencentcloud.guangzhou
  name       = "guangzhou-vpc"
  cidr_block = "10.0.0.0/16"
}
```

### 12.5 国内 Provider 资源对照表

| 资源类型 | 阿里云 | 腾讯云 | 华为云 |
|----------|--------|--------|--------|
| VPC | `alicloud_vpc` | `tencentcloud_vpc` | `huaweicloud_vpc_v1` |
| 子网 | `alicloud_vswitch` | `tencentcloud_subnet` | `huaweicloud_vpc_subnet_v1` |
| 云服务器 | `alicloud_instance` | `tencentcloud_instance` | `huaweicloud_compute_instance_v2` |
| 负载均衡 | `alicloud_slb_load_balancer` | `tencentcloud_clb_instance` | `huaweicloud_lb_loadbalancer_v2` |
| RDS | `alicloud_db_instance` | `tencentcloud_mysql_instance` | `huaweicloud_rds_instance_v3` |
| Redis | `alicloud_redis_instance` | `tencentcloud_redis_instance` | `huaweicloud_redis_instance` |
| OSS/COS/OBS | `alicloud_oss_bucket` | `tencentcloud_cos_bucket` | `huaweicloud_obs_bucket` |
| K8s | `alicloud_cs_managed_kubernetes` | `tencentcloud_kubernetes_cluster` | `huaweicloud_cce_cluster` |
| CDN | `alicloud_cdn_domain_new` | `tencentcloud_cdn_domain` | `huaweicloud_cdn_domain` |
| DNS | `alicloud_dns_record` | `tencentcloud_dns_record` | `huaweicloud_dns_recordset_v2` |

---

## 13. IaC 最佳实践

### 13.1 代码组织

```
terraform/
├── modules/                    # 可复用模块
│   ├── networking/
│   ├── compute/
│   ├── database/
│   └── monitoring/
├── environments/               # 环境配置
│   ├── dev/
│   ├── staging/
│   └── production/
├── global/                     # 全局资源
│   ├── iam/
│   ├── dns/
│   └── terraform.tfstate.d/
└── scripts/                    # 辅助脚本
    ├── init-backend.sh
    └── migrate-state.sh
```

### 13.2 命名规范

```hcl
# 资源命名规范
# 格式：{project}-{environment}-{component}-{resource_type}

# 示例
resource "alicloud_vpc" "main" {
  vpc_name = "myapp-production-vpc"
}

resource "alicloud_instance" "web" {
  instance_name = "myapp-production-web-01"
}

# 标签规范
locals {
  common_tags = {
    Project     = "myapp"
    Environment = "production"
    ManagedBy   = "terraform"
    Team        = "platform"
    CostCenter  = "engineering"
  }
}
```

### 13.3 版本锁定

```hcl
# 使用精确版本
terraform {
  required_version = "= 1.7.0"
  
  required_providers {
    alicloud = {
      source  = "aliyun/alicloud"
      version = "= 1.220.0"
    }
  }
}

# 或使用版本约束
terraform {
  required_version = ">= 1.5.0, < 2.0.0"
  
  required_providers {
    alicloud = {
      source  = "aliyun/alicloud"
      version = "~> 1.220"  # 允许 1.220.x 补丁版本
    }
  }
}
```

### 13.4 状态管理最佳实践

```hcl
# 1. 使用远程 Backend
terraform {
  backend "oss" {
    bucket = "my-terraform-state"
    prefix = "my-project"
  }
}

# 2. 启用状态加密
terraform {
  backend "s3" {
    bucket  = "my-terraform-state"
    key     = "terraform.tfstate"
    encrypt = true
  }
}

# 3. 使用状态锁定
# 阿里云：使用 TableStore
# AWS：使用 DynamoDB
# 腾讯云：COS 自动锁定
```

### 13.5 安全最佳实践

```hcl
# 1. 不要在代码中硬编码敏感信息
variable "db_password" {
  description = "数据库密码"
  type        = string
  sensitive   = true  # 标记为敏感
}

# 2. 使用 Secret Manager
resource "alicloud_kms_secret" "db_password" {
  secret_name = "db-password"
  secret_data = var.db_password
}

# 3. 最小权限原则
# 为 Terraform 创建专用的 RAM 角色，只授予必要的权限

# 4. 使用 OIDC 认证
# 避免使用长期 AccessKey
```

### 13.6 CI/CD 最佳实践

```yaml
# 1. PR 自动 Plan
# 2. 合并后自动 Apply
# 3. 使用环境审批
# 4. 保存 Plan 产物
# 5. 添加安全扫描

name: Terraform Best Practices

on:
  pull_request:
    branches: [main]

jobs:
  terraform:
    runs-on: ubuntu-latest
    
    steps:
    # 格式检查
    - name: Terraform Format
      run: terraform fmt -check -recursive
    
    # 安全扫描
    - name: Security Scan
      uses: aquasecurity/tfsec-action@v1.0.3
    
    # Lint 检查
    - name: TFLint
      run: tflint --recursive
    
    # Plan
    - name: Terraform Plan
      run: terraform plan -out=tfplan
    
    # 评论 PR
    - name: Comment Plan
      uses: actions/github-script@v7
```

### 13.7 文档最佳实践

```hcl
# 1. 为每个变量添加描述
variable "instance_type" {
  description = "ECS 实例类型，推荐使用 ecs.g6 系列"
  type        = string
  default     = "ecs.g6.large"
}

# 2. 为每个输出添加描述
output "vpc_id" {
  description = "创建的 VPC ID，用于其他模块引用"
  value       = alicloud_vpc.main.id
}

# 3. 使用 README 文档化模块
# modules/vpc/README.md

# 4. 使用 terraform-docs 自动生成文档
```

### 13.8 成本优化

```hcl
# 1. 使用抢占式/Spot 实例
resource "alicloud_instance" "spot" {
  spot_strategy    = "SpotAsPriceGo"
  spot_price_limit = "0.5"
}

# 2. 自动伸缩
resource "alicloud_ess_scaling_group" "main" {
  min_size = 1
  max_size = 10
}

# 3. 资源标签用于成本追踪
locals {
  cost_tags = {
    CostCenter = "engineering"
    Team       = "platform"
  }
}

# 4. 使用 terraform-cost-estimation
# https://github.com/infracost/infracost
```

### 13.9 回滚策略

```bash
# 1. 使用版本控制回滚
git revert <commit-hash>
git push

# 2. 使用状态回滚
terraform state pull > terraform.tfstate.backup
terraform apply -target=resource.name

# 3. 使用 Terraform Cloud/Enterprise 的版本化状态

# 4. 保留足够的状态历史
```

---

## 附录：完整项目示例

```
my-terraform-project/
├── .github/
│   └── workflows/
│       ├── terraform-plan.yml
│       ├── terraform-apply.yml
│       └── terraform-destroy.yml
├── modules/
│   ├── vpc/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── outputs.tf
│   │   └── README.md
│   ├── ecs/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── outputs.tf
│   │   └── README.md
│   └── rds/
│       ├── main.tf
│       ├── variables.tf
│       ├── outputs.tf
│       └── README.md
├── environments/
│   ├── dev/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── outputs.tf
│   │   ├── terraform.tfvars
│   │   └── backend.tf
│   ├── staging/
│   │   └── ...
│   └── production/
│       └── ...
├── tests/
│   ├── go.mod
│   ├── go.sum
│   └── vpc_test.go
├── scripts/
│   ├── init-backend.sh
│   └── setup-credentials.sh
├── .tfsec/
│   └── config.yml
├── .tflint.hcl
├── .gitignore
└── README.md
```

**.gitignore 文件：**

```gitignore
# Terraform
*.tfstate
*.tfstate.backup
*.tfstate.*.backup
.terraform/
.terraform.lock.hcl
crash.log
crash.*.log
*.tfvars
!*.tfvars.example
override.tf
override.tf.json
*_override.tf
*_override.tf.json
.terraformrc
terraform.rc

# IDE
.idea/
.vscode/
*.swp
*.swo

# OS
.DS_Store
Thumbs.db
```

---

## 总结

本教程详细介绍了 Terraform IaC 与 GitHub 集成的完整实践方案，涵盖了：

- **IaC 概念**：基础设施即代码的核心理念和优势
- **Terraform 基础**：HCL 语法、Provider 配置、核心资源
- **CI/CD 集成**：GitHub Actions 自动化 Plan/Apply 流程
- **State 管理**：远程 Backend、状态锁定、加密存储
- **模块化开发**：可复用模块设计、版本管理、测试
- **GitHub Provider**：使用 Terraform 管理 GitHub 资源
- **Runner 部署**：自托管 Runner 的云上部署方案
- **多环境管理**：目录结构、Workspace、Terragrunt 等方案
- **安全扫描**：tfsec、Checkov、TFLint 工具链
- **Terratest**：基础设施自动化测试框架
- **OpenTofu**：Terraform 的开源替代方案
- **国内云厂商**：阿里云、腾讯云、华为云 Provider 详解
- **最佳实践**：代码组织、命名规范、安全、成本优化

通过这些实践，中国开发者可以构建高效、安全、可维护的基础设施即代码体系。
