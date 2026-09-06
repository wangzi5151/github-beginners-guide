# 数据库 CI/CD 工作流

## 数据库迁移管理

### 使用 Prisma

```yaml
# .github/workflows/database.yml
name: Database Migration

on:
  push:
    branches: [main]

jobs:
  migrate:
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: test
        ports:
          - 5432:5432
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '20'
    
    - run: npm ci
    
    - name: Run migrations
      run: npx prisma migrate deploy
      env:
        DATABASE_URL: postgresql://postgres:postgres@localhost:5432/test
    
    - name: Generate Prisma Client
      run: npx prisma generate
      env:
        DATABASE_URL: postgresql://postgres:postgres@localhost:5432/test
    
    - name: Run tests
      run: npm test
      env:
        DATABASE_URL: postgresql://postgres:postgres@localhost:5432/test
```

### 使用 Flyway

```yaml
# .github/workflows/flyway.yml
name: Flyway Migration

on:
  push:
    branches: [main]
    paths:
      - 'db/migrations/**'

jobs:
  migrate:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Flyway migrate
      uses: flyway/flyway-actions@v4
      with:
        url: jdbc:postgresql://your-db-host:5432/your-db
        user: ${{ secrets.DB_USER }}
        password: ${{ secrets.DB_PASSWORD }}
        locations: filesystem:db/migrations
```

### 使用 Liquibase

```yaml
# .github/workflows/liquibase.yml
name: Liquibase Migration

on:
  push:
    branches: [main]

jobs:
  migrate:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Liquibase update
      uses: liquibase/liquibase-github-action@v4
      with:
        command: update
        changelog-file: db/changelog.yaml
        url: jdbc:postgresql://your-db-host:5432/your-db
        username: ${{ secrets.DB_USER }}
        password: ${{ secrets.DB_PASSWORD }}
```

## 数据库测试

### 集成测试

```yaml
# .github/workflows/db-test.yml
name: Database Integration Test

on:
  pull_request:

jobs:
  test:
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: test
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '20'
    
    - run: npm ci
    
    - name: Run migrations
      run: npx prisma migrate deploy
      env:
        DATABASE_URL: postgresql://postgres:postgres@localhost:5432/test
    
    - name: Seed database
      run: npx prisma db seed
      env:
        DATABASE_URL: postgresql://postgres:postgres@localhost:5432/test
    
    - name: Run integration tests
      run: npm run test:integration
      env:
        DATABASE_URL: postgresql://postgres:postgres@localhost:5432/test
```

## 数据库备份和恢复

### 自动备份

```yaml
# .github/workflows/backup.yml
name: Database Backup

on:
  schedule:
    - cron: '0 2 * * *'  # 每天凌晨 2 点

jobs:
  backup:
    runs-on: ubuntu-latest
    
    steps:
    - name: Backup database
      run: |
        pg_dump -h ${{ secrets.DB_HOST }} \
                -U ${{ secrets.DB_USER }} \
                -d ${{ secrets.DB_NAME }} \
                -F c \
                -f backup-$(date +%Y%m%d).dump
    
    - name: Upload backup
      uses: actions/upload-artifact@v4
      with:
        name: backup-${{ github.run_id }}
        path: backup-*.dump
        retention-days: 30
```

### 恢复备份

```bash
# 恢复数据库
pg_restore -h $DB_HOST \
           -U $DB_USER \
           -d $DB_NAME \
           -c backup-20240101.dump
```

## Schema 变更审查

### 使用 Prisma Review

```yaml
# .github/workflows/schema-review.yml
name: Schema Review

on:
  pull_request:
    paths:
      - 'prisma/schema.prisma'

jobs:
  review:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Check schema changes
      run: |
        # 检查是否有破坏性变更
        npx prisma format
        npx prisma validate
        
        # 生成迁移预览
        npx prisma migrate diff --from-schema-datamodel prisma/schema.prisma --to-schema-datamodel prisma/schema.prisma --exit-code
```

## 最佳实践

1. **使用版本控制**：所有数据库变更都应该有版本控制
2. **自动化测试**：在 CI 中运行数据库测试
3. **备份策略**：定期备份数据库
4. **审查 Schema 变更**：在 PR 中审查数据库变更
5. **使用迁移工具**：使用专业的数据库迁移工具

## 相关资源

- [Prisma 文档](https://www.prisma.io/docs)
- [Flyway 文档](https://flywaydb.org/documentation)
- [Liquibase 文档](https://www.liquibase.org/documentation)

---

**上一篇：[GitHub Code Search 高级搜索](W18-code-search.md) | 下一篇：[Feature Flags 功能开关](W20-feature-flags.md)**
