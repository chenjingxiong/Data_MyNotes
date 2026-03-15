# steel-browser - AI 智能体开源浏览器 API

> 官方仓库：https://github.com/steel-dev/steel-browser
> 官方网站：https://www.steel.dev/

---

## 项目简介

**Steel Browser** 是一个开源的浏览器 API，专为构建 AI 智能体和应用程序而设计。它提供了一个"开箱即用"（batteries-included）的浏览器沙盒环境，让你能够在无需担心基础设施的情况下自动化 Web 操作。

### 核心特性

- **🌐 完整的浏览器控制** - 使用 Puppeteer 和 CDP 协议完全控制 Chrome 实例
- **🔄 会话管理** - 跨请求维护浏览器状态、Cookie 和本地存储
- **🔌 代理支持** - 内置代理链管理，支持 IP 轮换
- **🧩 扩展支持** - 加载自定义 Chrome 扩展以增强功能
- **🐛 调试工具** - 内置请求日志记录和 UI 用于查看/调试会话
- **🛡️ 反检测功能** - 包含隐身插件和指纹管理
- **🧹 资源管理** - 自动清理和浏览器生命周期管理
- **📊 浏览器工具** - 快速将页面转换为 Markdown、可读文本、截图或 PDF

---

## 快速部署

### 方式一：Docker 部署（推荐）

```bash
# 拉取并运行 Docker 镜像
docker run -p 3000:3000 -p 9223:9223 ghcr.io/steel-dev/steel-browser
```

这将启动：
- **API 服务器**：http://localhost:3000
- **Web UI**：http://localhost:3000/ui
- **调试端口**：9223

### 方式二：云平台一键部署

