# 供应链安全

## 为什么供应链安全很重要？

现代软件大量依赖第三方依赖，供应链攻击已成为主要威胁之一。

## SLSA (Supply chain Levels for Software Artifacts)

### SLSA 级别

| 级别 | 描述 | 要求 |
|------|------|------|
| Level 1 | 构建过程有文档记录 | 构建脚本、版本控制 |
| Level 2 | 使用托管构建服务 | 防篡改构建、审计日志 |
| Level 3 | 构建平台受保护 | 环境隔离、不可伪造的元数据 |
| Level 4 | 双人审查 | 最高级别安全 |

### SLSA 工作流

```yaml
# .github/workflows/slsa-build.yml
name: SLSA Build

on:
  push:
    tags: ['v*']

jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      hashes: ${{ steps.hash.outputs.hashes }}
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Build
      run: |
        npm ci
        npm run build
    
    - name: Generate artifact hashes
      id: hash
      run: |
        find dist -type f -exec sha256sum {} \; | base64 -w0 > hashes.txt
        echo "hashes=$(cat hashes.txt)" >> $GITHUB_OUTPUT
    
    - name: Upload artifacts
      uses: actions/upload-artifact@v4
      with:
        name: release-artifacts
        path: dist/

  provenance:
    needs: build
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      actions: read
    
    steps:
    - name: Generate SLSA provenance
      uses: slsa-framework/slsa-github-generator@v1.9.0
      with:
        base64-subjects: "${{ needs.build.outputs.hashes }}"
```

## SBOM (Software Bill of Materials)

### 生成 SBOM

```yaml
# .github/workflows/sbom.yml
name: Generate SBOM

on:
  push:
    branches: [main]

jobs:
  sbom:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    
    - name: Generate SBOM (SPDX)
      uses: anchore/sbom-action@v0
      with:
        artifact-name: sbom.spdx.json
        output-file: sbom.spdx.json
    
    - name: Generate SBOM (CycloneDX)
      uses: CycloneDX/gh-github-dx-action@master
      with:
        path: .
        output: sbom.cdx.json
    
    - name: Upload SBOM
      uses: actions/upload-artifact@v4
      with:
        name: sbom
        path: |
          sbom.spdx.json
          sbom.cdx.json
```

### 使用 Syft 生成

```bash
# 安装 Syft
curl -sSfL https://raw.githubusercontent.com/anchore/syft/main/install.sh | sh -s -- -b /usr/local/bin

# 生成 SPDX 格式
syft dir:. -o spdx-json > sbom.spdx.json

# 生成 CycloneDX 格式
syft dir:. -o cyclonedx-json > sbom.cdx.json
```

## Artifact Attestation

### 签名构建产物

```yaml
# .github/workflows/sign-artifacts.yml
name: Sign Artifacts

on:
  release:

permissions:
  id-token: write
  contents: read

jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      hashes: ${{ steps.hash.outputs.hashes }}
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Build
      run: npm run build
    
    - name: Generate hashes
      id: hash
      run: |
        sha256sum dist/* > hashes.txt
        echo "hashes<<EOF" >> $GITHUB_OUTPUT
        cat hashes.txt | base64 -w0 >> $GITHUB_OUTPUT
        echo "EOF" >> $GITHUB_OUTPUT
    
    - name: Upload artifacts
      uses: actions/upload-artifact@v4
      with:
        name: release
        path: dist/

  attest:
    needs: build
    runs-on: ubuntu-latest
    
    steps:
    - name: Attest
      uses: actions/attest-build-provenance@v1
      with:
        subject-name: my-app
        subject-digest: sha256:${{ needs.build.outputs.hashes }}
        push-to-registry: true
```

### 验证签名

```bash
# 安装 ghattest
go install github.com/sigstore/ghattest@latest

# 验证 artifact
ghattest verify \
  --artifact dist/my-app \
  --subject-name my-app \
  --source-repo owner/repo
```

## Dependabot 安全更新

### 配置

```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "daily"
    open-pull-requests-limit: 10
    reviewers:
      - "security-team"
    labels:
      - "security"
      - "dependencies"
    groups:
      security-updates:
        patterns:
          - "*"
        update-types:
          - "patch"
```

### 自动合并安全更新

```yaml
# .github/workflows/auto-merge.yml
name: Auto Merge Dependabot

on:
  pull_request:

permissions:
  contents: write
  pull-requests: write

jobs:
  auto-merge:
    if: github.actor == 'dependabot[bot]'
    runs-on: ubuntu-latest
    
    steps:
    - name: Fetch Dependabot metadata
      id: metadata
      uses: dependabot/fetch-metadata@v1
      with:
        github-token: "${{ secrets.GITHUB_TOKEN }}"
    
    - name: Auto-merge minor and patch updates
      if: steps.metadata.outputs.update-type != 'version-update:semver-major'
      run: gh pr merge --auto --squash "$PR_URL"
      env:
        PR_URL: ${{github.event.pull_request.html_url}}
        GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

## 容器安全

### 扫描容器镜像

```yaml
# .github/workflows/container-scan.yml
name: Container Security

on:
  push:
    branches: [main]

jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    
    - name: Build image
      run: docker build -t my-app:${{ github.sha }} .
    
    - name: Scan with Trivy
      uses: aquasecurity/trivy-action@master
      with:
        image-ref: my-app:${{ github.sha }}
        format: sarif
        output: trivy-results.sarif
    
    - name: Upload scan results
      uses: github/codeql-action/upload-sarif@v3
      with:
        sarif_file: trivy-results.sarif
```

## 最佳实践

1. **生成 SBOM**：跟踪所有依赖
2. **签名产物**：确保构建完整性
3. **使用 SLSA**：提高供应链安全级别
4. **自动更新依赖**：使用 Dependabot
5. **扫描容器**：检测镜像漏洞
6. **审计日志**：跟踪所有更改

## 相关资源

- [SLSA 文档](https://slsa.dev/)
- [SBOM 文档](https://spdx.org/)
- [Sigstore 文档](https://www.sigstore.dev/)

---

**上一篇：[GitHub Advanced Security](W10-advanced-security.md) | 下一篇：[GitHub App 开发](W14-github-app-dev.md)**
