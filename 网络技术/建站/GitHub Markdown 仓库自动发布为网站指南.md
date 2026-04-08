# GitHub Markdown 仓库自动发布为网站完全指南

> **适用场景**：将 GitHub 仓库中的 Markdown 笔记/文档自动发布为可访问的网站
> **更新日期**：2026-04-08

---

## 一、方案对比

| 方案 | 适用场景 | 成本 | 技术难度 | 自动化程度 |
|------|---------|------|---------|-----------|
| **VitePress + GitHub Pages** | 技术文档/笔记站 | 免费 | 低 | 高（推送自动部署） |
| **Docusaurus + GitHub Pages** | 文档中心/博客 | 免费 | 中 | 高 |
| **Hugo + GitHub Pages** | 博客/个人站 | 免费 | 低 | 高 |
| **WordPress + Git it Write** | 已有 WordPress 站 | 需服务器 | 中 | 中（定时拉取） |
| **WordPress + REST API** | 精细控制发布 | 需服务器 | 高 | 高 |
| **Cloudflare Pages** | 全球加速站 | 免费 | 低 | 高 |

**推荐**：对于 Markdown 笔记仓库，**VitePress + GitHub Pages** 是最佳方案——零成本、自动化、速度快、中文支持好。

---

## 二、方案一：VitePress + GitHub Pages（推荐）

VitePress 是 Vue 驱动的静态站点生成器，专为 Markdown 设计，构建极快，原生支持中文。

### 1. 初始化项目

在仓库根目录执行：

```bash
# 安装 VitePress
npm add -D vitepress

# 初始化向导（会生成 .vitepress/config.ts 等文件）
npx vitepress init
```

初始化向导问答：

```
┌  Welcome to VitePress!
│
◇  Where should VitePress initialize the config?
│  ./docs          ← 推荐放在 docs 目录
│
◇  Site title:
│  My Notes        ← 你的站点名称
│
◇  Site description:
│  技术笔记合集      ← 站点描述
│
◇  Theme:
│  Default Theme   ← 使用默认主题
│
◇  Use TypeScript for config file?
│  Yes
│
◇  Add VitePress npm scripts to package.json?
│  Yes
```

### 2. 项目结构

```
Data_MyNotes/
├── .vitepress/           # VitePress 配置
│   └── config.ts
├── docs/                  # 站点入口（可选）
├── AI技术/                # 你的 Markdown 目录
├── 网络技术/
├── 笔记本技术/
├── package.json
└── .github/
    └── workflows/
        └── deploy.yml    # GitHub Actions 部署脚本
```

### 3. 配置侧边栏和导航

编辑 `.vitepress/config.ts`：

```typescript
import { defineConfig } from 'vitepress'

export default defineConfig({
  title: '技术笔记',
  description: '技术笔记合集',
  lang: 'zh-CN',

  themeConfig: {
    nav: [
      { text: '首页', link: '/' },
      { text: 'AI 技术', link: '/AI技术/Firecrawl 介绍与部署使用指南' },
      { text: '网络技术', link: '/网络技术/建站/Spacedrive 介绍与部署使用指南' },
    ],

    sidebar: {
      '/AI技术/': [
        {
          text: 'AI 技术',
          items: [
            { text: 'Firecrawl 指南', link: '/AI技术/Firecrawl 介绍与部署使用指南' },
            { text: 'Claude Code 浏览器自动化', link: '/AI技术/ClaudeCode/ClaudeCode浏览器自动化指南' },
          ]
        }
      ],
      '/网络技术/': [
        {
          text: '建站',
          items: [
            { text: 'Spacedrive 指南', link: '/网络技术/建站/Spacedrive 介绍与部署使用指南' },
            { text: 'VitePress 教程', link: '/网络技术/建站/1Panel 部署 VitePress 教程' },
          ]
        }
      ]
    },

    search: {
      provider: 'local',   // 内置本地搜索
      options: {
        locales: {
          root: {
            translations: {
              button: { buttonText: '搜索文档' }
            }
          }
        }
      }
    },

    socialLinks: [
      { icon: 'github', link: 'https://github.com/chenjingxiong/Data_MyNotes' }
    ]
  }
})
```

### 4. 创建首页

在仓库根目录创建 `index.md`：

```markdown
---
layout: home

hero:
  name: 技术笔记
  text: 个人知识库
  tagline: 记录、整理、分享
  actions:
    - theme: brand
      text: 开始浏览
      link: /AI技术/Firecrawl 介绍与部署使用指南
    - theme: alt
      text: GitHub
      link: https://github.com/chenjingxiong/Data_MyNotes
---
```

