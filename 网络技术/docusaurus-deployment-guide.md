# Docusaurus 部署教程 - 支持 GitHub Markdown 同步

本文介绍如何部署 Docusaurus 网站，并实现从 GitHub 仓库同步 Markdown 文件的功能。

## 目录

- [前置要求](#前置要求)
- [快速开始](#快速开始)
- [配置 GitHub Markdown 同步](#配置-github-markdown-同步)
- [部署到 GitHub Pages](#部署到-github-pages)
- [自动化工作流](#自动化工作流)

---

## 前置要求

- [Node.js](https://nodejs.org/) 18.0 或更高版本
- [Git](https://git-scm.com/)
- GitHub 账号
- (可选) npm 或 yarn 或 pnpm

检查 Node.js 版本：
```bash
node --version
```

---

## 快速开始

### 1. 创建 Docusaurus 项目

```bash
# 使用 npm
npx create-docusaurus@latest my-website classic

# 或使用 yarn
yarn create docusaurus my-website classic

# 或使用 pnpm
pnpm create docusaurus my-website classic
```

项目结构：
```
my-website/
├── blog/
│   ├── 2019-05-28-hola.md
│   └── 2019-05-29-welcome.md
├── docs/
│   ├── intro.md
│   └── mdx.md
├── src/
│   ├── components/
│   ├── css/
│   └── pages/
├── static/
│   └── img/
├── docusaurus.config.js
├── sidebars.js
├── package.json
└── README.md
```

### 2. 本地开发

```bash
cd my-website
npm run start
```

访问 http://localhost:3000 查看网站。

### 3. 构建生产版本

```bash
npm run build
```

---

## 配置 GitHub Markdown 同步

Docusaurus 支持多种方式同步 GitHub 上的 Markdown 文件：

### 方法一：Git Submodule（推荐）

将外部 GitHub 仓库作为子模块添加到 `docs` 目录：

```bash
# 移除默认的 docs 文件夹
rm -rf docs

# 添加子模块（替换为你的 GitHub 仓库地址）
git submodule add https://github.com/username/docs-repo.git docs
```

更新子模块内容：
```bash
# 拉取子模块最新内容
git submodule update --remote --merge

# 或初始化并更新所有子模块
git submodule update --init --recursive
```

### 方法二：Git Subtree

使用 subtree 合并外部仓库：

```bash
# 添加远程仓库
git remote add docs-remote https://github.com/username/docs-repo.git

# 使用 subtree 拉取内容
git subtree add --prefix=docs docs-remote main

# 更新 subtree
git subtree pull --prefix=docs docs-remote main
```

### 方法三：自定义脚本同步

创建同步脚本 `scripts/sync-docs.js`：

```javascript
const { execSync } = require('child_process');
const fs = require('fs');
const path = require('path');

const DOCS_REPO = 'https://github.com/username/docs-repo.git';
const DOCS_DIR = path.join(__dirname, '../docs-temp');
const TARGET_DIR = path.join(__dirname, '../docs');

function syncDocs() {
  console.log('🔄 开始同步文档...');

  // 克隆或更新仓库
  if (fs.existsSync(DOCS_DIR)) {
    console.log('📥 拉取最新内容...');
    execSync('git pull', { cwd: DOCS_DIR });
  } else {
    console.log('📥 克隆仓库...');
    execSync(`git clone ${DOCS_REPO} ${DOCS_DIR}`);
  }

  // 复制文件到 docs 目录
  console.log('📋 复制文件...');
  if (fs.existsSync(TARGET_DIR)) {
    fs.rmSync(TARGET_DIR, { recursive: true, force: true });
  }
  fs.cpSync(DOCS_DIR, TARGET_DIR, { recursive: true });

  console.log('✅ 同步完成！');
}

syncDocs();
```

在 `package.json` 中添加脚本：

```json
{
  "scripts": {
    "sync-docs": "node scripts/sync-docs.js",
    "start": "npm run sync-docs && docusaurus start",
    "build": "npm run sync-docs && docusaurus build"
  }
}
```

---

## 部署到 GitHub Pages

### 方法一：使用 Docusaurus 部署命令

1. 在 `docusaurus.config.js` 中配置：

```javascript
module.exports = {
  // ...
  url: 'https://username.github.io',
  baseUrl: '/my-website/',

  organizationName: 'username', // GitHub 用户名
  projectName: 'my-website',    // 仓库名称
  deploymentBranch: 'gh-pages', // 部署分支
  trailingSlash: false,
};
```

2. 部署到 GitHub Pages：

```bash
npm run deploy
```

### 方法二：GitHub Actions 自动部署

创建 `.github/workflows/deploy.yml`：

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

jobs:
  deploy:
    name: Deploy to GitHub Pages
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive  # 同步子模块

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Build website
        run: npm run build

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./build

      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

---

## 自动化工作流

### 自动同步外部仓库

创建 `.github/workflows/sync-docs.yml`，每天自动同步文档：

```yaml
name: Sync Documentation

on:
  schedule:
    - cron: '0 2 * * *'  # 每天 UTC 2:00
  workflow_dispatch:

permissions:
  contents: write

jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          submodules: recursive

      - name: Pull latest submodule changes
        run: |
          git submodule update --remote --merge

      - name: Commit and push
        run: |
          git config user.name 'github-actions[bot]'
          git config user.email 'github-actions[bot]@users.noreply.github.com'
          git add docs/
          git diff --staged --quiet || git commit -m "docs: auto-sync documentation"
          git push
```

### 推送时自动构建部署

结合文档同步和部署的完整工作流：

```yaml
name: Sync and Deploy

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Build website
        run: npm run build

      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./build
```

---

## 高级配置

### 自定义导航栏

在 `docusaurus.config.js` 中配置：

```javascript
module.exports = {
  themeConfig: {
    navbar: {
      title: 'My Site',
      logo: {
        alt: 'My Site Logo',
        src: 'img/logo.svg',
      },
      items: [
        {
          type: 'doc',
          docId: 'intro',
          label: '文档',
          position: 'left',
        },
        { to: '/blog', label: '博客', position: 'left' },
        {
          href: 'https://github.com/username/my-website',
          label: 'GitHub',
          position: 'right',
        },
      ],
    },
    footer: {
      style: 'dark',
      links: [
        {
          title: '文档',
          items: [
            { label: '开始使用', to: '/docs/intro' },
          ],
        },
        {
          title: '社区',
          items: [
            { label: 'GitHub', href: 'https://github.com/username/my-website' },
          ],
        },
      ],
      copyright: `Copyright © ${new Date().getFullYear()} My Project.`,
    },
  },
};
```

### 配置侧边栏

编辑 `sidebars.js`：

```javascript
module.exports = {
  tutorialSidebar: [
    {
      type: 'category',
      label: '开始',
      items: ['intro', 'installation'],
    },
    {
      type: 'category',
      label: '高级指南',
      items: ['advanced/config', 'advanced/deployment'],
    },
  ],
};
```

---

## 常见问题

### Q1: 子模块更新后网站没变化？

确保在构建前拉取最新子模块：
```bash
git submodule update --remote
npm run build
```

### Q2: GitHub Pages 部署后 404？

检查 `docusaurus.config.js` 中的 `baseUrl` 配置是否正确，结尾需要加 `/`：
```javascript
baseUrl: '/my-website/',
```

### Q3: 如何使用自定义域名？

1. 在 `docusaurus.config.js` 中设置：
```javascript
url: 'https://yourdomain.com',
baseUrl: '/',
```

2. 在仓库根目录创建 `CNAME` 文件：
```
yourdomain.com
```

3. 在域名提供商处添加 DNS 记录。

### Q4: 如何支持数学公式？

安装并配置 KaTeX：

```bash
npm install remark-math rehype-katex
```

在 `docusaurus.config.js` 中添加：
```javascript
const math = require('remark-math');
const katex = require('rehype-katex');

module.exports = {
  presets: [
    [
      '@docusaurus/preset-classic',
      {
        docs: {
          remarkPlugins: [math],
          rehypePlugins: [katex],
        },
      },
    ],
  ],
  stylesheets: [
    {
      href: 'https://cdn.jsdelivr.net/npm/katex@0.13.24/dist/katex.min.css',
      type: 'text/css',
      integrity: 'sha384-odtC+0UGzzFL/6PNoE8rX/SPcQDXBJ+uRepguP4QKPCk2xkX+h4/eE9l4bJnK',
      crossorigin: 'anonymous',
    },
  ],
};
```

---

## 参考资源

- [Docusaurus 官方文档](https://docusaurus.io/)
- [部署到 GitHub Pages](https://docusaurus.io/docs/deployment)
- [Git 子模块教程](https://git-scm.com/book/en/v2/Git-Tools-Submodules)

---

**最后更新**: 2026-02-09
