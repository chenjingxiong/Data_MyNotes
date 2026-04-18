---
title: Scrapling 完全指南
date: 2026-04-18
tags:
  - web-scraping
  - python
  - ai
  - mcp
  - clawhub
  - claude-code
  - data-extraction
description: Scrapling —— 自适应 Web 爬虫框架完全指南，含安装、使用、MCP Server 及 AI Agent Skill 配置
---

# Scrapling 完全指南

> [!info] 一句话简介
> **Scrapling** 是一个自适应 Web 爬虫框架，能从单次请求扩展到全站并发爬取。其解析器可以**学习网站变化**，在页面更新后自动重新定位元素，堪称"写一次爬虫、永久有效"。

| 项目信息 | 详情 |
|---|---|
| **作者** | Karim Shoair ([D4Vinci](https://github.com/D4Vinci)) |
| **GitHub** | [D4Vinci/Scrapling](https://github.com/D4Vinci/Scrapling) |
| **文档** | [scrapling.readthedocs.io](https://scrapling.readthedocs.io/en/latest/) |
| **Stars** | 37.8k+ ⭐ |
| **Python 版本** | ≥ 3.10（MCP Server 需要 ≥ 3.12） |
| **许可证** | BSD-3-Clause |
| **最新版本** | v0.4.7（2026-04-17） |

---

## 一、为什么选择 Scrapling？

传统爬虫的痛点是 **网站一改版，爬虫就报废**。Scrapling 通过智能相似度算法解决了这个问题：

1. **自适应解析** —— 解析器会"记住"元素特征，网站改版后通过 `adaptive=True` 自动找到它们
2. **反检测能力** —— `StealthyFetcher` 开箱即可绕过 Cloudflare Turnstile 等反爬系统
3. **全栈式方案** —— HTTP 请求、动态渲染、隐身模式、并发爬取、暂停/恢复，一个库搞定
4. **极致性能** —— 文本提取速度是 BeautifulSoup 的 ~784 倍

### 性能基准（5000 嵌套元素文本提取）

| 排名 | 库 | 耗时 (ms) | 对比 Scrapling |
|---|---|---|---|
| 1 | **Scrapling** | **2.02** | **1.0x** |
| 2 | Parsel/Scrapy | 2.04 | 1.01x |
| 3 | Raw Lxml | 2.54 | 1.26x |
| 4 | PyQuery | 24.17 | ~12x |
| 5 | BS4 + Lxml | 1584.31 | ~784x |

---

## 二、核心功能一览

### 2.1 Fetcher（数据获取器）

| Fetcher | 用途 | 特点 |
|---|---|---|
| `Fetcher` | 基础 HTTP 请求 | 快速、可模拟浏览器 TLS 指纹、支持 HTTP/3 |
| `DynamicFetcher` | 动态页面渲染 | 基于 Playwright 的 Chromium/Chrome 浏览器自动化 |
| `StealthyFetcher` | 反检测爬取 | 指纹伪装，可自动解决 Cloudflare 验证码 |
| `AsyncFetcher` | 异步 HTTP 请求 | 完整的 async/await 支持 |

每种 Fetcher 都有对应的 Session 类，支持跨请求的 Cookie 和状态管理。

### 2.2 Spider（爬虫框架）

类似 Scrapy 的全功能爬虫 API：

- **并发控制** —— 可配置的并发限制、域名限速、下载延迟
- **多 Session 支持** —— HTTP、隐身浏览器可混用在一个 Spider 中
- **暂停 & 恢复** —— 基于 Checkpoint 的断点续爬（`Ctrl+C` 优雅停止）
- **流式输出** —— `async for item in spider.stream()` 实时获取数据
- **自动导出** —— 内置 JSON/JSONL 导出

### 2.3 选择器与自适应解析

```python
# 多种选择方式
page.css('.quote')                          # CSS 选择器
page.xpath('//div[@class="quote"]')         # XPath
page.find_all('div', class_='quote')        # BeautifulSoup 风格
page.find_by_text('quote', tag='div')       # 按文本内容查找

# 自适应元素追踪 —— 网站改版后自动重新定位
products = page.css('.product', auto_save=True)   # 首次：保存元素特征
products = page.css('.product', adaptive=True)     # 后续：自动找到对应元素
```

### 2.4 代理轮换

内置 `ProxyRotator`，支持循环或自定义轮换策略，所有 Session 类型均可使用。

### 2.5 CLI & 交互式 Shell

```bash
# 启动交互式爬虫 Shell
scrapling shell

# 直接从命令行提取网页内容（无需写代码）
scrapling extract get 'https://example.com' content.md
scrapling extract get 'https://example.com' content.txt --css-selector '#main'
scrapling extract stealthy-fetch 'https://protected-site.com' data.html --solve-cloudflare
```

---

## 三、安装指南

### 3.1 基础安装（仅解析器）

```bash
pip install scrapling
```

> [!note] 这只包含解析引擎，不包含 Fetcher 和 CLI。

### 3.2 完整安装（推荐）

```bash
# 安装所有 Fetcher 依赖
pip install "scrapling[fetchers]"

# 下载浏览器和系统依赖
scrapling install
```

### 3.3 按需安装

```bash
# MCP Server（AI 集成）
pip install "scrapling[ai]"

# 交互式 Shell 和 CLI extract 命令
pip install "scrapling[shell]"

# 全部功能
pip install "scrapling[all]"
scrapling install
```

### 3.4 Docker 安装

```bash
# 从 DockerHub
docker pull pyd4vinci/scrapling

# 从 GitHub Container Registry
docker pull ghcr.io/d4vinci/scrapling:latest
```

---

## 四、使用示例

### 4.1 基础 HTTP 请求

```python
from scrapling.fetchers import Fetcher, FetcherSession

# 单次请求
page = Fetcher.get('https://quotes.toscrape.com/')
quotes = page.css('.quote .text::text').getall()

# Session 模式（保持 Cookie/状态）
with FetcherSession(impersonate='chrome') as session:
    page = session.get('https://quotes.toscrape.com/', stealthy_headers=True)
    quotes = page.css('.quote .text::text').getall()
```

### 4.2 隐身模式（绕过反爬）

```python
from scrapling.fetchers import StealthyFetcher, StealthySession

# 自动解决 Cloudflare 验证码
with StealthySession(headless=True, solve_cloudflare=True) as session:
    page = session.fetch('https://nopecha.com/demo/cloudflare')
    data = page.css('#padded_content a').getall()

# 单次请求模式
page = StealthyFetcher.fetch('https://protected-site.com', headless=True)
data = page.css('.content').getall()
```

### 4.3 动态页面渲染

```python
from scrapling.fetchers import DynamicFetcher, DynamicSession

with DynamicSession(headless=True, network_idle=True) as session:
    page = session.fetch('https://spa-website.com/')
    data = page.xpath('//span[@class="text"]/text()').getall()
```

### 4.4 Spider（全站爬取）

```python
from scrapling.spiders import Spider, Request, Response

class QuotesSpider(Spider):
    name = "quotes"
    start_urls = ["https://quotes.toscrape.com/"]
    concurrent_requests = 10

    async def parse(self, response: Response):
        for quote in response.css('.quote'):
            yield {
                "text": quote.css('.text::text').get(),
                "author": quote.css('.author::text').get(),
            }

        # 自动翻页
        next_page = response.css('.next a')
        if next_page:
            yield response.follow(next_page[0].attrib['href'])

# 启动爬虫
result = QuotesSpider().start()
print(f"Scraped {len(result.items)} quotes")
result.items.to_json("quotes.json")
```

### 4.5 暂停/恢复长爬虫

```python
# 使用 crawldir 保存断点
QuotesSpider(crawldir="./crawl_data").start()
# Ctrl+C 暂停 → 再次运行相同命令即可恢复
```

---

## 五、作为 AI 工具集成（重点）

Scrapling 提供了 **三种** AI 集成方式，适合不同的使用场景：

### 5.1 方式一：MCP Server（推荐）

> [!tip] 适用场景
> Claude Desktop、Cursor AI、VS Code Copilot 等 MCP 兼容客户端。

Scrapling 内置了 MCP Server，AI 可以直接调用 Scrapling 的抓取能力，先在 Scrapling 端做内容提取再传给 AI，**大幅减少 Token 消耗**。

#### 安装

```bash
# 方式 A：安装 scrapling 的 AI 扩展
pip install "scrapling[ai]"
scrapling install

# 方式 B：安装独立 MCP 包
pip install scrapling-fetch-mcp
```

#### Claude Desktop 配置

编辑配置文件（路径取决于操作系统）：

| 系统 | 配置文件路径 |
|---|---|
| **macOS** | `~/Library/Application Support/Claude/claude_desktop_config.json` |
| **Windows** | `%APPDATA%\Claude\claude_desktop_config.json` |

添加以下内容：

```json
{
  "mcpServers": {
    "scrapling": {
      "command": "uvx",
      "args": ["scrapling-fetch-mcp"]
    }
  }
}
```

或使用 Python 直接运行：

```json
{
  "mcpServers": {
    "scrapling": {
      "command": "python",
      "args": ["-m", "scrapling.mcp"]
    }
  }
}
```

#### Cursor AI 配置

编辑 `~/.cursor/mcp.json`（全局）或项目根目录 `.cursor/mcp.json`（项目级）：

```json
{
  "mcpServers": {
    "scrapling": {
      "command": "uvx",
      "args": ["scrapling-fetch-mcp"]
    }
  }
}
```

#### 注意事项

> [!warning] Python 版本要求
> MCP Server 需要 **Python ≥ 3.12**。低版本会启动失败（参考 [Issue #163](https://github.com/D4Vinci/Scrapling/issues/163)）。

配置完成后**重启客户端**即可生效。

---

### 5.2 方式二：Agent Skill（Claude Code / OpenClaw）

> [!tip] 适用场景
> Claude Code CLI、OpenClaw 等支持 AgentSkill 规范的工具。

Scrapling 官方提供了一个 **Agent Skill**，将几乎所有文档内容打包成 Markdown，使 AI Agent 能准确回答 ~90% 的 Scrapling 使用问题，无需猜测 API。

#### 通过 Clawhub 安装（推荐）

```bash
clawhub install scrapling-official
```

#### 手动安装

1. 下载 Skill ZIP 文件：[Scrapling-Skill.zip](https://github.com/D4Vinci/Scrapling/tree/main/agent-skill/Scrapling-Skill.zip)
2. 解压到你的 Claude Code Skill 目录（通常为 `~/.claude/skills/`）
3. 重启 Claude Code

#### Skill 包含内容

- Scrapling 完整 API 文档（Markdown 格式）
- Fetcher / Spider / Parser 使用指南
- 常见问题解答和最佳实践
- 反检测配置说明

---

### 5.3 方式三：Claude Code 自定义 Skill（.claude/commands/）

如果你使用的是 Claude Code CLI，还可以通过自定义命令的方式集成：

```bash
# 在项目根目录创建自定义命令
mkdir -p .claude/commands
```

然后创建 `.claude/commands/scrape.md`：

```markdown
---
description: 使用 Scrapling 抓取网页数据
---

使用 Python Scrapling 库完成以下抓取任务：

1. 目标 URL：$ARGUMENTS
2. 使用 Fetcher 进行 HTTP 请求
3. 用 CSS 选择器提取关键数据
4. 将结果保存为 JSON 文件

请根据目标网站的结构，自动选择合适的选择器和提取策略。
如果遇到反爬机制，切换到 StealthyFetcher。
```

使用方式：

```bash
claude /scrape https://example.com
```

---

## 六、AI 集成方式对比

| 特性           | MCP Server              | Agent Skill (Clawhub)  | 自定义命令       |
| ------------ | ----------------------- | ---------------------- | ----------- |
| **适合工具**     | Claude Desktop / Cursor | Claude Code / OpenClaw | Claude Code |
| **实时抓取**     | ✅ 是                     | ❌ 知识型                  | ✅ 是         |
| **Token 节省** | ✅ 先提取再传给 AI             | ❌ 加载知识                 | ❌ 按需        |
| **安装复杂度**    | 中等                      | 简单                     | 简单          |
| **离线可用**     | ❌ 需要网络                  | ✅ 是                    | ✅ 是         |
| **功能覆盖**     | 抓取 + 提取                 | 文档知识                   | 灵活自定义       |

**推荐组合**：**同时安装 MCP Server + Agent Skill**，这样 AI 既有实时抓取能力，又了解 Scrapling 的完整 API。

---

## 七、常用代码片段

### 7.1 仅解析 HTML（不需要网络）

```python
from scrapling.parser import Selector

page = Selector("<html><body><h1>Hello</h1></body></html>")
title = page.css('h1::text').get()  # "Hello"
```

### 7.2 异步并发抓取

```python
import asyncio
from scrapling.fetchers import AsyncStealthySession

async def main():
    async with AsyncStealthySession(max_pages=2) as session:
        tasks = [
            session.fetch('https://example.com/page1'),
            session.fetch('https://example.com/page2'),
        ]
        results = await asyncio.gather(*tasks)
        for page in results:
            print(page.css('h1::text').get())

asyncio.run(main())
```

### 7.3 代理轮换

```python
from scrapling.fetchers import FetcherSession, ProxyRotator

proxies = ["http://proxy1:8080", "http://proxy2:8080"]
rotator = ProxyRotator(proxies)

with FetcherSession(proxy_rotator=rotator) as session:
    page = session.get('https://example.com')
```

### 7.4 广告拦截 + DNS 防泄漏

```python
from scrapling.fetchers import DynamicFetcher

page = DynamicFetcher.fetch(
    'https://example.com',
    headless=True,
    block_ads=True,               # 内置 ~3500 广告域名拦截
    block_domains=['tracker.com'],  # 自定义域名拦截
    network_idle=True,
)
```

---

## 八、最佳实践

1. **从简单开始** —— 先用 `Fetcher` 测试，遇到反爬再升级到 `StealthyFetcher`
2. **善用自适应** —— 对需要长期监控的页面，首次用 `auto_save=True`，后续用 `adaptive=True`
3. **用 Session 复用连接** —— 多个请求用 Session 类而非重复创建
4. **开发模式加速** —— Spider 的开发模式会缓存响应，避免反复请求目标服务器
5. **遵守 robots.txt** —— Spider 中设置 `robots_txt_obey = True`
6. **合理设置并发** —— `concurrent_requests` 不要过高，尊重目标服务器

---

## 九、相关资源

| 资源 | 链接 |
|---|---|
| GitHub 仓库 | [D4Vinci/Scrapling](https://github.com/D4Vinci/Scrapling) |
| 官方文档 | [scrapling.readthedocs.io](https://scrapling.readthedocs.io/en/latest/) |
| PyPI | [pypi.org/project/scrapling](https://pypi.org/project/scrapling/) |
| 独立 MCP 包 | [scrapling-fetch-mcp](https://pypi.org/project/scrapling-fetch-mcp/) |
| Agent Skill 目录 | [agent-skill/](https://github.com/D4Vinci/Scrapling/tree/main/agent-skill) |
| Clawhub 页面 | [clawhub.ai/D4Vinci/scrapling-official](https://clawhub.ai/D4Vinci/scrapling-official) |
| Discord 社区 | [discord.gg/EMgGbDceNQ](https://discord.gg/EMgGbDceNQ) |
| 中文 README | [README_CN.md](https://github.com/D4Vinci/Scrapling/blob/main/docs/README_CN.md) |

---

> [!quote] 引用格式
> ```bibtex
> @misc{scrapling,
>   author = {Karim Shoair},
>   title = {Scrapling},
>   year = {2024},
>   url = {https://github.com/D4Vinci/Scrapling},
>   note = {An adaptive Web Scraping framework that handles everything from a single request to a full-scale crawl!}
> }
> ```
