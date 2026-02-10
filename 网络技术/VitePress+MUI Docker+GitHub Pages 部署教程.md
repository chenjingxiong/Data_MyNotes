# VitePress + MUI Docker + GitHub Pages 自动部署教程

本教程介绍如何将 VitePress 与 Material-UI (MUI) 结合使用，通过 Docker 构建并自动部署到 GitHub Pages。

---

## 目录

1. [项目初始化](#1-项目初始化)
2. [配置 VitePress](#2-配置-vitepress)
3. [集成 MUI](#3-集成-mui)
4. [Docker 部署](#4-docker-部署)
5. [GitHub Actions 自动化](#5-github-actions-自动化)
6. [常见问题](#6-常见问题)

---

## 1. 项目初始化

### 1.1 创建项目目录

```bash
mkdir vitepress-mui-docs
cd vitepress-mui-docs
npm init -y
```

### 1.2 安装依赖

```bash
# 安装 VitePress
npm install -D vitepress

# 安装 MUI 和相关依赖
npm install @mui/material @emotion/react @emotion/styled

# 安装 Vue 相关依赖（VitePress 基于 Vue）
npm install vue
```

### 1.3 项目结构

```
vitepress-mui-docs/
├── .github/
│   └── workflows/
│       └── deploy.yml
├── docs/
│   ├── .vitepress/
│   │   ├── theme/
│   │   │   ├── index.ts
│   │   │   └── components/
│   │   │       └── MuiLayout.vue
│   │   └── config.ts
│   ├── guide/
│   │   └── index.md
│   └── index.md
├── Dockerfile
├── docker-compose.yml
├── .dockerignore
├── package.json
└── package-lock.json
```

---

## 2. 配置 VitePress

### 2.1 初始化 VitePress

```bash
npx vitepress init
```

按提示选择：
- **Where should the documentation be written?** → `./docs`
- **Title of your documentation?** → 你的文档标题
- **Description of your documentation?** → 描述
- **Theme?** → `Default Theme`
- **Use TypeScript for config and theme files?** → `Yes`
- **Add VitePress scripts to package.json?** → `Yes`

### 2.2 配置文件 `docs/.vitepress/config.ts`

```typescript
import { defineConfig } from 'vitepress'

export default defineConfig({
  title: '我的文档',
  description: 'VitePress + MUI 文档站点',

  // 构建配置
  base: '/vitepress-mui-docs/', // 替换为你的仓库名
  outDir: '.vitepress/dist',

  // 主题配置
  themeConfig: {
    nav: [
      { text: '指南', link: '/guide/' },
      { text: 'GitHub', link: 'https://github.com/your-username/vitepress-mui-docs' }
    ],

    sidebar: [
      {
        text: '开始',
        items: [
          { text: '介绍', link: '/guide/' },
          { text: '快速开始', link: '/guide/getting-started' }
        ]
      }
    ],

    socialLinks: [
      { icon: 'github', link: 'https://github.com/your-username/vitepress-mui-docs' }
    ]
  }
})
```

### 2.3 更新 `package.json`

```json
{
  "name": "vitepress-mui-docs",
  "version": "1.0.0",
  "scripts": {
    "docs:dev": "vitepress dev docs",
    "docs:build": "vitepress build docs",
    "docs:preview": "vitepress preview docs"
  },
  "devDependencies": {
    "vitepress": "^1.0.0"
  },
  "dependencies": {
    "@emotion/react": "^11.11.0",
    "@emotion/styled": "^11.11.0",
    "@mui/material": "^5.15.0",
    "vue": "^3.4.0"
  }
}
```

---

## 3. 集成 MUI

### 3.1 创建主题入口文件 `docs/.vitepress/theme/index.ts`

```typescript
import Theme from 'vitepress/theme'
import MuiLayout from './components/MuiLayout.vue'
import './mui.css'

export default {
  ...Theme,
  Layout: MuiLayout
}
```

### 3.2 创建 MUI 样式文件 `docs/.vitepress/theme/mui.css`

```css
/* 确保 MUI 样式优先级 */
.vitepress-theme {
  font-family: 'Roboto', sans-serif;
}

/* 自定义 MUI 组件样式适配 VitePress */
.Mui-container {
  /* 你的自定义样式 */
}
```

### 3.3 创建 MUI 布局组件 `docs/.vitepress/theme/components/MuiLayout.vue`

```vue
<script setup lang="ts">
import { useData } from 'vitepress'
import DefaultTheme from 'vitepress/theme'
import { onMounted, watch } from 'vue'

const { isDark } = useData()

// 监听主题变化，同步到 MUI
watch(isDark, (newVal) => {
  document.body.setAttribute('data-mui-color-scheme', newVal ? 'dark' : 'light')
})

onMounted(() => {
  // 初始化 MUI 主题
  document.body.setAttribute('data-mui-color-scheme', isDark.value ? 'dark' : 'light')
})
</script>

<template>
  <DefaultTheme.Layout />
</template>

<style>
@import '@mui/material/styles.css';
</style>
```

### 3.4 在 Markdown 中使用 MUI 组件

创建 `.vitepress/theme/components/MuiDemo.vue`：

```vue
<script setup lang="ts">
import { Button, Card, CardContent, Typography } from '@mui/material'
</script>

<template>
  <Card class="mui-demo-card">
    <CardContent>
      <Typography variant="h5" component="div">
        MUI 组件演示
      </Typography>
      <Typography variant="body2" color="text.secondary">
        这是一个集成在 VitePress 中的 Material-UI 组件
      </Typography>
      <Button variant="contained" color="primary">
        点击我
      </Button>
    </CardContent>
  </Card>
</template>

<style scoped>
.mui-demo-card {
  margin: 20px 0;
  max-width: 400px;
}
</style>
```

### 3.5 注册全局组件

在 `.vitepress/theme/index.ts` 中注册：

```typescript
import Theme from 'vitepress/theme'
import MuiLayout from './components/MuiLayout.vue'
import MuiDemo from './components/MuiDemo.vue'
import './mui.css'

export default {
  ...Theme,
  Layout: MuiLayout,
  enhanceApp({ app }) {
    app.component('MuiDemo', MuiDemo)
  }
}
```

现在你可以在任何 Markdown 文件中使用：

```markdown
# 欢迎使用

<MuiDemo />
```

---

## 4. Docker 部署

### 4.1 创建 `Dockerfile`

多阶段构建，优化镜像大小：

```dockerfile
# ===========================
# 第一阶段：构建阶段
# ===========================
FROM node:20-alpine AS builder

# 设置工作目录
WORKDIR /app

# 复制 package 文件
COPY package*.json ./

# 安装依赖
RUN npm ci

# 复制源代码
COPY . .

# 构建文档
RUN npm run docs:build

# ===========================
# 第二阶段：生产阶段
# ===========================
FROM nginx:alpine

# 复制构建产物到 Nginx
COPY --from=builder /app/docs/.vitepress/dist /usr/share/nginx/html

# 复制自定义 Nginx 配置
COPY nginx.conf /etc/nginx/conf.d/default.conf

# 暴露端口
EXPOSE 80

# 启动 Nginx
CMD ["nginx", "-g", "daemon off;"]
```

### 4.2 创建 `nginx.conf`

自定义 Nginx 配置以支持 SPA 路由：

```nginx
server {
    listen 80;
    server_name localhost;
    root /usr/share/nginx/html;
    index index.html;

    # 启用 gzip 压缩
    gzip on;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;
    gzip_min_length 1000;

    # 处理 SPA 路由
    location / {
        try_files $uri $uri/ /index.html;
    }

    # 缓存静态资源
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2|ttf|eot)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }

    # 禁止访问隐藏文件
    location ~ /\. {
        deny all;
    }
}
```

### 4.3 创建 `.dockerignore`

```dockerignore
node_modules
.git
.gitignore
.vscode
.idea
*.log
.DS_Store
docs/.vitepress/dist
```

### 4.4 创建 `docker-compose.yml`

用于本地开发和测试：

```yaml
version: '3.8'

services:
  # 生产环境服务
  docs-prod:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "8080:80"
    restart: unless-stopped
    container_name: vitepress-mui-prod

  # 开发环境服务（热重载）
  docs-dev:
    image: node:20-alpine
    working_dir: /app
    volumes:
      - .:/app
      - node_modules:/app/node_modules
    ports:
      - "5173:5173"
    command: sh -c "npm install && npm run docs:dev -- --host 0.0.0.0"
    restart: unless-stopped
    container_name: vitepress-mui-dev
    profiles:
      - dev

volumes:
  node_modules:
```

### 4.5 Docker 使用命令

```bash
# ====================================
# 本地开发（使用 Docker）
# ====================================
# 启动开发环境（支持热重载）
docker-compose --profile dev up docs-dev

# 后台运行
docker-compose --profile dev up -d docs-dev

# 查看日志
docker-compose logs -f docs-dev

# ====================================
# 构建和运行生产环境
# ====================================
# 构建镜像
docker build -t vitepress-mui-docs .

# 运行容器
docker run -d -p 8080:80 vitepress-mui-docs

# 或使用 docker-compose
docker-compose up docs-prod

# ====================================
# 其他常用命令
# ====================================
# 停止所有服务
docker-compose down

# 重新构建并启动
docker-compose up -d --build docs-prod

# 清理容器和镜像
docker-compose down -v --rmi all

# 进入容器调试
docker exec -it vitepress-mui-prod sh
```

### 4.6 部署到 Docker 容器平台

#### 部署到 Vercel / Railway / Render

这些平台支持直接从 Dockerfile 部署：

```bash
# 推送代码到 GitHub
git add .
git commit -m "Add Docker configuration"
git push

# 在平台选择 "Deploy from GitHub" 并选择你的仓库
# 平台会自动检测 Dockerfile 并构建
```

#### 部署到自己的服务器

```bash
# 在服务器上拉取代码
git clone https://github.com/your-username/vitepress-mui-docs.git
cd vitepress-mui-docs

# 构建并运行
docker-compose up -d docs-prod

# 配置 Nginx 反向代理（可选）
sudo nano /etc/nginx/conf.d/docs.conf
```

Nginx 反向代理配置：

```nginx
server {
    listen 80;
    server_name docs.example.com;

    location / {
        proxy_pass http://localhost:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

---

## 5. GitHub Actions 自动化

### 5.1 方式一：直接构建部署（不使用 Docker）

`.github/workflows/deploy.yml`：

```yaml
name: Deploy VitePress to GitHub Pages

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: pages
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Build VitePress
        run: npm run docs:build

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: docs/.vitepress/dist

  deploy:
    needs: build
    runs-on: ubuntu-latest

    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}

    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

### 5.2 方式二：使用 Docker 构建 + GitHub Pages

`.github/workflows/docker-deploy.yml`：

```yaml
name: Docker Build & Deploy to GitHub Pages

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: pages
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Build Docker image
        uses: docker/build-push-action@v5
        with:
          context: .
          file: ./Dockerfile
          target: builder  # 只使用构建阶段
          cache-from: type=gha
          cache-to: type=gha,mode=max
          outputs: type=local,dest=./dist

      - name: Copy built files
        run: |
          # Docker 输出的是完整的构建上下文，需要提取 dist
          ls -la ./dist
          cp -r ./dist/docs/.vitepress/dist ./output

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./output

  deploy:
    needs: build
    runs-on: ubuntu-latest

    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}

    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

### 5.3 方式三：构建 Docker 镜像并推送到 Docker Hub

`.github/workflows/docker-push.yml`：

```yaml
name: Build and Push Docker Image

on:
  push:
    branches: [main]
    tags:
      - 'v*.*.*'
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: your-dockerhub-username/vitepress-mui-docs
          tags: |
            type=ref,event=branch
            type=semver,pattern={{version}}
            type=semver,pattern={{major}}.{{minor}}
            type=sha

      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

使用此工作流前需要在 GitHub 设置中添加 Secrets：
- `DOCKER_USERNAME`: 你的 Docker Hub 用户名
- `DOCKER_PASSWORD`: 你的 Docker Hub 密码或访问令牌

---

## 6. 常见问题

### Q1: Docker 构建失败 "Cannot find module"

**解决方法**：确保 `package.json` 和 `package-lock.json` 已提交到仓库，检查 `Dockerfile` 中的 COPY 命令。

### Q2: Docker 容器运行后访问空白页

**解决方法**：检查 `nginx.conf` 配置和 `config.ts` 中的 `base` 路径：

```typescript
// Docker 部署时，base 通常设为根路径
base: '/',
```

### Q3: GitHub Pages 显示 404

**解决方法**：
- 检查 `config.ts` 中的 `base` 配置（GitHub Pages 需要仓库名）
- Docker 部署使用 `/`，GitHub Pages 使用 `/repo-name/`

```typescript
// 根据部署环境动态设置 base
const base = process.env.DEPLOY_TARGET === 'docker' ? '/' : '/vitepress-mui-docs/'
```

### Q4: MUI 样式不生效

**解决方法**：确保在 `theme/index.ts` 中导入了样式：

```typescript
import '@mui/material/styles.css'
```

### Q5: Docker 镜像太大

**解决方法**：使用多阶段构建（已在 Dockerfile 中实现），可以考虑：

```dockerfile
# 使用更小的基础镜像
FROM node:20-alpine AS builder

# 清理不必要的文件
RUN npm ci --production=false && \
    npm cache clean --force
```

### Q6: 热重载不工作（Docker 开发模式）

**解决方法**：确保在 `docker-compose.yml` 中正确挂载卷：

```yaml
volumes:
  - .:/app
  - node_modules:/app/node_modules  # 保留 node_modules
```

---

## 快速开始命令总结

```bash
# ====================================
# 本地开发（不使用 Docker）
# ====================================
npm run docs:dev

# ====================================
# Docker 开发环境
# ====================================
docker-compose --profile dev up docs-dev

# ====================================
# Docker 生产环境
# ====================================
# 方式 1: 直接构建
docker build -t vitepress-mui-docs .
docker run -d -p 8080:80 vitepress-mui-docs

# 方式 2: 使用 docker-compose
docker-compose up docs-prod

# ====================================
# 部署到 GitHub Pages
# ====================================
git add .
git commit -m "docs: update"
git push
```

---

## 部署方案对比

| 方案 | 优点 | 缺点 | 适用场景 |
|------|------|------|----------|
| **GitHub Pages** | 免费托管、自动 HTTPS | 只支持静态站点、自定义域名需手动配置 | 个人博客、文档站点 |
| **Docker + Nginx** | 完全控制、可扩展 | 需要服务器维护 | 企业内部文档、需要后端集成的场景 |
| **容器平台 (Vercel/Railway)** | 零配置、全球 CDN | 可能有费用限制 | 快速部署、演示项目 |

---

## 参考资源

- [VitePress 官方文档](https://vitepress.dev/)
- [MUI 官方文档](https://mui.com/)
- [Docker 官方文档](https://docs.docker.com/)
- [GitHub Pages 官方文档](https://docs.github.com/pages)
- [GitHub Actions 文档](https://docs.github.com/actions)
- [Nginx 配置指南](https://nginx.org/en/docs/)

---

## 更新日志

| 日期 | 版本 | 说明 |
|------|------|------|
| 2025-02-09 | 1.0.0 | 初始版本，支持 Docker 和 GitHub Pages 部署 |

---

> 💡 **提示**：
> - **GitHub Pages 部署**：推送代码后自动触发，几分钟后即可在 `https://your-username.github.io/vitepress-mui-docs/` 访问
> - **Docker 部署**：适合部署到自己的服务器或容器平台，获得更好的控制力和扩展性
> - **本地开发**：推荐使用 Docker 开发模式，保持环境一致性
