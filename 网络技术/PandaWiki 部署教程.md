# PandaWiki 部署教程

> PandaWiki 是由长亭科技开源的 AI 大模型驱动的知识库系统，支持 Markdown 编辑、AI 问答、AI 搜索等功能。

## 📋 目录

- [系统要求](#系统要求)
- [部署方式选择](#部署方式选择)
- [快速部署（Docker）](#快速部署docker)
- [Git 同步发布配置](#git-同步发布配置)
- [Markdown 笔记管理](#markdown-笔记管理)
- [常见问题](#常见问题)

---

## 系统要求

| 项目             | 要求                                  |
| -------------- | ----------------------------------- |
| 操作系统           | Linux（推荐 Ubuntu 20.04+ / CentOS 7+） |
| 内存             | 至少 2GB                              |
| 磁盘空间           | 至少 10GB                             |
| 架构             | x86_64                              |
| 网络             | 需要访问外网（拉取镜像和 AI 模型）                 |
| Docker         | 20.x+                               |
| Docker Compose | 2.0.0+                              |

---

## 部署方式选择

PandaWiki 支持多种部署方式：

1. **一键安装脚本** - 最简单，适合快速体验
2. **Docker Compose** - 推荐生产环境，便于管理
3. **Kubernetes** - 大规模部署场景

本教程重点介绍 **Docker Compose** 方式，并配合 Git 仓库进行内容管理。

---

## 快速部署（Docker）

### 方法一：一键安装脚本

```bash
# 使用 root 权限执行
bash -c "$(curl -fsSLk https://github.com/chaitin/PandaWiki/raw/main/scripts/install.sh)"
```

按照提示完成安装，完成后会显示访问地址、用户名和密码。

### 方法二：Docker Compose（推荐）

#### 1. 创建部署目录

```bash
mkdir -p ~/pandawiki
cd ~/pandawiki
```

#### 2. 创建 docker-compose.yml

```yaml
version: '3.8'

services:
  pandawiki:
    image: chaitin/pandawiki:latest
    container_name: pandawiki
    ports:
      - "8080:8080"
    volumes:
      # 挂载数据目录
      - ./data:/app/data
      # 挂载 Git 仓库目录（用于同步笔记）
      - ./wiki-repo:/app/wiki-repo
    environment:
      - TZ=Asia/Shanghai
    restart: unless-stopped
    networks:
      - pandawiki-net

networks:
  pandawiki-net:
    driver: bridge
```

#### 3. 启动服务

```bash
# 创建必要的目录
mkdir -p data wiki-repo

# 启动容器
docker compose up -d

# 查看日志
docker compose logs -f pandawiki
```

#### 4. 访问系统

打开浏览器访问：`http://your-server-ip:8080`

默认管理员账号（首次登录需创建）。

---

## Git 同步发布配置

### 方案概述

```
┌─────────────┐     Git Push      ┌─────────────┐
│  本地编辑    │ ────────────────▶ │  Git 仓库   │
│  (Markdown) │                   │ (GitHub/Gitea)│
└─────────────┘                   └─────────────┘
                                          │
                                          │ Git Pull
                                          ▼
                                   ┌─────────────┐
                                   │  PandaWiki  │
                                   │  服务器端    │
                                   └─────────────┘
```

### 配置步骤

#### 1. 在服务器上配置 Git

```bash
# 安装 Git（如果未安装）
apt update && apt install -y git

# 配置 Git 用户信息
git config --global user.name "PandaWiki Bot"
git config --global user.email "pandawiki@example.com"

# 生成 SSH 密钥（用于免密拉取）
ssh-keygen -t rsa -b 4096 -C "pandawiki" -f ~/.ssh/pandawiki -N ""
```

将生成的 `~/.ssh/pandawiki.pub` 公钥添加到你的 Git 仓库（GitHub/GitLab/Gitea）的 SSH Keys 中。

#### 2. 创建 Wiki 仓库

在你的 Git 托管平台（推荐 Gitea/自建 GitLab）创建一个私有仓库：

```bash
# 在服务器上克隆仓库
cd ~/pandawiki
git clone git@your-git-server:your-wiki-repo.git wiki-content
```

#### 3. 配置自动同步脚本

创建 `~/pandawiki/sync-wiki.sh`：

```bash
#!/bin/bash

# PandaWiki Git 同步脚本

REPO_PATH="/root/pandawiki/wiki-content"
LOG_FILE="/root/pandawiki/sync.log"
WEBHOOK_URL="http://localhost:8080/api/webhook/sync"  # PandaWiki webhook

echo "=== $(date) ===" >> $LOG_FILE

cd $REPO_PATH || exit 1

# 拉取最新内容
echo "拉取最新内容..." >> $LOG_FILE
git fetch origin
git reset --hard origin/main

# 可选：触发 PandaWiki 重新索引
# curl -X POST $WEBHOOK_URL >> $LOG_FILE 2>&1

echo "同步完成！" >> $LOG_FILE
echo "" >> $LOG_FILE
```

赋予执行权限：

```bash
chmod +x ~/pandawiki/sync-wiki.sh
```

#### 4. 设置定时同步（Cron）

```bash
# 编辑 crontab
crontab -e

# 添加定时任务（每 5 分钟同步一次）
*/5 * * * * /root/pandawiki/sync-wiki.sh
```

#### 5. 配置 Webhook 自动触发（可选）

在你的 Git 仓库中设置 Push Webhook：

1. 进入仓库设置 → Webhooks
2. 添加 Webhook URL：`http://your-server-ip:8080/api/webhook/git-sync`
3. 选择触发事件：Push events

---

## Markdown 笔记管理

### 目录结构建议

```
wiki-content/
├── README.md              # 首页
├── 01.快速开始/
│   └── 入门指南.md
├── 02.产品文档/
│   ├── 功能介绍.md
│   └── API文档.md
├── 03.技术文档/
│   ├── 架构设计.md
│   └── 部署指南.md
└── assets/
    ├── images/
    └── files/
```

### Markdown 编写规范

PandaWiki 完全兼容标准 Markdown 语法：

```markdown
# 一级标题

## 二级标题

### 三级标题

**粗体** 和 *斜体*

- 无序列表项 1
- 无序列表项 2

1. 有序列表项 1
2. 有序列表项 2

| 列1 | 列2 |
|-----|-----|
| 内容 | 内容 |

`行内代码`

```代码块```

[链接文字](https://example.com)

![图片描述](./assets/images/example.png)

> 引用内容

---

[Wiki链接](./其他文档.md)  <!-- PandaWiki 支持相对路径链接 -->
```

### 本地编辑工作流

```bash
# 1. 克隆仓库到本地
git clone git@your-git-server:your-wiki-repo.git

# 2. 使用你喜欢的编辑器编辑 Markdown 文件
# 推荐：VS Code, Obsidian, Typora

# 3. 提交更改
git add .
git commit -m "docs: 更新产品文档"

# 4. 推送到远程仓库
git push

# PandaWiki 服务器会自动拉取更新（通过 Cron 或 Webhook）
```

### 推荐的编辑器配置

| 编辑器 | 优点 | 适用场景 |
|--------|------|----------|
| **Obsidian** | 双链、插件丰富、本地优先 | 知识库构建 |
| **VS Code** | 扩展强大、Git 集成好 | 开发者文档 |
| **Typora** | 所见即所得、简洁优雅 | 单文档编辑 |
| **Mark Text** | 开源免费、实时预览 | 轻量编辑 |

---

## 常见问题

### Q1: 如何备份数据？

```bash
# 备份 PandaWiki 数据
docker exec pandawiki tar czf /tmp/backup.tar.gz /app/data
docker cp pandawiki:/tmp/backup.tar.gz ~/pandawiki-backup-$(date +%Y%m%d).tar.gz

# Git 仓库内容已经通过远程仓库备份
```

### Q2: 如何升级到最新版本？

```bash
cd ~/pandawiki
docker compose pull
docker compose up -d
```

### Q3: 如何自定义端口？

修改 `docker-compose.yml` 中的 ports 配置：

```yaml
ports:
  - "自定义端口:8080"
```

### Q4: Git 同步冲突怎么办？

```bash
# 强制使用远程版本（谨慎操作）
cd ~/pandawiki/wiki-content
git fetch origin
git reset --hard origin/main
```

### Q5: 如何启用 HTTPS？

推荐使用 Nginx 反向代理 + Let's Encrypt：

```nginx
server {
    listen 443 ssl http2;
    server_name wiki.yourdomain.com;

    ssl_certificate /etc/letsencrypt/live/wiki.yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/wiki.yourdomain.com/privkey.pem;

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

## 参考资源

- [PandaWiki GitHub 官方仓库](https://github.com/chaitin/PandaWiki)
- [PandaWiki 官方文档](https://pandawiki.docs.baizhi.cloud/)
- [5分钟快速部署教程 - 知乎](https://zhuanlan.zhihu.com/p/1954124233337708715)
- [Docker Compose 部署教程 - CSDN](https://blog.csdn.net/weimeilayer/article/details/153786291)
- [腾讯云部署教程](https://cloud.tencent.com/developer/article/2530858)

---

**文档创建时间**: 2026-02-09
**最后更新**: 2026-02-09
