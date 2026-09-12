# 微服务架构的 GitHub 管理

> 微服务项目的仓库组织、CI/CD 实践、服务治理与可观测性完整指南

---

## 目录

1. [微服务项目仓库组织](#1-微服务项目仓库组织)
2. [微服务 CI/CD 最佳实践](#2-微服务-cicd-最佳实践)
3. [服务网格部署](#3-服务网格部署)
4. [API Gateway 配置与管理](#4-api-gateway-配置与管理)
5. [微服务测试策略](#5-微服务测试策略)
6. [服务发现与注册](#6-服务发现与注册)
7. [分布式追踪](#7-分布式追踪)
8. [日志聚合](#8-日志聚合)
9. [微服务安全](#9-微服务安全)
10. [微服务监控与告警](#10-微服务监控与告警)
11. [蓝绿部署与金丝雀发布](#11-蓝绿部署与金丝雀发布)
12. [微服务拆分与重构策略](#12-微服务拆分与重构策略)
13. [国内微服务框架](#13-国内微服务框架)

---

## 1. 微服务项目仓库组织

### 1.1 Monorepo vs Multirepo

微服务架构面临的第一个重要决策就是仓库组织方式：

**Monorepo（单一仓库）**

```
my-microservices/
├── .github/
│   └── workflows/
│       ├── ci-user-service.yml
│       ├── ci-order-service.yml
│       └── deploy-all.yml
├── services/
│   ├── user-service/
│   │   ├── src/
│   │   ├── Dockerfile
│   │   ├── pom.xml
│   │   └── README.md
│   ├── order-service/
│   │   ├── src/
│   │   ├── Dockerfile
│   │   ├── pom.xml
│   │   └── README.md
│   ├── payment-service/
│   │   ├── src/
│   │   ├── Dockerfile
│   │   ├── pom.xml
│   │   └── README.md
│   └── notification-service/
│       ├── src/
│       ├── Dockerfile
│       ├── pom.xml
│       └── README.md
├── shared/
│   ├── common-lib/
│   ├── proto/
│   └── contracts/
├── infrastructure/
│   ├── kubernetes/
│   ├── terraform/
│   └── helm/
└── docs/
    ├── architecture/
    └── api-docs/
```

**Multirepo（多仓库）**

```
github.com/myorg/
├── user-service/          # 独立仓库
├── order-service/         # 独立仓库
├── payment-service/       # 独立仓库
├── notification-service/  # 独立仓库
├── common-lib/           # 共享库仓库
├── proto-definitions/    # Proto 定义仓库
└── infrastructure/       # 基础设施仓库
```

### 1.2 Monorepo CI/CD 配置

Monorepo 的关键挑战是只构建和部署变更的服务：

```yaml
# .github/workflows/monorepo-ci.yml
name: Monorepo CI

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  detect-changes:
    runs-on: ubuntu-latest
    outputs:
      user-service: ${{ steps.filter.outputs.user-service }}
      order-service: ${{ steps.filter.outputs.order-service }}
      payment-service: ${{ steps.filter.outputs.payment-service }}
      notification-service: ${{ steps.filter.outputs.notification-service }}
      shared: ${{ steps.filter.outputs.shared }}
    steps:
      - uses: actions/checkout@v4
      - uses: dorny/paths-filter@v3
        id: filter
        with:
          filters: |
            user-service:
              - 'services/user-service/**'
              - 'shared/**'
            order-service:
              - 'services/order-service/**'
              - 'shared/**'
            payment-service:
              - 'services/payment-service/**'
              - 'shared/**'
            notification-service:
              - 'services/notification-service/**'
              - 'shared/**'
            shared:
              - 'shared/**'

  build-user-service:
    needs: detect-changes
    if: needs.detect-changes.outputs.user-service == 'true'
    uses: ./.github/workflows/build-service.yml
    with:
      service-name: user-service
    secrets: inherit

  build-order-service:
    needs: detect-changes
    if: needs.detect-changes.outputs.order-service == 'true'
    uses: ./.github/workflows/build-service.yml
    with:
      service-name: order-service
    secrets: inherit

  build-payment-service:
    needs: detect-changes
    if: needs.detect-changes.outputs.payment-service == 'true'
    uses: ./.github/workflows/build-service.yml
    with:
      service-name: payment-service
    secrets: inherit

  deploy-all:
    needs: [build-user-service, build-order-service, build-payment-service]
    if: always() && github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Deploy changed services
        run: |
          # 根据构建结果决定部署哪些服务
          echo "Deploying changed services..."
```

### 1.3 可复用的工作流

创建可复用的工作流模板，避免重复配置：

```yaml
# .github/workflows/build-service.yml
name: Build Service (Reusable)

on:
  workflow_call:
    inputs:
      service-name:
        required: true
        type: string
      java-version:
        required: false
        type: string
        default: '17'
    secrets:
      DOCKER_REGISTRY:
        required: true
      REGISTRY_USERNAME:
        required: true
      REGISTRY_PASSWORD:
        required: true

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-java@v4
        with:
          java-version: ${{ inputs.java-version }}
          distribution: 'temurin'
          cache: 'maven'

      - name: Build with Maven
        working-directory: services/${{ inputs.service-name }}
        run: mvn clean package -DskipTests

      - name: Run tests
        working-directory: services/${{ inputs.service-name }}
        run: mvn test

      - name: Build Docker image
        working-directory: services/${{ inputs.service-name }}
        run: |
          docker build -t ${{ secrets.DOCKER_REGISTRY }}/${{ inputs.service-name }}:${{ github.sha }} .
          docker tag ${{ secrets.DOCKER_REGISTRY }}/${{ inputs.service-name }}:${{ github.sha }} \
            ${{ secrets.DOCKER_REGISTRY }}/${{ inputs.service-name }}:latest

      - name: Push Docker image
        run: |
          echo ${{ secrets.REGISTRY_PASSWORD }} | docker login ${{ secrets.DOCKER_REGISTRY }} -u ${{ secrets.REGISTRY_USERNAME }} --password-stdin
          docker push ${{ secrets.DOCKER_REGISTRY }}/${{ inputs.service-name }}:${{ github.sha }}
          docker push ${{ secrets.DOCKER_REGISTRY }}/${{ inputs.service-name }}:latest
```

---

## 2. 微服务 CI/CD 最佳实践

### 2.1 完整的微服务 CI/CD 流水线

```yaml
# .github/workflows/microservice-pipeline.yml
name: Microservice Pipeline

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

env:
  SERVICE_NAME: user-service
  DOCKER_REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}/user-service

jobs:
  code-quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'
      
      - name: Run Checkstyle
        run: mvn checkstyle:check
      
      - name: Run SpotBugs
        run: mvn spotbugs:check
      
      - name: Run JaCoCo coverage
        run: mvn test jacoco:report
      
      - name: Check coverage threshold
        run: |
          COVERAGE=$(cat target/site/jacoco/jacoco.csv | tail -1 | cut -d',' -f7)
          echo "Coverage: $COVERAGE%"
          if [ "$COVERAGE" -lt 80 ]; then
            echo "Coverage is below 80%"
            exit 1
          fi

  unit-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'
      - run: mvn test

  integration-tests:
    runs-on: ubuntu-latest
    needs: unit-tests
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_DB: testdb
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
      redis:
        image: redis:7
        ports:
          - 6379:6379
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'
      - run: mvn verify -P integration-tests

  contract-tests:
    runs-on: ubuntu-latest
    needs: unit-tests
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'
      - name: Run Pact contract tests
        run: mvn test -P contract-tests
      - name: Publish pacts
        run: |
          mvn pact:publish -Dpact.provider.version=${{ github.sha }}

  build-and-push:
    needs: [code-quality, integration-tests]
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    permissions:
      contents: read
      packages: write
    steps:
      - uses: actions/checkout@v4
      
      - name: Log in to Container Registry
        uses: docker/login-action@v3
        with:
          registry: ${{ env.DOCKER_REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: |
            ${{ env.DOCKER_REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}
            ${{ env.DOCKER_REGISTRY }}/${{ env.IMAGE_NAME }}:latest
          cache-from: type=gha
          cache-to: type=gha,mode=max

  deploy-staging:
    needs: build-and-push
    runs-on: ubuntu-latest
    environment: staging
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup kubectl
        uses: azure/setup-kubectl@v3
      
      - name: Configure kubeconfig
        run: |
          echo "${{ secrets.KUBE_CONFIG }}" | base64 -d > kubeconfig
          export KUBECONFIG=kubeconfig
      
      - name: Deploy to staging
        run: |
          kubectl set image deployment/${{ env.SERVICE_NAME }} \
            ${{ env.SERVICE_NAME }}=${{ env.DOCKER_REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }} \
            -n staging
      
      - name: Wait for rollout
        run: |
          kubectl rollout status deployment/${{ env.SERVICE_NAME }} -n staging --timeout=300s

  smoke-tests:
    needs: deploy-staging
    runs-on: ubuntu-latest
    steps:
      - name: Run smoke tests
        run: |
          # 等待服务就绪
          sleep 30
          
          # 健康检查
          curl -f https://staging-api.example.com/${{ env.SERVICE_NAME }}/actuator/health || exit 1
          
          # 运行端到端测试
          npm install -g newman
          newman run tests/smoke-tests.json \
            --env-var "base_url=https://staging-api.example.com"

  deploy-production:
    needs: smoke-tests
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    environment: production
    steps:
      - uses: actions/checkout@v4
      
      - name: Deploy to production
        run: |
          kubectl set image deployment/${{ env.SERVICE_NAME }} \
            ${{ env.SERVICE_NAME }}=${{ env.DOCKER_REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }} \
            -n production
      
      - name: Wait for rollout
        run: |
          kubectl rollout status deployment/${{ env.SERVICE_NAME }} -n production --timeout=300s
```

### 2.2 Dockerfile 最佳实践

```dockerfile
# services/user-service/Dockerfile
# 多阶段构建
FROM maven:3.9-eclipse-temurin-17 AS builder
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline

COPY src ./src
RUN mvn package -DskipTests

# 运行阶段
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app

# 创建非 root 用户
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# 安装必要工具
RUN apk add --no-cache curl

COPY --from=builder /app/target/*.jar app.jar

# 切换到非 root 用户
USER appuser

# 健康检查
HEALTHCHECK --interval=30s --timeout=3s --start-period=60s --retries=3 \
  CMD curl -f http://localhost:8080/actuator/health || exit 1

EXPOSE 8080

ENTRYPOINT ["java", "-XX:+UseContainerSupport", "-XX:MaxRAMPercentage=75.0", "-jar", "app.jar"]
```

---

## 3. 服务网格部署

### 3.1 Istio 部署

```yaml
# .github/workflows/istio-deploy.yml
name: Deploy with Istio

on:
  push:
    branches: [ main ]

jobs:
  deploy-with-istio:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup kubectl
        uses: azure/setup-kubectl@v3
      
      - name: Setup Istioctl
        run: |
          curl -L https://istio.io/downloadIstio | ISTIO_VERSION=1.20.0 sh -
          export PATH=$PWD/istio-1.20.0/bin:$PATH
      
      - name: Configure kubeconfig
        run: |
          echo "${{ secrets.KUBE_CONFIG }}" | base64 -d > $HOME/.kube/config
      
      - name: Enable sidecar injection
        run: |
          kubectl label namespace production istio-injection=enabled --overwrite
      
      - name: Deploy services with Istio
        run: |
          # 应用 Istio 配置
          kubectl apply -f infrastructure/istio/
          
          # 部署服务
          kubectl apply -f infrastructure/kubernetes/production/
          
          # 等待所有 Pod 就绪
          kubectl wait --for=condition=ready pod -l app=user-service -n production --timeout=300s
      
      - name: Verify Istio configuration
        run: |
          istioctl analyze -n production
```

Istio 流量管理配置：

```yaml
# infrastructure/istio/virtual-service.yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: user-service
  namespace: production
spec:
  hosts:
    - user-service
  http:
    - match:
        - headers:
            x-canary:
              exact: "true"
      route:
        - destination:
            host: user-service
            subset: canary
          weight: 100
    - route:
        - destination:
            host: user-service
            subset: stable
          weight: 90
        - destination:
            host: user-service
            subset: canary
          weight: 10
      retries:
        attempts: 3
        perTryTimeout: 2s
      timeout: 10s
---
# infrastructure/istio/destination-rule.yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: user-service
  namespace: production
spec:
  host: user-service
  subsets:
    - name: stable
      labels:
        version: stable
    - name: canary
      labels:
        version: canary
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 100
      http:
        h2UpgradePolicy: DEFAULT
        http1MaxPendingRequests: 100
        http2MaxRequests: 1000
    outlierDetection:
      consecutive5xxErrors: 5
      interval: 30s
      baseEjectionTime: 30s
      maxEjectionPercent: 50
```

### 3.2 Linkerd 部署

```yaml
# .github/workflows/linkerd-deploy.yml
name: Deploy with Linkerd

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Install Linkerd CLI
        run: |
          curl -fsL https://run.linkerd.io/install | sh
          export PATH=$HOME/.linkerd2/bin:$PATH
      
      - name: Install Linkerd CRDs
        run: |
          linkerd install --crds | kubectl apply -f -
      
      - name: Install Linkerd control plane
        run: |
          linkerd install | kubectl apply -f -
          linkerd check
      
      - name: Inject Linkerd proxy and deploy
        run: |
          kubectl get deploy -n production -o yaml | \
            linkerd inject - | \
            kubectl apply -f -
```

---

## 4. API Gateway 配置与管理

### 4.1 Kong API Gateway

```yaml
# .github/workflows/kong-deploy.yml
name: Kong API Gateway

on:
  push:
    branches: [ main ]
    paths:
      - 'infrastructure/kong/**'

jobs:
  deploy-kong:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Apply Kong configuration
        run: |
          kubectl apply -f infrastructure/kong/
      
      - name: Configure routes via decK
        run: |
          brew install kong/deck/deck
          deck gateway sync infrastructure/kong/kong.yaml \
            --kong-addr https://kong-admin.example.com
```

Kong 配置文件：

```yaml
# infrastructure/kong/kong.yaml
_format_version: "3.0"
services:
  - name: user-service
    url: http://user-service.production.svc.cluster.local:8080
    routes:
      - name: user-routes
        paths:
          - /api/v1/users
        strip_path: false
    plugins:
      - name: rate-limiting
        config:
          minute: 100
          policy: redis
          redis:
            host: redis.production.svc.cluster.local
      - name: jwt
        config:
          uri_param_names:
            - jwt
          header_names:
            - Authorization

  - name: order-service
    url: http://order-service.production.svc.cluster.local:8080
    routes:
      - name: order-routes
        paths:
          - /api/v1/orders
    plugins:
      - name: rate-limiting
        config:
          minute: 200

consumers:
  - username: api-client
    jwt_secrets:
      - key: client-key
        secret: client-secret
```

### 4.2 Spring Cloud Gateway

```yaml
# infrastructure/spring-cloud-gateway/gateway-configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: gateway-config
  namespace: production
data:
  application.yml: |
    spring:
      cloud:
        gateway:
          routes:
            - id: user-service
              uri: lb://user-service
              predicates:
                - Path=/api/v1/users/**
              filters:
                - StripPrefix=0
                - name: CircuitBreaker
                  args:
                    name: user-service-cb
                    fallbackUri: forward:/fallback/users
                - name: RequestRateLimiter
                  args:
                    redis-rate-limiter.replenishRate: 10
                    redis-rate-limiter.burstCapacity: 20

            - id: order-service
              uri: lb://order-service
              predicates:
                - Path=/api/v1/orders/**
              filters:
                - StripPrefix=0
                - name: Retry
                  args:
                    retries: 3
                    statuses: BAD_GATEWAY,SERVICE_UNAVAILABLE

      loadbalancer:
        ribbon:
          enabled: false
```

---

## 5. 微服务测试策略

### 5.1 测试金字塔

```
          /  E2E  \           <- 少量端到端测试
         /----------\
        / Integration \       <- 适量集成测试
       /--------------\
      /   Unit Tests    \     <- 大量单元测试
     /------------------\
    /   Contract Tests   \    <- 契约测试
   /----------------------\
```

### 5.2 契约测试（Pact）

```java
// services/user-service/src/test/java/contract/UserProviderPactTest.java
@Provider("user-service")
@PactFolder("pacts")
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class UserProviderPactTest {

    @LocalServerPort
    private int port;

    @TestTemplate
    @ExtendWith(PactVerificationInvocationContextProvider.class)
    void verifyPact(Pact pact, Interaction interaction, HttpRequest request, PactVerificationContext context) {
        context.verifyInteraction();
    }

    @BeforeEach
    void setup(PactVerificationContext context) {
        context.setTarget(new HttpTestTarget("localhost", port));
    }

    @State("user exists")
    void userExists() {
        // 准备测试数据
        userRepository.save(new User("123", "testuser", "test@example.com"));
    }

    @State("user does not exist")
    void userDoesNotExist() {
        userRepository.deleteAll();
    }
}
```

消费者端测试：

```java
// services/order-service/src/test/java/contract/UserConsumerPactTest.java
@Consumer("order-service")
@PactTestFor(pactVersion = PactVerision.V3)
public class UserConsumerPactTest {

    @Pact(consumer = "order-service", provider = "user-service")
    public RequestResponsePact getUserById(PactDslWithProvider builder) {
        return builder
            .given("user exists")
            .uponReceiving("a request for user")
            .path("/api/v1/users/123")
            .method("GET")
            .willRespondWith()
            .status(200)
            .headers(Map.of("Content-Type", "application/json"))
            .body(new PactDslJsonBody()
                .stringType("id", "123")
                .stringType("username", "testuser")
                .stringType("email", "test@example.com"))
            .toPact();
    }

    @Test
    @PactTestFor(pactMethod = "getUserById")
    void testGetUserById(MockServer mockServer) {
        UserClient client = new UserClient(mockServer.getUrl());
        User user = client.getUserById("123");
        
        assertNotNull(user);
        assertEquals("testuser", user.getUsername());
    }
}
```

### 5.3 集成测试

```yaml
# .github/workflows/integration-tests.yml
name: Integration Tests

on:
  pull_request:
    branches: [ main ]

jobs:
  integration-test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_DB: testdb
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
        ports:
          - 5432:5432
      
      redis:
        image: redis:7
        ports:
          - 6379:6379
      
      kafka:
        image: confluentinc/cp-kafka:7.5.0
        ports:
          - 9092:9092
        env:
          KAFKA_BROKER_ID: 1
          KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
          KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092

    steps:
      - uses: actions/checkout@v4
      
      - name: Start dependent services with Docker Compose
        run: |
          docker-compose -f docker-compose.test.yml up -d
          sleep 30
      
      - name: Run integration tests
        run: mvn verify -P integration-tests
        env:
          SPRING_DATASOURCE_URL: jdbc:postgresql://localhost:5432/testdb
          SPRING_REDIS_HOST: localhost
          SPRING_KAFKA_BOOTSTRAP_SERVERS: localhost:9092
      
      - name: Cleanup
        if: always()
        run: docker-compose -f docker-compose.test.yml down
```

### 5.4 端到端测试

```yaml
# .github/workflows/e2e-tests.yml
name: E2E Tests

on:
  push:
    branches: [ main ]

jobs:
  e2e:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Deploy test environment
        run: |
          kubectl create namespace e2e-test
          kubectl apply -f infrastructure/kubernetes/e2e/ -n e2e-test
          kubectl wait --for=condition=ready pod --all -n e2e-test --timeout=600s
      
      - name: Run E2E tests
        run: |
          npm install -g newman
          newman run tests/e2e/collection.json \
            --env-var "base_url=https://e2e.example.com"
      
      - name: Cleanup
        if: always()
        run: |
          kubectl delete namespace e2e-test
```

---

## 6. 服务发现与注册

### 6.1 Consul 服务发现

```yaml
# infrastructure/consul/consul-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: consul
  namespace: infrastructure
spec:
  replicas: 3
  selector:
    matchLabels:
      app: consul
  template:
    metadata:
      labels:
        app: consul
    spec:
      containers:
        - name: consul
          image: hashicorp/consul:1.17
          args:
            - agent
            - -server
            - -bootstrap-expect=3
            - -ui
            - -client=0.0.0.0
          ports:
            - containerPort: 8500
            - containerPort: 8300
            - containerPort: 8301
            - containerPort: 8302
---
# infrastructure/consul/consul-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: consul
  namespace: infrastructure
spec:
  selector:
    app: consul
  ports:
    - port: 8500
      targetPort: 8500
  type: ClusterIP
```

### 6.2 Nacos 服务发现（阿里开源）

```yaml
# infrastructure/nacos/nacos-deployment.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: nacos
  namespace: infrastructure
spec:
  serviceName: nacos
  replicas: 3
  selector:
    matchLabels:
      app: nacos
  template:
    metadata:
      labels:
        app: nacos
    spec:
      containers:
        - name: nacos
          image: nacos/nacos-server:v2.2.3
          ports:
            - containerPort: 8848
            - containerPort: 9848
          env:
            - name: MODE
              value: "cluster"
            - name: NACOS_SERVERS
              value: "nacos-0.nacos:8848 nacos-1.nacos:8848 nacos-2.nacos:8848"
            - name: SPRING_DATASOURCE_PLATFORM
              value: "mysql"
            - name: MYSQL_SERVICE_HOST
              value: "mysql.infrastructure.svc.cluster.local"
            - name: MYSQL_SERVICE_DB_NAME
              value: "nacos"
            - name: MYSQL_SERVICE_USER
              valueFrom:
                secretKeyRef:
                  name: nacos-db-secret
                  key: username
            - name: MYSQL_SERVICE_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: nacos-db-secret
                  key: password
          volumeMounts:
            - name: nacos-data
              mountPath: /home/nacos/data
  volumeClaimTemplates:
    - metadata:
        name: nacos-data
      spec:
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 10Gi
```

---

## 7. 分布式追踪

### 7.1 Jaeger 部署

```yaml
# infrastructure/jaeger/jaeger-deployment.yaml
apiVersion: jaegertracing.io/v1
kind: Jaeger
metadata:
  name: jaeger
  namespace: monitoring
spec:
  strategy: production
  collector:
    maxReplicas: 5
    resources:
      limits:
        cpu: 500m
        memory: 512Mi
  storage:
    type: elasticsearch
    options:
      es:
        server-urls: http://elasticsearch.monitoring.svc.cluster.local:9200
    esIndexCleaner:
      enabled: true
      numberOfDays: 7
      schedule: "55 23 * * *"
  query:
    replicas: 2
```

### 7.2 Spring Boot 集成 Jaeger

```yaml
# application.yml
management:
  tracing:
    sampling:
      probability: 1.0
  zipkin:
    tracing:
      endpoint: http://jaeger-collector.monitoring.svc.cluster.local:9411/api/v2/spans

# 使用 OpenTelemetry
otel:
  service:
    name: user-service
  exporter:
    otlp:
      endpoint: http://otel-collector.monitoring.svc.cluster.local:4317
```

### 7.3 OpenTelemetry Collector 配置

```yaml
# infrastructure/otel/otel-collector-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: otel-collector-config
  namespace: monitoring
data:
  config.yaml: |
    receivers:
      otlp:
        protocols:
          grpc:
            endpoint: 0.0.0.0:4317
          http:
            endpoint: 0.0.0.0:4318
      jaeger:
        protocols:
          grpc:
            endpoint: 0.0.0.0:14250
          thrift_http:
            endpoint: 0.0.0.0:14268

    processors:
      batch:
        timeout: 5s
        send_batch_size: 1000
      memory_limiter:
        limit_mib: 512
        spike_limit_mib: 128

    exporters:
      prometheus:
        endpoint: "0.0.0.0:8889"
      jaeger:
        endpoint: jaeger-collector.monitoring.svc.cluster.local:14250
        tls:
          insecure: true
      elasticsearch:
        endpoints: ["http://elasticsearch.monitoring.svc.cluster.local:9200"]
        traces_index: otel-traces

    service:
      pipelines:
        traces:
          receivers: [otlp, jaeger]
          processors: [memory_limiter, batch]
          exporters: [jaeger, elasticsearch]
        metrics:
          receivers: [otlp]
          processors: [memory_limiter, batch]
          exporters: [prometheus]
```

---

## 8. 日志聚合

### 8.1 ELK Stack 部署

```yaml
# infrastructure/elk/elasticsearch.yaml
apiVersion: elasticsearch.k8s.elastic.co/v1
kind: Elasticsearch
metadata:
  name: elasticsearch
  namespace: monitoring
spec:
  version: 8.11.0
  nodeSets:
    - name: default
      count: 3
      config:
        node.store.allow_mmap: false
      podTemplate:
        spec:
          containers:
            - name: elasticsearch
              resources:
                requests:
                  memory: 2Gi
                  cpu: 1
                limits:
                  memory: 4Gi
                  cpu: 2
      volumeClaimTemplates:
        - metadata:
            name: elasticsearch-data
          spec:
            accessModes: ["ReadWriteOnce"]
            resources:
              requests:
                storage: 100Gi
---
# infrastructure/elk/kibana.yaml
apiVersion: kibana.k8s.elastic.co/v1
kind: Kibana
metadata:
  name: kibana
  namespace: monitoring
spec:
  version: 8.11.0
  count: 1
  elasticsearchRef:
    name: elasticsearch
```

### 8.2 Fluentd 日志收集

```yaml
# infrastructure/elk/fluentd-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: fluentd-config
  namespace: monitoring
data:
  fluent.conf: |
    <source>
      @type tail
      path /var/log/containers/*.log
      pos_file /var/log/fluentd-containers.log.pos
      tag kubernetes.*
      read_from_head true
      <parse>
        @type json
        time_key time
        time_format %Y-%m-%dT%H:%M:%S.%NZ
      </parse>
    </source>

    <filter kubernetes.**>
      @type kubernetes_metadata
      @id filter_kube_metadata
    </filter>

    <filter kubernetes.**>
      @type record_transformer
      <record>
        service_name ${record.dig("kubernetes", "labels", "app")}
        namespace ${record.dig("kubernetes", "namespace_name")}
        pod_name ${record.dig("kubernetes", "pod_name")}
      </record>
    </filter>

    <match **>
      @type elasticsearch
      host elasticsearch.monitoring.svc.cluster.local
      port 9200
      logstash_format true
      logstash_prefix fluentd-${record["namespace"]}-${record["service_name"]}
      <buffer>
        flush_thread_count 8
        flush_interval 5s
        chunk_limit_size 2M
        queue_limit_length 32
        retry_max_interval 30
        retry_forever true
      </buffer>
    </match>
```

### 8.3 统一日志格式

```json
{
  "timestamp": "2024-01-15T10:30:00.000Z",
  "level": "INFO",
  "service": "user-service",
  "traceId": "abc123def456",
  "spanId": "span789",
  "userId": "user-123",
  "message": "User created successfully",
  "extra": {
    "email": "***@example.com",
    "registrationSource": "web"
  }
}
```

---

## 9. 微服务安全

### 9.1 OAuth2 + JWT 认证

```yaml
# infrastructure/keycloak/keycloak-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: keycloak
  namespace: security
spec:
  replicas: 2
  selector:
    matchLabels:
      app: keycloak
  template:
    metadata:
      labels:
        app: keycloak
    spec:
      containers:
        - name: keycloak
          image: quay.io/keycloak/keycloak:23.0
          args: ["start"]
          env:
            - name: KEYCLOAK_ADMIN
              valueFrom:
                secretKeyRef:
                  name: keycloak-secret
                  key: admin-user
            - name: KEYCLOAK_ADMIN_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: keycloak-secret
                  key: admin-password
            - name: KC_DB
              value: postgres
            - name: KC_DB_URL
              value: jdbc:postgresql://postgres.security.svc.cluster.local:5432/keycloak
          ports:
            - containerPort: 8080
```

Spring Security OAuth2 配置：

```java
// services/user-service/src/main/java/config/SecurityConfig.java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(authorize -> authorize
                .requestMatchers("/actuator/**").permitAll()
                .requestMatchers("/api/v1/public/**").permitAll()
                .requestMatchers("/api/v1/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            .oauth2ResourceServer(oauth2 -> oauth2
                .jwt(jwt -> jwt
                    .jwtAuthenticationConverter(jwtAuthenticationConverter())
                )
            )
            .cors(cors -> cors.configurationSource(corsConfigurationSource()));
        return http.build();
    }

    @Bean
    public JwtAuthenticationConverter jwtAuthenticationConverter() {
        JwtGrantedAuthoritiesConverter grantedAuthoritiesConverter = new JwtGrantedAuthoritiesConverter();
        grantedAuthoritiesConverter.setAuthoritiesClaimName("roles");
        grantedAuthoritiesConverter.setAuthorityPrefix("ROLE_");

        JwtAuthenticationConverter authConverter = new JwtAuthenticationConverter();
        authConverter.setJwtGrantedAuthoritiesConverter(grantedAuthoritiesConverter);
        return authConverter;
    }
}
```

### 9.2 mTLS 双向认证

```yaml
# infrastructure/istio/peer-authentication.yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: production
spec:
  mtls:
    mode: STRICT
---
# infrastructure/istio/authorization-policy.yaml
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: order-service-policy
  namespace: production
spec:
  selector:
    matchLabels:
      app: order-service
  rules:
    - from:
        - source:
            principals:
              - cluster.local/ns/production/sa/user-service
              - cluster.local/ns/production/sa/api-gateway
      to:
        - operation:
            methods: ["GET", "POST"]
            paths: ["/api/v1/orders/*"]
```

### 9.3 API 密钥管理

```yaml
# .github/workflows/secret-rotation.yml
name: Secret Rotation

on:
  schedule:
    - cron: '0 0 1 */3 *'  # 每三个月轮换一次

jobs:
  rotate-secrets:
    runs-on: ubuntu-latest
    steps:
      - name: Generate new API keys
        run: |
          NEW_KEY=$(openssl rand -hex 32)
          echo "NEW_API_KEY=$NEW_KEY" >> $GITHUB_ENV
      
      - name: Update Kubernetes secrets
        run: |
          kubectl create secret generic api-keys \
            --from-literal=api-key=$NEW_API_KEY \
            -n production \
            --dry-run=client -o yaml | kubectl apply -f -
      
      - name: Restart pods to pick up new secrets
        run: |
          kubectl rollout restart deployment -n production
      
      - name: Notify team
        run: |
          # 发送通知
          curl -X POST "${{ secrets.SLACK_WEBHOOK }}" \
            -H 'Content-Type: application/json' \
            -d '{"text": "API keys have been rotated successfully"}'
```

---

## 10. 微服务监控与告警

### 10.1 Prometheus + Grafana

```yaml
# infrastructure/monitoring/prometheus-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-config
  namespace: monitoring
data:
  prometheus.yml: |
    global:
      scrape_interval: 15s
      evaluation_interval: 15s

    rule_files:
      - /etc/prometheus/rules/*.yml

    scrape_configs:
      - job_name: 'kubernetes-pods'
        kubernetes_sd_configs:
          - role: pod
        relabel_configs:
          - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
            action: keep
            regex: true
          - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
            action: replace
            target_label: __metrics_path__
            regex: (.+)
          - source_labels: [__address__, __meta_kubernetes_pod_annotation_prometheus_io_port]
            action: replace
            regex: ([^:]+)(?::\d+)?;(\d+)
            replacement: $1:$2
            target_label: __address__
```

### 10.2 告警规则

```yaml
# infrastructure/monitoring/alert-rules.yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: microservice-alerts
  namespace: monitoring
spec:
  groups:
    - name: microservice.rules
      rules:
        - alert: HighErrorRate
          expr: |
            sum(rate(http_server_requests_seconds_count{status=~"5.."}[5m])) by (service)
            /
            sum(rate(http_server_requests_seconds_count[5m])) by (service)
            > 0.05
          for: 5m
          labels:
            severity: critical
          annotations:
            summary: "High error rate on {{ $labels.service }}"
            description: "Service {{ $labels.service }} has error rate > 5%"

        - alert: HighLatency
          expr: |
            histogram_quantile(0.95, sum(rate(http_server_requests_seconds_bucket[5m])) by (le, service))
            > 2
          for: 5m
          labels:
            severity: warning
          annotations:
            summary: "High latency on {{ $labels.service }}"
            description: "P95 latency for {{ $labels.service }} is > 2s"

        - alert: PodCrashLooping
          expr: |
            rate(kube_pod_container_status_restarts_total{namespace="production"}[15m]) > 0
          for: 5m
          labels:
            severity: critical
          annotations:
            summary: "Pod {{ $labels.pod }} is crash looping"
```

### 10.3 Grafana Dashboard 配置

```json
{
  "dashboard": {
    "title": "Microservices Overview",
    "panels": [
      {
        "title": "Request Rate",
        "type": "graph",
        "targets": [
          {
            "expr": "sum(rate(http_server_requests_seconds_count[5m])) by (service)",
            "legendFormat": "{{ service }}"
          }
        ]
      },
      {
        "title": "Error Rate",
        "type": "gauge",
        "targets": [
          {
            "expr": "sum(rate(http_server_requests_seconds_count{status=~'5..'}[5m])) / sum(rate(http_server_requests_seconds_count[5m]))",
            "thresholds": {
              "steps": [
                {"color": "green", "value": 0},
                {"color": "yellow", "value": 0.01},
                {"color": "red", "value": 0.05}
              ]
            }
          }
        ]
      },
      {
        "title": "P95 Latency",
        "type": "graph",
        "targets": [
          {
            "expr": "histogram_quantile(0.95, sum(rate(http_server_requests_seconds_bucket[5m])) by (le, service))",
            "legendFormat": "{{ service }}"
          }
        ]
      }
    ]
  }
}
```

---

## 11. 蓝绿部署与金丝雀发布

### 11.1 蓝绿部署

```yaml
# .github/workflows/blue-green-deploy.yml
name: Blue-Green Deploy

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/checkout@v4
      
      - name: Determine current active environment
        id: current
        run: |
          CURRENT=$(kubectl get svc user-service -n production -o jsonpath='{.spec.selector.version}')
          echo "current=$CURRENT" >> $GITHUB_ENV
          if [ "$CURRENT" = "blue" ]; then
            echo "target=green" >> $GITHUB_ENV
          else
            echo "target=blue" >> $GITHUB_ENV
          fi
      
      - name: Deploy to target environment
        run: |
          kubectl set image deployment/user-service-$TARGET \
            user-service=${{ env.DOCKER_REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }} \
            -n production
          
          kubectl rollout status deployment/user-service-$TARGET -n production --timeout=300s
      
      - name: Run smoke tests
        run: |
          # 针对目标环境运行测试
          TARGET_PORT=$(kubectl get svc user-service-$TARGET -n production -o jsonpath='{.spec.ports[0].port}')
          curl -f http://user-service-$TARGET:$TARGET_PORT/actuator/health
      
      - name: Switch traffic
        run: |
          # 更新 Service selector 指向目标环境
          kubectl patch svc user-service -n production \
            -p '{"spec":{"selector":{"version":"'$TARGET'"}}}'
      
      - name: Verify deployment
        run: |
          sleep 30
          curl -f https://api.example.com/api/v1/users/health
      
      - name: Rollback on failure
        if: failure()
        run: |
          kubectl patch svc user-service -n production \
            -p '{"spec":{"selector":{"version":"'$CURRENT'"}}}'
```

### 11.2 金丝雀发布

```yaml
# infrastructure/kubernetes/canary/deployment-canary.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: user-service-canary
  namespace: production
spec:
  replicas: 1  # 少量副本用于金丝雀测试
  selector:
    matchLabels:
      app: user-service
      version: canary
  template:
    metadata:
      labels:
        app: user-service
        version: canary
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "8080"
    spec:
      containers:
        - name: user-service
          image: ghcr.io/myorg/user-service:canary
          ports:
            - containerPort: 8080
          readinessProbe:
            httpGet:
              path: /actuator/health
              port: 8080
            initialDelaySeconds: 30
            periodSeconds: 10
          livenessProbe:
            httpGet:
              path: /actuator/health
              port: 8080
            initialDelaySeconds: 60
            periodSeconds: 30
```

使用 Istio 进行精细流量控制：

```yaml
# infrastructure/istio/canary-virtual-service.yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: user-service-canary
  namespace: production
spec:
  hosts:
    - user-service
  http:
    - match:
        - headers:
            x-canary:
              exact: "true"
      route:
        - destination:
            host: user-service
            subset: canary
    - route:
        - destination:
            host: user-service
            subset: stable
          weight: 95
        - destination:
            host: user-service
            subset: canary
          weight: 5
      retries:
        attempts: 3
        perTryTimeout: 2s
---
# 自动金丝雀分析
apiVersion: flagger.app/v1beta1
kind: Canary
metadata:
  name: user-service
  namespace: production
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: user-service
  progressDeadlineSeconds: 600
  service:
    port: 8080
  analysis:
    interval: 1m
    threshold: 5
    maxWeight: 50
    stepWeight: 10
    metrics:
      - name: request-success-rate
        thresholdRange:
          min: 99
        interval: 1m
      - name: request-duration
        thresholdRange:
          max: 500
        interval: 1m
    webhooks: []
```

---

## 12. 微服务拆分与重构策略

### 12.1 绞杀者模式（Strangler Fig Pattern）

```yaml
# .github/workflows/strangler-pattern-migration.yml
name: Strangler Pattern Migration

on:
  push:
    branches: [ main ]

jobs:
  # 阶段1：部署新服务
  deploy-new-service:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Deploy new microservice
        run: |
          kubectl apply -f infrastructure/kubernetes/new-service/
          
  # 阶段2：配置 API Gateway 路由迁移
  configure-routing:
    needs: deploy-new-service
    runs-on: ubuntu-latest
    steps:
      - name: Update routing rules
        run: |
          # 逐步将流量从单体迁移到新服务
          # 开始：10% 流量到新服务
          kubectl apply -f infrastructure/istio/migration-phase-1.yaml
          
  # 阶段3：监控和验证
  monitor-migration:
    needs: configure-routing
    runs-on: ubuntu-latest
    steps:
      - name: Monitor error rates
        run: |
          # 查询 Prometheus 检查新服务错误率
          ERROR_RATE=$(curl -s "http://prometheus:9090/api/v1/query" \
            --data-urlencode "query=sum(rate(http_server_requests_seconds_count{service='new-service',status=~'5..'}[5m]))/sum(rate(http_server_requests_seconds_count{service='new-service'}[5m]))" \
            | jq '.data.result[0].value[1]' -r)
          
          if (( $(echo "$ERROR_RATE > 0.01" | bc -l) )); then
            echo "Error rate too high, triggering rollback"
            kubectl apply -f infrastructure/istio/rollback.yaml
            exit 1
          fi
```

### 12.2 数据库拆分策略

```yaml
# infrastructure/kubernetes/db-migration-job.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: db-split-migration
  namespace: production
spec:
  template:
    spec:
      containers:
        - name: migration
          image: myorg/db-migration:latest
          env:
            - name: SOURCE_DB_URL
              valueFrom:
                secretKeyRef:
                  name: db-secrets
                  key: source-db-url
            - name: TARGET_DB_URL
              valueFrom:
                secretKeyRef:
                  name: db-secrets
                  key: target-db-url
          command:
            - /bin/sh
            - -c
            - |
              # 1. 复制数据到新数据库
              pg_dump $SOURCE_DB_URL -t users | psql $TARGET_DB_URL
              
              # 2. 验证数据一致性
              SOURCE_COUNT=$(psql $SOURCE_DB_URL -t -c "SELECT COUNT(*) FROM users")
              TARGET_COUNT=$(psql $TARGET_DB_URL -t -c "SELECT COUNT(*) FROM users")
              
              if [ "$SOURCE_COUNT" != "$TARGET_COUNT" ]; then
                echo "Data count mismatch!"
                exit 1
              fi
              
              # 3. 更新应用配置指向新数据库
              kubectl set env deployment/user-service \
                DATABASE_URL=$TARGET_DB_URL \
                -n production
      restartPolicy: Never
  backoffLimit: 3
```

---

## 13. 国内微服务框架

### 13.1 Spring Cloud Alibaba

Spring Cloud Alibaba 是国内最流行的微服务框架之一，提供了 Nacos、Sentinel、Seata 等组件的集成。

```yaml
# .github/workflows/spring-cloud-alibaba.yml
name: Spring Cloud Alibaba CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    services:
      nacos:
        image: nacos/nacos-server:v2.2.3
        ports:
          - 8848:8848
        env:
          MODE: standalone
      
      sentinel:
        image: bladex/sentinel-dashboard:1.8.7
        ports:
          - 8858:8858
      
      seata:
        image: seataio/seata-server:1.7.1
        ports:
          - 8091:8091

    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'
      
      - name: Build with Maven
        run: mvn clean package -DskipTests
      
      - name: Run tests
        run: mvn test
        env:
          NACOS_SERVER_ADDR: localhost:8848
          SENTINEL_DASHBOARD: localhost:8858
```

Nacos 配置管理示例：

```yaml
# application.yml
spring:
  application:
    name: user-service
  cloud:
    nacos:
      discovery:
        server-addr: ${NACOS_SERVER_ADDR:nacos.infrastructure.svc.cluster.local:8848}
        namespace: production
        group: DEFAULT_GROUP
      config:
        server-addr: ${NACOS_SERVER_ADDR:nacos.infrastructure.svc.cluster.local:8848}
        namespace: production
        group: DEFAULT_GROUP
        file-extension: yaml
        shared-configs:
          - data-id: common.yaml
            group: DEFAULT_GROUP
            refresh: true
```

Sentinel 限流降级配置：

```yaml
# Sentinel 规则配置
spring:
  cloud:
    sentinel:
      transport:
        dashboard: ${SENTINEL_DASHBOARD:localhost:8858}
      datasource:
        flow:
          nacos:
            server-addr: ${NACOS_SERVER_ADDR}
            dataId: ${spring.application.name}-flow-rules
            groupId: SENTINEL_GROUP
            rule-type: flow
        degrade:
          nacos:
            server-addr: ${NACOS_SERVER_ADDR}
            dataId: ${spring.application.name}-degrade-rules
            groupId: SENTINEL_GROUP
            rule-type: degrade
```

### 13.2 Dubbo 微服务

Apache Dubbo 是另一个广泛使用的 RPC 框架：

```yaml
# .github/workflows/dubbo-service.yml
name: Dubbo Service CI

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'
      
      - name: Build and test
        run: |
          mvn clean package
      
      - name: Deploy to Kubernetes
        run: |
          kubectl apply -f infrastructure/kubernetes/dubbo/
```

Dubbo 服务配置：

```yaml
# application-dubbo.yml
dubbo:
  application:
    name: user-service
    qos-enable: false
  registry:
    address: nacos://nacos.infrastructure.svc.cluster.local:8848
    parameters:
      namespace: production
  protocol:
    name: dubbo
    port: 20880
  provider:
    timeout: 5000
    retries: 2
  consumer:
    check: false
    timeout: 5000
  scan:
    base-packages: com.mycompany.userservice
```

### 13.3 国内镜像加速配置

```yaml
# .github/workflows/china-mirror-ci.yml
name: China Mirror CI

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'
          cache: maven
      
      - name: Configure Maven mirror
        run: |
          mkdir -p ~/.m2
          cat > ~/.m2/settings.xml << 'EOF'
          <settings>
            <mirrors>
              <mirror>
                <id>aliyun</id>
                <mirrorOf>central</mirrorOf>
                <name>Aliyun Maven Mirror</name>
                <url>https://maven.aliyun.com/repository/public</url>
              </mirror>
            </mirrors>
          </settings>
          EOF
      
      - name: Build with mirror
        run: mvn clean package
```

### 13.4 国内微服务网关选择

国内常用的微服务网关包括：

| 网关 | 特点 | 适用场景 |
|------|------|----------|
| **Spring Cloud Gateway** | 基于 WebFlux，响应式 | Spring 生态项目 |
| **Apache Shenyu** | 高性能，插件化 | 多协议支持场景 |
| **APISIX** | 动态路由，高性能 | 大规模流量场景 |
| **Kong** | 插件丰富，社区活跃 | 企业级 API 管理 |
| **Soul** | 轻量级，易于扩展 | 中小型项目 |

---

## 总结

本指南涵盖了微服务架构在 GitHub 上进行管理的完整生命周期：

### 架构决策

| 方面 | Monorepo | Multirepo |
|------|----------|-----------|
| 代码共享 | 容易 | 需要包管理 |
| CI/CD 配置 | 需要路径过滤 | 独立配置 |
| 团队协作 | 统一视图 | 独立管理 |
| 版本管理 | 统一版本 | 独立版本 |

### 技术栈选择

```
┌─────────────────────────────────────────────────────────────┐
│                     API Gateway                              │
│        (Spring Cloud Gateway / Kong / APISIX)               │
├─────────────────────────────────────────────────────────────┤
│                    Service Mesh                              │
│              (Istio / Linkerd)                               │
├─────────────────────────────────────────────────────────────┤
│                    服务层                                      │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐      │
│  │User Svc  │ │Order Svc │ │Pay Svc   │ │Notify Svc│      │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘      │
├─────────────────────────────────────────────────────────────┤
│                    基础设施层                                  │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐      │
│  │  Nacos   │ │ Sentinel │ │  Seata   │ │  SkyWalking│    │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘      │
└─────────────────────────────────────────────────────────────┘
```

### 关键实践

1. **仓库组织**：根据团队规模和项目复杂度选择 Monorepo 或 Multirepo
2. **CI/CD**：实现自动化的构建、测试、部署流水线
3. **可观测性**：分布式追踪 + 日志聚合 + 监控告警三位一体
4. **安全**：OAuth2/JWT 认证 + mTLS 通信加密
5. **发布策略**：蓝绿部署保证零停机，金丝雀发布降低风险
6. **国内适配**：选择合适的镜像源和框架

微服务架构的成功不仅依赖于技术选型，更依赖于合理的仓库管理和自动化的 CI/CD 流程。通过 GitHub Actions 结合 Kubernetes 和服务网格，可以构建出高可用、可观测、易维护的微服务系统。

---

*最后更新：2026 年 9 月*