### 5. 自动生成侧边栏（进阶）

手动维护侧边栏太麻烦，可以用脚本自动生成。创建 `scripts/sidebar.ts`：

```typescript
import fs from 'fs'
import path from 'path'

interface SidebarItem {
  text: string
  link: string
}

interface SidebarGroup {
  text: string
  items: SidebarItem[]
}

function generateSidebar(dir: string, prefix: string = ''): SidebarGroup[] {
  const fullPath = path.join(process.cwd(), dir)
  const groups: SidebarGroup[] = []

  // 直接扫描目录下的子目录
  const entries = fs.readdirSync(fullPath, { withFileTypes: true })

  for (const entry of entries) {
    if (entry.name.startsWith('.') || entry.name === 'node_modules') continue

    if (entry.isDirectory()) {
      const subDir = path.join(fullPath, entry.name)
      const mdFiles = fs.readdirSync(subDir).filter(f => f.endsWith('.md'))

      if (mdFiles.length > 0) {
        groups.push({
          text: entry.name,
          items: mdFiles.map(f => ({
            text: f.replace('.md', ''),
            link: `/${prefix}${entry.name}/${f}`
          }))
        })
      }
    }
  }

  return groups
}

// 导出配置用
export const sidebar = {
  '/AI技术/': generateSidebar('AI技术', 'AI技术/'),
  '/网络技术/': generateSidebar('网络技术', '网络技术/'),
}
```

更简单的方式——使用社区插件 `vitepress-sidebar`：

```bash
npm add -D vitepress-sidebar
```

```typescript
// .vitepress/config.ts
import { defineConfig } from 'vitepress'
import { generateSidebar } from 'vitepress-sidebar'

const sidebar = generateSidebar([
  {
    documentRootPath: '/AI技术',
    routeRootPath: '/AI技术',
  },
  {
    documentRootPath: '/网络技术',
    routeRootPath: '/网络技术',
  }
])

export default defineConfig({
  // ... 其他配置
  themeConfig: {
    sidebar,
  }
})
```

### 6. 配置 GitHub Actions 自动部署

创建 `.github/workflows/deploy.yml`：

```yaml
name: Deploy VitePress site to Pages

on:
  push:
    branches: [master]
  workflow_dispatch:   # 允许手动触发

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

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm

      - name: Setup Pages
        uses: actions/configure-pages@v4

      - name: Install dependencies
        run: npm ci

      - name: Build with VitePress
        run: npm run docs:build

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: .vitepress/dist

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

### 7. 配置 GitHub Pages

1. 进入仓库 **Settings → Pages**
2. **Build and deployment → Source** 选择 **GitHub Actions**
3. 推送代码到 `master` 分支，Actions 自动构建部署
4. 访问地址：`https://<username>.github.io/<repo-name>/`

### 8. package.json 脚本

```json
{
  "devDependencies": {
    "vitepress": "^1.6.0"
  },
  "scripts": {
    "docs:dev": "vitepress dev",
    "docs:build": "vitepress build",
    "docs:preview": "vitepress preview"
  }
}
```

---

## 三、方案二：Docusaurus（React 生态）

Docusaurus 是 Meta 开发的文档站点生成器，功能丰富，插件生态好。

### 1. 初始化

```bash
npx create-docusaurus@latest my-site classic
cd my-site
```

### 2. 配置中文和侧边栏

编辑 `docusaurus.config.js`：

```javascript
module.exports = {
  title: '技术笔记',
  url: 'https://yourusername.github.io',
  baseUrl: '/Data_MyNotes/',
  i18n: {
    defaultLocale: 'zh-CN',
    locales: ['zh-CN'],
  },
  presets: [
    [
      'classic',
      {
        docs: {
          sidebarPath: require.resolve('./sidebars.js'),
          // 指向你的 markdown 目录
          path: '../AI技术',
          routeBasePath: 'ai',
        },
      },
    ],
  ],
};
```

### 3. GitHub Actions 部署

```yaml
name: Deploy Docusaurus

on:
  push:
    branches: [master]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm
      - run: npm ci
      - run: npm run build
      - uses: peaceiris/actions-gh-pages@v4
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./build
```

### 4. Docusaurus 的优势

