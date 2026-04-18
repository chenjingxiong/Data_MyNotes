# Playwright CLI 完全指南

> 官方文档：https://playwright.dev/docs/cli
> 官方仓库：https://github.com/microsoft/playwright

---

## 项目简介

**Playwright CLI** 是 Microsoft 开源的端到端测试框架 Playwright 附带的命令行工具集。它不仅能运行测试，还提供了**截图、PDF 生成、代码录制、浏览器调试**等实用功能，是浏览器自动化和 Web 测试的一站式工具。

### 核心特性

- **🌐 跨浏览器** - 支持 Chromium、Firefox、WebKit 三大引擎
- **🎬 代码录制** - `codegen` 录制用户操作，自动生成测试脚本
- **📸 截图 / PDF** - 一条命令即可对网页截图或导出 PDF
- **🔧 调试工具** - Inspector + UI Mode + Trace Viewer 三位一体
- **⚡ 自动等待** - 内置智能等待机制，无需手动 `sleep`
- **📦 零配置** - `npx playwright install` 即可安装所有浏览器

---

## 安装

### 前置条件

- Node.js >= 18
- npm / yarn / pnpm

### 安装 Playwright

```bash
# 方式一：全局安装（推荐 CLI 用户）
npm install -g playwright

# 方式二：项目内安装（推荐测试项目）
npm init playwright@latest

# 方式三：仅安装 CLI
npx playwright --version   # npx 会自动拉取
```

### 安装浏览器二进制文件

```bash
# 安装所有浏览器（Chromium + Firefox + WebKit）
npx playwright install

# 仅安装指定浏览器
npx playwright install chromium
npx playwright install firefox
npx playwright install webkit

# 安装浏览器所需的系统依赖（Linux 上必需）
npx playwright install-deps

# 仅安装系统依赖，不下载浏览器
npx playwright install-deps --dry-run
```

> [!tip] 国内加速
> 如果下载慢，可以设置镜像源：
> ```bash
> export PLAYWRIGHT_DOWNLOAD_HOST=https://npmmirror.com/mirrors/playwright
> npx playwright install
> ```

---

## 命令总览

```text
npx playwright <command>

Commands:
  open          在指定浏览器中打开 URL（交互调试）
  codegen       录制操作并自动生成测试代码
  screenshot    对网页截图
  pdf           将网页导出为 PDF（仅 Chromium）
  test          运行测试用例
  show-report   打开 HTML 测试报告
  show-trace    打开 Trace Viewer 查看执行轨迹
  install       安装浏览器二进制文件
  install-deps  安装系统依赖
```

---

## 1. `open` — 打开浏览器调试

在本地浏览器中打开一个 URL，用于**手动调试和检查**页面行为。会启动一个**持久化上下文**（persistent context），保留 Cookie、localStorage 等。

```bash
# 基本用法
npx playwright open https://example.com

# 指定浏览器
npx playwright open --browser chromium https://example.com
npx playwright open --browser firefox https://example.com
npx playwright open --browser webkit https://example.com

# 模拟移动设备
npx playwright open --device "iPhone 15" https://example.com

# 指定视口大小
npx playwright open --viewport-size "1280,720" https://example.com

# 指定配色方案
npx playwright open --color-scheme dark https://example.com
```

> [!note] 适用场景
> - 快速检查页面在 Playwright 浏览器中的渲染效果
> - 手动验证登录流程、Cookie 行为
> - 配合 Inspector 进行逐步调试

---

## 2. `codegen` — 录制并生成代码

**最强大的功能之一**。打开浏览器窗口，录制你的所有操作（点击、输入、导航等），并实时生成对应的测试代码。

