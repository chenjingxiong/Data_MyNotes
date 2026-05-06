---
title: Wiki.js 部署与使用指南
date: 2026-05-06
tags:
  - wiki
  - documentation
  - docker
  - github
  - markdown
  - devops
  - knowledge-management
---

# Wiki.js 部署与使用指南

## 一、Wiki.js 概述

**Wiki.js** 是一款现代化、开源的 wiki 引擎，基于 **Node.js** 构建，以 **Git** 为核心版本控制，原生支持 **Markdown**。它拥有美观的界面、强大的权限系统和丰富的存储后端，是团队知识管理的优秀选择。

> **GitHub**: [requarks/wiki](https://github.com/requarks/wiki)
> **官网**: [js.wiki](https://js.wiki)
> **文档**: [docs.requarks.io](https://docs.requarks.io)
> **Stars**: 25k+ ⭐ | **License**: AGPL-3.0
> **技术栈**: Node.js | Vue.js | PostgreSQL / MySQL / SQLite / MSSQL

### 1.1 核心特性

| 特性 | 说明 |
|------|------|
| **多编辑器** | Markdown (可视化/源码)、WYSIWYG、HTML、API |
| **Git 版本控制** | 内置 Git 引擎，每次编辑自动产生版本记录 |
| **多存储后端** | 本地 Git、GitHub、GitLab、Bitbucket、Azure DevOps 等 |
| **权限管理** | 基于路径的精细化权限控制（页面级别） |
| **多认证方式** | 本地、LDAP/AD、OAuth2、SAML、GitHub、Google、Microsoft 等 |
| **多语言** | 支持 50+ 语言，含完整中文支持 |
| **搜索** | 内置全文搜索引擎（数据库 / Elasticsearch / Algolia） |
| **主题** | 可自定义 CSS，支持明暗模式 |
| **API** | 完整的 GraphQL API，支持自动化操作 |

### 1.2 版本现状

| 版本 | 状态 | 说明 |
|------|------|------|
| **v2.x** | ✅ 稳定版 | 当前主力版本，功能完善，社区广泛使用 |
| **v3.x (Heron)** | 🔧 开发中 | 重大架构重写，基于 Tauri 桌面同步，新后端架构 |

> ⚠️ 本指南基于 **v2.x** 稳定版撰写。v3 尚未正式发布，不建议生产环境使用。

---

## 二、Docker Compose 部署

### 2.1 架构说明

```
┌──────────────────────────────────────────────────┐
│              Wiki.js 部署架构                      │
├──────────────────────────────────────────────────┤
│                                                  │
│   用户浏览器 ──→ Nginx/Caddy (反代 + SSL)         │
│                     │                            │
│                     ▼                            │
│              Wiki.js 容器 (:3000)                │
│              │              │                     │
│              ▼              ▼                     │
│        PostgreSQL         Git Storage            │
│        (数据库)       (GitHub/本地仓库)            │
│                                                  │
└──────────────────────────────────────────────────┘
```

### 2.2 目录结构

```bash
wikijs/
├── docker-compose.yml
├── .env                      # 环境变量（密码等敏感信息）
├── config.yml                # Wiki.js 配置文件（可选）
├── data/                     # 持久化数据
│   ├── db/                   # PostgreSQL 数据卷
│   └── wiki/                 # Wiki.js 数据卷
└── nginx/                    # Nginx 配置（如需反代）
    └── nginx.conf
```

### 2.3 Docker Compose 配置

创建 `.env` 文件：

```bash
# .env
POSTGRES_DB=wiki
POSTGRES_USER=wikijs
POSTGRES_PASSWORD=ChangeMe_StrongPassword_2026
WIKI_ADMIN_EMAIL=admin@example.com
```

创建 `docker-compose.yml`：

```yaml
version: "3.8"

services:
  db:
    image: postgres:16-alpine
    container_name: wikijs-db
    environment:
      POSTGRES_DB: ${POSTGRES_DB:-wiki}
      POSTGRES_USER: ${POSTGRES_USER:-wikijs}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:-wikijsrocks}
    volumes:
      - ./data/db:/var/lib/postgresql/data
    restart: unless-stopped
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER:-wikijs}"]
      interval: 10s
      timeout: 5s
      retries: 5

  wiki:
    image: ghcr.io/requarks/wiki:2
    container_name: wikijs
    depends_on:
      db:
        condition: service_healthy
    environment:
      DB_TYPE: postgres
      DB_HOST: db
      DB_PORT: 5432
      DB_USER: ${POSTGRES_USER:-wikijs}
      DB_PASS: ${POSTGRES_PASSWORD:-wikijsrocks}
      DB_NAME: ${POSTGRES_DB:-wiki}
    ports:
      - "3000:3000"
    volumes:
      - ./data/wiki:/wiki/data
      - ./config.yml:/wiki/config.yml
    restart: unless-stopped
```

### 2.4 轻量化部署：SQLite（无需独立数据库）

> 📌 根据官方文档确认，Wiki.js **原生支持 SQLite**，无需部署 PostgreSQL / MySQL 即可运行。

Wiki.js 支持的数据库类型：`postgres`、`mysql`、`mariadb`、`mssql`、**`sqlite`**

对于个人博客、小型团队（<10人）或测试环境，使用 **SQLite** 可以大幅简化部署——只需一个 Docker 容器即可运行：

```yaml
# docker-compose.sqlite.yml — SQLite 轻量化部署

services:
  wiki:
    image: ghcr.io/requarks/wiki:2
    container_name: wikijs
    environment:
      DB_TYPE: sqlite
      DB_FILEPATH: /wiki/data/db.sqlite
    ports:
      - "3000:3000"
    volumes:
      - ./data/wiki:/wiki/data
      - ./config.yml:/wiki/config.yml
    restart: unless-stopped
```

或使用单条 `docker run` 命令：

```bash
docker run -d -p 3000:3000 --name wiki \
  -v ${PWD}/data:/wiki/data \
  -e "DB_TYPE=sqlite" \
  -e "DB_FILEPATH=/wiki/data/db.sqlite" \
  ghcr.io/requarks/wiki:2
```

**资源占用**：

| 指标 | 值 |
|------|-----|
| RAM 常驻 | ~70 MB |
| RAM 最低要求 | 1 GB（Linux） |
| CPU | 单核即可运行 |
| 存储 | 文本为主时仅数 MB |
| 容器数量 | **1 个**（无需额外数据库容器） |

**SQLite vs PostgreSQL 对比**：

| 维度 | SQLite | PostgreSQL |
|------|--------|------------|
| 部署复杂度 | ⭐ 极简（1 个容器） | ⭐⭐⭐ 需额外数据库 |
| 并发写入 | ❌ 单写者（不适合高并发编辑） | ✅ 多用户并发写入 |
| 高可用 (HA) | ❌ 不支持 | ✅ 支持 |
| 搜索性能 | 中等（<1000 页面足够） | 高（配合 pg_trgm） |
| 备份 | 复制 `.sqlite` 文件即可 | `pg_dump` 命令 |
| 生产环境 | ⚠️ 官方**不推荐**用于生产 | ✅ 官方推荐 |
| v3 升级 | ❌ v3 将**不支持** SQLite | ✅ v3 推荐引擎 |

> ⚠️ **重要**：官方明确表示 **SQLite 不推荐用于生产环境**，且 **v3 将不再支持 SQLite**。如果你计划未来升级到 v3，请直接使用 PostgreSQL。
>
> 💡 **适用场景**：个人笔记、文档预览、CI/CD 中的文档站点、小型团队（<10 人）、资源受限环境（树莓派等 ARM64 设备）

### 2.5 可选：config.yml 配置文件

```yaml
# config.yml - Wiki.js 高级配置

# 数据库连接（已通过环境变量配置时可省略）
# db:
#   type: postgres
#   host: db
#   port: 5432
#   user: wikijs
#   pass: wikijsrocks
#   db: wiki

# 日志级别
logLevel: info

# 离线模式（禁止外部 CDN）
offline: false

# HA 模式（多实例部署时开启）
# ha: true
```

### 2.6 启动与初始化

```bash
# 创建目录
mkdir -p wikijs/data/db wikijs/data/wiki

# 启动服务
cd wikijs
docker compose up -d

# 查看日志
docker compose logs -f wiki

# 等待约 30 秒后访问
# http://localhost:3000
```

首次访问会进入 **安装向导**：

| 步骤 | 说明 |
|------|------|
| 1. 系统检查 | 自动检查环境依赖 |
| 2. 管理员账户 | 设置管理员邮箱和密码 |
| 3. 站点配置 | 设置站点名称、URL、语言等 |
| 4. 完成 | 进入管理后台 |

### 2.7 生产环境：添加 Nginx 反向代理

```nginx
# nginx/wikijs.conf

server {
    listen 80;
    server_name wiki.yourdomain.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name wiki.yourdomain.com;

    ssl_certificate     /etc/nginx/ssl/fullchain.pem;
    ssl_certificate_key /etc/nginx/ssl/privkey.pem;

    # SSL 优化
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;

    # 安全头
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;

    location / {
        proxy_pass http://wikijs:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_read_timeout 86400;
    }

    # 上传文件大小限制
    client_max_body_size 50M;
}
```

将 Nginx 加入 `docker-compose.yml`：

```yaml
  nginx:
    image: nginx:alpine
    container_name: wikijs-nginx
    depends_on:
      - wiki
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/wikijs.conf:/etc/nginx/conf.d/default.conf:ro
      - ./nginx/ssl:/etc/nginx/ssl:ro
    restart: unless-stopped
```

---

## 三、GitHub Markdown 仓库集成

Wiki.js 的一大亮点是可以将 GitHub 仓库中的 Markdown 文件直接同步为 Wiki 页面。这意味着你可以用熟悉的 Git 工作流来管理知识库。

### 3.1 工作原理

```
┌──────────────────────────────────────────────────────┐
│            Wiki.js Git Storage 同步流程                │
├──────────────────────────────────────────────────────┤
│                                                      │
│   GitHub 仓库 (docs/)                                │
│   ├── README.md  ──────────→  首页                   │
│   ├── getting-started/                                │
│   │   ├── install.md ────→  getting-started/install   │
│   │   └── config.md  ────→  getting-started/config    │
│   ├── api/                                            │
│   │   ├── auth.md   ────→  api/auth                   │
│   │   └── users.md  ────→  api/users                  │
│   └── _assets/                                        │
│       └── logo.png   ────→  _assets/logo (资源文件)    │
│                                                      │
│   Wiki.js 定时拉取 → 自动创建/更新对应页面             │
│                                                      │
└──────────────────────────────────────────────────────┘
```

### 3.2 GitHub 准备工作

#### 3.2.1 创建 Personal Access Token

1. 登录 GitHub → Settings → Developer settings → **Personal access tokens** → **Fine-grained tokens**
2. 点击 **Generate new token**
3. 配置权限：

| 权限 | 范围 | 说明 |
|------|------|------|
| Contents | Read-only | 拉取仓库内容 |
| Contents | Read and write | 如需双向同步（Wiki → GitHub） |
| Metadata | Read-only | 仓库元数据（自动授予） |

4. 复制生成的 Token（只显示一次）

#### 3.2.2 仓库结构规范

建议的文档仓库结构：

```bash
your-docs-repo/
├── README.md                    # 对应 Wiki 首页
├── sidebar.md                   # 侧边栏导航（可选）
├── home.md                      # 首页内容（如果不用 README.md）
├── _footer.md                   # 页脚（可选）
├── getting-started/
│   ├── installation.md
│   ├── configuration.md
│   └── quick-start.md
├── guides/
│   ├── permissions.md
│   ├── authentication.md
│   └── git-sync.md
├── api/
│   ├── overview.md
│   └── graphql.md
├── _assets/                     # 资源文件夹
│   ├── images/
│   │   ├── architecture.png
│   │   └── diagram.svg
│   └── files/
│       └── template.xlsx
└── .wiki/                       # Wiki.js 特殊配置（可选）
    └── sidebar.yml
```

> **命名规则**：文件夹名和文件名中的空格会被转为 `-`，文件名即为页面标题。

### 3.3 Wiki.js 中配置 Git Storage

#### 步骤 1：进入管理后台

导航到 **Administration** → **Storage** (或 **Modules** → **Storage**)

#### 步骤 2：启用 Git Storage

| 配置项 | 值 | 说明 |
|--------|-----|------|
| **Active** | ✅ 开启 | 激活 Git 存储模块 |
| **Storage Target** | Git | 选择 Git 作为存储后端 |

#### 步骤 3：配置 Git 连接

| 配置项 | 示例值 | 说明 |
|--------|--------|------|
| **Repository URL** | `https://github.com/your-org/your-docs-repo.git` | GitHub 仓库 HTTPS 地址 |
| **Branch** | `main` | 同步的分支 |
| **Authentication** | Token / SSH | 推荐 Token |
| **Access Token** | `ghp_xxxxxxxxxxxx` | 3.2.1 步骤创建的 Token |
| **Sync Direction** | 选择以下之一 | 见下方说明 |
| **Path** | `/` | 仓库根目录，或子目录如 `/docs` |

#### 步骤 4：选择同步方向

| 模式 | 说明 | 适用场景 |
|------|------|---------|
| **Git → Wiki** | 单向拉取，GitHub 为主 | 文档仓库是唯一真相源，Wiki.js 只做展示 |
| **Wiki → Git** | 单向推送，Wiki 为主 | 团队通过 Wiki.js 编辑，Git 做备份 |
| **双向同步** | 两个方向同步 | 需要两边都能编辑（⚠️ 冲突风险较高） |

> 💡 **推荐**：大多数场景使用 **Git → Wiki（单向拉取）**，让 Git 仓库作为文档的唯一真相源（Single Source of Truth），配合 CI/CD 流水线自动更新。

#### 步骤 5：配置自动同步

| 配置项 | 推荐值 | 说明 |
|--------|--------|------|
| **Sync Mode** | Auto | 自动定时同步 |
| **Sync Interval** | 5 minutes | 同步频率（可选 1min / 5min / 15min / 30min / 1h / 6h / 12h / 24h） |
| **Verify SSL** | ✅ | 验证 SSL 证书 |

### 3.4 GitHub Webhook 实时同步（推荐）

除了定时拉取，还可以配置 Webhook 实现 push 后立即同步：

1. 在 Wiki.js 中：**Administration** → **Storage** → **Git** → 记下 **Webhook URL**（形如 `https://wiki.yourdomain.com/h/git`)
2. 在 GitHub 仓库中：**Settings** → **Webhooks** → **Add webhook**
3. 配置：

| 配置项 | 值 |
|--------|-----|
| Payload URL | `https://wiki.yourdomain.com/h/git` |
| Content type | `application/json` |
| Secret | 留空或设置后填入 Wiki.js |
| Events | **Just the push event** |
| Active | ✅ |

配置完成后，每次 `git push` 到 GitHub，Wiki.js 将在数秒内自动更新。

### 3.5 Markdown 文件规范

为了让 GitHub 上的 Markdown 文件在 Wiki.js 中正确渲染，需要注意：

```markdown
---
title: 自定义页面标题
description: 页面描述（用于 SEO）
tags:
  - guide
  - setup
published: true
date: 2026-05-06
---

# 页面标题

正文内容...

## 二级标题

内容段落...

![图片描述](/_assets/images/screenshot.png)

[链接文本](/getting-started/installation)
```

| 注意事项 | 说明 |
|----------|------|
| **Frontmatter** | Wiki.js 识别 YAML frontmatter 中的 `title`、`description`、`tags` 等字段 |
| **图片路径** | 使用仓库相对路径，Wiki.js 会自动映射 `_assets` 目录 |
| **内部链接** | 使用 Wiki.js 的页面路径格式，而非 `.md` 后缀 |
| **文件编码** | 必须是 **UTF-8** |

---

## 四、权限设置

### 4.1 权限模型概述

```
┌───────────────────────────────────────────────────┐
│             Wiki.js 权限模型                        │
├───────────────────────────────────────────────────┤
│                                                   │
│   用户 (User)                                     │
│     └── 属于 → 组 (Group)                         │
│              └── 拥有 → 权限规则 (Permission)      │
│                       └── 作用于 → 路径 (Path)     │
│                                                   │
│   层级：                                          │
│   全局权限 → 路径权限 → 页面权限                    │
│   (高优先级)              (低优先级，可覆盖)         │
│                                                   │
└───────────────────────────────────────────────────┘
```

### 4.2 预定义组

| 组名 | 说明 | 默认权限 |
|------|------|---------|
| **Administrators** | 系统管理员 | 所有权限，含管理后台访问 |
| **Guests** | 未登录访客 | 默认无权限（可配置只读） |
| **All Users** | 所有已登录用户 | 继承全局权限设置 |

### 4.3 创建自定义组

导航到 **Administration** → **Groups** → **New Group**

建议的组结构：

| 组名 | 用途 | 建议权限范围 |
|------|------|-------------|
| `editors` | 内容编辑者 | 指定路径下读写 |
| `reviewers` | 审核人员 | 读写 + 管理页面 |
| `developers` | 开发文档维护 | 技术文档区域读写 |
| `viewers` | 只读用户 | 全站只读 |
| `external` | 外部协作者 | 限定路径只读 |

### 4.4 权限类型详解

导航到 **Administration** → **Permissions**

| 权限 | 说明 | 适用场景 |
|------|------|---------|
| **Read Pages** | 查看页面内容 | 所有需要看到内容的组 |
| **Write Pages** | 创建和编辑页面 | 编辑者、开发者 |
| **Manage Pages** | 删除、移动、管理页面 | 审核人员、管理员 |
| **Read Assets** | 访问上传的资源文件 | 所有需要看到图片/文件的用户 |
| **Write Assets** | 上传新资源文件 | 编辑者、开发者 |
| **Manage Assets** | 删除或修改资源文件 | 管理员 |
| **Read Comments** | 查看评论 | 所有用户 |
| **Write Comments** | 发表评论 | 已登录用户 |
| **Manage Comments** | 审核和删除评论 | 管理员、审核人员 |
| **View History** | 查看页面修订历史 | 编辑者以上 |
| **View Source** | 查看原始 Markdown 源码 | 开发者 |
| **Admin: Pages** | 管理后台 - 页面管理 | 仅管理员 |
| **Admin: Users** | 管理后台 - 用户管理 | 仅管理员 |
| **Admin: System** | 管理后台 - 系统设置 | 仅管理员 |
| **Admin: Theme** | 管理后台 - 主题设置 | 仅管理员 |
| **Admin: API** | 管理 API Keys | 仅管理员 |

### 4.5 路径权限配置

权限按路径层级继承，越具体的路径优先级越高：

```
/                        ← 全局权限（影响所有页面）
├── /engineering         ← 工程团队区域
│   ├── /engineering/backend    ← 后端文档（可单独设权限）
│   └── /engineering/frontend   ← 前端文档
├── /product             ← 产品区域
│   └── /product/roadmap ← 产品路线图（可设为仅管理层可见）
├── /hr                  ← 人事区域（可设为仅 HR 组可见）
└── /public              ← 公开区域（所有人均可读）
```

**配置示例**：

| 路径 | 组 | 权限 |
|------|-----|------|
| `/` | `viewers` | Read Pages, Read Assets |
| `/` | `editors` | Read/Write Pages, Read/Write Assets, View History |
| `/engineering` | `developers` | Read/Write Pages, Read/Write Assets |
| `/engineering` | `viewers` | Read Pages（覆盖全局设置） |
| `/hr` | `editors` | ❌ 无权限（覆盖全局，禁止访问） |
| `/hr` | `hr-team` | 全部权限 |
| `/product/roadmap` | `management` | Read/Write Pages |
| `/product/roadmap` | `all-users` | ❌ 无权限 |

### 4.6 页面级权限覆盖

单个页面可以覆盖组权限：

1. 打开目标页面
2. 点击 **Page Actions** (右上角 `⋮` 菜单) → **Page Permissions**
3. 添加特定组的权限规则

```
全局权限: editors → / → Read/Write ✅
           ↓ 继承
路径权限: editors → /hr → ❌ None (覆盖全局)
           ↓ 继承
页面权限: editors → /hr/onboarding → Read ✅ (覆盖路径权限)
```

### 4.7 最佳实践

| 实践 | 说明 |
|------|------|
| **组优于个人** | 始终通过组分配权限，避免给单个用户直接授权 |
| **最小权限原则** | 只授予完成任务所需的最少权限 |
| **路径分层** | 按组织结构/项目划分顶级路径，每个路径设独立权限 |
| **审计日志** | 定期检查 **Administration → Logs** 了解权限变更 |
| **测试权限** | 创建测试用户验证权限配置是否正确 |
| **文档化** | 记录权限矩阵，便于团队理解访问规则 |

### 4.8 权限矩阵模板

```
┌──────────────────┬─────────┬──────────┬────────────┬───────────┬──────────┐
│ 路径 \ 组        │ admins  │ editors  │ developers │ viewers   │ external │
├──────────────────┼─────────┼──────────┼────────────┼───────────┼──────────┤
│ /                │ CRUD    │ CR       │ R          │ R         │ ❌       │
│ /engineering     │ CRUD    │ CR       │ CR         │ R         │ ❌       │
│ /product         │ CRUD    │ CR       │ R          │ R         │ R        │
│ /product/roadmap │ CRUD    │ CR       │ ❌         │ ❌        │ ❌       │
│ /hr              │ CRUD    │ ❌       │ ❌         │ ❌        │ ❌       │
│ /public          │ CRUD    │ CR       │ CR         │ R         │ R        │
└──────────────────┴─────────┴──────────┴────────────┴───────────┴──────────┘
C=创建 R=读取 U=更新 D=删除 ❌=无权限
```

---

## 五、认证集成

### 5.1 支持的认证方式

| 认证方式 | 适用场景 | 配置复杂度 |
|----------|---------|-----------|
| **本地认证** | 小团队、测试环境 | ⭐ 简单 |
| **GitHub** | 开发团队 | ⭐⭐ 中等 |
| **Google** | 使用 Google Workspace 的团队 | ⭐⭐ 中等 |
| **Microsoft** | 使用 Microsoft 365 的团队 | ⭐⭐ 中等 |
| **LDAP / Active Directory** | 企业内网 | ⭐⭐⭐ 较复杂 |
| **OAuth2 通用** | 自有认证服务 | ⭐⭐⭐ 较复杂 |
| **SAML 2.0** | 企业 SSO | ⭐⭐⭐⭐ 复杂 |
| **Keycloak** | 开源 SSO 方案 | ⭐⭐⭐ 较复杂 |

### 5.2 GitHub OAuth 认证配置

#### 步骤 1：在 GitHub 创建 OAuth App

1. GitHub → Settings → Developer settings → **OAuth Apps** → **New OAuth App**
2. 配置：

| 字段 | 值 |
|------|-----|
| Application name | `Wiki.js` |
| Homepage URL | `https://wiki.yourdomain.com` |
| Authorization callback URL | `https://wiki.yourdomain.com/login/github/callback` |
3. 记录 **Client ID** 和 **Client Secret**

#### 步骤 2：在 Wiki.js 中配置

导航到 **Administration** → **Authentication** → **GitHub**

| 配置项 | 值 |
|--------|-----|
| Client ID | 从 GitHub 复制 |
| Client Secret | 从 GitHub 复制 |
| Auto-create users | ✅（首次登录自动创建账户） |
| Map to group | 可选：指定自动加入的组 |

### 5.3 LDAP / Active Directory 配置

导航到 **Administration** → **Authentication** → **LDAP / Active Directory**

```yaml
# 关键配置参数

# LDAP 服务器
host: ldap.yourdomain.com
port: 389                  # 或 636 (SSL)

# 绑定账户（用于搜索 LDAP 目录）
bindDN: "CN=wikijs,OU=ServiceAccounts,DC=yourdomain,DC=com"
bindCredentials: "service_account_password"

# 搜索基础
searchBase: "OU=Users,DC=yourdomain,DC=com"
searchFilter: "(sAMAccountName={{username}})"

# 字段映射
mapping:
  displayName: "displayName"
  email: "mail"
  avatar: "thumbnailPhoto"

# 组映射（可选）
groupMapping:
  baseDN: "OU=Groups,DC=yourdomain,DC=com"
  filter: "(member={{dn}})"
  map:
    "CN=WikiAdmins,OU=Groups,DC=yourdomain,DC=com": "Administrators"
    "CN=WikiEditors,OU=Groups,DC=yourdomain,DC=com": "editors"
    "CN=WikiViewers,OU=Groups,DC=yourdomain,DC=com": "viewers"
```

---

## 六、搜索配置

### 6.1 搜索引擎选择

| 引擎 | 适用场景 | 性能 | 配置复杂度 |
|------|---------|------|-----------|
| **Database** (内置) | 小型 Wiki（<1000 页面） | 中等 | ⭐ 零配置 |
| **Elasticsearch** | 中大型（1000+ 页面） | 高 | ⭐⭐⭐ |
| **Algolia** | 对搜索体验要求高 | 极高 | ⭐⭐ |

### 6.2 Elasticsearch 配置（推荐）

导航到 **Administration** → **Search** → **Elasticsearch**

| 配置项 | 值 | 说明 |
|--------|-----|------|
| Host | `http://elasticsearch:9200` | ES 服务地址 |
| Index Prefix | `wiki` | 索引名前缀 |
| Authentication | 可选 | 如 ES 需要认证 |

在 `docker-compose.yml` 中添加 Elasticsearch：

```yaml
  elasticsearch:
    image: elasticsearch:8.12.0
    container_name: wikijs-es
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    volumes:
      - ./data/es:/usr/share/elasticsearch/data
    restart: unless-stopped
```

---

## 七、运维与备份

### 7.1 数据备份策略

```bash
#!/bin/bash
# backup-wikijs.sh

BACKUP_DIR="/backups/wikijs/$(date +%Y%m%d_%H%M%S)"
mkdir -p "$BACKUP_DIR"

# 1. PostgreSQL 数据库备份
docker exec wikijs-db pg_dump -U wikijs wiki > "$BACKUP_DIR/wiki_db.sql"

# 2. 压缩数据库备份
gzip "$BACKUP_DIR/wiki_db.sql"

# 3. 备份 Wiki.js 数据卷
tar czf "$BACKUP_DIR/wiki_data.tar.gz" -C ./data/wiki .

# 4. 备份配置文件
cp docker-compose.yml .env config.yml "$BACKUP_DIR/"

echo "✅ 备份完成: $BACKUP_DIR"
```

### 7.2 定时备份（Crontab）

```bash
# 每天凌晨 2 点自动备份
0 2 * * * /path/to/backup-wikijs.sh >> /var/log/wikijs-backup.log 2>&1
```

### 7.3 恢复流程

```bash
# 1. 停止服务
docker compose down

# 2. 恢复数据库
gunzip < /backups/wikijs/YYYYMMDD_HHMMSS/wiki_db.sql.gz | \
  docker exec -i wikijs-db psql -U wikijs wiki

# 3. 恢复数据卷
tar xzf /backups/wikijs/YYYYMMDD_HHMMSS/wiki_data.tar.gz -C ./data/wiki

# 4. 启动服务
docker compose up -d
```

### 7.4 升级流程

```bash
# 1. 备份（先！）
./backup-wikijs.sh

# 2. 拉取新版本
docker compose pull wiki

# 3. 重启（Wiki.js 会自动处理数据库迁移）
docker compose up -d wiki

# 4. 验证
docker compose logs -f wiki
```

### 7.5 常用运维命令

```bash
# 查看容器状态
docker compose ps

# 查看 Wiki.js 日志
docker compose logs -f --tail=100 wiki

# 查看数据库日志
docker compose logs -f --tail=50 db

# 进入 Wiki.js 容器
docker exec -it wikijs sh

# 数据库连接
docker exec -it wikijs-db psql -U wikijs wiki

# 重建并启动（配置变更后）
docker compose up -d --build --force-recreate

# 清理旧镜像
docker image prune -f
```

---

## 八、常用配置速查

### 8.1 管理后台导航

| 路径 | 功能 |
|------|------|
| Administration → General | 站点名称、URL、语言、时区 |
| Administration → Users | 用户管理（创建/禁用/删除） |
| Administration → Groups | 组管理（创建/权限分配） |
| Administration → Permissions | 全局路径权限配置 |
| Administration → Authentication | 认证方式配置 |
| Administration → Storage | Git/其他存储后端配置 |
| Administration → Search | 搜索引擎配置 |
| Administration → Modules | 功能模块管理 |
| Administration → Logging | 系统日志查看 |
| Administration → Theme | 外观和 CSS 自定义 |
| Administration → Mail | 邮件服务配置（用于通知） |

### 8.2 环境变量速查

| 变量 | 说明 | 示例 |
|------|------|------|
| `DB_TYPE` | 数据库类型 | `postgres` / `mysql` / `sqlite` / `mssql` |
| `DB_HOST` | 数据库主机 | `db` |
| `DB_PORT` | 数据库端口 | `5432` |
| `DB_USER` | 数据库用户 | `wikijs` |
| `DB_PASS` | 数据库密码 | `yourpassword` |
| `DB_NAME` | 数据库名 | `wiki` |

### 8.3 端口速查

| 端口 | 服务 | 说明 |
|------|------|------|
| 3000 | Wiki.js HTTP | 默认 Web 端口 |
| 3443 | Wiki.js HTTPS | 自带 SSL 端口（不推荐，建议用 Nginx） |
| 5432 | PostgreSQL | 数据库端口（仅容器内部） |
| 80/443 | Nginx | 反向代理 SSL 终止 |

---

## 九、常见问题排查

### 9.1 页面不更新（Git 同步失败）

```bash
# 检查 Git Storage 状态
Administration → Storage → Git → 查看同步日志

# 手动触发同步
Administration → Storage → Git → "Sync Now"

# 检查 Webhook 是否正常
GitHub → 仓库 → Settings → Webhooks → 查看最近投递记录
```

常见原因：
- Token 过期 → 重新生成并更新
- 仓库路径配置错误 → 检查 Path 和 Branch
- 网络连通性问题 → 检查容器能否访问 GitHub

### 9.2 权限不生效

- 检查用户是否在正确的组中
- 检查路径权限是否被更具体的子路径覆盖
- 检查页面级权限是否覆盖了路径权限
- 使用 **Administration → Users → 选中用户 → Permissions** 查看有效权限

### 9.3 数据库连接失败

```bash
# 检查数据库是否就绪
docker exec wikijs-db pg_isready -U wikijs

# 检查网络连通性
docker exec wikijs ping -c 3 db

# 重启数据库
docker compose restart db

# 等待健康检查通过后重启 Wiki.js
docker compose restart wiki
```

---

## 十、生产环境 Checklist

```
□ 数据库密码已修改为强密码
□ 已配置反向代理 (Nginx/Caddy) + SSL
□ 已配置认证方式 (GitHub OAuth / LDAP / SSO)
□ 已配置 Git Storage 同步 + Webhook
□ 已创建自定义组并分配权限
□ 已配置搜索后端 (Elasticsearch)
□ 已配置邮件服务 (用于通知和密码重置)
□ 已设置定时备份脚本
□ 已测试恢复流程
□ 已限制管理后台访问 (仅 Administrators 组)
□ 已配置日志级别和监控
□ 已设置 `client_max_body_size` (上传限制)
```

---

## 参考链接

- [Wiki.js 官方文档](https://docs.requarks.io)
- [Wiki.js GitHub](https://github.com/requarks/wiki)
- [Wiki.js Docker 安装指南](https://docs.requarks.io/install/docker)
- [Wiki.js Git Storage 配置](https://docs.requarks.io/storage/git)
- [Wiki.js 权限系统](https://docs.requarks.io/groups)
- [Wiki.js 认证配置](https://docs.requarks.io/auth)
- [Docker Hub - requarks/wiki](https://hub.docker.com/r/requarks/wiki)
