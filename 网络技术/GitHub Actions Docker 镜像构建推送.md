---
title: GitHub Actions Docker 镜像构建推送
created: 2026-03-14
source: Cherry Studio
tags: 
---
# GitHub Actions Docker 镜像构建推送

## 🧑‍💻 User

Github docker 构建 推送

---
## 🤖 Assistant

在 GitHub 上构建 Docker 镜像并推送到 Docker Hub 或 GitHub Container Registry (GHCR) 通常使用 **GitHub Actions** 来实现自动化 CI/CD。

下面我将分步骤介绍两种最常用的方式：
1. **推送到 Docker Hub** (最常用的公共仓库)
2. **推送到 GitHub Container Registry (GHCR)** (GitHub 自带的仓库，推荐)

---

### 准备工作

无论推送到哪里，你的项目仓库中必须包含一个 `Dockerfile`。

**示例 Dockerfile:**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY . .
RUN npm install
CMD ["node", "index.js"]
```

---

### 方案一：推送到 Docker Hub

#### 1. 获取 Docker Hub Token
为了安全，不要直接使用密码。
1. 登录 [Docker Hub](https://hub.docker.com/)。
2. 点击右上角头像 -> **Account Settings** -> **Security** -> **New Access Token**。
3. 输入描述（如 `github-actions`），点击 Generate。
4. **复制**生成的 Token（只显示一次）。

#### 2. 在 GitHub 仓库中配置 Secrets
1. 打开你的 GitHub 仓库。
2. 点击 **Settings** -> **Secrets and variables** -> **Actions** -> **New repository secret**。
3. 添加以下两个 Secret：
 * `DOCKER_USERNAME`: 你的 Docker Hub 用户名。
 * `DOCKER_TOKEN`: 刚才复制的 Access Token。

#### 3. 创建 GitHub Actions Workflow
在项目根目录创建文件 `.github/workflows/docker-publish.yml`：

```yaml
name: Build and Push Docker Image

on:
  push:
    branches:
      - "main" # 当推送到 main 分支时触发
    tags:
      - "v*.*.*" # 当推送标签时触发

env:
  # 设置 Docker 镜像名称
  IMAGE_NAME: your-dockerhub-username/your-repo-name

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    permissions:
      contents: read

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Log in to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_TOKEN }}

      - name: Extract metadata (tags, labels) for Docker
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.IMAGE_NAME }}
          # 自动生成 tag 标签，例如：latest, git commit sha, branch name
          tags: |
            type=ref,event=branch
            type=ref,event=pr
            type=semver,pattern={{version}}
            type=semver,pattern={{major}}.{{minor}}
            type=sha

      - name: Build and push Docker image
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

---

### 方案二：推送到 GitHub Container Registry (GHCR)

**推荐理由**：与 GitHub 原生集成，支持 GitHub Token 免密登录，私有仓库免费额度大。

#### 1. 配置权限 (最重要的一步)
如果你使用的是 Public 仓库，通常不需要这一步。如果是 Private 仓库，必须给 Actions 工作流赋予写权限：
1. 仓库 **Settings** -> **Actions** -> **General**。
2. 滚动到底部的 **Workflow permissions**。
3. 选择 **Read and write permissions**。
4. 勾选 **Allow GitHub Actions to create and approve pull requests** (可选)。
5. 点击 Save。

#### 2. 创建 GitHub Actions Workflow
你**不需要**手动配置 Secrets Token，GitHub 会自动提供 `GITHUB_TOKEN`。

修改 `.github/workflows/docker-publish.yml` 如下：

```yaml
name: Build and Push to GHCR

on:
  push:
    branches: ["main"]

env:
  # 格式必须为：ghcr.io/<你的GitHub用户名>/<仓库名>
  IMAGE_NAME: ghcr.io/${{ github.repository_owner }}/my-app

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    # 必须授予 contents: write 权限才能推送到 GHCR
    permissions:
      contents: read
      packages: write

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      # 登录到 GHCR
      - name: Log in to the Container registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }} # 自动触发此 Action 的用户
          password: ${{ secrets.GITHUB_TOKEN }} # GitHub 自动提供的 Token

      - name: Extract metadata (tags, labels)
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.IMAGE_NAME }}
          tags: |
            type=ref,event=branch
            type=sha

      - name: Build and push Docker image
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
```

---

### 关键配置解释

1. **`docker/setup-buildx-action`**: Docker 的构建增强工具，支持缓存和多平台构建。
2. **`docker/login-action`**: 标准的登录步骤。
 * Docker Hub 默认不需要 `registry` 参数。
 * GHCR 必须指定 `registry: ghcr.io`。
3. **`docker/metadata-action`**: 这是一个非常棒的工具，它自动帮你生成 Tag 标签。
 * 比如你推送到 `main` 分支，它会自动打上 `main` 标签。
 * 如果你打 Git Tag 为 `v1.0.0`，它会自动识别并生成 `1.0.0` 和 `latest` 等标签。
4. **`docker/build-push-action`**: 实际的构建和推送步骤。
 * `push: true`: 表示构建完立即推送。
 * `cache-from` / `cache-to`: 启用 GitHub Actions 缓存，大幅加快构建速度（第二次构建通常只需几秒）。

### 如何拉取镜像？

构建成功后：

* **Docker Hub:**
 ```bash
 docker pull your-dockerhub-username/your-repo-name:latest
 ```
* **GHCR:**
 ```bash
 docker pull ghcr.io/your-username/my-app:latest
 ```
 *(注意：如果是私有镜像，拉取前需要先 `docker login ghcr.io`)*