```bash
# 基本用法：录制并输出到终端
npx playwright codegen https://example.com

# 保存到文件
npx playwright codegen --output test.spec.ts https://example.com

# 指定浏览器
npx playwright codegen --browser firefox https://example.com

# 指定生成语言 / 框架
npx playwright codegen --target python https://example.com
npx playwright codegen --target java https://example.com
npx playwright codegen --target csharp https://example.com
npx playwright codegen --target playwright-test https://example.com   # 默认

# 模拟设备
npx playwright codegen --device "Pixel 7" https://example.com

# 设置配色方案
npx playwright codegen --color-scheme dark https://example.com

# 设置语言环境
npx playwright codegen --lang "zh-CN" https://example.com

# 指定视口
npx playwright codegen --viewport-size "1920,1080" https://example.com

# 组合使用
npx playwright codegen \
  --browser chromium \
  --device "iPhone 15" \
  --color-scheme light \
  --output tests/mobile-home.spec.ts \
  https://example.com
```

### 支持的 `--target` 参数

| 值 | 说明 |
|----|------|
| `playwright-test` | Playwright Test（默认，推荐） |
| `python` | Python (async API) |
| `python-async` | Python (async，同 `python`) |
| `java` | Java (JUnit) |
| `csharp` | C# (NUnit) |
| `csharp-mstest` | C# (MSTest) |
| `csharp-xunit` | C# (xUnit) |

> [!tip] 技巧
> 录制完成后，直接将生成的代码复制到你的测试项目中，稍作修改即可使用。这是编写测试脚本的**最快方式**。

---

## 3. `screenshot` — 网页截图

一条命令对网页进行截图，支持全页截图和视口截图。

```bash
# 基本用法
npx playwright screenshot https://example.com screenshot.png

# 全页截图（滚动截取完整页面）
npx playwright screenshot --full-page https://example.com full.png

# 指定浏览器
npx playwright screenshot --browser firefox https://example.com shot.png

# 指定视口大小
npx playwright screenshot --viewport-size "1920,1080" https://example.com shot.png

# 指定配色方案
npx playwright screenshot --color-scheme dark https://example.com dark.png

# 等待超时（毫秒）
npx playwright screenshot --wait-for-timeout 3000 https://example.com shot.png

# 模拟设备
npx playwright screenshot --device "iPhone 15" https://example.com mobile.png

# 输出格式：支持 png / jpeg / webp
npx playwright screenshot https://example.com shot.webp
```

### 截图选项一览

| 选项 | 说明 | 示例 |
|------|------|------|
| `--browser` | 浏览器引擎 | `chromium`, `firefox`, `webkit` |
| `--full-page` | 完整页面截图 | — |
| `--viewport-size` | 视口尺寸 | `"1280,720"` |
| `--color-scheme` | 配色 | `light`, `dark` |
| `--device` | 模拟设备 | `"iPhone 15"` |
| `--wait-for-timeout` | 等待时间(ms) | `3000` |
| `--output` | 输出文件路径 | `./shots/page.png` |

> [!example] 批量截图脚本
> ```bash
> #!/bin/bash
> # 对多个页面同时截图
> URLS=("https://example.com" "https://example.com/about" "https://example.com/contact")
> for url in "${URLS[@]}"; do
>   name=$(echo "$url" | sed 's|.*/||')
>   npx playwright screenshot --full-page "$url" "screenshots/${name:-home}.png"
> done
> ```

---

## 4. `pdf` — 导出 PDF

将网页保存为 PDF 文件。**仅支持 Chromium 浏览器**。

```bash
# 基本用法
npx playwright pdf https://example.com output.pdf

# 指定纸张格式
npx playwright pdf --format A4 https://example.com output.pdf

# 指定纸张方向（横向）
npx playwright pdf --landscape https://example.com output.pdf

# 显示页眉页脚
npx playwright pdf --display-header-footer https://example.com output.pdf

# 打印背景色
npx playwright pdf --print-background https://example.com output.pdf

# 设置缩放
npx playwright pdf --scale 0.8 https://example.com output.pdf

# 设置页边距
npx playwright pdf --margin "top=20mm,right=15mm,bottom=20mm,left=15mm" https://example.com output.pdf

# 组合使用
npx playwright pdf \
  --format A4 \
  --print-background \
  --display-header-footer \
  --margin "top=20mm,bottom=20mm,left=15mm,right=15mm" \
  https://example.com report.pdf
```

