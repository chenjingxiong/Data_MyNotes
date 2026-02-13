# VitePress Docker 部署教程

使用官方推荐的 Node.js + Nginx 方式部署 VitePress。

---

## 快速开始

```bash
mkdir vitepress-docs && cd vitepress-docs
npm init -y

# 安装 VitePress
npm install -D vitepress

# 初始化
npx vitepress init
```

---

## Docker 部署方案

### 方案一：构建 + Nginx（推荐）

这是 VitePress 官方推荐的部署方式，使用官方 Node.js 和 Nginx 镜像。

**Dockerfile**
```dockerfile
# ========== 构建阶段 ==========
FROM node:20-alpine AS builder

WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run docs:build

# ========== 生产阶段 ==========
FROM nginx:alpine

# 复制构建产物
COPY --from=builder /app/docs/.vitepress/dist /usr/share/nginx/html

# 官方推荐的 Nginx 配置
RUN cat > /etc/nginx/conf.d/default.conf << 'EOF'
server {
    gzip on;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

    listen 80;
    server_name _;
    index index.html;

    location / {
        root /usr/share/nginx/html;
        try_files $uri $uri.html $uri/ =404;
        error_page 404 /404.html;
    }

    location ~* ^/assets/ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
}
EOF

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

**.dockerignore**
```
node_modules
.git
docs/.vitepress/dist
```

**使用命令**
```bash
# 构建
docker build -t vitepress-docs .

# 运行
docker run -d -p 8080:80 vitepress-docs
```

---

### 方案二：docker-compose（简化版）

**docker-compose.yml**
```yaml
services:
  docs:
    build: .
    ports:
      - "8080:80"
    restart: unless-stopped
```

```bash
docker-compose up -d
```

---

### 方案三：纯运行时（开发用）

使用 VitePress 内置的 preview 服务器：

```bash
# 直接运行
docker run -it --rm -v $(pwd):/app -w /app -p 4173:4173 node:20-alpine sh -c "npm install && npx vitepress preview docs --host 0.0.0.0"
```

---

## 配置要点

### config.ts 设置

```typescript
export default defineConfig({
  // Docker 部署使用根路径
  base: '/',

  // 输出目录（默认）
  outDir: '.vitepress/dist'
})
```

---

## 常见问题

| 问题 | 解决方法 |
|------|----------|
| 空白页 | 检查 `base: '/'` |
| 404 错误 | 确认 `try_files` 配置正确 |
| 样式丢失 | 检查构建产物路径 |

---

## 一键部署到服务器

```bash
git clone <your-repo>
cd <repo-name>
docker-compose up -d
```

---

## 同步 GitHub Markdown 文档

### 方案一：Git Submodule（推荐）

将你的笔记仓库作为子模块引入：

```bash
# 在 VitePress 项目中添加子模块
git submodule add https://github.com/your-username/your-notes.git docs/content

# 更新子模块
git submodule update --remote --merge

# 克隆时包含子模块
git clone --recurse-submodules <your-vitepress-repo>
```

**项目结构：**
```
vitepress-docs/
├── docs/
│   ├── content/          # 子模块：你的笔记
│   │   ├── guide.md
│   │   └── api.md
│   ├── .vitepress/
│   │   └── config.ts
│   └── index.md
```

**config.ts 配置：**
```typescript
export default defineConfig({
  title: '我的文档',
  // 从 content 目录读取
  srcDir: 'content'
})
```

### 方案二：GitHub Actions 自动同步

在构建时自动从另一个仓库拉取内容：

`.github/workflows/sync-and-build.yml`:
```yaml
name: Sync & Build

on:
  push:
    branches: [main]
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout main repo
        uses: actions/checkout@v4

      - name: Checkout notes repo
        uses: actions/checkout@v4
        with:
          repository: your-username/your-notes
          path: docs/content

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Build
        run: |
          npm ci
          npm run docs:build

      - name: Deploy
        run: |
          # 部署到你的服务器或上传 artifact
          echo "Deploy..."
```

### 方案三：同步脚本

创建 `scripts/sync.sh`:

```bash
#!/bin/bash
NOTES_REPO="https://github.com/your-username/your-notes.git"
CONTENT_DIR="docs/content"

# 拉取最新内容
if [ -d "$CONTENT_DIR/.git" ]; then
    cd "$CONTENT_DIR"
    git pull origin main
else
    git clone "$NOTES_REPO" "$CONTENT_DIR"
fi

echo "Sync completed!"
```

```bash
# 添加到 package.json
{
  "scripts": {
    "sync": "bash scripts/sync.sh",
    "docs:build": "npm run sync && vitepress build docs"
  }
}
```

### 方案四：Docker 构建时同步

修改 Dockerfile 支持动态拉取：

```dockerfile
FROM node:20-alpine AS builder

# 安装 git
RUN apk add --no-cache git

WORKDIR /app
COPY package*.json ./
RUN npm ci

# 复制主项目
COPY . .

# 拉取笔记仓库
RUN git clone https://github.com/your-username/your-notes.git docs/content

RUN npm run docs:build

FROM nginx:alpine
COPY --from=builder /app/docs/.vitepress/dist /usr/share/nginx/html
# ... nginx 配置
```

**使用环境变量（更灵活）：**

```dockerfile
ARG NOTES_REPO=https://github.com/your-username/your-notes.git
RUN git clone "${NOTES_REPO}" docs/content
```

```bash
# 构建时指定仓库
docker build --build-arg NOTES_REPO=https://github.com/other/repo.git -t docs .
```

---

## 参考资源

Sources:
- [VitePress 官方部署指南](https://vitepress.dev/zh/guide/deploy)
- [2026年Docker常用镜像及其推荐稳定版本](https://www.cnblogs.com/aiparallelworld/p/19589451)