| 特性 | 说明 |
|------|------|
| 版本化文档 | 支持多版本文档 |
| 搜索 | 内置 Algolia DocSearch 集成 |
| 博客支持 | 内置博客功能，支持 RSS |
| 国际化 | 完善的多语言支持 |
| MDX | 支持 React 组件嵌入 Markdown |

---

## 四、方案三：Hugo（最快构建）

Hugo 用 Go 编写，构建速度极快，适合大量文件。

### 1. 安装 Hugo

```bash
# macOS
brew install hugo

# Linux
sudo snap install hugo

# Debian/Ubuntu
sudo apt install hugo
```

### 2. 初始化站点

```bash
hugo new site my-notes
cd my-notes
git init
git submodule add https://github.com/thegeeklab/hugo-geekdoc.git themes/geekdoc
```

### 3. 配置

`hugo.toml`：

```toml
baseURL = 'https://yourusername.github.io/Data_MyNotes/'
languageCode = 'zh-CN'
title = '技术笔记'
theme = 'geekdoc'

[params]
  geekdocSearch = true
```

### 4. 将 Markdown 文件链接到 content 目录

```bash
# 符号链接你的 markdown 目录
ln -s ../AI技术 content/AI技术
ln -s ../网络技术 content/网络技术
```

### 5. GitHub Actions 部署

```yaml
name: Deploy Hugo

on:
  push:
    branches: [master]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          submodules: true

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v3
        with:
          hugo-version: 'latest'

      - name: Build
        run: hugo --minify

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v4
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
```

---

## 五、方案四：WordPress 发布

已有 WordPress 站点的用户，可以通过插件自动从 GitHub 仓库同步 Markdown 文章。

### 方法 A：Git it Write 插件