### PDF 选项一览

| 选项 | 说明 | 默认值 |
|------|------|--------|
| `--format` | 纸张大小 | `Letter` |
| `--landscape` | 横向模式 | `false` |
| `--display-header-footer` | 显示页眉页脚 | `false` |
| `--print-background` | 打印背景 | `false` |
| `--scale` | 缩放比例 | `1` |
| `--margin` | 页边距 | — |
| `--viewport-size` | 视口尺寸 | `1280,720` |

> [!warning] 仅 Chromium
> PDF 功能底层依赖 Chromium 的打印 API，Firefox 和 WebKit 不支持。

---

## 5. `test` — 运行测试

Playwright 的核心命令，用于运行端到端测试。

```bash
# 运行所有测试
npx playwright test

# 运行指定文件
npx playwright test tests/login.spec.ts

# 运行匹配模式的测试
npx playwright test --grep "登录"

# UI 模式（推荐开发时使用）
npx playwright test --ui

# 调试模式
npx playwright test --debug

# 指定项目（浏览器）
npx playwright test --project=chromium
npx playwright test --project="Mobile Safari"

# 并行/串行
npx playwright test --workers=4      # 4 个 worker 并行
npx playwright test --workers=1      # 串行执行

# 失败重试
npx playwright test --retries=3

# 输出详细日志
npx playwright test --reporter=list
npx playwright test --reporter=html
npx playwright test --reporter=json

# 仅运行失败的测试
npx playwright test --last-failed

# 查看测试报告
npx playwright show-report
```

### 测试配置文件 `playwright.config.ts`

```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests',
  fullyParallel: true,
  retries: process.env.CI ? 2 : 0,
  reporter: 'html',
  use: {
    baseURL: 'http://localhost:3000',
    trace: 'on-first-retry',       // 失败时自动录制 trace
    screenshot: 'only-on-failure',  // 失败时自动截图
    video: 'retain-on-failure',     // 失败时保留视频
  },
  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
    { name: 'firefox',  use: { ...devices['Desktop Firefox'] } },
    { name: 'mobile',   use: { ...devices['iPhone 15'] } },
  ],
});
```

---

## 6. `show-trace` — 查看执行轨迹

测试失败时查看详细的执行轨迹（DOM 快照、网络请求、控制台日志等）。

```bash
# 打开 Trace Viewer
npx playwright show-trace trace.zip

# 查看测试执行后自动生成的 trace
npx playwright test --trace on
npx playwright show-trace   # 会自动定位到 trace 文件
```

> [!info] Trace 内容
> Trace Viewer 包含：
> - 📸 每一步操作的 DOM 快照
> - 🌐 完整的网络请求/响应记录
> - 📋 控制台日志
> - ⏱️ 时间线视图
> - 🔍 元素选择器高亮

---

## 7. `show-report` — 查看测试报告

```bash
# 生成并打开 HTML 报告
npx playwright show-report

# 指定报告目录
npx playwright show-report ./test-results
```

---

## 8. 保存与复用登录状态（Storage State）

Playwright 默认每次启动都是**干净的浏览器上下文**（没有 Cookie、没有 localStorage），这意味着每次测试都需要重新登录。这在实际项目中效率极低，而且某些登录流程涉及验证码、二步验证等难以自动化的步骤。

Playwright 提供了 **Storage State** 机制来解决这个问题：**一次登录，到处复用**。

### 工作原理

