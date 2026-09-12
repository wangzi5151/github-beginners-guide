# Kubernetes + GitHub Actions 实战指南

> 本教程面向中国开发者，系统讲解如何将 Kubernetes 与 GitHub Actions 深度集成，实现从代码提交到生产部署的全自动化流水线。

---

## 目录

1. [Kubernetes 基础概念回顾](#1-kubernetes-基础概念回顾)
2. [GitHub Actions 部署到 K8s 的方案](#2-github-actions-部署到-k8s-的方案)
3. [kubectl 配置与 GitHub Secrets](#3-kubectl-配置与-github-secrets)
4. [Helm Chart + GitHub Actions 部署](#4-helm-chart--github-actions-部署)
5. [Kustomize + GitHub Actions 部署](#5-kustomize--github-actions-部署)
6. [ArgoCD + GitOps 工作流](#6-argocd--gitops-工作流)
7. [Flux CD + GitHub 集成](#7-flux-cd--github-集成)
8. [K8s 多环境管理（dev/staging/prod）](#8-k8s-多环境管理devstagingprod)
9. [K8s 自动扩缩容配置](#9-k8s-自动扩缩容配置)
10. [监控与告警（Prometheus + Grafana）](#10-监控与告警prometheus--grafana)
11. [K8s 安全最佳实践](#11-k8s-安全最佳实践)
12. [云厂商 K8s 服务](#12-云厂商-k8s-服务)
13. [K8s 成本优化](#13-k8s-成本优化)
14. [国内 K8s 部署注意事项](#14-国内-k8s-部署注意事项)

---

## 1. Kubernetes 基础概念回顾

### 1.1 什么是 Kubernetes

Kubernetes（简称 K8s）是由 Google 开源的容器编排平台，用于自动化部署、扩展和管理容器化应用程序。它已经成为云原生领域的事实标准，被国内外各大企业广泛采用。

### 1.2 核心架构

Kubernetes 采用主从架构，主要包含以下组件：

**控制平面（Control Plane）：**

- `kube-apiserver`：API 服务器，所有操作的入口
- `etcd`：分布式键值存储，保存集群状态
- `kube-scheduler`：调度器，决定 Pod 运行在哪个节点
- `kube-controller-manager`：控制器管理器，维护期望状态

**工作节点（Worker Node）：**

- `kubelet`：节点代理，管理 Pod 生命周期
- `kube-proxy`：网络代理，实现 Service 负载均衡
- `Container Runtime`：容器运行时（Docker、containerd、CRI-O）

### 1.3 核心资源对象

```yaml
# Pod - 最小部署单元
apiVersion: v1
kind: Pod
metadata:
  name: my-app
  labels:
    app: my-app
spec:
  containers:
  - name: my-app
    image: my-registry.com/my-app:v1.0.0
    ports:
    - containerPort: 8080
    resources:
      requests:
        cpu: "100m"
        memory: "128Mi"
      limits:
        cpu: "500m"
        memory: "512Mi"
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
      initialDelaySeconds: 30
      periodSeconds: 10
    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 5
---
# Deployment - 无状态应用部署
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app
        image: my-registry.com/my-app:v1.0.0
        ports:
        - containerPort: 8080
---
# Service - 服务发现与负载均衡
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  selector:
    app: my-app
  ports:
  - port: 80
    targetPort: 8080
  type: ClusterIP
---
# Ingress - HTTP 路由
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: my-app.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-app-service
            port:
              number: 80
```

### 1.4 Kubernetes 对象管理方式

| 方式 | 说明 | 适用场景 |
|------|------|----------|
| 命令式命令 | `kubectl run/create/delete` | 临时测试 |
| 命令式对象配置 | `kubectl create -f manifest.yaml` | 简单部署 |
| 声明式对象配置 | `kubectl apply -f manifest.yaml` | 生产环境（推荐） |

### 1.5 为什么选择 Kubernetes + GitHub Actions

GitHub Actions 作为 CI/CD 平台的优势：

- **原生集成**：与 GitHub 仓库无缝集成，代码推送即触发
- **丰富的生态**：Marketplace 提供数千个现成的 Action
- **矩阵构建**：支持多平台、多版本并行测试
- **自托管 Runner**：可在内网 K8s 集群中运行 Runner
- **安全机制**：Secrets、OIDC、环境审批等完善的安全体系

---

## 2. GitHub Actions 部署到 K8s 的方案

### 2.1 常见部署方案对比

| 方案 | 复杂度 | 安全性 | 适用场景 |
|------|--------|--------|----------|
| kubectl 直接部署 | 低 | 中 | 小型项目、快速验证 |
| Helm Chart 部署 | 中 | 中 | 中大型项目、多环境 |
| Kustomize 部署 | 中 | 中 | 多环境变体管理 |
| ArgoCD (GitOps) | 高 | 高 | 企业级生产环境 |
| Flux CD (GitOps) | 高 | 高 | 企业级生产环境 |

### 2.2 基本流水线架构

```
代码提交 → GitHub Actions CI → 构建镜像 → 推送镜像仓库 → 部署到 K8s
    ↓              ↓              ↓            ↓              ↓
  触发器        测试/扫描      Docker Build   阿里云ACR/     kubectl/Helm
                                               腾讯云CCR/
                                               华为云SWR
```

### 2.3 完整 CI/CD 流水线示例

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
    tags: ['v*']
  pull_request:
    branches: [main]

env:
  REGISTRY: registry.cn-hangzhou.aliyuncs.com
  NAMESPACE: my-namespace
  IMAGE_NAME: my-app

jobs:
  # 阶段1：代码测试
  test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    
    - name: Set up Go
      uses: actions/setup-go@v5
      with:
        go-version: '1.22'
    
    - name: Run tests
      run: go test -v -race -coverprofile=coverage.out ./...
    
    - name: Upload coverage
      uses: codecov/codecov-action@v4
      with:
        file: ./coverage.out

  # 阶段2：构建与推送镜像
  build:
    needs: test
    runs-on: ubuntu-latest
    if: github.event_name == 'push'
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v3
    
    - name: Login to Alibaba Cloud ACR
      uses: docker/login-action@v3
      with:
        registry: ${{ env.REGISTRY }}
        username: ${{ secrets.ACR_USERNAME }}
        password: ${{ secrets.ACR_PASSWORD }}
    
    - name: Extract metadata
      id: meta
      uses: docker/metadata-action@v5
      with:
        images: ${{ env.REGISTRY }}/${{ env.NAMESPACE }}/${{ env.IMAGE_NAME }}
        tags: |
          type=sha,prefix=
          type=ref,event=branch
          type=semver,pattern={{version}}
    
    - name: Build and push
      uses: docker/build-push-action@v5
      with:
        context: .
        push: true
        tags: ${{ steps.meta.outputs.tags }}
        cache-from: type=gha
        cache-to: type=gha,mode=max

  # 阶段3：部署到 Kubernetes
  deploy:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main' || startsWith(github.ref, 'refs/tags/v')
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Set up kubectl
      uses: azure/setup-kubectl@v3
      with:
        version: 'v1.29.0'
    
    - name: Configure kubeconfig
      run: |
        mkdir -p $HOME/.kube
        echo "${{ secrets.KUBECONFIG }}" | base64 -d > $HOME/.kube/config
        chmod 600 $HOME/.kube/config
    
    - name: Deploy to K8s
      run: |
        IMAGE_TAG=${GITHUB_SHA::8}
        kubectl set image deployment/my-app \
          my-app=${{ env.REGISTRY }}/${{ env.NAMESPACE }}/${{ env.IMAGE_NAME }}:${IMAGE_TAG} \
          -n production
        kubectl rollout status deployment/my-app -n production --timeout=300s
    
    - name: Notify on success
      if: success()
      run: |
        # 发送钉钉/飞书/企业微信通知
        curl -X POST "${{ secrets.DINGTALK_WEBHOOK }}" \
          -H 'Content-Type: application/json' \
          -d '{"msgtype":"text","text":{"content":"部署成功: '${{ github.repository }}' @ '${GITHUB_SHA::8}'"}}'
```

---

## 3. kubectl 配置与 GitHub Secrets

### 3.1 获取 kubeconfig

**方式一：从云厂商控制台获取**

以阿里云 ACK 为例：

```bash
# 安装 aliyun CLI
curl -O https://aliyuncli.alicdn.com/aliyun-cli-linux-latest-amd64.tgz
tar xzvf aliyun-cli-linux-latest-amd64.tgz
sudo mv aliyun /usr/local/bin/

# 配置阿里云账号
aliyun configure

# 获取集群 kubeconfig
aliyun cs GET /k8s/{cluster_id}/user_config | jq -r '.config' > kubeconfig.yaml
```

**方式二：使用 kubectl 直接配置**

```bash
# 阿里云 ACK
aliyun cs GET /k8s/{cluster_id}/user_config | jq -r '.config' > ~/.kube/config

# 腾讯云 TKE
# 从控制台下载 kubeconfig 或使用 tke 命令行工具

# 华为云 CCE
# 从控制台下载 kubeconfig 文件
```

**方式三：使用 OIDC Token（更安全）**

```yaml
# 使用 aws-iam-authenticator（EKS）
# 使用 gcp-auth-plugin（GKE）
# 使用 OIDC Token
```

### 3.2 配置 GitHub Secrets

在 GitHub 仓库中设置 Secrets：

1. 进入仓库 → Settings → Secrets and variables → Actions
2. 点击 "New repository secret"
3. 添加以下 Secrets：

| Secret 名称 | 说明 | 获取方式 |
|-------------|------|----------|
| `KUBECONFIG` | Base64 编码的 kubeconfig | `cat ~/.kube/config \| base64 -w 0` |
| `ACR_USERNAME` | 镜像仓库用户名 | 阿里云控制台 |
| `ACR_PASSWORD` | 镜像仓库密码 | 阿里云控制台 |
| `SLACK_WEBHOOK` | 通知 Webhook | Slack/钉钉/飞书设置 |

### 3.3 Base64 编码 kubeconfig

```bash
# Linux
cat ~/.kube/config | base64 -w 0

# macOS
cat ~/.kube/config | base64

# Windows PowerShell
[Convert]::ToBase64String([IO.File]::ReadAllBytes("$env:USERPROFILE\.kube\config"))
```

### 3.4 使用 GitHub Environments 管理多环境

```yaml
# 创建环境：Settings → Environments → New environment
# - development
# - staging
# - production（添加审批规则）

jobs:
  deploy-prod:
    runs-on: ubuntu-latest
    environment: production  # 需要审批
    steps:
    - name: Deploy
      run: kubectl apply -f k8s/prod/
```

### 3.5 使用 OIDC 替代长期凭证（推荐）

GitHub Actions 支持 OIDC（OpenID Connect）与云厂商联合认证，避免存储长期凭证：

```yaml
# 阿里云 OIDC 配置
jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read
    
    steps:
    - name: Configure Alibaba Cloud credentials
      uses: aliyun/oss-upload-action@v1
      with:
        access-key-id: ${{ secrets.ALIYUN_ACCESS_KEY_ID }}
        access-key-secret: ${{ secrets.ALIYUN_ACCESS_KEY_SECRET }}
    
    # 或使用 OIDC（更安全）
    - name: Assume Role via OIDC
      run: |
        # 获取 OIDC Token
        OIDC_TOKEN=$(curl -H "Authorization: bearer $ACTIONS_ID_TOKEN_REQUEST_TOKEN" \
          "$ACTIONS_ID_TOKEN_REQUEST_URL&audience=sts.aliyuncs.com" | jq -r '.value')
        
        # 使用 STS Assume Role
        aliyun sts AssumeRoleWithOIDC \
          --RoleArn "acs:ram::123456789:role/github-actions-role" \
          --OIDCProviderArn "acs:ram::123456789:oidc-provider/github" \
          --OIDCToken "$OIDC_TOKEN" \
          --RoleSessionName "github-actions"
```

**AWS EKS OIDC 配置示例：**

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read
    
    steps:
    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v4
      with:
        role-to-assume: arn:aws:iam::123456789012:role/github-actions-role
        aws-region: cn-northwest-1
    
    - name: Update kubeconfig
      run: aws eks update-kubeconfig --name my-cluster --region cn-northwest-1
```

---

## 4. Helm Chart + GitHub Actions 部署

### 4.1 Helm 基础概念

Helm 是 Kubernetes 的包管理器，将相关资源打包为 Chart：

```
my-chart/
├── Chart.yaml          # Chart 元数据
├── values.yaml         # 默认配置值
├── templates/          # 模板文件
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── hpa.yaml
│   └── _helpers.tpl
├── charts/             # 依赖的子 Chart
└── README.md
```

### 4.2 创建 Helm Chart

```bash
# 创建新的 Chart
helm create my-app

# Chart.yaml
cat > my-app/Chart.yaml << 'EOF'
apiVersion: v2
name: my-app
description: A Helm chart for my application
type: application
version: 0.1.0
appVersion: "1.0.0"
dependencies:
- name: postgresql
  version: "12.x.x"
  repository: "https://charts.bitnami.com/bitnami"
  condition: postgresql.enabled
EOF
```

### 4.3 values.yaml 多环境配置

```yaml
# values.yaml - 默认值
replicaCount: 1

image:
  repository: registry.cn-hangzhou.aliyuncs.com/my-namespace/my-app
  pullPolicy: IfNotPresent
  tag: "latest"

service:
  type: ClusterIP
  port: 80

ingress:
  enabled: false
  className: nginx
  annotations: {}
  hosts:
  - host: my-app.local
    paths:
    - path: /
      pathType: Prefix

resources:
  requests:
    cpu: 100m
    memory: 128Mi
  limits:
    cpu: 500m
    memory: 512Mi

autoscaling:
  enabled: false
  minReplicas: 1
  maxReplicas: 10
  targetCPUUtilizationPercentage: 80

nodeSelector: {}
tolerations: []
affinity: {}
```

```yaml
# values-production.yaml - 生产环境覆盖值
replicaCount: 3

image:
  tag: ""  # 由 CI/CD 动态设置

ingress:
  enabled: true
  className: nginx
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
    nginx.ingress.kubernetes.io/rate-limit: "100"
  hosts:
  - host: my-app.example.com
    paths:
    - path: /
      pathType: Prefix
  tls:
  - secretName: my-app-tls
    hosts:
    - my-app.example.com

resources:
  requests:
    cpu: 500m
    memory: 512Mi
  limits:
    cpu: "2"
    memory: 2Gi

autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 20
  targetCPUUtilizationPercentage: 70

postgresql:
  enabled: true
  auth:
    existingSecret: postgres-credentials
```

### 4.4 GitHub Actions Helm 部署工作流

```yaml
name: Helm Deploy

on:
  push:
    branches: [main]

env:
  RELEASE_NAME: my-app
  NAMESPACE: production
  CHART_PATH: ./helm/my-app

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Set up Helm
      uses: azure/setup-helm@v3
      with:
        version: 'v3.14.0'
    
    - name: Configure kubectl
      run: |
        mkdir -p $HOME/.kube
        echo "${{ secrets.KUBECONFIG }}" | base64 -d > $HOME/.kube/config
    
    - name: Add Helm repos
      run: |
        helm repo add bitnami https://charts.bitnami.com/bitnami
        helm repo update
    
    - name: Update Chart dependencies
      run: |
        helm dependency update ${{ env.CHART_PATH }}
    
    - name: Helm Lint
      run: |
        helm lint ${{ env.CHART_PATH }} -f ${{ env.CHART_PATH }}/values-production.yaml
    
    - name: Helm Diff (dry-run)
      run: |
        helm diff upgrade ${{ env.RELEASE_NAME }} ${{ env.CHART_PATH }} \
          -f ${{ env.CHART_PATH }}/values-production.yaml \
          --namespace ${{ env.NAMESPACE }} \
          --allow-unreleased || true
    
    - name: Deploy with Helm
      run: |
        IMAGE_TAG=${GITHUB_SHA::8}
        helm upgrade --install ${{ env.RELEASE_NAME }} ${{ env.CHART_PATH }} \
          -f ${{ env.CHART_PATH }}/values-production.yaml \
          --namespace ${{ env.NAMESPACE }} \
          --create-namespace \
          --set image.tag=${IMAGE_TAG} \
          --wait \
          --timeout 5m
    
    - name: Verify deployment
      run: |
        kubectl rollout status deployment/${{ env.RELEASE_NAME }} \
          -n ${{ env.NAMESPACE }} --timeout=300s
        kubectl get pods -n ${{ env.NAMESPACE }} -l app.kubernetes.io/name=my-app
```

### 4.5 Helm Chart 测试

```yaml
# 在 GitHub Actions 中运行 Helm 测试
- name: Run Helm tests
  run: |
    helm test ${{ env.RELEASE_NAME }} -n ${{ env.NAMESPACE }} --timeout 5m
```

```yaml
# templates/tests/test-connection.yaml
apiVersion: v1
kind: Pod
metadata:
  name: "{{ include "my-app.fullname" . }}-test-connection"
  labels:
    {{- include "my-app.labels" . | nindent 4 }}
  annotations:
    "helm.sh/hook": test
spec:
  containers:
  - name: wget
    image: busybox
    command: ['wget']
    args: ['{{ include "my-app.fullname" . }}:{{ .Values.service.port }}']
  restartPolicy: Never
```

---

## 5. Kustomize + GitHub Actions 部署

### 5.1 Kustomize 简介

Kustomize 是 Kubernetes 原生的配置管理工具，通过 overlay 机制实现多环境配置管理，无需模板引擎。

### 5.2 项目结构

```
k8s/
├── base/
│   ├── kustomization.yaml
│   ├── deployment.yaml
│   ├── service.yaml
│   └── ingress.yaml
└── overlays/
    ├── dev/
    │   ├── kustomization.yaml
    │   └── namespace.yaml
    ├── staging/
    │   ├── kustomization.yaml
    │   ├── replica-count.yaml
    │   └── resource-limits.yaml
    └── prod/
        ├── kustomization.yaml
        ├── replica-count.yaml
        ├── resource-limits.yaml
        └── hpa.yaml
```

### 5.3 Base 配置

```yaml
# base/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

commonLabels:
  app: my-app
  managed-by: kustomize

resources:
- deployment.yaml
- service.yaml
- ingress.yaml

configMapGenerator:
- name: app-config
  literals:
  - APP_ENV=production
  - LOG_LEVEL=info

secretGenerator:
- name: app-secrets
  type: Opaque
  literals:
  - DB_PASSWORD=changeme

images:
- name: my-app
  newName: registry.cn-hangzhou.aliyuncs.com/my-namespace/my-app
  newTag: latest
```

```yaml
# base/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app
        image: my-app
        ports:
        - containerPort: 8080
        envFrom:
        - configMapRef:
            name: app-config
        - secretRef:
            name: app-secrets
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
```

### 5.4 Overlay 配置

```yaml
# overlays/prod/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

namespace: production

resources:
- ../../base
- hpa.yaml

patches:
- path: replica-count.yaml
- path: resource-limits.yaml

configMapGenerator:
- name: app-config
  behavior: merge
  literals:
  - APP_ENV=production
  - LOG_LEVEL=warn

images:
- name: my-app
  newTag: ""  # 由 CI/CD 动态设置
```

```yaml
# overlays/prod/replica-count.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 5
```

```yaml
# overlays/prod/hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-app
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  minReplicas: 3
  maxReplicas: 20
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

### 5.5 Kustomize + GitHub Actions 工作流

```yaml
name: Kustomize Deploy

on:
  push:
    branches: [main]

env:
  IMAGE: registry.cn-hangzhou.aliyuncs.com/my-namespace/my-app

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Set up kubectl
      uses: azure/setup-kubectl@v3
    
    - name: Configure kubeconfig
      run: |
        mkdir -p $HOME/.kube
        echo "${{ secrets.KUBECONFIG }}" | base64 -d > $HOME/.kube/config
    
    - name: Set image tag
      id: tag
      run: echo "tag=${GITHUB_SHA::8}" >> $GITHUB_OUTPUT
    
    - name: Kustomize build and apply
      run: |
        cd k8s/overlays/prod
        
        # 使用 kustomize edit 设置镜像标签
        kustomize edit set image my-app=${{ env.IMAGE }}:${{ steps.tag.outputs.tag }}
        
        # 构建并应用
        kustomize build . | kubectl apply -f -
        
        # 等待部署完成
        kubectl rollout status deployment/my-app -n production --timeout=300s
```

### 5.6 使用 kubectl 的内置 kustomize

```yaml
# 直接使用 kubectl apply -k
- name: Deploy with kubectl kustomize
  run: |
    cd k8s/overlays/prod
    kustomize edit set image my-app=${{ env.IMAGE }}:${{ steps.tag.outputs.tag }}
    kubectl apply -k .
```

---

## 6. ArgoCD + GitOps 工作流

### 6.1 GitOps 原则

GitOps 是一种以 Git 仓库为唯一事实来源的运维模式：

1. **声明式**：所有配置以声明式方式存储在 Git 中
2. **版本化**：所有变更通过 Git 追踪
3. **自动化**：变更自动应用到集群
4. **自愈**：集群状态与 Git 声明不一致时自动修复

### 6.2 ArgoCD 架构

```
Git Repository (期望状态)
        ↓
   ArgoCD Server
        ↓
   Application Controller → Kubernetes Cluster (实际状态)
        ↓
   Notification Controller → 钉钉/飞书/Slack
```

### 6.3 安装 ArgoCD

```bash
# 创建命名空间
kubectl create namespace argocd

# 安装 ArgoCD
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# 获取初始密码
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

# 访问 UI（端口转发）
kubectl port-forward svc/argocd-server -n argocd 8080:443

# 安装 CLI
curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
```

### 6.4 ArgoCD Application 配置

```yaml
# argocd-application.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
  namespace: argocd
  finalizers:
  - resources-finalizer.argocd.argoproj.io
spec:
  project: default
  source:
    repoURL: https://github.com/my-org/my-app-config.git
    targetRevision: HEAD
    path: overlays/prod
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true        # 删除 Git 中不存在的资源
      selfHeal: true     # 自动修复手动更改
    syncOptions:
    - CreateNamespace=true
    - PrunePropagationPolicy=foreground
    retry:
      limit: 5
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m
```

### 6.5 GitHub Actions + ArgoCD 集成

**方式一：ArgoCD 自动检测 Git 变更（Pull 模式）**

ArgoCD 默认每 3 分钟检查 Git 仓库变更，无需额外配置。

**方式二：GitHub Actions 触发 ArgoCD 同步（Push 模式）**

```yaml
name: Build and Notify ArgoCD

on:
  push:
    branches: [main]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Build and push image
      run: |
        IMAGE_TAG=${GITHUB_SHA::8}
        docker build -t registry.cn-hangzhou.aliyuncs.com/my-namespace/my-app:${IMAGE_TAG} .
        docker push registry.cn-hangzhou.aliyuncs.com/my-namespace/my-app:${IMAGE_TAG}
    
    - name: Update image tag in config repo
      run: |
        # 克隆配置仓库
        git clone https://x-access-token:${{ secrets.CONFIG_REPO_TOKEN }}@github.com/my-org/my-app-config.git
        cd my-app-config
        
        # 更新镜像标签
        cd overlays/prod
        kustomize edit set image my-app=registry.cn-hangzhou.aliyuncs.com/my-namespace/my-app:${GITHUB_SHA::8}
        
        # 提交并推送
        git config user.name "GitHub Actions"
        git config user.email "actions@github.com"
        git add .
        git commit -m "Update image to ${GITHUB_SHA::8}"
        git push
    
    - name: Trigger ArgoCD sync
      run: |
        # 方式一：使用 ArgoCD CLI
        argocd app sync my-app --server argocd.example.com \
          --auth-token ${{ secrets.ARGOCD_TOKEN }} \
          --insecure
        
        # 方式二：使用 ArgoCD API
        curl -X POST "https://argocd.example.com/api/v1/applications/my-app/sync" \
          -H "Authorization: Bearer ${{ secrets.ARGOCD_TOKEN }}" \
          -H "Content-Type: application/json" \
          -d '{}' --insecure
```

### 6.6 ArgoCD Notifications

```yaml
# 配置 ArgoCD 通知
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-notifications-cm
  namespace: argocd
data:
  service.webhook.dingtalk: |
    url: https://oapi.dingtalk.com/robot/send?access_token=xxx
    headers:
    - name: Content-Type
      value: application/json
  
  template.app-sync-succeeded: |
    webhook:
      dingtalk:
        method: POST
        body: |
          {
            "msgtype": "markdown",
            "markdown": {
              "title": "ArgoCD 部署通知",
              "text": "## ArgoCD 部署成功\n\n- **应用**: {{.app.metadata.name}}\n- **状态**: {{.app.status.sync.status}}\n- **提交**: {{.app.status.sync.revision}}"
            }
          }
  
  trigger.on-sync-succeeded: |
    - when: app.status.operationState.phase in ['Succeeded']
      send: [app-sync-succeeded]
```

---

## 7. Flux CD + GitHub 集成

### 7.1 Flux CD 简介

Flux CD 是 CNCF 毕业项目，原生支持 GitOps，与 Kubernetes API 深度集成。

### 7.2 安装 Flux CD

```bash
# 安装 Flux CLI
curl -s https://fluxcd.io/install.sh | sudo bash

# 检查集群是否满足要求
flux check --pre

# Bootstrap Flux（以 GitHub 为例）
flux bootstrap github \
  --owner=my-org \
  --repository=my-cluster-config \
  --branch=main \
  --path=clusters/production \
  --personal
```

### 7.3 Flux GitRepository 配置

```yaml
# git-repository.yaml
apiVersion: source.toolkit.fluxcd.io/v1
kind: GitRepository
metadata:
  name: my-app
  namespace: flux-system
spec:
  interval: 1m
  url: https://github.com/my-org/my-app-config.git
  ref:
    branch: main
  secretRef:
    name: github-credentials
```

### 7.4 Flux Kustomization 配置

```yaml
# kustomization.yaml
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: my-app
  namespace: flux-system
spec:
  interval: 5m
  path: ./overlays/prod
  prune: true
  sourceRef:
    kind: GitRepository
    name: my-app
  healthChecks:
  - apiVersion: apps/v1
    kind: Deployment
    name: my-app
    namespace: production
  timeout: 3m
```

### 7.5 GitHub Actions + Flux 集成

```yaml
name: Update Flux Image

on:
  push:
    branches: [main]

jobs:
  update-image:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Setup Flux CLI
      uses: fluxcd/flux2/action@main
    
    - name: Update image in config repo
      run: |
        IMAGE_TAG=${GITHUB_SHA::8}
        flux create image policy my-app \
          --image=my-app \
          --select-semver=">=1.0.0" \
          --interval=5m \
          --export > policy.yaml
        
        # 或直接使用 flux 命令更新
        flux update kustomization my-app \
          --source=GitRepository/my-app
```

### 7.6 Flux Image Automation

```yaml
# image-repository.yaml
apiVersion: image.toolkit.fluxcd.io/v1beta2
kind: ImageRepository
metadata:
  name: my-app
  namespace: flux-system
spec:
  image: registry.cn-hangzhou.aliyuncs.com/my-namespace/my-app
  interval: 5m

---
# image-policy.yaml
apiVersion: image.toolkit.fluxcd.io/v1beta2
kind: ImagePolicy
metadata:
  name: my-app
  namespace: flux-system
spec:
  imageRepositoryRef:
    name: my-app
  policy:
    semver:
      range: ">=1.0.0"
```

---

## 8. K8s 多环境管理（dev/staging/prod）

### 8.1 环境管理策略

| 策略 | 说明 | 适用场景 |
|------|------|----------|
| 单集群多命名空间 | 一个集群用 Namespace 隔离 | 成本敏感、小团队 |
| 多集群 | 每个环境独立集群 | 安全要求高、大团队 |
| 混合模式 | dev/staging 同集群，prod 独立 | 平衡成本与安全 |

### 8.2 GitHub Actions 多环境部署

```yaml
name: Multi-Environment Deploy

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  # 开发环境 - develop 分支自动部署
  deploy-dev:
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/develop'
    environment: development
    steps:
    - uses: actions/checkout@v4
    - name: Deploy to Dev
      run: |
        kubectl config use-context dev-cluster
        kustomize build k8s/overlays/dev | kubectl apply -f -
        kubectl rollout status deployment/my-app -n development

  # 预发布环境 - main 分支自动部署
  deploy-staging:
    runs-on: ubuntu-latest
    needs: [test, build]
    if: github.ref == 'refs/heads/main'
    environment: staging
    steps:
    - uses: actions/checkout@v4
    - name: Deploy to Staging
      run: |
        kubectl config use-context staging-cluster
        kustomize build k8s/overlays/staging | kubectl apply -f -
        kubectl rollout status deployment/my-app -n staging
    
    - name: Run integration tests
      run: |
        # 运行集成测试
        npm run test:integration -- --base-url=https://staging.example.com

  # 生产环境 - Tag 触发，需要审批
  deploy-prod:
    runs-on: ubuntu-latest
    needs: [deploy-staging]
    if: startsWith(github.ref, 'refs/tags/v')
    environment: production  # 需要人工审批
    steps:
    - uses: actions/checkout@v4
    - name: Deploy to Production
      run: |
        kubectl config use-context prod-cluster
        kustomize build k8s/overlays/prod | kubectl apply -f -
        kubectl rollout status deployment/my-app -n production
    
    - name: Notify success
      run: |
        curl -X POST "${{ secrets.DINGTALK_WEBHOOK }}" \
          -H 'Content-Type: application/json' \
          -d '{"msgtype":"text","text":{"content":"生产部署成功: ${{ github.ref_name }}"}}'
```

### 8.3 使用 Matrix 策略多环境部署

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        environment: [dev, staging]
        include:
        - environment: dev
          namespace: development
          replicas: 1
        - environment: staging
          namespace: staging
          replicas: 2
    
    environment: ${{ matrix.environment }}
    
    steps:
    - uses: actions/checkout@v4
    - name: Deploy to ${{ matrix.environment }}
      run: |
        kustomize build k8s/overlays/${{ matrix.environment }} | kubectl apply -f -
        kubectl scale deployment/my-app --replicas=${{ matrix.replicas }} -n ${{ matrix.namespace }}
```

### 8.4 环境配置管理

```yaml
# 使用 GitHub Environments 配置环境变量
# Settings → Environments → 选择环境 → Environment variables

# 在工作流中使用
- name: Use environment config
  run: |
    echo "Deploying to ${{ vars.CLUSTER_NAME }}"
    echo "Namespace: ${{ vars.NAMESPACE }}"
    echo "Replicas: ${{ vars.REPLICAS }}"
```

---

## 9. K8s 自动扩缩容配置

### 9.1 HPA（水平 Pod 自动扩缩容）

```yaml
# hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-app
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  minReplicas: 2
  maxReplicas: 50
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
  - type: Pods
    pods:
      metric:
        name: http_requests_per_second
      target:
        type: AverageValue
        averageValue: 1000
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
      - type: Pods
        value: 4
        periodSeconds: 60
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 10
        periodSeconds: 60
```

### 9.2 VPA（垂直 Pod 自动扩缩容）

```yaml
# vpa.yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: my-app
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  updatePolicy:
    updateMode: "Auto"
  resourcePolicy:
    containerPolicies:
    - containerName: my-app
      minAllowed:
        cpu: 100m
        memory: 128Mi
      maxAllowed:
        cpu: 4
        memory: 8Gi
      controlledResources: ["cpu", "memory"]
```

### 9.3 Cluster Autoscaler

```yaml
# 阿里云 ACK 集群自动伸缩配置
# 在 ACK 控制台配置节点池的自动伸缩策略

# AWS EKS Cluster Autoscaler
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cluster-autoscaler
  namespace: kube-system
spec:
  replicas: 1
  selector:
    matchLabels:
      app: cluster-autoscaler
  template:
    metadata:
      labels:
        app: cluster-autoscaler
    spec:
      containers:
      - image: registry.aliyuncs.com/acs/autoscaler:v1.26.3-eks
        name: cluster-autoscaler
        command:
        - ./cluster-autoscaler
        - --v=4
        - --cloud-provider=aws
        - --skip-nodes-with-local-storage=false
        - --expander=least-waste
        - --node-group-auto-discovery=asg:tag=k8s.io/cluster-autoscaler/enabled,k8s.io/cluster-autoscaler/my-cluster
```

### 9.4 KEDA（Kubernetes Event-Driven Autoscaling）

```yaml
# 安装 KEDA
helm repo add kedacore https://kedacore.github.io/charts
helm install keda kedacore/keda --namespace keda --create-namespace

# ScaledObject 配置
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: my-app
spec:
  scaleTargetRef:
    name: my-app
  minReplicaCount: 1
  maxReplicaCount: 100
  triggers:
  - type: kafka
    metadata:
      bootstrapServers: kafka:9092
      consumerGroup: my-group
      topic: my-topic
      lagThreshold: "100"
  - type: redis
    metadata:
      address: redis:6379
      listName: my-queue
      listLength: "10"
```

---

## 10. 监控与告警（Prometheus + Grafana）

### 10.1 Prometheus 安装

```bash
# 使用 Helm 安装 kube-prometheus-stack
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

helm install prometheus prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace \
  --set grafana.adminPassword=your-password \
  --set prometheus.prometheusSpec.retention=30d
```

### 10.2 自定义 ServiceMonitor

```yaml
# service-monitor.yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: my-app
  labels:
    release: prometheus
spec:
  selector:
    matchLabels:
      app: my-app
  endpoints:
  - port: metrics
    interval: 15s
    path: /metrics
```

### 10.3 PrometheusRule 告警规则

```yaml
# alert-rules.yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: my-app-alerts
  labels:
    release: prometheus
spec:
  groups:
  - name: my-app
    rules:
    - alert: HighErrorRate
      expr: |
        rate(http_requests_total{service="my-app", status=~"5.."}[5m])
        / rate(http_requests_total{service="my-app"}[5m]) > 0.05
      for: 5m
      labels:
        severity: critical
      annotations:
        summary: "高错误率告警"
        description: "my-app 的 5xx 错误率超过 5%，当前值: {{ $value }}"
    
    - alert: HighLatency
      expr: |
        histogram_quantile(0.95, rate(http_request_duration_seconds_bucket{service="my-app"}[5m])) > 1
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: "高延迟告警"
        description: "my-app 的 P95 延迟超过 1 秒，当前值: {{ $value }}"
    
    - alert: PodCrashLooping
      expr: |
        rate(kube_pod_container_status_restarts_total{namespace="production"}[15m]) > 0
      for: 5m
      labels:
        severity: critical
      annotations:
        summary: "Pod 重启告警"
        description: "Pod {{ $labels.pod }} 正在频繁重启"
```

### 10.4 Grafana Dashboard 配置

```yaml
# grafana-dashboard.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-app-dashboard
  labels:
    grafana_dashboard: "1"
data:
  my-app.json: |
    {
      "dashboard": {
        "title": "My App Dashboard",
        "panels": [
          {
            "title": "请求速率",
            "type": "graph",
            "targets": [
              {
                "expr": "rate(http_requests_total{service=\"my-app\"}[5m])",
                "legendFormat": "{{method}} {{status}}"
              }
            ]
          },
          {
            "title": "延迟分布",
            "type": "heatmap",
            "targets": [
              {
                "expr": "rate(http_request_duration_seconds_bucket{service=\"my-app\"}[5m])",
                "legendFormat": "{{le}}"
              }
            ]
          }
        ]
      }
    }
```

### 10.5 GitHub Actions 部署监控栈

```yaml
name: Deploy Monitoring Stack

on:
  push:
    branches: [main]
    paths:
    - 'monitoring/**'

jobs:
  deploy-monitoring:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    
    - name: Configure kubectl
      run: |
        mkdir -p $HOME/.kube
        echo "${{ secrets.KUBECONFIG }}" | base64 -d > $HOME/.kube/config
    
    - name: Deploy Prometheus
      run: |
        helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
        helm upgrade --install prometheus prometheus-community/kube-prometheus-stack \
          --namespace monitoring \
          --create-namespace \
          -f monitoring/values.yaml \
          --wait
    
    - name: Apply custom rules
      run: |
        kubectl apply -f monitoring/alert-rules.yaml
        kubectl apply -f monitoring/service-monitors.yaml
        kubectl apply -f monitoring/dashboards/
```

---

## 11. K8s 安全最佳实践

### 11.1 RBAC 配置

```yaml
# rbac.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-app-sa
  namespace: production
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: my-app-role
  namespace: production
rules:
- apiGroups: [""]
  resources: ["configmaps", "secrets"]
  verbs: ["get", "list", "watch"]
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: my-app-rolebinding
  namespace: production
subjects:
- kind: ServiceAccount
  name: my-app-sa
  namespace: production
roleRef:
  kind: Role
  name: my-app-role
  apiGroup: rbac.authorization.k8s.io
```

**GitHub Actions 使用最小权限：**

```yaml
# 为 GitHub Actions 创建专用 ServiceAccount
apiVersion: v1
kind: ServiceAccount
metadata:
  name: github-actions-deployer
  namespace: production
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: github-actions-deployer-role
  namespace: production
rules:
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["get", "list", "update", "patch"]
- apiGroups: [""]
  resources: ["pods", "services"]
  verbs: ["get", "list"]
- apiGroups: [""]
  resources: ["configmaps", "secrets"]
  verbs: ["get", "list", "create", "update"]
```

### 11.2 NetworkPolicy

```yaml
# network-policy.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: my-app-netpol
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: my-app
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: ingress-nginx
    ports:
    - protocol: TCP
      port: 8080
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          name: database
    ports:
    - protocol: TCP
      port: 5432
  - to:  # 允许 DNS 查询
    - namespaceSelector: {}
    ports:
    - protocol: UDP
      port: 53
```

### 11.3 Pod Security Standards

```yaml
# pod-security.yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    runAsGroup: 1000
    fsGroup: 1000
    seccompProfile:
      type: RuntimeDefault
  containers:
  - name: my-app
    image: my-app:latest
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop:
        - ALL
    volumeMounts:
    - name: tmp
      mountPath: /tmp
  volumes:
  - name: tmp
    emptyDir: {}
```

### 11.4 镜像安全扫描

```yaml
# 在 GitHub Actions 中集成 Trivy 扫描
- name: Run Trivy vulnerability scanner
  uses: aquasecurity/trivy-action@master
  with:
    image-ref: 'my-app:${{ github.sha }}'
    format: 'sarif'
    output: 'trivy-results.sarif'
    severity: 'CRITICAL,HIGH'
    exit-code: '1'  # 发现高危漏洞时失败

- name: Upload Trivy scan results
  uses: github/codeql-action/upload-sarif@v3
  if: always()
  with:
    sarif_file: 'trivy-results.sarif'
```

### 11.5 Sealed Secrets

```bash
# 安装 Sealed Secrets
kubectl apply -f https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.24.0/controller.yaml

# 安装 kubeseal CLI
wget https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.24.0/kubeseal-0.24.0-linux-amd64.tar.gz
tar xvfz kubeseal-0.24.0-linux-amd64.tar.gz
sudo mv kubeseal /usr/local/bin/

# 创建 Sealed Secret
echo -n mypassword | kubectl create secret generic my-secret \
  --dry-run=client --from-file=password=/dev/stdin -o yaml | \
  kubeseal -o yaml > sealed-secret.yaml
```

---

## 12. 云厂商 K8s 服务

### 12.1 阿里云 ACK（容器服务 Kubernetes 版）

**创建集群：**

```bash
# 使用 aliyun CLI 创建 ACK 集群
aliyun cs POST /clusters --header "Content-Type=application/json" --body '{
  "name": "my-cluster",
  "cluster_type": "ManagedKubernetes",
  "region_id": "cn-hangzhou",
  "kubernetes_version": "1.28.9-aliyun.1",
  "worker_instance_types": ["ecs.g6.large"],
  "num_of_nodes": 3,
  "pod_cidr": "172.20.0.0/16",
  "service_cidr": "172.21.0.0/20"
}'
```

**配置 GitHub Actions：**

```yaml
- name: Setup kubeconfig for ACK
  run: |
    # 使用阿里云 OIDC 或 AccessKey
    aliyun cs GET /k8s/${{ secrets.ACK_CLUSTER_ID }}/user_config | jq -r '.config' > kubeconfig.yaml
    export KUBECONFIG=kubeconfig.yaml
    kubectl get nodes
```

### 12.2 腾讯云 TKE（容器服务）

```yaml
# 使用腾讯云 CLI 配置
- name: Configure TKE
  run: |
    # 安装 tke 命令行工具
    pip install tencentcloud-sdk-python
    
    # 配置 kubeconfig
    # 从控制台下载或使用 API 获取
    echo "${{ secrets.TKE_KUBECONFIG }}" | base64 -d > $HOME/.kube/config
```

### 12.3 华为云 CCE（云容器引擎）

```yaml
# 使用华为云 CLI 配置
- name: Configure CCE
  run: |
    # 安装 cce 命令行工具
    pip install huaweicloudsdkcce
    
    # 配置 kubeconfig
    echo "${{ secrets.CCE_KUBECONFIG }}" | base64 -d > $HOME/.kube/config
```

### 12.4 多云部署策略

```yaml
# 使用 GitHub Actions Matrix 多云部署
jobs:
  deploy-multi-cloud:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        cloud: [alibaba, tencent, huawei]
        include:
        - cloud: alibaba
          cluster: ack-cluster
          registry: registry.cn-hangzhou.aliyuncs.com
        - cloud: tencent
          cluster: tke-cluster
          registry: ccr.ccs.tencentyun.com
        - cloud: huawei
          cluster: cce-cluster
          registry: swr.cn-north-4.myhuaweicloud.com
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Deploy to ${{ matrix.cloud }}
      run: |
        # 配置对应的云厂商凭证
        echo "${{ secrets[format('{0}_KUBECONFIG', matrix.cloud)] }}" | base64 -d > $HOME/.kube/config
        
        # 部署
        kubectl apply -f k8s/
```

---

## 13. K8s 成本优化

### 13.1 资源配额管理

```yaml
# resource-quota.yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: production-quota
  namespace: production
spec:
  hard:
    requests.cpu: "20"
    requests.memory: 40Gi
    limits.cpu: "40"
    limits.memory: 80Gi
    pods: "50"
    services: "20"
    persistentvolumeclaims: "10"
```

### 13.2 LimitRange 配置

```yaml
# limit-range.yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: production-limits
  namespace: production
spec:
  limits:
  - default:
      cpu: "500m"
      memory: "512Mi"
    defaultRequest:
      cpu: "100m"
      memory: "128Mi"
    max:
      cpu: "4"
      memory: "8Gi"
    min:
      cpu: "50m"
      memory: "64Mi"
    type: Container
```

### 13.3 成本监控

```yaml
# 使用 kubecost 进行成本监控
helm install kubecost cost-analyzer \
  --repo https://kubecost.github.io/cost-analyzer/ \
  --namespace kubecost \
  --create-namespace \
  --set kubecostToken="your-token"
```

### 13.4 节点优化策略

```yaml
# 使用 Spot/抢占式实例
# 阿里云抢占式实例配置
apiVersion: v1
kind: Node
metadata:
  labels:
    node-type: spot
spec:
  taints:
  - key: spot
    value: "true"
    effect: NoSchedule

# Pod 配置容忍
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  tolerations:
  - key: spot
    operator: Equal
    value: "true"
    effect: NoSchedule
  nodeSelector:
    node-type: spot
```

### 13.5 GitHub Actions 成本优化

```yaml
# 使用缓存减少构建时间
- name: Cache Docker layers
  uses: actions/cache@v4
  with:
    path: /tmp/.buildx-cache
    key: ${{ runner.os }}-buildx-${{ github.sha }}
    restore-keys: |
      ${{ runner.os }}-buildx-

# 使用自托管 Runner 降低成本
jobs:
  build:
    runs-on: self-hosted  # 使用自托管 Runner
    steps:
    - uses: actions/checkout@v4
```

---

## 14. 国内 K8s 部署注意事项

### 14.1 镜像加速配置

```yaml
# 配置 Docker 镜像加速器
# /etc/docker/daemon.json
{
  "registry-mirrors": [
    "https://mirror.ccs.tencentyun.com",
    "https://registry.cn-hangzhou.aliyuncs.com",
    "https://hub-mirror.c.163.com"
  ]
}
```

### 14.2 使用国内镜像仓库

```yaml
# 阿里云容器镜像服务
image: registry.cn-hangzhou.aliyuncs.com/my-namespace/my-app:v1.0.0

# 腾讯云容器镜像服务
image: ccr.ccs.tencentyun.com/my-namespace/my-app:v1.0.0

# 华为云容器镜像服务
image: swr.cn-north-4.myhuaweicloud.com/my-namespace/my-app:v1.0.0
```

### 14.3 使用国内 Helm 仓库

```bash
# 阿里云 Helm 仓库
helm repo add aliyun https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts

# 微软中国镜像
helm repo add azurecn-mirror https://kubernetesartifacts.azureedge.net/stable

# 腾讯云 Helm 仓库
helm repo add tencentcloud https://mirror.ccs.tencentyun.com
```

### 14.4 解决 GitHub 访问问题

```yaml
# 使用国内 Git 仓库镜像
# Gitee 镜像同步 GitHub 仓库

# 或使用代理
- name: Git proxy
  run: |
    git config --global http.proxy http://proxy.example.com:8080
    git config --global https.proxy http://proxy.example.com:8080

# 使用 ghproxy 加速下载
- name: Download binary
  run: |
    wget https://ghproxy.com/https://github.com/owner/repo/releases/download/v1.0.0/binary-linux-amd64
```

### 14.5 国内 CI/CD 替代方案

| 平台 | 说明 | 特点 |
|------|------|------|
| 云效（阿里云） | 阿里云 DevOps 平台 | 深度集成阿里云服务 |
| CODING（腾讯云） | 腾讯云 DevOps 平台 | 深度集成腾讯云服务 |
| 华为云 DevCloud | 华为云 DevOps 平台 | 深度集成华为云服务 |
| Gitee Go | Gitee CI/CD | 与 Gitee 集成 |
| Jenkins | 开源 CI/CD | 自托管、灵活 |

### 14.6 合规与数据安全

```yaml
# 数据驻留要求
# - 确保 K8s 集群部署在国内区域
# - 镜像仓库使用国内节点
# - 日志和监控数据存储在国内

# 阿里云区域选择
region: cn-hangzhou   # 杭州
region: cn-shanghai   # 上海
region: cn-beijing    # 北京
region: cn-shenzhen   # 深圳
region: cn-guangzhou  # 广州
region: cn-chengdu    # 成都

# 腾讯云区域选择
region: ap-guangzhou  # 广州
region: ap-shanghai   # 上海
region: ap-beijing    # 北京
region: ap-nanjing    # 南京
```

---

## 附录：完整项目结构示例

```
my-k8s-app/
├── .github/
│   └── workflows/
│       ├── ci.yml                    # 持续集成
│       ├── cd-dev.yml                # 开发环境部署
│       ├── cd-prod.yml               # 生产环境部署
│       └── monitoring.yml            # 监控栈部署
├── src/
│   └── ...                          # 应用源代码
├── Dockerfile
├── helm/
│   └── my-app/
│       ├── Chart.yaml
│       ├── values.yaml
│       ├── values-dev.yaml
│       ├── values-staging.yaml
│       ├── values-prod.yaml
│       └── templates/
├── k8s/
│   ├── base/
│   └── overlays/
├── monitoring/
│   ├── alert-rules.yaml
│   ├── service-monitors.yaml
│   └── dashboards/
├── terraform/
│   └── ...                          # 基础设施代码
└── README.md
```

---

## 总结

本教程详细介绍了 Kubernetes 与 GitHub Actions 集成的完整方案，从基础概念到高级实践，涵盖了：

- **CI/CD 流水线**：从代码提交到生产部署的自动化流程
- **部署策略**：kubectl、Helm、Kustomize、ArgoCD、Flux CD 等多种方案
- **多环境管理**：dev/staging/prod 环境隔离与配置管理
- **自动扩缩容**：HPA、VPA、KEDA 等弹性伸缩方案
- **监控告警**：Prometheus + Grafana 可观测性体系
- **安全实践**：RBAC、NetworkPolicy、镜像扫描等安全措施
- **云厂商集成**：阿里云 ACK、腾讯云 TKE、华为云 CCE
- **成本优化**：资源配额、Spot 实例、缓存策略
- **国内实践**：镜像加速、网络优化、合规要求

通过这些实践，中国开发者可以构建高效、安全、可靠的云原生 CI/CD 流水线。