**项目地址**：[github.com/vaakash/git-it-write](https://github.com/vaakash/git-it-write)

#### 安装

1. WordPress 后台 → 插件 → 安装插件
2. 搜索 "Git it Write" → 安装并激活

#### 配置

1. 进入 **Settings → Git it Write**
2. 添加仓库：
   - **Repository URL**：`https://github.com/chenjingxiong/Data_MyNotes`
   - **Branch**：`master`
   - **Directory**：`AI技术`（或留空同步全部）
3. 配置映射规则：
   - Markdown 文件 → WordPress 文章/页面
   - Frontmatter 字段映射到 WordPress 文章属性
4. 设置同步频率（每小时/每天/手动）

#### Frontmatter 格式

在 Markdown 文件头部添加 YAML 元数据：

```yaml
---
title: Firecrawl 介绍与部署使用指南
date: 2026-04-07
categories: AI技术
tags: [AI, 爬虫, Docker]
status: publish
---
```

### 方法 B：Publish Documents from Git 插件

**项目地址**：[github.com/gis-ops/wordpress-markdown-git](https://github.com/gis-ops/wordpress-markdown-git)

支持 Markdown 和 Jupyter Notebook 文件，适合技术文档站点。

### 方法 C：GitHub Actions + WordPress REST API

完全自定义的自动化方案，适合精细控制。

#### 1. 生成 WordPress Application Password

WordPress 后台 → 用户 → 个人资料 → 应用密码 → 添加新应用密码

#### 2. 配置 GitHub Secrets

在仓库 Settings → Secrets and variables → Actions 中添加：

| Secret 名称 | 值 |
|-------------|---|
| `WP_URL` | `https://your-wordpress-site.com` |
| `WP_USERNAME` | WordPress 用户名 |
| `WP_APP_PASSWORD` | 应用密码 |

#### 3. 创建部署工作流

```yaml
name: Publish to WordPress

on:
  push:
    paths:
      - '**.md'
    branches: [master]

jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Publish Markdown to WordPress
        env:
          WP_URL: ${{ secrets.WP_URL }}
          WP_USER: ${{ secrets.WP_USERNAME }}
          WP_PASS: ${{ secrets.WP_APP_PASSWORD }}
        run: |
          # 遍历所有变更的 markdown 文件并发布
          git diff --name-only --diff-filter=ACM HEAD~1 HEAD -- '*.md' | while read file; do
            echo "Publishing: $file"

            # 提取标题（第一行 # 标题）
            title=$(head -5 "$file" | grep -oP '(?<=# ).+' | head -1)
            if [ -z "$title" ]; then
              title=$(basename "$file" .md)
            fi

            # 读取文件内容
            content=$(cat "$file")

            # 通过 REST API 发布
            curl -s -X POST \
              "${WP_URL}/wp-json/wp/v2/posts" \
              -u "${WP_USER}:${WP_PASS}" \
              -H "Content-Type: application/json" \
              -d "$(jq -n \
                --arg title "$title" \
                --arg content "$content" \
                '{title: $title, content: $content, status: "draft"}')"
          done
```

这个脚本会将新提交的 Markdown 文件以草稿形式发布到 WordPress，你可以在后台审核后发布。

---

## 六、方案五：Cloudflare Pages（全球加速）

如果需要全球访问速度更快，可以用 Cloudflare Pages 替代 GitHub Pages。

### 1. 配置 Cloudflare Pages

1. 登录 [Cloudflare Dashboard](https://dash.cloudflare.com/)
2. Workers & Pages → Create → Pages → Connect to Git
3. 选择你的 GitHub 仓库

### 2. 构建设置

| 设置项 | 值 |
|-------|---|
| Framework preset | VitePress |
| Build command | `npm run docs:build` |
| Build output directory | `.vitepress/dist` |
| Root directory | `/` |

### 3. 自动部署

每次推送到 `master` 分支，Cloudflare 自动构建部署。还可以配置预览部署（PR 自动生成预览链接）。

### 4. 自定义域名

在 Cloudflare Pages 设置中添加自定义域名，自动配置 SSL。

---

## 七、中文文件名处理

Markdown 文件含中文路径，需要注意以下配置：

### VitePress

```typescript
// .vitepress/config.ts
export default defineConfig({
  // 启用干净的 URL（无 .html 后缀）
  cleanUrls: true,

  // 构建时编码处理
  vite: {
    build: {
      chunkSizeWarningLimit: 2000,
    },
    server: {
      fs: {
        allow: ['..']  // 允许访问上级目录
      }
    }
  }
})
```

### Nginx 反向代理配置

如果自托管，Nginx 需要处理中文 URL：

```nginx
server {
    listen 80;
    server_name notes.example.com;
    root /var/www/notes;

    # 处理中文 URL 编码
    location / {
        try_files $uri $uri/ $uri.html /index.html;
        charset utf-8;
    }
}
```

---

## 八、方案选择建议

### 你的场景（技术笔记仓库）

```
Data_MyNotes/
├── AI技术/ClaudeCode/...
├── AI技术/Firecrawl 介绍与部署使用指南.md
├── 网络技术/建站/...
├── 笔记本技术/...
└── README.md
```

**推荐路线**：

| 需求 | 推荐方案 |
|------|---------|
| 快速上线、零成本 | VitePress + GitHub Pages |
| 全球加速、自定义域名 | VitePress + Cloudflare Pages |
| 已有 WordPress 站 | Git it Write 插件 |
| 需要博客 + 评论功能 | Docusaurus + Giscus |
| 超大仓库（1000+ 文件） | Hugo + GitHub Pages |

### 最简实施步骤（VitePress）

```bash
# 1. 安装依赖
npm add -D vitepress

# 2. 创建最小配置
cat > .vitepress/config.ts << 'EOF'
import { defineConfig } from 'vitepress'
export default defineConfig({
  title: '技术笔记',
  lang: 'zh-CN',
  cleanUrls: true,
})
EOF

# 3. 创建首页
cat > index.md << 'EOF'
---
layout: home
hero:
  name: 技术笔记
  text: 个人知识库
---
EOF

# 4. 本地预览
npx vitepress dev

# 5. 创建 GitHub Actions 部署文件（见上方）
# 6. 推送到 GitHub，自动部署
```

从安装到上线，最快 **10 分钟**搞定。

---

## 九、参考链接

- **VitePress 官方部署文档**：[vitepress.dev/guide/deploy](https://vitepress.dev/guide/deploy)
- **VitePress 中文部署文档**：[vitepress.dev/zh/guide/deploy](https://vitepress.dev/zh/guide/deploy)
- **Docusaurus 官方文档**：[docusaurus.io](https://docusaurus.io)
- **Hugo 官方文档**：[gohugo.io](https://gohugo.io)
- **Git it Write 插件**：[github.com/vaakash/git-it-write](https://github.com/vaakash/git-it-write)
- **Publish Documents from Git**：[github.com/gis-ops/wordpress-markdown-git](https://github.com/gis-ops/wordpress-markdown-git)
- **vitepress-sidebar 插件**：[github.com/jooy2/vitepress-sidebar](https://github.com/jooy2/vitepress-sidebar)
- **Cloudflare Pages 文档**：[developers.cloudflare.com/pages](https://developers.cloudflare.com/pages)

---

*最后更新：2026-04-08*