```
┌──────────────┐    登录 + 操作     ┌──────────────────┐
│  手动/自动登录  │ ───────────────→ │  浏览器上下文       │
│  (首次)       │                   │  (Cookie + Storage)│
└──────────────┘                   └────────┬─────────┘
                                            │
                                   storageState.save()
                                            │
                                            ▼
                                   ┌──────────────────┐
                                   │  auth.json        │
                                   │  {                │
                                   │    cookies: [...],│
                                   │    origins: [...] │
                                   │  }                │
                                   └────────┬─────────┘
                                            │
                                   storageState 加载
                                            │
                                            ▼
┌──────────────┐    自动跳过登录   ┌──────────────────┐
│  后续测试/脚本 │ ───────────────→ │  已认证的上下文     │
│  (复用状态)   │                   │  (直接开始操作)     │
└──────────────┘                   └──────────────────┘
```

### 方式一：全局 Setup 保存状态（推荐）

**步骤 1** — 创建全局 Setup 脚本 `global-setup.ts`：

```typescript
import { chromium, type FullConfig } from '@playwright/test';

async function globalSetup(config: FullConfig) {
  // 启动浏览器
  const browser = await chromium.launch();
  const page = await browser.newPage();

  // 执行登录流程
  await page.goto('https://example.com/login');
  await page.fill('[name="username"]', 'myuser');
  await page.fill('[name="password"]', 'mypassword');
  await page.click('button[type="submit"]');

  // 等待登录完成
  await page.waitForURL('**/dashboard');

  // ★ 保存登录状态到文件
  await page.context().storageState({ path: 'auth.json' });

  await browser.close();
}

export default globalSetup;
```

**步骤 2** — 在 `playwright.config.ts` 中引用：

```typescript
import { defineConfig } from '@playwright/test';

export default defineConfig({
  globalSetup: require.resolve('./global-setup'),
  use: {
    // ★ 让所有测试复用保存的登录状态
    storageState: 'auth.json',
  },
});
```

**步骤 3** — 测试文件中无需再登录：

```typescript
import { test, expect } from '@playwright/test';

test('已登录状态下的操作', async ({ page }) => {
  // 直接访问需要认证的页面，无需登录！
  await page.goto('https://example.com/dashboard');
  await expect(page.locator('.welcome')).toContainText('Welcome');
});
```

### 方式二：多角色状态（项目级配置）

当系统有多种角色（管理员、普通用户、VIP 等）时，可以保存多份状态：

```typescript
// playwright.config.ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  projects: [
    {
      name: 'admin-setup',
      testMatch: /admin-setup\.ts/,
    },
    {
      name: 'admin-tests',
      dependencies: ['admin-setup'],   // 先运行 setup
      testMatch: /admin.*\.spec\.ts/,
      use: {
        storageState: '.auth/admin.json',  // 使用管理员状态
      },
    },
    {
      name: 'user-setup',
      testMatch: /user-setup\.ts/,
    },
    {
      name: 'user-tests',
      dependencies: ['user-setup'],
      testMatch: /user.*\.spec\.ts/,
      use: {
        storageState: '.auth/user.json',   // 使用普通用户状态
      },
    },
  ],
});
```

```typescript
// admin-setup.ts
import { test as setup } from '@playwright/test';

setup('管理员登录', async ({ page }) => {
  await page.goto('/login');
  await page.fill('[name="username"]', 'admin');
  await page.fill('[name="password"]', 'admin-pass');
  await page.click('button[type="submit"]');
  await page.waitForURL('**/admin/dashboard');
  await page.context().storageState({ path: '.auth/admin.json' });
});
```

### 方式三：手动录制登录状态

如果登录流程涉及验证码、短信验证等**无法自动化的步骤**，可以手动完成登录后保存状态：

```bash
# 1. 打开浏览器（有头模式），手动操作
npx playwright open --browser chromium https://example.com/login

# 2. 手动完成登录（输入验证码、扫码等）
# ...

# 3. 登录成功后，在浏览器 Console 中执行：
# （打开 DevTools → Console）
```

