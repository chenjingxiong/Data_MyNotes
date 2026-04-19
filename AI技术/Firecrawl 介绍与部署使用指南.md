# Firecrawl 介绍与部署使用指南

> **项目地址**：[github.com/firecrawl/firecrawl](https://github.com/firecrawl/firecrawl)
> **官方网站**：[firecrawl.dev](https://firecrawl.dev)
> **许可证**：AGPL-3.0（核心）/ MIT（SDK 和部分 UI）
> **最新版本**：v2.8.0
> **GitHub Stars**：37k+

---

## 一、Firecrawl 是什么？

Firecrawl 是一款专为 AI 应用设计的 **Web 数据 API**——将任何网页转换为 LLM 可直接使用的干净 Markdown、结构化 JSON 或截图。只需一个 API 调用，即可搜索、抓取、爬取和交互整个 Web。

### 核心定位

| 定位 | 说明 |
|------|------|
| **Web Data API for AI** | 为 AI 代理和应用提供清洁的 Web 数据 |
| **LLM-Ready 输出** | 直接输出 Markdown、JSON、截图，减少 Token 消耗 |
| **全网页覆盖** | 覆盖 96% 的网页，包括 JS 重度渲染页面 |
| **高性能** | P95 延迟 3.4 秒，可处理数百万页面 |

### 为什么选择 Firecrawl？

- **行业领先可靠性**：覆盖 96% 的网页，包括 JS 重度页面——无需操心代理问题
- **极速响应**：P95 延迟 3.4 秒，适合实时 AI 代理和动态应用
- **LLM 原生输出**：干净的 Markdown、结构化 JSON、截图——更少 Token，更好的 AI 应用
- **零配置**：自动处理旋转代理、速率限制、JS 屏蔽内容等
- **AI 代理就绪**：一条命令即可连接任何 AI 代理或 MCP 客户端
- **媒体解析**：解析 PDF、DOCX 等 Web 托管文件
- **交互操作**：点击、滚动、输入、等待、按键后再提取内容

---

## 二、功能概览

### 核心端点

| 功能 | 描述 |
|------|------|
| **Search** | 搜索 Web 并获取结果的完整页面内容 |
| **Scrape** | 将任何 URL 转为 Markdown、HTML、截图或结构化 JSON |
| **Interact** | 抓取页面后，使用 AI 提示或代码与页面交互 |

### 扩展功能

| 功能               | 描述               |
| ---------------- | ---------------- |
| **Agent**        | 自动数据采集，只需描述你需要什么 |
| **Crawl**        | 单个请求爬取网站的所有 URL  |
| **Map**          | 即时发现网站上的所有 URL   |
| **Batch Scrape** | 异步抓取数千个 URL      |

### 开源版 vs 云端版

| 能力 | 云端版 | 自托管版 |
|------|--------|---------|
| 所有 API 端点 | 支持 | 部分不支持（`/agent`、`/browser` 不支持） |
| 截图支持 | 支持 | 支持（需运行 Playwright 服务） |
| 本地 LLM | 不支持 | 支持（通过 `OLLAMA_BASE_URL`，实验性） |
| 高级反检测 | 支持（Fire-engine） | 不支持 |
| 搜索 API | 内置 | 需配置 SearXNG |

---

## 三、部署方式

### 方式一：Docker Compose 自托管（推荐）

这是自托管最简单的方式。

#### 1. 克隆仓库

```bash
git clone https://github.com/firecrawl/firecrawl.git
cd firecrawl
```

#### 2. 配置环境变量

```bash
# 复制环境变量模板
cp apps/api/.env.example .env
```

编辑 `.env` 文件，最小配置如下：

```bash
# ===== 必需配置 =====
PORT=3002
HOST=0.0.0.0

# 关闭数据库认证（自托管无需 Supabase）
USE_DB_AUTHENTICATION=false

# ===== 队列管理面板密钥 =====
# 请修改此值，用于访问 Bull Queue Manager
BULL_AUTH_KEY=your_secret_key_here

# ===== 可选：AI 功能 =====
# 启用 JSON 格式抓取、/extract API 等
# OPENAI_API_KEY=sk-your-openai-key

# ===== 可选：使用 Ollama 本地 LLM（实验性）=====
# OLLAMA_BASE_URL=http://localhost:11434/api
# MODEL_NAME=deepseek-r1:7b
# MODEL_EMBEDDING_NAME=nomic-embed-text

# ===== 可选：使用 OpenAI 兼容 API =====
# OPENAI_BASE_URL=https://example.com/v1
# OPENAI_API_KEY=your-key

# ===== 可选：代理配置 =====
# PROXY_SERVER=http://0.1.2.3:1234
# PROXY_USERNAME=
# PROXY_PASSWORD=

# ===== 可选：搜索 API（替代 Google）=====
# SEARXNG_ENDPOINT=http://your.searxng.server

# ===== 系统资源限制 =====
# MAX_CPU=0.8    # CPU 使用上限（0.0-1.0）
# MAX_RAM=0.8    # 内存使用上限（0.0-1.0）
```

#### 3. 构建并启动

```bash
# 构建 Docker 镜像
docker compose build

# 启动所有服务
docker compose up -d

# 查看日志
docker compose logs -f
```

启动后访问：
- **API 地址**：`http://localhost:3002`
- **队列管理面板**：`http://localhost:3002/admin/{BULL_AUTH_KEY}/queues`

#### 4. 验证部署

```bash
# 测试抓取
curl -X POST http://localhost:3002/v2/scrape \
  -H 'Content-Type: application/json' \
  -d '{
    "url": "https://firecrawl.dev"
  }'

# 测试爬取
curl -X POST http://localhost:3002/v2/crawl \
  -H 'Content-Type: application/json' \
  -d '{
    "url": "https://docs.firecrawl.dev"
  }'
```

### 方式二：带 SearXNG 搜索的完整部署

如果需要 `/search` API 功能，需要搭配 SearXNG。

创建 `docker-compose.override.yml`：

```yaml
version: '3.8'

services:
  searxng:
    image: searxng/searxng:latest
    container_name: searxng
    restart: unless-stopped
    ports:
      - "8888:8080"
    volumes:
      - ./searxng-settings:/etc/searxng:rw
    environment:
      - SEARXNG_BASE_URL=http://localhost:8888/
    cap_drop:
      - ALL
    cap_add:
      - CHOWN
      - SETGID
      - SETUID
```

在 `.env` 中添加：

```bash
SEARXNG_ENDPOINT=http://searxng:8080
```

然后启动：

```bash
docker compose -f docker-compose.yml -f docker-compose.override.yml up -d
```

### 方式三：Kubernetes 部署

适用于生产环境的大规模部署，参考仓库中的 `examples/kubernetes-cluster-install/` 目录。

```bash
cd examples/kubernetes-cluster-install/
# 按照 README.md 中的说明操作
kubectl apply -f firecrawl.yaml
```

### 方式四：使用云端 API（无需部署）

最简单的方式——直接使用 Firecrawl 托管服务：

1. 访问 [firecrawl.dev](https://firecrawl.dev) 注册账号
2. 获取 API Key（`fc-YOUR_API_KEY`）
3. 直接调用 API：`https://api.firecrawl.dev/v2/`

---

## 四、SDK 安装与使用

### Python SDK

```bash
pip install firecrawl-py
```

#### 基础用法

```python
from firecrawl import Firecrawl

# 使用云端 API
app = Firecrawl(api_key="fc-YOUR_API_KEY")

# 使用自托管实例
# app = Firecrawl(api_key="", api_url="http://localhost:3002")

# ---------- 抓取单个页面 ----------
result = app.scrape('https://firecrawl.dev', formats=['markdown'])
print(result.markdown)

# ---------- 搜索 Web ----------
results = app.search("Firecrawl web scraping", limit=5)
for r in results:
    print(r['title'], r['url'])

# ---------- 爬取整个网站 ----------
docs = app.crawl('https://docs.firecrawl.dev', limit=50)
for doc in docs.data:
    print(doc.metadata.source_url, doc.markdown[:100])

# ---------- 发现网站 URL ----------
links = app.map('https://firecrawl.dev')
print(links)

# ---------- 批量抓取 ----------
job = app.batch_scrape([
    'https://firecrawl.dev',
    'https://docs.firecrawl.dev',
], formats=['markdown'])
for doc in job.data:
    print(doc.metadata.source_url)
```

#### 结构化数据提取

```python
from firecrawl import Firecrawl
from pydantic import BaseModel, Field
from typing import List, Optional

app = Firecrawl(api_key="fc-YOUR_API_KEY")

# 定义输出结构
class Founder(BaseModel):
    name: str = Field(description="创始人全名")
    role: Optional[str] = Field(None, description="职位")

class FoundersSchema(BaseModel):
    founders: List[Founder] = Field(description="创始人列表")

# 使用 Agent 自动提取
result = app.agent(
    prompt="找出 Firecrawl 的创始人",
    schema=FoundersSchema
)
print(result.data)
# {
#   "founders": [
#     {"name": "Eric Ciarla", "role": "Co-founder"},
#     {"name": "Nicolas Camara", "role": "Co-founder"},
#     {"name": "Caleb Peffer", "role": "Co-founder"}
#   ]
# }
```

#### 页面交互

```python
from firecrawl import Firecrawl

app = Firecrawl(api_key="fc-YOUR_API_KEY")

# 先抓取页面
result = app.scrape("https://amazon.com")
scrape_id = result.metadata.scrape_id

# 与页面交互
app.interact(scrape_id, prompt="搜索 'mechanical keyboard'")
app.interact(scrape_id, prompt="点击第一个结果")
```

### Node.js SDK

```bash
npm install @mendable/firecrawl-js
```

```javascript
import Firecrawl from '@mendable/firecrawl-js';

const app = new Firecrawl({
  apiKey: 'fc-YOUR_API_KEY',
  // 自托管时：
  // apiUrl: 'http://localhost:3002'
});

// 抓取
const doc = await app.scrape('https://firecrawl.dev', { formats: ['markdown'] });
console.log(doc.markdown);

// 搜索
const results = await app.search('best web scraping tools 2026', { limit: 10 });
results.data.web.forEach(r => {
  console.log(`${r.title}: ${r.url}`);
});

// 爬取
const docs = await app.crawl('https://docs.firecrawl.dev', { limit: 50 });
docs.data.forEach(doc => {
  console.log(doc.metadata.sourceURL, doc.markdown.substring(0, 100));
});
```

### cURL 直接调用

```bash
# 抓取
curl -X POST 'https://api.firecrawl.dev/v2/scrape' \
  -H 'Authorization: Bearer fc-YOUR_API_KEY' \
  -H 'Content-Type: application/json' \
  -d '{"url": "https://firecrawl.dev"}'

# 搜索
curl -X POST 'https://api.firecrawl.dev/v2/search' \
  -H 'Authorization: Bearer fc-YOUR_API_KEY' \
  -H 'Content-Type: application/json' \
  -d '{"query": "firecrawl web scraping", "limit": 5}'

# 爬取
curl -X POST 'https://api.firecrawl.dev/v2/crawl' \
  -H 'Authorization: Bearer fc-YOUR_API_KEY' \
  -H 'Content-Type: application/json' \
  -d '{"url": "https://docs.firecrawl.dev", "limit": 100}'

# 查看爬取状态
curl -X GET 'https://api.firecrawl.dev/v2/crawl/{JOB_ID}' \
  -H 'Authorization: Bearer fc-YOUR_API_KEY'

# Map
curl -X POST 'https://api.firecrawl.dev/v2/map' \
  -H 'Authorization: Bearer fc-YOUR_API_KEY' \
  -H 'Content-Type: application/json' \
  -d '{"url": "https://firecrawl.dev"}'
```

### CLI 工具

```bash
# 安装
npm install -g firecrawl-cli

# 抓取
firecrawl scrape https://firecrawl.dev
firecrawl https://firecrawl.dev --only-main-content

# 搜索
firecrawl search "firecrawl web scraping" --limit 5

# 交互
firecrawl scrape https://amazon.com
firecrawl interact exec --prompt "搜索 mechanical keyboard"
```

---

## 五、AI 代理集成

### MCP 服务器

Firecrawl 提供 MCP 服务器，可连接任何 MCP 兼容客户端：

```json
{
  "mcpServers": {
    "firecrawl-mcp": {
      "command": "npx",
      "args": ["-y", "firecrawl-mcp"],
      "env": {
        "FIRECRAWL_API_KEY": "fc-YOUR_API_KEY"
      }
    }
  }
}
```

### Claude Code Skill

一条命令为 AI 代理安装 Web 数据能力：

```bash
npx -y firecrawl-cli@latest init --all --browser
```

支持 Claude Code、Antigravity、OpenCode 等。重启代理后生效。

### Agent 端点

使用 AI Agent 自动采集数据——无需知道 URL，只需描述需求：

```bash
curl -X POST 'https://api.firecrawl.dev/v2/agent' \
  -H 'Authorization: Bearer fc-YOUR_API_KEY' \
  -H 'Content-Type: application/json' \
  -d '{"prompt": "查找 Notion 的定价方案"}'
```

**模型选择**：

| 模型 | 成本 | 适用场景 |
|------|------|---------|
| `spark-1-mini`（默认） | 便宜 60% | 大多数任务 |
| `spark-1-pro` | 标准 | 复杂研究、关键提取 |

```python
result = app.agent(
    prompt="比较 Firecrawl、Apify、ScrapingBee 的企业功能",
    model="spark-1-pro"
)
```

### 平台集成

| 平台 | 类型 |
|------|------|
| Lovable | 低代码平台 |
| Zapier | 自动化工作流 |
| n8n | 工作流自动化 |
| LangChain | AI 框架 |
| LlamaIndex | AI 框架 |

---

## 六、自托管配置详解

### 代理配置

```bash
# .env 中配置代理
PROXY_SERVER=http://proxy-ip:port
PROXY_USERNAME=username
PROXY_PASSWORD=password
```

### 本地 LLM（Ollama）

```bash
# .env 配置
OLLAMA_BASE_URL=http://localhost:11434/api
MODEL_NAME=deepseek-r1:7b
MODEL_EMBEDDING_NAME=nomic-embed-text
```

先启动 Ollama：

```bash
# 安装 Ollama
curl -fsSL https://ollama.com/install.sh | sh

# 拉取模型
ollama pull deepseek-r1:7b
ollama pull nomic-embed-text

# 启动服务
ollama serve
```

### TypeScript Playwright 服务

如需使用 TypeScript 版 Playwright 服务（默认是 Go 版）：

1. 修改 `docker-compose.yml`：

```yaml
# 将
build: apps/playwright-service
# 改为
build: apps/playwright-service-ts
```

2. 在 `.env` 中设置：

```bash
PLAYWRIGHT_MICROSERVICE_URL=http://localhost:3000/scrape
```

---

## 七、常见问题排查

### Supabase 客户端未配置

**症状**：
```
ERROR - Attempted to access Supabase client when it's not configured.
```

**说明**：自托管版本不需要 Supabase，可以正常使用抓取和爬取功能。

### 绕过认证警告

**症状**：
```
WARN - You're bypassing authentication
```

**说明**：正常现象。自托管版本关闭了数据库认证（`USE_DB_AUTHENTICATION=false`）。

### Docker 容器启动失败

```bash
# 查看日志
docker logs [container_name]

# 检查环境变量
cat .env

# 重建容器
docker compose down
docker compose build --no-cache
docker compose up -d
```

### Redis 连接问题

- 确保 Redis 服务正在运行：`docker compose ps`
- 检查 `REDIS_URL` 配置是否正确
- 检查网络设置和防火墙规则

### API 端点无响应

- 确认 Firecrawl 容器正在运行
- 检查 `PORT` 和 `HOST` 设置
- 确认端口未被占用：`ss -tlnp | grep 3002`

---

## 八、与同类工具对比

| 特性       | Firecrawl     | Scrapy     | Crawl4AI | Playwright |
| -------- | ------------- | ---------- | -------- | ---------- |
| 定位       | AI 数据 API     | 通用爬虫框架     | AI 爬虫    | 浏览器自动化     |
| LLM 原生输出 | 原生支持          | 需自行处理      | 支持       | 需自行处理      |
| JS 渲染    | 自动处理          | 需 Splash 等 | 支持       | 原生支持       |
| API 服务   | REST API      | 需自行搭建      | 基础 API   | 无          |
| 代理管理     | 自动            | 需配置        | 需配置      | 需配置        |
| 结构化提取    | AI 驱动         | 需编写选择器     | 支持       | 需编写代码      |
| 学习曲线     | 低             | 中高         | 低        | 中          |
| 语言       | TypeScript/Go | Python     | Python   | 多语言        |

---

## 九、参考链接

- **GitHub**：[github.com/firecrawl/firecrawl](https://github.com/firecrawl/firecrawl)
- **官网**：[firecrawl.dev](https://firecrawl.dev)
- **文档**：[docs.firecrawl.dev](https://docs.firecrawl.dev)
- **API 参考**：[docs.firecrawl.dev/api-reference](https://docs.firecrawl.dev/api-reference)
- **Playground**：[firecrawl.dev/playground](https://firecrawl.dev/playground)
- **自托管指南**：[docs.firecrawl.dev/contributing/self-host](https://docs.firecrawl.dev/contributing/self-host)
- **Discord**：[discord.gg/firecrawl](https://discord.gg/949090953497567312)

---

*最后更新：2026-04-07*
