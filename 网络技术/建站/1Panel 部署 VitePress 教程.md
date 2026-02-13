# 1Panel 部署 VitePress 教程

本教程介绍如何使用 **Obsidian + Git + 1Panel + VitePress** 构建自动化的知识库发布系统。

## 整体工作流

```
┌─────────────────┐      Git 推送      ┌─────────────────┐
│   本地 Obsidian │ ─────────────────► │  GitHub 仓库    │
│   (写作 + 提交)  │                    │  (Markdown)     │
└─────────────────┘                    └────────┬────────┘
                                                │
                                                │ 拉取 + 构建
                                                ▼
                                         ┌─────────────────┐
                                         │  1Panel 服务器  │
                                         │  (VitePress)    │
                                         └─────────────────┘
```

---

## 目录

1. [前置准备](#1-前置准备)
2. [本地配置：Obsidian + Git 自动提交](#2-本地配置obsidian--git-自动提交)
3. [服务器配置：1Panel 部署 VitePress](#3-服务器配置1panel-部署-vitepress)
4. [自动更新：GitHub 变更自动同步](#4-自动更新github-变更自动同步)
5. [域名配置](#5-域名配置)
6. [常见问题](#6-常见问题)
7. [完整工作流总结](#7-完整工作流总结)
8. [进阶配置](#8-进阶配置)

---

## 1. 前置准备

### 1.1 环境要求

| 项目            | 要求                           |
| ------------- | ---------------------------- |
| **本地**        | Obsidian + Git               |
| **服务器**       | Linux (Ubuntu/CentOS/Debian) |
| **1Panel**    | 已安装 1Panel 面板                |
| **OpenResty** | 在 1Panel 应用商店安装              |
| **Node.js**   | 在 1Panel 应用商店安装 (v18+)       |
| **GitHub**    | 一个 GitHub 仓库                 |

### 1.2 安装 1Panel

如果还未安装，执行以下命令：

```bash
curl -sSL https://resource.fit2cloud.com/1panel/package/quick_start.sh -o quick_start.sh && bash quick_start.sh
```

安装完成后访问 `http://你的服务器IP:端口` 登录面板。

### 1.3 准备 GitHub 仓库

1. 在 GitHub 创建新仓库（如 `my-notes-vitepress`）
2. 初始化为仓库并添加 README
3. 记录仓库地址：`https://github.com/username/my-notes-vitepress.git`

---

## 方案选择：单仓库 vs 双仓库

在开始配置前，你需要选择仓库结构：

### 方案 A：单仓库（简单）

```
my-notes-vitepress/
├── docs/              # Markdown 内容 + VitePress 配置
├── .obsidian/         # Obsidian 配置
└── package.json
```

**优点**：配置简单，适合个人项目
**缺点**：内容和配置混在一起

### 方案 B：双仓库（推荐）

使用 **Git Submodule** 分离内容和配置：

```
# 仓库 1：notes-content（Markdown 内容）
notes-content/
├── tech/
├── life/
└── index.md

# 仓库 2：vitepress-site（网站配置）
vitepress-site/
├── docs/              → 通过 submodule 指向 notes-content
├── .vitepress/        # VitePress 配置
├── package.json
└── .gitmodules        # submodule 配置
```

**优点**：
- 内容和配置完全分离
- Obsidian 只需关注 `notes-content`
- 网站配置独立管理，可单独升级

**缺点**：初次配置稍复杂

---

## 2. 本地配置：Obsidian + Git 自动提交

### 方案 A：单仓库配置

### 2.1 安装 Obsidian Git 插件

### 2.1 安装 Obsidian Git 插件

1. 打开 Obsidian 设置 → **第三方插件** → **浏览**
2. 搜索并安装 **Obsidian Git** 插件
3. 启用插件

### 2.2 配置 Git 插件自动提交

进入 **插件设置** → **Obsidian Git**，配置如下：

| 设置项                      | 推荐值                     | 说明                |
| ------------------------ | ----------------------- | ----------------- |
| **Auto commit interval** | 10 分钟                   | 自动提交时间间隔          |
| **Auto push interval**   | 15 分钟                   | 自动推送到 GitHub      |
| **Commit message**       | `Auto commit: {{date}}` | 提交信息（可自定义）        |
| **Commit on save**       | ✅ 启用                    | 保存文件时自动提交         |
| **Push on commit**       | ✅ 启用                    | 提交后自动推送           |
| **Pull on startup**      | ✅ 启用                    | 启动 Obsidian 时自动拉取 |

### 2.3 初始化本地 Git 仓库

在你的 Obsidian 笔记库根目录执行：

```bash
# 初始化 Git 仓库
git init

# 添加远程仓库
git remote add origin https://github.com/username/my-notes-vitepress.git

# 添加 VitePress 配置（见下一节）
```

### 2.4 创建 VitePress 项目结构

在笔记库根目录创建以下结构：

```
你的笔记库/
├── docs/                    # VitePress 文档目录
│   ├── index.md            # 首页
│   ├── .vitepress/
│   │   ├── config.ts       # VitePress 配置
│   │   └── dist/           # 构建产物（自动生成）
│   └── ...                 # 你的笔记内容
├── .obsidian/              # Obsidian 配置
├── package.json            # Node.js 依赖
└── .git/                   # Git 仓库
```

### 2.5 初始化 VitePress

```bash
# 在笔记库根目录
npm init -y
npm install -D vitepress

# 初始化 VitePress（选择 docs 目录）
npx vitepress init
```

按提示选择：
- `Where should your documentation be placed?` → `docs`
- `Does your documentation use Markdown with frontmatter?` → `Yes`
- `Will you use TypeScript to configure your VitePress site?` → `Yes`
- `Add VitePress npm scripts to package.json?` → `Yes`

### 2.5.1 安装 VitePress Theme Teek（可选）

`vitepress-theme-teek` 是一个轻量、简洁高效的 VitePress 主题，基于默认主题进行拓展。

```bash
# 安装 teek 主题
npm install vitepress-theme-teek -D
```

### 2.6 配置 VitePress

#### 方案 A：使用默认主题

编辑 `docs/.vitepress/config.ts`：

```typescript
import { defineConfig } from 'vitepress'

export default defineConfig({
  title: '我的知识库',
  description: '基于 Obsidian + VitePress 的知识库',

  // 根据部署路径调整，根目录部署用 '/'
  base: '/',

  // 配置侧边栏
  themeConfig: {
    nav: [
      { text: '首页', link: '/' },
      { text: 'GitHub', link: 'https://github.com/username/my-notes-vitepress' }
    ],

    sidebar: [
      {
        text: '笔记',
        items: [
          { text: '首页', link: '/' },
          // 自动扫描 docs 目录下的文件
        ]
      }
    ]
  }
})
```

#### 方案 B：使用 VitePress Theme Teek（推荐）

**1. 创建主题入口文件**

创建 `docs/.vitepress/theme/index.ts`：

```typescript
import Teek from 'vitepress-theme-teek'
import 'vitepress-theme-teek/index.css'

export default {
  extends: Teek,
}
```

**2. 配置文件引入 Teek**

编辑 `docs/.vitepress/config.ts`：

```typescript
import { defineConfig } from 'vitepress'
import { defineTeekConfig } from 'vitepress-theme-teek/config'

// Teek 主题配置
const teekConfig = defineTeekConfig({
  // 主题配置选项
  appearance: true,        // 支持深色模式
  logo: '/logo.png',       // 网站 logo
  nav: [
    { text: '首页', link: '/' },
    { text: 'GitHub', link: 'https://github.com/username/my-notes-vitepress' }
  ],
  sidebar: {
    '/': [
      {
        text: '指南',
        items: [
          { text: '开始', link: '/guide/start' },
          { text: '配置', link: '/guide/config' },
        ]
      }
    ]
  }
})

// VitePress 配置
export default defineConfig({
  extends: teekConfig,

  title: '我的知识库',
  description: '基于 Obsidian + VitePress + Teek 的知识库',

  // 根据部署路径调整
  base: '/',
})
```

**Teek 主题特点**：
- ✅ 轻量级，基于默认主题拓展
- ✅ 支持所有 VitePress 功能
- ✅ 三种典型预设配置
- ✅ 灵活易扩展

### 2.7 本地测试构建

```bash
# 开发模式（预览）
npm run docs:dev

# 构建
npm run docs:build
```

构建成功后，`docs/.vitepress/dist/` 目录包含静态文件。

### 2.8 首次提交到 GitHub（单仓库）

```bash
git add .
git commit -m "Initial VitePress setup"
git push -u origin main
```

---

### 方案 B：双仓库配置（推荐）

使用 Git Submodule 分离内容和配置。

#### 2.9 创建内容仓库

1. **在 GitHub 创建内容仓库** `notes-content`

2. **将 Obsidian 笔记库连接到此仓库**

```bash
# 在你的 Obsidian 笔记库根目录
cd /path/to/your/obsidian/vault
git init
git remote add origin https://github.com/username/notes-content.git

# 创建示例目录结构
mkdir -p tech life
echo "# 我的笔记" > index.md

# 提交内容
git add .
git commit -m "Initial notes"
git push -u origin main
```

3. **配置 Obsidian Git 插件**

   进入 **插件设置** → **Obsidian Git**：

   | 设置项                      | 推荐值                              |
   | ------------------------ | --------------------------------- |
   | **Auto commit interval** | 10 分钟                              |
   | **Auto push interval**   | 15 分钟                              |
   | **Commit message**       | `Update notes: {{date}}`            |
   | **Commit on save**       | ✅ 启用                               |
   | **Push on commit**       | ✅ 启用                               |
   | **Pull on startup**      | ✅ 启用                               |

#### 2.10 创建网站配置仓库

1. **在另一个目录创建网站项目**

```bash
# 创建网站项目目录（与 Obsidian 笔记库分离）
mkdir ~/vitepress-site && cd ~/vitepress-site

# 初始化 Git
git init
git remote add origin https://github.com/username/vitepress-site.git

# 初始化项目
npm init -y
npm install -D vitepress vitepress-theme-teek

# 创建配置目录
mkdir -p docs/.vitepress
```

2. **添加内容仓库为 Submodule**

```bash
# 将 notes-content 作为 submodule 添加到 docs 目录
git submodule add https://github.com/username/notes-content.git docs

# 或者使用 SSH（推荐，配置了 SSH 密钥后）
# git submodule add git@github.com:username/notes-content.git docs
```

此时目录结构：
```
vitepress-site/
├── docs/              → Submodule，指向 notes-content 仓库
│   ├── index.md
│   ├── tech/
│   └── life/
├── .gitmodules        # Submodule 配置文件
├── package.json
└── .git/
```

3. **创建 VitePress 配置文件**

创建 `docs/.vitepress/config.ts`：

```typescript
import { defineConfig } from 'vitepress'
import { defineTeekConfig } from 'vitepress-theme-teek/config'

// Teek 主题配置
const teekConfig = defineTeekConfig({
  appearance: true,
  logo: '/logo.png',
  nav: [
    { text: '首页', link: '/' },
    { text: 'GitHub', link: 'https://github.com/username/vitepress-site' }
  ],
  sidebar: {
    '/': [
      {
        text: '技术笔记',
        items: [
          { text: '所有文章', link: '/tech/' },
        ]
      }
    ]
  }
})

export default defineConfig({
  extends: teekConfig,
  title: '我的知识库',
  description: '基于 Obsidian + VitePress 的知识库',
  base: '/',
})
```

4. **创建主题入口文件**

创建 `docs/.vitepress/theme/index.ts`：

```typescript
import Teek from 'vitepress-theme-teek'
import 'vitepress-theme-teek/index.css'

export default {
  extends: Teek,
}
```

5. **提交网站配置仓库**

```bash
# 在 vitepress-site 目录
cd ~/vitepress-site

# 添加所有配置文件
git add .
git commit -m "Initial VitePress config"
git push -u origin main
```

#### 2.11 双仓库工作流

**日常写作流程：**

```
1. 打开 Obsidian（notes-content 目录）
2. 编辑 Markdown 笔记
3. Obsidian Git 插件自动提交到 notes-content 仓库
```

**更新网站配置：**

```
1. 进入 vitepress-site 目录
2. 修改配置文件或主题
3. 手动提交到 vitepress-site 仓库
```

**同步更新 Submodule：**

```bash
# 在 vitepress-site 目录
cd ~/vitepress-site

# 拉取最新的笔记内容
git submodule update --remote docs

# 或进入 docs 目录手动拉取
cd docs
git pull origin main
```

---

### 2.8 首次提交到 GitHub

```bash
git add .
git commit -m "Initial VitePress setup"
git push -u origin main
```

---

## 3. 服务器配置：1Panel 部署 VitePress

### 3.1 安装 Node.js（通过 1Panel 应用商店）

1. 登录 1Panel 面板
2. 进入 **应用商店** → 搜索 `Node.js`
3. 点击 **安装**
4. 选择版本（推荐 `20.x` 或更高）
5. 安装完成后验证：在 **主机** → **终端** 执行 `node -v`

### 3.2 安装 OpenResty

1. 进入 **应用商店** → 搜索 `OpenResty`
2. 点击 **安装**
3. 配置端口（默认 `80`）

### 3.3 克隆并构建项目

#### 单仓库部署

```bash
# 进入网站目录
cd /opt/1panel/apps/openresty/openresty/www

# 克隆仓库
git clone https://github.com/username/my-notes-vitepress.git vitepress
cd vitepress

# 安装依赖
npm install

# 构建项目
npm run docs:build

# 复制构建产物到网站根目录
cp -r docs/.vitepress/dist/* ./
```

#### 双仓库部署（使用 Submodule）

```bash
# 进入网站目录
cd /opt/1panel/apps/openresty/openresty/www

# 克隆网站配置仓库（会自动拉取 submodule）
git clone https://github.com/username/vitepress-site.git vitepress
cd vitepress

# 更新 submodule（获取最新内容）
git submodule update --init --recursive

# 安装依赖
npm install

# 构建项目
npm run docs:build

# 复制构建产物到网站根目录
cp -r docs/.vitepress/dist/* ./
```

**注意**：如果使用私有仓库，需要先配置 SSH 密钥或使用 Personal Access Token。

### 3.4 创建 1Panel 网站

1. 进入 **网站** → **创建网站**
2. 配置如下：
   - **类型**：静态网站
   - **主域名**：`your-domain.com` 或服务器 IP
   - **根目录**：`/opt/1panel/apps/openresty/openresty/www/vitepress`
   - **PHP 版本**：纯静态（无需选择）

3. 点击 **创建**

### 3.5 配置 SPA 路由（重要）

进入网站 **设置** → **配置文件**，确保包含以下配置：

```nginx
location / {
    try_files $uri $uri/ /index.html;
}
```

### 3.6 配置 HTTPS（可选）

1. 进入网站 **设置** → **SSL**
2. 选择 **Let's Encrypt** 或上传自有证书
3. 开启 **强制 HTTPS**

---

## 4. 自动更新：GitHub 变更自动同步

本节实现：**GitHub 仓库有变更时，1Panel 服务器自动拉取并重建网站**

### 方案一：使用 Cron 定时任务（推荐，简单可靠）

#### 4.1.1 创建更新脚本

在服务器创建自动更新脚本：

```bash
# 创建脚本目录
mkdir -p /opt/scripts

# 创建更新脚本
cat > /opt/scripts/update-vitepress.sh << 'EOF'
#!/bin/bash

# VitePress 项目路径
SITE_PATH="/opt/1panel/apps/openresty/openresty/www/vitepress"
LOG_FILE="/opt/scripts/update-vitepress.log"

# 记录日志函数
log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" >> "$LOG_FILE"
}

log "=== 开始检查更新 ==="

cd "$SITE_PATH" || exit 1

# 拉取最新代码
git fetch origin
LOCAL=$(git rev-parse HEAD)
REMOTE=$(git rev-parse origin/main)

if [ "$LOCAL" != "$REMOTE" ]; then
    log "发现新版本，开始更新..."

    # 拉取代码
    git pull origin main

    # 安装依赖（如有 package.json 变更）
    # npm install

    # 构建
    npm run docs:build

    # 复制构建产物
    cp -r docs/.vitepress/dist/* ./

    log "更新完成！"
else
    log "已是最新版本，无需更新"
fi

log "=== 检查完成 ==="
EOF

# 添加执行权限
chmod +x /opt/scripts/update-vitepress.sh
```

**双仓库版本更新脚本**：

```bash
# 创建双仓库更新脚本
cat > /opt/scripts/update-vitepress-dual.sh << 'EOF'
#!/bin/bash

SITE_PATH="/opt/1panel/apps/openresty/openresty/www/vitepress"
LOG_FILE="/opt/scripts/update-vitepress-dual.log"

log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" >> "$LOG_FILE"
}

log "=== 开始检查更新 ==="

cd "$SITE_PATH" || exit 1

# 检查主仓库更新
git fetch origin
MAIN_LOCAL=$(git rev-parse HEAD)
MAIN_REMOTE=$(git rev-parse origin/main)

# 检查 submodule 更新
cd docs
git fetch origin
SUB_LOCAL=$(git rev-parse HEAD)
SUB_REMOTE=$(git rev-parse origin/main)
cd ..

if [ "$MAIN_LOCAL" != "$MAIN_REMOTE" ] || [ "$SUB_LOCAL" != "$SUB_REMOTE" ]; then
    log "发现新版本，开始更新..."

    # 拉取主仓库（配置）
    git pull origin main

    # 更新 submodule（内容）
    git submodule update --remote docs

    # 构建
    npm run docs:build

    # 复制构建产物
    cp -r docs/.vitepress/dist/* ./

    log "更新完成！"
else
    log "已是最新版本，无需更新"
fi

log "=== 检查完成 ==="
EOF

chmod +x /opt/scripts/update-vitepress-dual.sh
```

#### 4.1.2 配置 1Panel 计划任务

1. 进入 1Panel **计划任务**
2. 点击 **创建计划任务**
3. 配置如下：
   - **任务名称**：`Auto Update VitePress`
   - **执行周期**：每 `5` 分钟
   - **任务类型**：Shell 脚本
   - **命令内容**：
     ```bash
     /opt/scripts/update-vitepress.sh
     ```

4. 点击 **确认**

### 方案二：使用 GitHub Webhook（实时性高）

此方案使用 Docker 运行的 Webhook 服务，通过 **Volume 挂载** 访问宿主机上的 Git 仓库和脚本。

#### 4.2.1 Docker 访问宿主机目录原理

Docker 容器可以通过 `-v` 参数将宿主机目录挂载到容器内：

```bash
docker run -v /host/path:/container/path ...
```

这样容器内的进程就可以访问宿主机的文件系统。

**本方案的挂载结构**：

```
宿主机                          Docker 容器
────────────────────────────────────────────────
/opt/scripts/          →    /opt/scripts/
/opt/1panel/apps/...   →    /opt/1panel/apps/...
/opt/webhook/hooks.yml →    /etc/webhook/hooks.yml
```

#### 4.2.2 安装 Webhook 服务

```bash
# 创建必要的目录
mkdir -p /opt/webhook /opt/scripts

# 使用 Docker 安装 webhook 服务
docker run -d \
  --name webhook-server \
  --restart unless-stopped \
  -p 9000:9000 \
  -v /opt/webhook/hooks.yml:/etc/webhook/hooks.yml \
  -v /opt/scripts:/opt/scripts \
  -v /opt/1panel/apps/openresty/openresty/www:/opt/1panel/apps/openresty/openresty/www \
  almir/webhook
```

**参数说明**：
| 参数 | 说明 |
|------|------|
| `-d` | 后台运行 |
| `--restart unless-stopped` | 自动重启（除非手动停止） |
| `-p 9000:9000` | 端口映射 |
| `-v /opt/webhook/...` | 挂载配置文件 |
| `-v /opt/scripts:/opt/scripts` | **挂载脚本目录**（关键） |
| `-v /opt/1panel/apps/...` | **挂载网站目录**（关键） |

#### 4.2.3 配置 Webhook

创建 `/opt/webhook/hooks.yml`：

```yaml
- id: update-vitepress
  execute-command: /opt/scripts/update-vitepress.sh
  command-working-directory: /opt/1panel/apps/openresty/openresty/www/vitepress
  trigger-rule:
    match:
      type: payload-hash-sha256
      secret: "your-webhook-secret"
      parameter:
        source: header
        name: X-Hub-Signature-256
```

**重要**：由于 `/opt/scripts` 和网站目录都已挂载到容器内，Webhook 执行的脚本可以直接访问宿主机上的：
- Git 仓库（执行 `git pull`）
- Node.js 环境（执行 `npm run docs:build`）
- 构建产物（执行复制操作）

#### 4.2.4 配置 GitHub Webhook

1. 进入 GitHub 仓库 → **Settings** → **Webhooks**
2. 点击 **Add webhook**
3. 配置：
   - **Payload URL**：`http://your-server-ip:9000/hooks/update-vitepress`
   - **Content type**：`application/json`
   - **Secret**：你设置的 `your-webhook-secret`
   - **触发事件**：选择 `Push` events

4. 点击 **Add webhook**

### 方案三：GitHub Actions + SSH 部署（无需服务器 Git）

如果你不希望服务器上有 Git 仓库，可以使用此方案。

#### 4.3.1 创建 GitHub Actions 配置

在仓库创建 `.github/workflows/deploy.yml`：



```yaml
name: Deploy to 1Panel Server

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Build VitePress
        run: |
          npm install
          npm run docs:build

      - name: Deploy to server
        uses: easingthemes/ssh-deploy@main
        env:
          SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
          REMOTE_HOST: ${{ secrets.REMOTE_HOST }}
          REMOTE_USER: ${{ secrets.REMOTE_USER }}
          TARGET: /opt/1panel/apps/openresty/openresty/www/vitepress
          SOURCE: "docs/.vitepress/dist/"
          EXCLUDE: "/node_modules/"
```

#### 4.3.2 配置 GitHub Secrets

在 GitHub 仓库 → **Settings** → **Secrets and variables** → **Actions** 添加：

| Secret 名称         | 值说明                      |
| ---------------- | ------------------------ |
| `SSH_PRIVATE_KEY` | 服务器的 SSH 私钥（`~/.ssh/id_rsa`） |
| `REMOTE_HOST`     | 服务器 IP 地址                |
| `REMOTE_USER`     | SSH 用户名（通常是 `root`）      |

---

## 5. 域名配置

### 5.1 DNS 解析

在域名服务商添加解析：

| 类型 | 主机记录 | 记录值        |
|------|----------|------------- |
| A | @        | 你的服务器 IP    |
| A | www      | 你的服务器 IP    |

### 5.2 1Panel 网站配置

在网站设置中添加域名绑定：
- 主域名：`your-domain.com`
- 子域名：`www.your-domain.com`

---

## 6. 常见问题

### Q1: 页面空白，404 错误

**原因**：`base` 路径配置错误

**解决**：检查 `.vitepress/config.ts`：

```typescript
// 如果部署在根目录
base: '/'

// 如果部署在子目录
base: '/your-subdirectory/'
```

### Q2: 刷新页面 404

**原因**：Nginx 未配置 SPA 路由回退

**解决**：在 1Panel 网站 **设置** → **配置文件** 中添加：

```nginx
location / {
    try_files $uri $uri/ /index.html;
}
```

### Q3: 资源加载失败

**解决**：检查 `outDir` 配置，确保构建产物路径正确：

```typescript
// .vitepress/config.ts
export default defineConfig({
  outDir: './dist',  // 确保是相对路径
})
```

### Q4: Obsidian Git 插件无法提交

**症状**：插件显示 "No changes detected" 但实际有修改

**解决方法**：
1. 检查 `.gitignore` 是否排除了你需要的文件
2. 确认插件有执行权限
3. 手动测试：`git status` 查看是否有变更

### Q5: Git 拉取时提示权限错误

**原因**：仓库是私有的，未配置认证

**解决方法**：

**方法一：使用 SSH 密钥**
```bash
# 生成 SSH 密钥
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

# 将公钥添加到 GitHub 账户
cat ~/.ssh/id_rsa.pub

# 修改远程地址为 SSH
git remote set-url origin git@github.com:username/my-notes-vitepress.git
```

**方法二：使用 Personal Access Token**
```bash
# 在 GitHub 设置中生成 PAT (repo 权限)
# 拉取时使用
git pull https://TOKEN@github.com/username/my-notes-vitepress.git
```

### Q6: 服务器 Node.js 版本过低

**症状**：`npm install` 报错

**解决**：

**方法一：通过 1Panel 更新（推荐）**
1. 进入 1Panel **应用商店** → **已安装**
2. 找到 Node.js → 点击 **更新** 或 **重装**
3. 选择更高版本（推荐 `20.x`）

**方法二：卸载后重装**
1. 在 1Panel **已安装** 中卸载旧版 Node.js
2. 进入 **应用商店** 重新安装最新版

### Q7: 自动更新脚本不执行

**排查步骤**：
1. 检查脚本权限：`ls -l /opt/scripts/update-vitepress.sh`
2. 手动执行测试：`bash /opt/scripts/update-vitepress.sh`
3. 查看 1Panel 计划任务日志
4. 检查 cron 服务状态：`systemctl status cron`

### Q8: 构建时内存不足

**症状**：`npm run docs:build` 时进程被杀死

**解决**：增加 Node.js 内存限制
```bash
# 创建 swap 空间
dd if=/dev/zero of=/swapfile bs=1M count=2048
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile

# 或增加 Node.js 内存
NODE_OPTIONS="--max_old_space_size=4096" npm run docs:build
```

### Q9: 图片资源无法显示

**原因**：图片路径配置问题

**解决**：
1. 将图片放在 `docs/public/` 目录（会被自动复制到构建产物）
2. 或使用绝对路径的图片链接
3. 在 `.gitignore` 中不要忽略图片文件

### Q10: Submodule 内容没有更新

**症状**：`docs` 目录的内容还是旧的

**解决**：
```bash
# 在 vitepress-site 目录
cd ~/vitepress-site

# 更新 submodule 到最新
git submodule update --remote docs

# 或进入 docs 目录手动拉取
cd docs
git pull origin main
```

### Q11: 克隆时 Submodule 为空

**症状**：克隆项目后 `docs` 目录是空的

**解决**：
```bash
# 克隆时初始化 submodule
git clone --recurse-submodules https://github.com/username/vitepress-site.git

# 或克隆后手动初始化
cd vitepress-site
git submodule init
git submodule update
```

### Q12: 双仓库如何配置 Obsidian Git 插件？

**解决**：
- Obsidian 笔记库只连接 `notes-content` 仓库
- 网站配置仓库 `vitepress-site` 单独管理，不在 Obsidian 中操作
- 需要更新网站时，在 `vitepress-site` 目录手动操作

### Q13: Docker Webhook 容器无法访问宿主机文件

**症状**：脚本执行报错 "No such file or directory"

**原因**：目录未正确挂载到容器内

**解决**：
1. 检查容器是否包含正确的挂载参数：
   ```bash
   docker inspect webhook-server | grep -A 10 "Mounts"
   ```
2. 确保以下目录都已挂载：
   - `/opt/scripts` → 脚本目录
   - `/opt/1panel/apps/...` → 网站目录
3. 重新创建容器（删除旧的）：
   ```bash
   docker stop webhook-server && docker rm webhook-server
   # 重新运行 docker run 命令（包含所有 -v 参数）
   ```

### Q14: Docker Webhook 脚本执行后没有效果

**症状**：Webhook 触发成功，但网站没有更新

**排查步骤**：
1. 查看容器日志：
   ```bash
   docker logs webhook-server
   ```
2. 手动测试脚本：
   ```bash
   docker exec webhook-server /opt/scripts/update-vitepress.sh
   ```
3. 检查脚本权限（宿主机上）：
   ```bash
   ls -l /opt/scripts/update-vitepress.sh
   # 应该有执行权限（-rwxr-xr-x）
   ```
4. 确认脚本中的路径是挂载后的路径

### Q15: 容器内无法执行 git/npm 命令

**症状**：脚本报错 "git: command not found" 或 "npm: command not found"

**原因**：Webhook 容器内没有安装这些工具

**解决方案**：

**方案 A：脚本直接使用宿主机路径**
```bash
# 更新脚本，使用完整路径
/opt/scripts/update-vitepress.sh
```
由于目录已挂载，脚本中的 git/npm 命令实际上是在宿主机环境执行的。

**方案 B：使用包含工具的镜像**
```bash
# 使用包含 Node.js 和 Git 的镜像
docker run -d \
  --name webhook-server \
  -p 9000:9000 \
  -v /opt/webhook/hooks.yml:/etc/webhook/hooks.yml \
  -v /opt/scripts:/opt/scripts \
  -v /opt/1panel/apps/openresty/openresty/www:/opt/1panel/apps/openresty/openresty/www \
  node:20 bash -c "npm install -g webhook && webhook -hooks /etc/webhook/hooks.yml -verbose"
```

---

## 7. 完整工作流总结

### 单仓库日常写作流程

```
┌─────────────────────────────────────────────────────────────┐
│                      1. 本地写作                              │
│                  打开 Obsidian → 编辑笔记                      │
└─────────────────────────┬───────────────────────────────────┘
                          │ 自动保存
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                2. Git 自动提交（Obsidian Git 插件）             │
│           每 10 分钟自动提交 → 每 15 分钟自动推送到 GitHub       │
└─────────────────────────┬───────────────────────────────────┘
                          │ 推送到 GitHub
                          ▼
┌─────────────────────────────────────────────────────────────┐
│              3. 服务器自动更新（1Panel Cron 定时任务）           │
│            每 5 分钟检查更新 → 发现新版本 → 拉取 → 构建 → 部署      │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                    4. 访问网站                                │
│              访问 http://your-domain.com 查看最新内容            │
└─────────────────────────────────────────────────────────────┘
```

### 双仓库日常写作流程

```
┌─────────────────────────────────────────────────────────────┐
│               1. 本地写作（Obsidian = notes-content）          │
│                  打开 Obsidian → 编辑 Markdown 笔记             │
└─────────────────────────┬───────────────────────────────────┘
                          │ 自动保存
                          ▼
┌─────────────────────────────────────────────────────────────┐
│          2. 内容自动提交（Obsidian Git → notes-content 仓库）    │
└─────────────────────────┬───────────────────────────────────┘
                          │ 推送到 GitHub
                          ▼
┌─────────────────────────────────────────────────────────────┐
│    3. 服务器自动更新（检测 notes-content 或 vitepress-site 变更） │
│    拉取 submodule → 拉取主仓库 → 构建 → 部署                     │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                    4. 访问网站                                │
└─────────────────────────────────────────────────────────────┘
```

**双仓库优势**：
- **内容专注**：Obsidian 只管理 Markdown，不涉及配置
- **配置独立**：网站配置可单独版本控制
- **灵活切换**：可以轻松替换主题或配置而不影响内容

### 初始化命令速查

```bash
# ===== 本地初始化 =====
cd /path/to/your/obsidian/vault
git init
git remote add origin https://github.com/username/my-notes-vitepress.git
npm init -y
npm install -D vitepress
npx vitepress init
git add .
git commit -m "Initial commit"
git push -u origin main

# ===== 服务器初始化 =====
# 1. 在 1Panel 应用商店安装 Node.js 和 OpenResty
# 2. 然后执行以下命令：

# 克隆并构建
cd /opt/1panel/apps/openresty/openresty/www
git clone https://github.com/username/my-notes-vitepress.git vitepress
cd vitepress
npm install
npm run docs:build
cp -r docs/.vitepress/dist/* ./

# 创建自动更新脚本
mkdir -p /opt/scripts
cat > /opt/scripts/update-vitepress.sh << 'EOF'
#!/bin/bash
SITE_PATH="/opt/1panel/apps/openresty/openresty/www/vitepress"
cd "$SITE_PATH" || exit 1
git fetch origin
LOCAL=$(git rev-parse HEAD)
REMOTE=$(git rev-parse origin/main)
if [ "$LOCAL" != "$REMOTE" ]; then
    git pull origin main
    npm run docs:build
    cp -r docs/.vitepress/dist/* ./
fi
EOF
chmod +x /opt/scripts/update-vitepress.sh
```

### 双仓库初始化命令速查

```bash
# ===== 1. 创建内容仓库（notes-content）=====
cd /path/to/your/obsidian/vault
git init
git remote add origin https://github.com/username/notes-content.git
git add .
git commit -m "Initial notes"
git push -u origin main

# ===== 2. 创建网站配置仓库（vitepress-site）=====
mkdir ~/vitepress-site && cd ~/vitepress-site
git init
git remote add origin https://github.com/username/vitepress-site.git
npm init -y
npm install -D vitepress vitepress-theme-teek

# 添加 submodule
git submodule add https://github.com/username/notes-content.git docs

# 创建 VitePress 配置
mkdir -p docs/.vitepress/theme
# （创建 config.ts 和 theme/index.ts，见上方教程）

git add .
git commit -m "Initial VitePress config"
git push -u origin main

# ===== 3. 服务器初始化（双仓库）=====
# 在 1Panel 应用商店安装 Node.js 和 OpenResty
cd /opt/1panel/apps/openresty/openresty/www
git clone --recurse-submodules https://github.com/username/vitepress-site.git vitepress
cd vitepress
npm install
npm run docs:build
cp -r docs/.vitepress/dist/* ./

# 使用双仓库更新脚本（见上方 4.1.1）
```

### 方案对比（仓库结构）

| 仓库结构       | 优点              | 缺点                | 适用场景     |
| ---------- | --------------- | ----------------- | -------- |
| **单仓库**     | 配置简单，一个仓库管理所有内容 | 内容和配置混在一起        | 小型个人项目  |
| **双仓库**     | 内容和配置完全分离，职责清晰  | 初次配置稍复杂，需了解 submodule | 中大型知识库项目 |

### 方案对比（自动更新）

**推荐选择**：
- **新手**：Cron 定时任务
- **追求实时**：GitHub Webhook
- **团队协作**：GitHub Actions + SSH

---

## 8. 进阶配置

### 8.1 配置 Git 忽略文件

在仓库根目录创建 `.gitignore`：

```gitignore
# Obsidian
.obsidian/workspace
.obsidian/workspace-mobile.json
.obsidian/graph.json

# Node.js
node_modules/
npm-debug.log

# VitePress
docs/.vitepress/cache/
docs/.vitepress/dist/

# 系统文件
.DS_Store
Thumbs.db

# 私人笔记（按需配置）
private/
drafts/
```

### 8.2 配置自定义域名

如果使用子目录部署（如 `https://example.com/docs/`）：

```typescript
// docs/.vitepress/config.ts
export default defineConfig({
  base: '/docs/',
  // ...
})
```

1Panel 网站根目录设置为 `/www/docs`

### 8.3 启用搜索功能

VitePress 内置搜索，在配置中启用：

```typescript
// docs/.vitepress/config.ts
export default defineConfig({
  themeConfig: {
    search: {
      provider: 'local'
    }
  }
})
```

### 8.4 配置评论系统（可选）

使用 [Giscus](https://giscus.app/) 添加评论：

```typescript
// docs/.vitepress/config.ts
export default defineConfig({
  themeConfig: {
    giscus: {
      repo: 'username/my-notes-vitepress',
      repoId: 'YOUR_REPO_ID',
      category: 'Announcements',
      categoryId: 'YOUR_CATEGORY_ID',
      mapping: 'pathname',
    }
  }
})
```

---

## 参考资源

- [1Panel 官方文档](https://1panel.cn/docs/)
- [VitePress 官方文档](https://vitepress.dev/)
- [VitePress Theme Teek 官方文档](https://teek.w3c.cool/)
- [VitePress Theme Teek GitHub](https://github.com/Kele-Bingtang/vitepress-theme-teek)
- [Obsidian Git 插件](https://github.com/denolehov/obsidian-git)
- [OpenResty 文档](https://openresty.org/en/)
- [GitHub Actions 文档](https://docs.github.com/en/actions)

---

## 更新日志

| 日期 | 版本 | 说明 |
|------|------|------|
| 2026-02-10 | 2.1.1 | 完善 Webhook 方案，添加 Docker 访问宿主机目录说明 |
| 2026-02-10 | 2.1.0 | 添加双仓库方案（Git Submodule）分离内容和配置 |
| 2026-02-10 | 2.0.0 | 重构教程，添加 Obsidian + Git 自动提交工作流 |
| 2026-02-10 | 1.0.0 | 初始版本 |

---

> 💡 **核心提示**：
> - **单仓库**：适合小型项目，配置简单
> - **双仓库**：适合中大型项目，内容和配置分离，职责清晰
> - **Docker Webhook**：通过 `-v` 挂载宿主机目录，容器可访问 Git 仓库和脚本
> - 本地使用 Obsidian 写作，Git 插件自动同步到 GitHub
> - 服务器通过定时任务或 Webhook 检查更新，自动构建部署
> - 整个流程完全自动化，无需手动干预
>
> ⚠️ **注意**：首次配置需要仔细检查每一步，特别是 Git 权限、`base` 路径配置、Submodule 更新和 Docker 挂载