```javascript
// 在浏览器 Console 中执行，导出当前 Cookie
(async () => {
  const cookies = await cookieStore.getAll();
  const state = {
    cookies: cookies.map(c => ({
      name: c.name,
      value: c.value,
      domain: c.domain || location.hostname,
      path: c.path || '/',
      expires: c.expires || -1,
      httpOnly: false,
      secure: c.secure || false,
      sameSite: c.sameSite || 'Lax',
    })),
    origins: [],
  };
  // 复制输出并保存为 auth.json
  console.log(JSON.stringify(state, null, 2));
})();
```

### 方式四：从真实浏览器导出 Cookie（借助 EditThisCookie 等）

```text
1. 在 Chrome 中安装 EditThisCookie 扩展
2. 登录目标网站
3. 点击扩展 → 导出 Cookie（JSON 格式）
4. 将导出的内容转换为 Playwright 的 storageState 格式
5. 保存为 auth.json
```

### Storage State 文件格式

```json
{
  "cookies": [
    {
      "name": "session_id",
      "value": "abc123xyz",
      "domain": ".example.com",
      "path": "/",
      "expires": 1735689600,
      "httpOnly": true,
      "secure": true,
      "sameSite": "Lax"
    }
  ],
  "origins": [
    {
      "origin": "https://example.com",
      "localStorage": [
        { "name": "token", "value": "eyJhbGc..." }
      ]
    }
  ]
}
```

> [!warning] 安全提醒
> - `auth.json` 包含敏感凭证，**切勿提交到 Git**
> - 添加到 `.gitignore`：`echo "auth.json" >> .gitignore`（也建议忽略 `.auth/` 目录）
> - Cookie 有过期时间，失效后需要重新生成
> - 定期更新 Storage State 文件

> [!tip] 状态刷新策略
> ```bash
> # 在 CI 中每次运行前重新生成
> npx playwright test --config=setup.config.ts    # 先运行登录脚本
> npx playwright test                              # 再运行测试
>
> # 或使用 globalSetup 自动处理（推荐）
> ```

### 常见登录状态问题

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| Cookie 过期 | Session 有 TTL | 定期重新生成 Storage State |
| 状态文件找不到 | 路径不匹配 | 使用相对路径，检查 `storageState` 配置 |
| CSRF Token 失效 | 单次使用 Token | 在 setup 中获取后立即使用 |
| localStorage 丢失 | 跨域限制 | 检查 `origins` 配置 |
| 验证码登录 | 无法自动化 | 使用手动录制方式（方式三） |

---

## 9. 与 Claude Code 结合使用

Playwright 与 Claude Code 的结合有两种主要方式：**Playwright MCP Server**（AI 直接操控）和 **Playwright 脚本生成**（辅助编写代码）。

### 方式一：Playwright MCP Server（推荐）

Playwright 官方提供了 MCP Server，让 Claude Code 可以直接通过自然语言操控浏览器，无需手写代码。

#### 架构

```
┌──────────────┐        MCP Protocol        ┌─────────────────────┐
│  Claude Code │ ◄──────────────────────────► │ Playwright MCP Server│
│  (AI Agent)  │                              │   (Node.js)          │
└──────────────┘                              └──────────┬──────────┘
                                                         │
                                              Playwright API (CDP)
                                                         │
                                                         ▼
                                              ┌─────────────────────┐
                                              │  Chromium / Firefox  │
                                              │  / WebKit 浏览器      │
                                              └─────────────────────┘
```

#### 安装与配置

**步骤 1** — 在 Claude Code 中配置 MCP Server：

