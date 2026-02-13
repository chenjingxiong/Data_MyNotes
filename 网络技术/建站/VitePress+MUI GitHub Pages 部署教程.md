# VitePress + MUI GitHub Pages 自动部署教程

本教程介绍如何将 VitePress 与 Material-UI (MUI) 结合使用，并通过 GitHub Actions 自动部署到 GitHub Pages。

---

## 目录

1. [项目初始化](#1-项目初始化)
2. [配置 VitePress](#2-配置-vitepress)
3. [集成 MUI](#3-集成-mui)
4. [GitHub Pages 部署配置](#4-github-pages-部署配置)
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

---

## 2. 配置 VitePress

### 2.1 项目结构

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
│   │   └── config.ts
│   ├── guide/
│   │   └── index.md
│   └── index.md
├── package.json
└── package-lock.json
```

### 2.2 初始化 VitePress

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

### 2.3 配置文件 `docs/.vitepress/config.ts`

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

### 2.4 更新 `package.json`

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
Mui-container {
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

## 4. GitHub Pages 部署配置

### 4.1 创建 GitHub 仓库

1. 在 GitHub 创建新仓库 `vitepress-mui-docs`
2. 初始化本地 Git：

```bash
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/your-username/vitepress-mui-docs.git
git push -u origin main
```

### 4.2 配置仓库设置

进入仓库 **Settings** → **Pages**：

- **Source**: 选择 `GitHub Actions`
- **Custom domain**: (可选) 添加你的自定义域名

---

## 5. GitHub Actions 自动化

### 5.1 创建工作流文件 `.github/workflows/deploy.yml`

```yaml
name: Deploy VitePress to GitHub Pages

on:
  # 推送到 main 分支时触发
  push:
    branches: [main]

  # 允许手动触发
  workflow_dispatch:

# 设置权限
permissions:
  contents: read
  pages: write
  id-token: write

# 防止并发部署
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
          fetch-depth: 0 # 启用 git history

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

### 5.2 工作流说明

| 触发条件 | 说明 |
|---------|------|
| `push: branches: [main]` | 每次 push 到 main 分支自动部署 |
| `workflow_dispatch` | 在 GitHub Actions 页面手动触发 |

### 5.3 高级配置：多环境部署

如果需要测试环境和生产环境：

```yaml
on:
  push:
    branches:
      - main    # 生产环境
      - develop # 测试环境

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to environment
        if: github.ref == 'refs/heads/main'
        # 生产环境部署...

      - name: Deploy to staging
        if: github.ref == 'refs/heads/develop'
        # 测试环境部署...
```

---

## 6. 常见问题

### Q1: 构建失败 "Cannot find module '@mui/material'"

**解决方法**：确保在 `package.json` 中正确配置了 `npm ci` 命令，并且 `package-lock.json` 已提交到仓库。

### Q2: GitHub Pages 显示 404

**解决方法**：检查 `config.ts` 中的 `base` 配置：

```typescript
// 正确格式：以 / 开头和结尾
base: '/your-repo-name/',

// 如果使用自定义域名
base: '/',
```

### Q3: MUI 样式不生效

**解决方法**：确保在 `theme/index.ts` 中导入了样式：

```typescript
import '@mui/material/styles.css'
```

### Q4: 自动更新不工作

**检查清单**：

- ✅ 工作流文件在 `.github/workflows/` 目录下
- ✅ 仓库 Settings → Pages → Source 选择 `GitHub Actions`
- ✅ 工作流文件有正确的权限配置 (`permissions`)
- ✅ 推送代码到正确的分支 (默认 `main`)

### Q5: 构建超时

**解决方法**：在工作流中添加超时设置：

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    timeout-minutes: 10
    steps:
      # ...
```

---

## 快速开始命令总结

```bash
# 1. 创建项目
npm init -y
npm install -D vitepress
npm install @mui/material @emotion/react @emotion/styled

# 2. 初始化 VitePress
npx vitepress init

# 3. 本地开发
npm run docs:dev

# 4. 本地构建预览
npm run docs:build
npm run docs:preview

# 5. 提交并自动部署
git add .
git commit -m "docs: update"
git push
```

---

## 参考资源

- [VitePress 官方文档](https://vitepress.dev/)
- [MUI 官方文档](https://mui.com/)
- [GitHub Pages 官方文档](https://docs.github.com/pages)
- [GitHub Actions 文档](https://docs.github.com/actions)

---

## 更新日志

| 日期 | 版本 | 说明 |
|------|------|------|
| 2025-02-09 | 1.0.0 | 初始版本 |

---

> 💡 **提示**：首次部署后，GitHub Actions 可能需要几分钟时间完成构建。你可以在仓库的 "Actions" 标签页查看部署进度。部署成功后，你的站点将在 `https://your-username.github.io/vitepress-mui-docs/` 可访问。
