# Hugo 技术博客搭建完全指南

> 本指南涵盖使用 Hugo 搭建技术博客的完整流程，包括 Docker/1Panel 部署、多 Git 仓库内容自动同步、SSL 证书配置和 CI/CD 自动化。

---

## 📋 目录

- [一、Hugo 简介](#一hugo-简介)
- [二、准备工作](#二准备工作)
- [三、Docker 部署 Hugo](#三docker-部署-hugo)
- [四、1Panel 部署方案](#四1panel-部署方案)
- [五、多仓库内容同步](#五多仓库内容同步)
- [六、Nginx 反向代理配置](#六nginx-反向代理配置)
- [七、SSL 证书配置](#七ssl-证书配置)
- [八、自动化部署脚本](#八自动化部署脚本)
- [九、常见问题排查](#九常见问题排查)

---

## 一、Hugo 简介

### 1.1 什么是 Hugo

Hugo 是由 Go 语言编写的静态网站生成器，具有以下特点：

- **极速构建**：毫秒级构建速度
- **零依赖**：单个二进制文件即可运行
- **丰富主题**：大量现成主题可供选择
- **Markdown 支持**：原生支持 Markdown 写作
- **灵活组织**：支持多内容源、分类、标签

### 1.2 多仓库内容同步架构

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│   主内容仓库    │     │   笔记仓库      │     │   资源仓库      │
│  (blog-content) │     │  (my-notes)     │     │  (resources)    │
└────────┬────────┘     └────────┬────────┘     └────────┬────────┘
         │                       │                       │
         └───────────────────────┼───────────────────────┘
                                 │
                    ┌────────────▼────────────┐
                    │   Git Submodule/Git     │
                    │   同步脚本 (Git Sync)   │
                    └────────────┬────────────┘
                                 │
                    ┌────────────▼────────────┐
                    │   Hugo 容器             │
                    │   内容目录: /content    │
                    └────────────┬────────────┘
                                 │
                    ┌────────────▼────────────┐
                    │   静态 HTML 输出        │
                    │   目录: /public         │
                    └────────────┬────────────┘
                                 │
                    ┌────────────▼────────────┐
                    │   Nginx 提供 Web 服务   │
                    └─────────────────────────┘
```

### 1.3 推荐技术栈

| 组件 | 推荐方案 | 说明 |
|------|----------|------|
| 容器 | Docker + Docker Compose | 便于管理和扩展 |
| 面板 | 1Panel | 可视化管理，内置 Nginx |
| Web 服务器 | Nginx | 高性能静态文件服务 |
| SSL | Let's Encrypt | 免费证书，自动续期 |
| 同步方式 | Git Submodule / 脚本 | 多仓库内容聚合 |

---

## 二、准备工作

### 2.1 域名与服务器准备

```bash
# 确保域名已解析到服务器
ping blog.yourdomain.com

# 检查服务器环境（参考 VPS 初始化指南）
# 已安装：Docker、1Panel、Nginx
```

### 2.2 目录结构规划

```bash
# 创建项目目录
mkdir -p /opt/hugo-blog
cd /opt/hugo-blog

# 目录结构
/opt/hugo-blog/
├── docker-compose.yml      # Docker 编排文件
├── config/                 # 配置文件
│   └── nginx/              # Nginx 配置
├── data/                   # 数据持久化
│   ├── content/            # 主内容目录
│   │   ├── posts/          # 文章
│   │   ├── notes/          # 笔记（同步）
│   │   └── resources/      # 资源（同步）
│   ├── themes/             # 主题
│   └── public/             # 生成的静态文件
├── scripts/                # 同步脚本
│   └── sync-repos.sh
└── .env                    # 环境变量
```

### 2.3 创建项目目录

```bash
# 创建完整目录结构
mkdir -p /opt/hugo-blog/{config/nginx,data/{content/{posts,notes,resources},themes,public},scripts}

# 设置权限
chown -R 1000:1000 /opt/hugo-blog/data
```

---

## 三、Docker 部署 Hugo

### 3.1 使用官方镜像创建站点

```bash
cd /opt/hugo-blog

# 使用 Hugo 官方镜像创建新站点
docker run --rm -v $(pwd)/data:/src \
    hugomods/hugo:extended-0.139.4 \
    hugo new site .

# 查看生成的结构
ls -la data/
```

### 3.2 安装主题

```bash
# 方式1：Git Clone 主题（推荐）
cd /opt/hugo-blog/data/themes
git clone https://github.com/thegeeklab/hugo-geekdoc.git

# 方式2：添加为 submodule
cd /opt/hugo-blog/data
git submodule add https://github.com/thegeeklab/hugo-geekdoc.git themes/hugo-geekdoc
```

### 3.3 配置 Hugo

创建 `/opt/hugo-blog/data/hugo.toml`：

```toml
baseURL = "https://blog.yourdomain.com/"
languageCode = "zh-cn"
defaultContentLanguage = "zh"
title = "我的技术博客"
theme = "hugo-geekdoc"

# 构建
disableKinds = ["taxonomy", "term"]
enableGitInfo = true
enableEmoji = true

# 菜单
[menu]
  [[menu.main]]
    name = "首页"
    url = "/"
    weight = 1

  [[menu.main]]
    name = "文章"
    url = "/posts/"
    weight = 2

  [[menu.main]]
    name = "笔记"
    url = "/notes/"
    weight = 3

  [[menu.main]]
    name = "关于"
    url = "/about/"
    weight = 4

# Markdown 配置
[markup]
  [markup.goldmark]
    [markup.goldmark.renderer]
      unsafe = true  # 允许 HTML

  [markup.highlight]
    style = "github"
    lineNos = true
    lineNumbersInTable = true
    noClasses = false

# 输出格式
[outputs]
  home = ["HTML"]
  section = ["HTML"]
  page = ["HTML"]

# 参数
[params]
  description = "技术博客，记录学习与思考"
  images = ["images/logo.png"]
```

### 3.4 创建 Docker Compose 配置

创建 `/opt/hugo-blog/docker-compose.yml`：

```yaml
version: '3.8'

services:
  hugo:
    image: hugomods/hugo:extended-0.139.4
    container_name: hugo-blog
    restart: unless-stopped
    working_dir: /src
    volumes:
      - ./data:/src
      - ./scripts:/scripts
    environment:
      - HUGO_ENV=production
    command: >
      sh -c "
        while true; do
          hugo --cleanDestinationDir
          sleep 300
        done
      "
    networks:
      - blog-network

  # Nginx 容器（如果不用 1Panel）
  nginx:
    image: nginx:alpine
    container_name: hugo-nginx
    restart: unless-stopped
    ports:
      - "8080:80"
    volumes:
      - ./data/public:/usr/share/nginx/html:ro
      - ./config/nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./config/nginx/conf.d:/etc/nginx/conf.d:ro
    depends_on:
      - hugo
    networks:
      - blog-network

  # Git 同步容器
  git-sync:
    image: alpine/git:latest
    container_name: hugo-git-sync
    restart: unless-stopped
    volumes:
      - ./data/content:/content
      - ./scripts:/scripts
    command: >
      sh -c "
        apk add --no-cache openssh-client &&
        chmod +x /scripts/sync-repos.sh &&
        while true; do
          /scripts/sync-repos.sh
          sleep 600
        done
      "
    environment:
      - GIT_SSH_COMMAND=ssh -o StrictHostKeyChecking=no
    networks:
      - blog-network

networks:
  blog-network:
    driver: bridge
```

### 3.5 启动服务

```bash
cd /opt/hugo-blog

# 启动服务
docker-compose up -d

# 查看日志
docker-compose logs -f hugo

# 检查生成的文件
ls -la data/public/
```

---

## 四、1Panel 部署方案

### 4.1 在 1Panel 中创建容器

1. **登录 1Panel 面板**

2. **容器 → 创建容器**

配置 Hugo 容器：

| 配置项  | 值                                |
| ---- | -------------------------------- |
| 镜像   | `hugomods/hugo:extended-0.139.4` |
| 容器名  | `hugo-blog`                      |
| 重启策略 | `unless-stopped`                 |
| 工作目录 | `/src`                           |

3. **挂载卷**

| 容器路径 | 主机路径 |
|----------|----------|
| `/src` | `/opt/hugo-blog/data` |
| `/scripts` | `/opt/hugo-blog/scripts` |

4. **命令**

```bash
sh -c "hugo server --bind 0.0.0.0 --port 1313 --buildDrafts --cleanDestinationDir"
```

### 4.2 创建网站

1. **网站 → 创建网站 → 反向代理**

| 配置项 | 值 |
|--------|-----|
| 主域名 | `blog.yourdomain.com` |
| 代理地址 | `http://hugo-blog:1313` |
| 开启 WebSocket | 否 |

2. **配置 Nginx**

在 1Panel 中编辑网站配置：

```nginx
location / {
    proxy_pass http://hugo-blog:1313;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;

    # 静态文件缓存
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2|ttf|eot)$ {
        proxy_pass http://hugo-blog:1313;
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
}
```

### 4.3 配置定时任务（自动构建）

1. **计划任务 → 创建任务**

| 配置项  | 值                                                  |
| ---- | -------------------------------------------------- |
| 任务名称 | Hugo 自动构建                                          |
| 执行周期 | `*/5 * * * *` (每5分钟)                               |
| 命令   | `docker exec hugo-blog hugo --cleanDestinationDir` |
| 状态   | 启用                                                 |

---

## 五、多仓库内容同步

### 5.1 方式一：Git Submodule（推荐）

```bash
cd /opt/hugo-blog/data

# 添加内容子模块
git submodule add https://github.com/yourusername/notes.git content/notes
git submodule add https://github.com/yourusername/resources.git content/resources

# 更新子模块
git submodule update --remote --merge

# 查看子模块状态
git submodule status
```

### 5.2 方式二：同步脚本

创建 `/opt/hugo-blog/scripts/sync-repos.sh`：

```bash
#!/bin/bash

################################################################################
# Git 仓库同步脚本
# 功能：从多个 Git 仓库同步 Markdown 内容到 Hugo
################################################################################

set -e

# 颜色输出
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m'

log_info() {
    echo -e "${GREEN}[INFO]${NC} $(date '+%Y-%m-%d %H:%M:%S') - $1"
}

log_warn() {
    echo -e "${YELLOW}[WARN]${NC} $(date '+%Y-%m-%d %H:%M:%S') - $1"
}

log_error() {
    echo -e "${RED}[ERROR]${NC} $(date '+%Y-%m-%d %H:%M:%S') - $1"
}

################################################################################
# 配置区域
################################################################################

# 主内容目录
CONTENT_DIR="/content"

# 仓库配置：格式 "目标目录|仓库URL|分支"
REPOS=(
    "posts|https://github.com/yourusername/blog-posts.git|main"
    "notes|https://github.com/yourusername/notes.git|main"
    "resources|https://github.com/yourusername/resources.git|main"
)

# Git 配置
GIT_CONFIG_GLOBAL="--git-dir=/git/config --work-tree=/git/worktree"

################################################################################
# 同步函数
################################################################################

sync_repo() {
    local target_dir="$1"
    local repo_url="$2"
    local branch="${3:-main}"

    local full_path="${CONTENT_DIR}/${target_dir}"

    log_info "开始同步: ${target_dir}"

    # 创建目标目录
    mkdir -p "${full_path}"

    # 检查是否已存在 Git 仓库
    if [ -d "${full_path}/.git" ]; then
        log_info "仓库已存在，执行拉取..."

        cd "${full_path}"

        # 获取最新变更
        git fetch origin

        # 检查本地是否有未提交的更改
        if ! git diff-index --quiet HEAD --; then
            log_warn "检测到未提交的更改，暂存当前更改..."
            git stash save "Auto-stash before sync $(date)"
        fi

        # 拉取最新代码
        git reset --hard "origin/${branch}"
        git clean -fd

        log_info "更新完成: ${target_dir}"
    else
        log_info "克隆新仓库..."

        # 删除目录内容（保留目录）
        find "${full_path}" -mindepth 1 -delete

        # 克隆仓库
        git clone --depth 1 --branch "${branch}" "${repo_url}" "${full_path}"

        log_info "克隆完成: ${target_dir}"
    fi
}

################################################################################
# 主函数
################################################################################

main() {
    log_info "========== 开始同步所有仓库 =========="

    for repo_config in "${REPOS[@]}"; do
        IFS='|' read -r target_dir repo_url branch <<< "${repo_config}"
        sync_repo "${target_dir}" "${repo_url}" "${branch}"
    done

    log_info "========== 所有仓库同步完成 =========="

    # 触发 Hugo 重建
    log_info "触发 Hugo 重建..."
    if [ -n "$HUGO_TRIGGER_URL" ]; then
        curl -s -X POST "$HUGO_TRIGGER_URL" || true
    fi

    log_info "同步任务完成"
}

# 执行主函数
main
```

赋予执行权限：

```bash
chmod +x /opt/hugo-blog/scripts/sync-repos.sh
```

### 5.3 配置 GitHub Actions 自动推送（可选）

在内容仓库创建 `.github/workflows/sync.yml`：

```yaml
name: Sync to Blog

on:
  push:
    branches: [main]

jobs:
  sync:
    runs-on: ubuntu-latest

    steps:
      - name: Trigger blog rebuild
        run: |
          curl -X POST "${{ secrets.HUGO_WEBHOOK_URL }}" || true
```

### 5.4 配置 Webhook（可选）

创建 webhook 接收服务：

```docker
# 添加到 docker-compose.yml
  webhook:
    image: almir/webhook:latest
    container_name: hugo-webhook
    restart: unless-stopped
    ports:
      - "9000:9000"
    volumes:
      - ./config/webhook/hooks.json:/etc/webhook/hooks.json
      - ./scripts:/scripts
    command: -hooks /etc/webhook/hooks.json -verbose
```

创建 `/opt/hugo-blog/config/webhook/hooks.json`：

```json
[
  {
    "id": "rebuild-hugo",
    "execute-command": "/scripts/rebuild.sh",
    "command-working-directory": "/",
    "trigger-rule": {
      "match": {
        "type": "payload-hash-sha256",
        "secret": "your-webhook-secret",
        "parameter": {
          "source": "header",
          "name": "X-Hub-Signature-256"
        }
      }
    }
  }
]
```

---

## 六、Nginx 反向代理配置

### 6.1 完整 Nginx 配置

`/opt/hugo-blog/config/nginx/conf.d/blog.conf`：

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name blog.yourdomain.com;

    # 强制 HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name blog.yourdomain.com;

    # SSL 证书配置（见下一节）
    ssl_certificate /etc/nginx/ssl/blog.yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/nginx/ssl/blog.yourdomain.com/privkey.pem;

    # SSL 优化
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256;
    ssl_prefer_server_ciphers off;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;

    # 安全头
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;

    # 日志
    access_log /var/log/nginx/blog_access.log;
    error_log /var/log/nginx/blog_error.log;

    # 静态文件直接服务（生产环境）
    location / {
        root /usr/share/nginx/html;
        try_files $uri $uri/ /index.html;

        # 静态资源缓存
        location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2|ttf|eot|webp)$ {
            expires 1y;
            add_header Cache-Control "public, immutable";
        }

        # HTML 不缓存
        location ~* \.html$ {
            expires -1;
            add_header Cache-Control "no-cache, no-store, must-revalidate";
        }
    }

    # Hugo 开发服务器代理（开发环境）
    # location / {
    #     proxy_pass http://hugo-blog:1313;
    #     proxy_set_header Host $host;
    #     proxy_set_header X-Real-IP $remote_addr;
    #     proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    #     proxy_set_header X-Forwarded-Proto $scheme;
    # }

    # Gzip 压缩
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_comp_level 6;
    gzip_types text/plain text/css text/xml text/javascript
               application/json application/javascript application/xml+rss
               application/rss+xml font/truetype font/opentype
               application/vnd.ms-fontobject image/svg+xml;
}
```

### 6.2 在 1Panel 中应用配置

1. **网站 → 选择你的博客网站 → 配置**

2. **修改配置**，将上述内容粘贴并调整路径

3. **重载配置**

---

## 七、SSL 证书配置

### 7.1 在 1Panel 中申请证书（推荐）

1. **网站 → SSL**

2. **选择 Let's Encrypt**

3. **配置**：
   - 邮箱：`your@email.com`
   - 域名：`blog.yourdomain.com`
   - 验证方式：HTTP

4. **申请**

### 7.2 使用 acme.sh 申请

```bash
# 申请证书
acme.sh --issue -d blog.yourdomain.com --webroot /opt/hugo-blog/data/public

# 安装证书
acme.sh --install-cert -d blog.yourdomain.com \
  --key-file /opt/hugo-blog/config/nginx/ssl/blog.yourdomain.com/privkey.pem \
  --fullchain-file /opt/hugo-blog/config/nginx/ssl/blog.yourdomain.com/fullchain.pem \
  --reloadcmd "docker exec hugo-nginx nginx -s reload"
```

### 7.3 DNS API 验证（通配符证书）

```bash
# Cloudflare
export CF_Token="your_api_token"
acme.sh --issue -d blog.yourdomain.com -d *.blog.yourdomain.com --dns dns_cf

# 阿里云
export Ali_Key="your_access_key"
export Ali_Secret="your_access_secret"
acme.sh --issue -d blog.yourdomain.com -d *.blog.yourdomain.com --dns dns_ali
```

---

## 八、自动化部署脚本

### 8.1 完整部署脚本

创建 `/opt/hugo-blog/scripts/deploy.sh`：

```bash
#!/bin/bash

################################################################################
# Hugo 博客一键部署脚本
# 功能：同步内容、构建站点、重启服务
################################################################################

set -e

# 颜色
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m'

info() { echo -e "${GREEN}[INFO]${NC} $1"; }
warn() { echo -e "${YELLOW}[WARN]${NC} $1"; }

# 项目目录
PROJECT_DIR="/opt/hugo-blog"
cd "$PROJECT_DIR"

# 1. 同步 Git 仓库
info "========== 步骤 1: 同步 Git 仓库 =========="
docker exec hugo-git-sync /scripts/sync-repos.sh

# 2. 构建 Hugo
info "========== 步骤 2: 构建 Hugo =========="
docker exec hugo-blog hugo --cleanDestinationDir

# 3. 清理 Nginx 缓存（可选）
info "========== 步骤 3: 清理缓存 =========="
if docker exec hugo-nginx test -d /var/cache/nginx; then
    docker exec hugo-nginx find /var/cache/nginx -type f -delete
fi

# 4. 重载 Nginx
info "========== 步骤 4: 重载 Nginx =========="
docker exec hugo-nginx nginx -s reload

info "========== 部署完成！=========="
```

### 8.2 添加到 Crontab

```bash
# 编辑 crontab
crontab -e

# 添加定时任务（每小时同步一次）
0 * * * * /opt/hugo-blog/scripts/deploy.sh >> /var/log/hugo-deploy.log 2>&1
```

### 8.3 一键安装脚本

创建 `/opt/hugo-blog/install.sh`：

```bash
#!/bin/bash

################################################################################
# Hugo 博客一键安装脚本
################################################################################

set -e

GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m'

info() { echo -e "${GREEN}[INFO]${NC} $1"; }

# 配置
DOMAIN="${1:-blog.example.com}"
EMAIL="${2:-admin@example.com}"

info "========== Hugo 博客安装 =========="
info "域名: $DOMAIN"
info "邮箱: $EMAIL"

# 创建目录
info "创建目录结构..."
mkdir -p /opt/hugo-blog/{config/{nginx,webhook},data/{content/{posts,notes,resources},themes,public},scripts}

# 创建 docker-compose.yml
info "创建 Docker Compose 配置..."
cat > /opt/hugo-blog/docker-compose.yml << 'EOF'
version: '3.8'

services:
  hugo:
    image: hugomods/hugo:extended-0.139.4
    container_name: hugo-blog
    restart: unless-stopped
    working_dir: /src
    volumes:
      - ./data:/src
    environment:
      - HUGO_ENV=production
    command: >
      sh -c "
        while true; do
          hugo --cleanDestinationDir
          sleep 300
        done
      "
    networks:
      - blog-network

  git-sync:
    image: alpine/git:latest
    container_name: hugo-git-sync
    restart: unless-stopped
    volumes:
      - ./data/content:/content
      - ./scripts:/scripts
    command: >
      sh -c "
        apk add --no-cache openssh-client &&
        chmod +x /scripts/sync-repos.sh &&
        while true; do
          /scripts/sync-repos.sh
          sleep 600
        done
      "
    networks:
      - blog-network

networks:
  blog-network:
    driver: bridge
EOF

# 创建同步脚本
info "创建同步脚本..."
cat > /opt/hugo-blog/scripts/sync-repos.sh << 'EOF'
#!/bin/bash
set -e
REPOS=(
    "posts|https://github.com/yourusername/blog-posts.git|main"
    "notes|https://github.com/yourusername/notes.git|main"
)
CONTENT_DIR="/content"
for repo_config in "${REPOS[@]}"; do
    IFS='|' read -r target_dir repo_url branch <<< "${repo_config}"
    full_path="${CONTENT_DIR}/${target_dir}"
    mkdir -p "${full_path}"
    if [ -d "${full_path}/.git" ]; then
        cd "${full_path}" && git pull origin "${branch}"
    else
        git clone --depth 1 --branch "${branch}" "${repo_url}" "${full_path}"
    fi
done
EOF

chmod +x /opt/hugo-blog/scripts/sync-repos.sh

# 创建 Hugo 配置
info "创建 Hugo 配置..."
cat > /opt/hugo-blog/data/hugo.toml << EOF
baseURL = "https://$DOMAIN/"
languageCode = "zh-cn"
title = "我的技术博客"
theme = "hugo-geekdoc"

[menu]
  [[menu.main]]
    name = "首页"
    url = "/"
    weight = 1
  [[menu.main]]
    name = "文章"
    url = "/posts/"
    weight = 2
EOF

# 克隆主题
info "克隆主题..."
git clone https://github.com/thegeeklab/hugo-geekdoc.git /opt/hugo-blog/data/themes/hugo-geekdoc

# 创建示例文章
info "创建示例文章..."
mkdir -p /opt/hugo-blog/data/content/posts
cat > /opt/hugo-blog/data/content/posts/first-post.md << EOF
---
title: "第一篇文章"
date: 2026-02-09
draft: false
---

欢迎来到我的技术博客！
EOF

# 设置权限
info "设置权限..."
chown -R 1000:1000 /opt/hugo-blog/data

# 启动服务
info "启动服务..."
cd /opt/hugo-blog
docker-compose up -d

info "========== 安装完成！=========="
info "1. 请在 1Panel 中配置反向代理"
info "2. 域名: $DOMAIN"
info "3. 代理地址: http://hugo-blog:1313"
info ""
warn "记得修改同步脚本中的仓库地址！"
```

使用方法：

```bash
chmod +x /opt/hugo-blog/install.sh
./opt/hugo-blog/install.sh blog.yourdomain.com admin@example.com
```

---

## 九、常见问题排查

### 9.1 Hugo 构建失败

```bash
# 查看日志
docker logs hugo-blog

# 检查配置文件
docker exec hugo-blog hugo config

# 手动构建查看错误
docker exec -it hugo-blog hugo --verbose --cleanDestinationDir
```

### 9.2 内容未更新

```bash
# 检查同步状态
docker exec hugo-git-sync /scripts/sync-repos.sh

# 检查 Git 子模块
cd /opt/hugo-blog/data
git submodule status
git submodule update --remote

# 手动触发构建
docker exec hugo-blog hugo --cleanDestinationDir
```

### 9.3 主题未生效

```bash
# 检查主题目录
ls -la /opt/hugo-blog/data/themes/

# 确认配置文件中的主题名称
docker exec hugo-blog cat hugo.toml | grep theme

# 重新克隆主题
cd /opt/hugo-blog/data/themes
rm -rf hugo-geekdoc
git clone https://github.com/thegeeklab/hugo-geekdoc.git
```

### 9.4 Nginx 403/404 错误

```bash
# 检查文件权限
ls -la /opt/hugo-blog/data/public/

# 修复权限
chown -R 1000:1000 /opt/hugo-blog/data

# 检查 Nginx 配置
docker exec hugo-nginx nginx -t
docker exec hugo-nginx cat /etc/nginx/conf.d/blog.conf
```

### 9.5 SSL 证书问题

```bash
# 检查证书有效期
docker exec hugo-nginx ls -la /etc/nginx/ssl/

# 手动续期
acme.sh --renew -d blog.yourdomain.com --force

# 检查证书链
openssl s_client -connect blog.yourdomain.com:443 -servername blog.yourdomain.com
```

---

## 📚 相关资源

### 官方文档
- [Hugo 官方文档](https://gohugo.io/)
- [Hugo 主题列表](https://themes.gohugo.io/)
- [1Panel 官方文档](https://1panel.cn/docs/)

### 推荐主题
- [hugo-geekdoc](https://github.com/thegeeklab/hugo-geekdoc) - 文档风格
- [hugo-theme-stack](https://github.com/CaiJimmy/hugo-theme-stack) - 博客风格
- [hugo-book](https://github.com/alex-shpak/hugo-book) - 书籍风格

### Git 同步工具
- [Git Submodule](https://git-scm.com/book/en/v2/Git-Tools-Submodules)
- [git-subrepo](https://github.com/ingydotnet/git-subrepo)

---

## 📝 更新日志

| 日期 | 版本 | 说明 |
|------|------|------|
| 2026-02-09 | v1.0 | 初始版本，包含完整部署流程 |

---

> 💡 **提示**: 本指南适用于已有 VPS 服务器并完成基础初始化的环境。如需服务器初始化，请参考 [[VPS云服务器初始化完全指南]]