编辑 `~/.claude/settings.json` 或项目根目录的 `.claude/settings.json`：

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["@anthropic-ai/mcp-playwright"]
    }
  }
}
```

或者使用 `claude` CLI 添加：

```bash
claude mcp add playwright -- npx @anthropic-ai/mcp-playwright
```

**步骤 2** — 验证安装：

在 Claude Code 中输入：

```
> 使用 Playwright 打开 https://example.com 并截图
```

如果配置成功，Claude 会调用 Playwright MCP 完成操作。

#### MCP 模式 vs Chrome DevTools MCP

| 特性 | Playwright MCP | Chrome DevTools MCP |
|------|---------------|---------------------|
| **浏览器** | Chromium / Firefox / WebKit | 仅 Chrome |
| **启动方式** | Playwright 管理的浏览器实例 | 连接已有 Chrome |
| **登录状态** | 需通过 Storage State 管理 | 天然保留（真实 Chrome Profile） |
| **隔离性** | ✅ 每次干净的沙盒环境 | ❌ 共享真实浏览器状态 |
| **跨浏览器** | ✅ 支持 | ❌ 仅 Chromium |
| **适用场景** | 测试自动化、干净环境 | 已登录网站操作、调试 |
| **安装复杂度** | ⭐ 一行命令 | ⭐⭐⭐ 需装扩展 |

> [!tip] 选型建议
> - **需要操作已登录的网站**（Gmail、GitHub 等）→ 选 **Chrome DevTools MCP**，直接使用真实浏览器 Profile
> - **需要干净测试环境 / 跨浏览器** → 选 **Playwright MCP**，配合 Storage State 管理登录
> - **两者可以同时配置**，Claude 会根据任务自动选择

#### Playwright MCP 常用操作

```text
# 页面导航
> 打开 https://example.com
> 刷新页面
> 返回上一页

# 元素交互
> 点击页面上的"登录"按钮
> 在搜索框中输入"Playwright"
> 填写表单：姓名 张三，邮箱 test@example.com

# 信息获取
> 截取当前页面
> 获取页面的文本内容
> 列出页面上的所有链接

# 测试辅助
> 验证页面标题是否包含"Welcome"
> 检查登录按钮是否可见
> 等待数据表格加载完成后截图
```

### 方式二：Claude Code 辅助编写 Playwright 脚本

不用 MCP，直接让 Claude Code 帮你编写、调试 Playwright 测试代码。

#### 典型工作流

```text
你: 帮我写一个 Playwright 测试，登录 GitHub 后检查是否有通知

Claude: [生成完整测试代码]
        → 使用 codegen 录制登录流程
        → 添加断言和等待
        → 保存为 tests/github-notification.spec.ts

你: 运行这个测试

Claude: npx playwright test tests/github-notification.spec.ts
        [查看输出，如有错误则修复]
```

#### 实战：让 Claude Code 帮你完成测试全流程

```text
1. 录制 → "用 playwright codegen 录制 https://myapp.com 的登录流程"
2. 生成 → "根据录制的操作生成测试代码，使用 Page Object Model"
3. 增强 → "给测试加上断言，验证登录后的用户名显示正确"
4. 调试 → "运行测试，失败了就分析截图和 trace"
5. 完善 → "添加多浏览器支持、失败重试、报告生成"
```

### 方式三：Playwright 脚本 + Claude Code Agent 循环

将 Playwright 作为 Claude Code 的**工具**，让 AI 自主完成复杂的浏览器任务：

```markdown
<!-- .claude/instructions.md -->
## 浏览器自动化规则

当需要操作浏览器时，编写并运行 Playwright 脚本：
1. 使用 TypeScript 编写脚本到 scripts/browser/
2. 使用 `npx tsx scripts/browser/xxx.ts` 运行
3. 脚本中使用 storageState 复用登录状态
4. 操作完成后截图验证结果
5. 如果失败，分析截图后自动修复
```

示例脚本 `scripts/browser/export-report.ts`：

```typescript
import { chromium } from 'playwright';