| 部署方式 | 链接 |
|---------|------|
| Docker 镜像 | ![GitHub Container Registry](https://img.shields.io/badge/GHCR-474646?style=for-the-badge&labelColor=333&logo=github&logoColor=white) |
| Railway | ![Railway](https://img.shields.io/badge/Railway-B0394B?style=for-the-badge&labelColor=B0394B&logo=railway&logoColor=white) |
| Render | ![Render](https://img.shields.io/badge/Deploy%20to%20Render-4E2C89?style=for-the-badge) |

### 方式三：本地开发运行

#### 使用 Docker Compose（开发者）

```bash
# Mac Silicon 用户需要指定平台
DOCKER_DEFAULT_PLATFORM=linux/arm64 docker compose up

# 或者使用开发模式
docker compose -f docker-compose.dev.yml up --build
```

#### 使用 Docker Compose（生产部署）

创建 `docker-compose.yml` 文件：

```yaml
services:
  api:
    image: ghcr.io/steel-dev/steel-browser-api:latest
    ports:
      - "3000:3000"
      - "9223:9223"
    volumes:
      - ./.cache:/app/.cache
    networks:
      - steel-network

  ui:
    image: ghcr.io/steel-dev/steel-browser-ui:latest
    ports:
      - "5173:80"
    depends_on:
      - api
    networks:
      - steel-network

networks:
  steel-network:
    name: steel-network
    driver: bridge
```

启动服务：

```bash
docker compose up -d
```

访问地址：
- **API 服务**：http://localhost:3000
- **Web UI**：http://localhost:5173
- **调试端口**：9223

#### 使用 Node.js

```bash
# 安装依赖
npm install

# 运行开发服务器
npm run dev
```

**Chrome 可执行文件路径要求**：
- **Linux**: `/usr/bin/google-chrome`
- **macOS**: `/Applications/Google Chrome.app/Contents/MacOS/Google Chrome`
- **Windows**:
  - `C:\Program Files\Google\Chrome\Application\chrome.exe`
  - `C:\Program Files (x86)\Google\Chrome\Application\chrome.exe`

**自定义 Chrome 路径**：
```bash
export CHROME_EXECUTABLE_PATH=/path/to/your/chrome
npm run dev
```

---

## 安装 SDK

### Node.js SDK

```bash
npm install steel-sdk
```

### Python SDK

```bash
pip install steel-sdk
```

---

## 快速开始

### 1. 使用会话（Sessions）

会话允许你使用自定义选项或扩展（例如自定义代理）重新启动浏览器，并重置浏览器状态。适合需要精细控制的复杂有状态工作流。

#### Node.js 示例

```javascript
import Steel from 'steel-sdk';

const client = new Steel({
  baseURL: "http://localhost:3000" // 自定义 API Base URL
});

(async () => {
  try {
    // 创建新的浏览器会话
    const session = await client.sessions.create({
      blockAds: true,
      proxyUrl: "user:pass@host:port", // 可选
      dimensions: { width: 1280, height: 800 }, // 可选
    });
    console.log("创建的会话 ID:", session.id);
  } catch (error) {
    console.error("创建会话错误:", error);
  }
})();
```

#### Python 示例

```python
from steel import Steel

client = Steel(
    base_url="http://localhost:3000"  # 自定义 API Base URL
)

try:
    # 创建新的浏览器会话
    session = client.sessions.create(
        block_ads=True,
        proxy_url="user:pass@host:port",  # 可选
        dimensions={"width": 1280, "height": 800},  # 可选
    )
    print("创建的会话 ID:", session.id)
except Exception as e:
    print("创建会话错误:", e)
```

#### cURL 示例

```bash
# 启动新的浏览器会话
curl -X POST http://localhost:3000/v1/sessions \
  -H "Content-Type: application/json" \
  -d '{
    "proxyUrl": "user:pass@host:port",
    "blockAds": true,
    "dimensions": { "width": 1280, "height": 800 }
  }'
```

### 2. Selenium 会话

对于现有 Selenium 工作流，Steel 提供了完全兼容的替代方案：

```javascript
// 使用 Node SDK
const session = await client.sessions.create({ isSelenium: true });
```

```python
# 使用 Python SDK
session = client.sessions.create(is_selenium=True)
```

```bash
# 使用 cURL
curl -X POST http://localhost:3000/v1/sessions \
  -H "Content-Type: application/json" \
  -d '{
    "isSelenium": true
  }'
```

### 3. 快速操作 API

`/scrape`、`/screenshot` 和 `/pdf` 端点让你从任何网页快速提取干净、格式良好的数据。

#### 抓取网页

```bash
curl -X POST http://0.0.0.0:3000/v1/scrape \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://example.com",
    "delay": 1000
  }'
```

#### 截取屏幕截图

```bash
curl -X POST http://0.0.0.0:3000/v1/screenshot \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://example.com",
    "fullPage": true
  }' --output screenshot.png
```

#### 下载 PDF

```bash
curl -X POST http://0.0.0.0:3000/v1/pdf \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://example.com"
  }' --output output.pdf
```

---

## AI 工具集成

### Claude Code 集成

将 Steel 作为技能添加到 Claude Code：

```bash
npx skills add steel-dev/steel-browser
```

这会将技能安装到 `.claude/skills/steel-browser/SKILL.md` 中，让 Claude Code 学会完整的 Steel 工作流程。

### OpenClaw 集成

在 OpenClaw 中，可以通过 REST API 直接调用 Steel：

```bash
# 创建会话
SESSION_ID=$(curl -X POST http://localhost:3000/v1/sessions \
  -H "Content-Type: application/json" \
  -d '{"blockAds": true}' | jq -r '.id')

# 使用会话进行浏览器操作
# ... 与 Puppeteer/Playwright 集成
```

### AGENTS.md / CLAUDE.md 配置

将以下内容添加到项目或全局指令文件：

```markdown
## 浏览器自动化

使用 `Steel Browser` 进行 Web 自动化。API 地址：`http://localhost:3000`

核心工作流程：

1. 创建会话：`POST /v1/sessions` - 获取会话 ID
2. 使用 Puppeteer/Playwright 连接到会话
3. 执行浏览器操作（导航、点击、填写表单等）
4. 快速操作：
   - `/v1/scrape` - 提取页面内容
   - `/v1/screenshot` - 截图
   - `/v1/pdf` - 生成 PDF

完整文档：http://localhost:3000/documentation
```

---

## 主要功能

### 完整浏览器控制

- 支持 Puppeteer、Playwright 和 Selenium 连接
- 使用 Chrome DevTools Protocol (CDP) 进行底层控制
- 完全控制 Chrome 实例

### 会话管理

- 跨请求维护浏览器状态
- Cookie 和本地存储持久化
- 支持多个并发会话

### 代理支持

- 内置代理链管理
- IP 轮换功能
- 支持认证代理（`user:pass@host:port`）

### 扩展支持

- 加载自定义 Chrome 扩展
- 增强浏览器功能
- 支持多个扩展

### 调试工具

- 内置请求日志记录
- Web UI 查看和调试会话
- Chrome DevTools 集成

### 反检测功能

- 隐身插件（stealth plugins）
- 指纹管理
- 避免机器人检测

---

## API 端点概览

| 端点 | 方法 | 描述 |
|------|------|------|
| `/v1/sessions` | POST | 创建新会话 |
| `/v1/sessions/:id` | GET | 获取会话信息 |
| `/v1/sessions/:id` | DELETE | 删除会话 |
| `/v1/scrape` | POST | 抓取网页内容 |
| `/v1/screenshot` | POST | 截取屏幕截图 |
| `/v1/pdf` | POST | 生成 PDF |

完整的 OpenAPI 文档可在 `http://localhost:3000/documentation` 查看。

---

## 与 Puppeteer/Playwright 集成

### Puppeteer 集成

创建会话后，可以使用 Puppeteer 连接：

```javascript
const puppeteer = require('puppeteer');

// 使用 Steel 提供的 WebSocket URL 连接
const browser = await puppeteer.connect({
  browserWSEndpoint: session.wsUrl
});

const page = await browser.newPage();
await page.goto('https://example.com');
```

### Playwright 集成

```javascript
const { chromium } = require('playwright');

// 使用 Steel 提供的 WebSocket URL 连接
const browser = await chromium.connect(session.wsUrl);

const page = await browser.newPage();
await page.goto('https://example.com');
```

---

## 配置选项

### 会话创建选项

```javascript
{
  blockAds: boolean,        // 是否屏蔽广告
  proxyUrl: string,         // 代理 URL（格式：user:pass@host:port）
  dimensions: {
    width: number,          // 浏览器宽度
    height: number          // 浏览器高度
  },
  isSelenium: boolean,      // 是否创建 Selenium 会话
  extensions: string[]      // 要加载的扩展列表
}
```

### 快速操作选项

```javascript
{
  url: string,              // 目标 URL
  delay: number,            // 延迟（毫秒）
  fullPage: boolean,        // 是否截取整个页面
  // ... 其他选项
}
```

---

## 社区与支持

- **文档**: https://www.steel.dev/docs
- **Cookbook**: https://www.steel.dev/cookbook
- **Discord**: ![Discord](https://img.shields.io/discord/1285696350117167226?label=discord)
- **Twitter**: [@steeldotdev](https://twitter.com/steeldotdev)

### 贡献指南

Steel Browser 是开源项目，欢迎贡献！
- 有问题/想法/反馈？加入 [Discord](https://discord.gg/steel)
- 发现 Bug？在 GitHub 上提 Issue

---

## 项目数据

- **许可协议**: Apache 2.0
- **GitHub Stars**: ![GitHub stars](https://img.shields.io/github/stars/steel-dev/steel-browser)
- **最新版本**: 查看 [Releases](https://github.com/steel-dev/steel-browser/releases)

---

## 与 agent-browser 对比

| 特性 | steel-browser | agent-browser |
|------|---------------|---------------|
| **架构** | REST API + Server | CLI 工具 |
| **协议** | Puppeteer/Playwright/Selenium | CDP |
| **部署** | Docker/云平台 | 本地安装 |
| **会话管理** | ✅ 内置 | ✅ 支持 |
| **代理支持** | ✅ 内置 | ✅ 支持 |
| **扩展支持** | ✅ 支持 | ✅ 支持 |
| **UI 界面** | ✅ Web UI | ❌ 命令行 |
| **反检测** | ✅ 隐身插件 | ✅ 支持 |
| **适合场景** | 服务器/API 部署 | 本地开发/CLI 工具 |

---

## 标签

`#browser-automation` `#AI-agent` `#API` `#Puppeteer` `#Playwright` `#Selenium` `#web-scraping` `#open-source`