async function exportReport() {
  // 复用已保存的登录状态
  const browser = await chromium.launch({ headless: true });
  const context = await browser.newContext({
    storageState: 'auth.json',       // 加载登录状态
    viewport: { width: 1920, height: 1080 },
  });
  const page = await context.newPage();

  try {
    // 导航到报表页面
    await page.goto('https://myapp.com/reports/monthly');

    // 等待报表加载
    await page.waitForSelector('.report-loaded');

    // 导出 PDF
    await page.pdf({
      path: 'monthly-report.pdf',
      format: 'A4',
      printBackground: true,
    });

    // 截图留档
    await page.screenshot({
      path: 'monthly-report.png',
      fullPage: true,
    });

    console.log('✅ 报表导出完成');
  } catch (error) {
    // 失败时截图帮助调试
    await page.screenshot({ path: 'error-screenshot.png' });
    console.error('❌ 导出失败:', error);
    throw error;
  } finally {
    await browser.close();
  }
}

exportReport();
```

### 组合方案：三种方式互补

```
┌─────────────────────────────────────────────────────────┐
│                   Claude Code                            │
│                                                          │
│  ┌─────────────────┐  快速探索 / 已登录网站操作           │
│  │ Chrome DevTools  │  → 直接操控真实 Chrome              │
│  │ MCP              │  → 保留所有登录状态                  │
│  └─────────────────┘                                     │
│                                                          │
│  ┌─────────────────┐  测试自动化 / 跨浏览器               │
│  │ Playwright MCP   │  → 干净隔离的浏览器环境              │
│  │                  │  → 配合 Storage State 管理登录       │
│  └─────────────────┘                                     │
│                                                          │
│  ┌─────────────────┐  复杂任务 / 定时脚本                  │
│  │ Playwright 脚本  │  → Claude 生成 + 自主运行            │
│  │ + Agent Loop     │  → 失败自动重试 + 截图分析           │
│  └─────────────────┘                                     │
└─────────────────────────────────────────────────────────┘
```

### 相关笔记

- [[AI技术/ClaudeCode/ClaudeCode浏览器自动化指南]] — Chrome DevTools MCP 详细配置
- [[AI技术/浏览器Agent技术/多台机器浏览器同步登录状态指南]] — 多设备 Cookie 同步方案

---

## 高级用法

### 与 CI/CD 集成

```yaml
# GitHub Actions 示例
name: Playwright Tests
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm ci
      - run: npx playwright install --with-deps
      - run: npx playwright test
      - uses: actions/upload-artifact@v4
        if: always()
        with:
          name: playwright-report
          path: playwright-report/
          retention-days: 30
```

### 使用环境变量

```bash
# 设置浏览器下载镜像
export PLAYWRIGHT_DOWNLOAD_HOST=https://npmmirror.com/mirrors/playwright

# 跳过浏览器下载（已预装时）
export PLAYWRIGHT_SKIP_BROWSER_DOWNLOAD=1

# 指定 Chromium 可执行文件路径
export CHROMIUM_EXECUTABLE_PATH=/usr/bin/chromium-browser

# 调试模式
export DEBUG=pw:api
npx playwright test
```

### Playwright 作为通用浏览器自动化工具

Playwright CLI 不局限于测试，也可以用于**日常自动化**：

```bash
# 监控网站可用性
npx playwright screenshot --wait-for-timeout 5000 https://myapp.com health.png

# 定期导出报告为 PDF
npx playwright pdf --format A4 https://myapp.com/dashboard daily-report.pdf

# 批量截图做视觉回归
npx playwright screenshot --full-page https://staging.myapp.com staging.png
npx playwright screenshot --full-page https://myapp.com production.png

# 录制测试用例
npx playwright codegen --target python https://myapp.com
```

---

## 常见问题

### Q: 安装浏览器失败 / 下载太慢？

```bash
# 使用国内镜像
export PLAYWRIGHT_DOWNLOAD_HOST=https://npmmirror.com/mirrors/playwright
npx playwright install

# 或者仅安装 Chromium（最小化安装）
npx playwright install chromium
```

### Q: Linux 服务器上浏览器启动失败？

```bash
# 安装系统依赖
npx playwright install-deps

# 无头模式下还需要这些库（Ubuntu/Debian）
sudo apt-get install libnss3 libnspr4 libatk1.0-0 libatk-bridge2.0-0 \
  libcups2 libdrm2 libxkbcommon0 libxcomposite1 libxdamage1 libxfixes3 \
  libxrandr2 libgbm1 libpango-1.0-0 libcairo2 libasound2
```

### Q: 如何在无头（headless）模式下运行？

```bash
# 默认就是无头模式
npx playwright test

# 如需有头模式（显示浏览器窗口）
npx playwright test --headed
```

### Q: codegen 录制的代码可以直接用吗？

录制生成的代码是**很好的起点**，但通常需要：
1. 添加断言（assertions）来验证结果
2. 使用 Page Object Model 重构重复代码
3. 添加适当的等待策略
4. 清理硬编码的测试数据

---

## 速查表

```text
┌─────────────────────────────────────────────────────┐
│              Playwright CLI 速查表                    │
├─────────────────────────────────────────────────────┤
│ 安装                                                 │
│   npx playwright install          # 安装所有浏览器   │
│   npx playwright install chromium # 仅 Chromium     │
│   npx playwright install-deps     # 系统依赖         │
├─────────────────────────────────────────────────────┤
│ 浏览器操作                                           │
│   npx playwright open <url>       # 打开调试         │
│   npx playwright codegen <url>    # 录制生成代码     │
│   npx playwright screenshot <url> <file>  # 截图    │
│   npx playwright pdf <url> <file> # 导出 PDF        │
├─────────────────────────────────────────────────────┤
│ 测试                                                 │
│   npx playwright test             # 运行测试         │
│   npx playwright test --ui        # UI 模式          │
│   npx playwright test --debug     # 调试模式         │
│   npx playwright test --headed    # 有头模式         │
│   npx playwright show-report      # 查看报告         │
│   npx playwright show-trace <file># 查看轨迹         │
├─────────────────────────────────────────────────────┤
│ 登录状态                                             │
│   storageState({ path: 'auth.json' })  # 保存状态     │
│   storageState: 'auth.json'            # 复用状态     │
├─────────────────────────────────────────────────────┤
│ Claude Code 集成                                     │
│   claude mcp add playwright -- npx @anthropic-ai/mcp-playwright  │
│   npx playwright codegen <url>  # 录制 → Claude 生成  │
│   npx tsx scripts/browser/*.ts  # Claude 编写的脚本   │
├─────────────────────────────────────────────────────┤
│ 常用选项                                             │
│   --browser <engine>    chromium|firefox|webkit      │
│   --device <name>       模拟设备                      │
│   --viewport-size <wxh> 视口大小                      │
│   --color-scheme <s>    light|dark                   │
│   --full-page           全页截图                      │
│   --output <file>       输出文件                      │
│   --target <lang>       生成代码语言                  │
└─────────────────────────────────────────────────────┘
```

---

## 相关资源

- [Playwright 官方文档](https://playwright.dev/docs/intro)
- [Playwright CLI 文档](https://playwright.dev/docs/cli)
- [Playwright GitHub](https://github.com/microsoft/playwright)
- [Playwright API 参考](https://playwright.dev/docs/api/class-playwright)
- [Playwright Storage State 文档](https://playwright.dev/docs/auth)
- [Playwright MCP Server](https://github.com/anthropics/mcp-playwright)
- [[AI技术/ClaudeCode/ClaudeCode浏览器自动化指南]] — Chrome DevTools MCP 配置指南
- [[AI技术/浏览器Agent技术/多台机器浏览器同步登录状态指南]] — 多设备 Cookie 同步方案
- [[AI技术/浏览器Agent技术/Agent-Browser 完全指南]] — agent-browser 详细指南及与 Playwright 对比

---

*最后更新：2026-04-13*
