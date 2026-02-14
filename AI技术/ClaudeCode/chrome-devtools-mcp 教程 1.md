# Chrome DevTools MCP 完整教程

> Chrome DevTools MCP 是一个 Model Context Protocol (MCP) 服务器，让 AI 助手能够直接与基于 Chromium 的浏览器（Chrome、Edge 等）和 DevTools 进行交互，实现自动化调试、性能分析、网络监控等功能。

---

## 📋 目录

- [什么是 Chrome DevTools MCP](#什么是-chrome-devtools-mcp)
- [支持的浏览器](#支持的浏览器)
- [系统要求](#系统要求)
- [安装步骤](#安装步骤)
- [配置方法](#配置方法)
- [使用不同浏览器](#使用不同浏览器)
- [核心功能](#核心功能)
- [使用案例](#使用案例)
- [常见问题](#常见问题)
- [参考资料](#参考资料)

---

## 什么是 Chrome DevTools MCP

**Chrome DevTools MCP** 是由 Chrome DevTools 团队开发的官方 MCP 服务器，它将基于 Chromium 浏览器的调试能力暴露给 MCP 客户端（如 Claude Code、Cursor IDE），使 AI 能够：

- 🔍 **检查页面元素** - 实时查看 DOM 结构和样式
- 🐛 **调试代码** - 设置断点、检查变量、单步执行
- 📊 **性能分析** - 分析页面加载性能、运行时性能
- 🌐 **网络监控** - 拦截和检查网络请求/响应
- 🎭 **设备模拟** - 模拟各种设备和用户代理
- 🤖 **自动化操作** - 模拟用户输入、点击、导航

---

## 支持的浏览器

虽然项目名为 "Chrome DevTools MCP"，但它支持所有基于 Chromium 且实现了 Chrome DevTools Protocol 的浏览器：

| 浏览器 | 兼容性 | 说明 |
|--------|--------|------|
| **Google Chrome** | ✅ 完全支持 | 官方主要测试浏览器 |
| **Microsoft Edge** | ✅ 完全支持 | 基于 Chromium，完全兼容 CDP |
| **Brave** | ✅ 支持 | 开源 Chromium 浏览器 |
| **Opera** | ⚠️ 部分支持 | 基于 Chromium，但可能有差异 |
| **Vivaldi** | ⚠️ 部分支持 | 基于 Chromium，但可能有差异 |

---

## 系统要求

| 组件 | 要求 |
|------|------|
| **Node.js** | v18 或更高版本 |
| **浏览器** | Chrome / Edge (最新稳定版) |
| **AI 客户端** | Claude Code / Cursor IDE / 其他 MCP 兼容工具 |
| **操作系统** | Windows / macOS / Linux |

---

## 安装步骤

### 1️⃣ 安装 MCP 服务器

```bash
# 全局安装 Chrome DevTools MCP 服务器
npm install -g @chromedevtools/mcp-server
```

验证安装：

```bash
chrome-devtools-mcp --version
```

---

## 使用不同浏览器

### 🌐 使用 Microsoft Edge

Edge 浏览器完全兼容 Chrome DevTools Protocol，可以直接使用 chrome-devtools-mcp。

#### Windows 启动 Edge 远程调试

```powershell
# 方法一：命令行启动
& "${env:ProgramFiles(x86)}\Microsoft\Edge\Application\msedge.exe" --remote-debugging-port=9222 --user-data-dir="C:\edge-debug-profile"

# 方法二：使用完整路径
"C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe" --remote-debugging-port=9222 --user-data-dir="C:\edge-debug-profile"
```

#### macOS 启动 Edge 远程调试

```bash
/Applications/Microsoft\ Edge.app/Contents/MacOS/Microsoft\ Edge --remote-debugging-port=9222 --user-data-dir="/tmp/edge-debug-profile"
```

#### Linux 启动 Edge 远程调试

```bash
microsoft-edge --remote-debugging-port=9222 --user-data-dir="/tmp/edge-debug-profile"
```

#### Edge 可执行文件位置参考

| 操作系统                      | 默认路径                                                             |
| ------------------------- | ---------------------------------------------------------------- |
| **Windows x64**           | `C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe`   |
| **Windows x86**           | `C:\Program Files\Microsoft\Edge\Application\msedge.exe`         |
| **macOS**                 | `/Applications/Microsoft Edge.app/Contents/MacOS/Microsoft Edge` |
| **Linux (Debian/Ubuntu)** | `/opt/microsoft/msedge/msedge`                                   |
| **Linux (Fedora)**        | `/usr/bin/microsoft-edge`                                        |

> 💡 **小贴士**: 可以在终端/命令行中输入 `where msedge` (Windows) 或 `which msedge` (Linux/macOS) 来查找 Edge 的确切安装路径。

---

### 🌐 使用 Google Chrome

#### Windows 启动 Chrome 远程调试

```powershell
# 方法一：命令行启动
& "${env:ProgramFiles}\Google\Chrome\Application\chrome.exe" --remote-debugging-port=9222 --user-data-dir="C:\chrome-debug-profile"

# 方法二：使用完整路径
"C:\Program Files\Google\Chrome\Application\chrome.exe" --remote-debugging-port=9222 --user-data-dir="C:\chrome-debug-profile"
```

#### macOS 启动 Chrome 远程调试

```bash
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --remote-debugging-port=9222 --user-data-dir="/tmp/chrome-debug-profile"
```

#### Linux 启动 Chrome 远程调试

```bash
google-chrome --remote-debugging-port=9222 --user-data-dir="/tmp/chrome-debug-profile"
```

---

### 🔌 同时使用 Chrome 和 Edge

如果需要同时使用两个浏览器（例如对比测试），可以使用不同的端口：

```powershell
# 终端 1 - 启动 Edge 在端口 9222
msedge.exe --remote-debugging-port=9222 --user-data-dir="C:\edge-debug-profile"

# 终端 2 - 启动 Chrome 在端口 9223
chrome.exe --remote-debugging-port=9223 --user-data-dir="C:\chrome-debug-profile"
```

然后在 MCP 配置中指定对应的 URL：

```json
{
  "mcpServers": {
    "edge-devtools": {
      "command": "npx",
      "args": ["@chromedevtools/mcp-server"],
      "env": {
        "CHROME_REMOTE_DEBUGGING_URL": "http://localhost:9222"
      }
    },
    "chrome-devtools": {
      "command": "npx",
      "args": ["@chromedevtools/mcp-server"],
      "env": {
        "CHROME_REMOTE_DEBUGGING_URL": "http://localhost:9223"
      }
    }
  }
}
```

---

### 2️⃣ 验证连接

访问 `http://localhost:9222/json`，如果看到 JSON 响应包含页面列表，说明浏览器远程调试已成功启用。

**Edge 验证示例**:
```bash
curl http://localhost:9222/json
```

**预期输出**:
```json
[
  {
    "description": "",
    "devtoolsFrontendUrl": "/devtools/inspector.html?ws=localhost:9222/...",
    "id": "A87B2C4D...",
    "title": "New Tab",
    "type": "page",
    "url": "edge://newtab",
    "webSocketDebuggerUrl": "ws://localhost:9222/..."
  }
]
```

---

## 配置方法

### Claude Code 配置

1. 打开 Claude Code 设置
2. 找到 **MCP Servers** 配置项
3. 添加以下配置：

#### 使用 Edge 浏览器

```json
{
  "mcpServers": {
    "edge-devtools": {
      "command": "npx",
      "args": ["@chromedevtools/mcp-server"],
      "env": {
        "CHROME_REMOTE_DEBUGGING_URL": "http://localhost:9222",
        "BROWSER_NAME": "Microsoft Edge"
      }
    }
  }
}
```

#### 使用 Chrome 浏览器

```json
{
  "mcpServers": {
    "chrome-devtools": {
      "command": "npx",
      "args": ["@chromedevtools/mcp-server"],
      "env": {
        "CHROME_REMOTE_DEBUGGING_URL": "http://localhost:9222"
      }
    }
  }
}
```

---

### Cursor IDE 配置

1. 打开 Cursor 设置 (Settings)
2. 搜索 "MCP" 或 "Model Context Protocol"
3. 添加服务器配置：

```json
{
  "mcpServers": [
    {
      "name": "edge-devtools",
      "command": "npx",
      "args": ["@chromedevtools/mcp-server"],
      "env": {
        "CHROME_REMOTE_DEBUGGING_URL": "http://localhost:9222"
      }
    }
  ]
}
```

---

### Cline (VS Code 扩展) 配置

在 VS Code 设置中添加：

```json
{
  "cline.mcpServers": {
    "edge-devtools": {
      "command": "npx",
      "args": ["@chromedevtools/mcp-server"],
      "env": {
        "CHROME_REMOTE_DEBUGGING_URL": "http://localhost:9222"
      }
    }
  }
}
```

---

## 核心功能

### 🔧 可用工具列表

| 工具名称 | 功能描述 |
|---------|---------|
| `chrome_getDocument` | 获取页面的完整 DOM 树 |
| `chrome_querySelector` | 使用 CSS 选择器查询元素 |
| `chrome_getElementProperties` | 获取元素的 CSS 属性 |
| `chrome_evaluate` | 在页面上下文中执行 JavaScript |
| `chrome_navigate` | 导航到指定 URL |
| `chrome_reload` | 重新加载页面 |
| `chrome_click` | 点击指定元素 |
| `chrome_type` | 在输入框中输入文本 |
| `chrome_screenshot` | 截取页面截图 |
| `chrome_enableNetworkMonitoring` | 启用网络监控 |
| `chrome_getNetworkRequests` | 获取网络请求列表 |
| `chrome_enableRuntimeMonitoring` | 启用运行时监控 |
| `chrome_setEmulation` | 设置设备/CPU 网络模拟 |
| `chrome_takeHeapSnapshot` | 获取堆快照 |
| `chrome_startPerformanceTrace` | 开始性能追踪 |
| `chrome_stopPerformanceTrace` | 停止性能追踪并获取结果 |

> ⚠️ **注意**: 虽然工具名称以 `chrome_` 开头，但这些工具同样适用于 Edge 浏览器。

---

## 使用案例

### 案例 1: 自动化表单填写

**场景**: 自动填写登录表单并提交

```markdown
# AI 指令示例

请帮我完成以下任务：
1. 导航到 https://example.com/login
2. 查找用户名输入框 (selector: #username)
3. 输入 "test@example.com"
4. 查找密码输入框 (selector: #password)
5. 输入 "password123"
6. 点击登录按钮 (selector: button[type="submit"])
7. 等待页面跳转后截图
```

**执行流程**:
```javascript
// AI 会自动调用以下工具序列
1. chrome_navigate("https://example.com/login")
2. chrome_querySelector("#username")
3. chrome_type("#username", "test@example.com")
4. chrome_type("#password", "password123")
5. chrome_click("button[type='submit']")
6. chrome_screenshot()
```

---

### 案例 2: 跨浏览器兼容性测试

**场景**: 在 Edge 和 Chrome 中同时测试网站兼容性

```markdown
# AI 指令示例

请帮我进行跨浏览器测试：
1. 在 Edge 浏览器中打开 https://example.com
2. 获取所有按钮元素的计算样式
3. 切换到 Chrome 浏览器
4. 打开同一页面
5. 对比两个浏览器中的样式差异
6. 生成兼容性报告
```

**配置要求**: 需要配置两个 MCP 服务器实例，分别连接 Edge (9222) 和 Chrome (9223)

---

### 案例 3: 性能分析与优化

**场景**: 分析页面加载性能，找出性能瓶颈

```markdown
# AI 指令示例

请分析当前页面的性能：
1. 开始性能追踪
2. 刷新页面
3. 停止追踪并分析结果
4. 识别最耗时的操作
5. 给出优化建议
```

**执行流程**:
```javascript
// AI 调用工具序列
1. chrome_startPerformanceTrace()
2. chrome_reload()
3. chrome_stopPerformanceTrace()
4. chrome_evaluate("performance.getEntriesByType('navigation')")
5. 分析并生成报告
```

---

### 案例 4: 网络请求调试

**场景**: 检查 API 请求的请求头和响应数据

```markdown
# AI 指令示例

帮我调试网络请求：
1. 启用网络监控
2. 刷新页面
3. 找到所有 /api/* 的请求
4. 显示每个请求的状态码、请求头和响应体
5. 找出失败的请求并分析原因
```

**执行流程**:
```javascript
// AI 调用工具序列
1. chrome_enableNetworkMonitoring()
2. chrome_reload()
3. chrome_getNetworkRequests()
4. 过滤分析 /api/* 请求
5. 生成调试报告
```

---

### 案例 5: CSS 样式调试

**场景**: 找出元素为什么没有应用预期的样式

```markdown
# AI 指令示例

帮我调试这个元素的样式问题：
1. 查找 selector: .my-button 元素
2. 获取所有计算后的 CSS 属性
3. 获取元素的所有样式规则（包括继承的）
4. 找出 color 属性的来源
5. 解释为什么颜色是 #000000 而不是红色
```

**执行流程**:
```javascript
// AI 调用工具序列
1. chrome_querySelector(".my-button")
2. chrome_getElementProperties(".my-button", ["color", "background-color"])
3. chrome_evaluate("getComputedStyle(document.querySelector('.my-button'))")
4. 分析样式层叠规则
5. 生成样式问题报告
```

---

### 案例 6: 设备模拟测试

**场景**: 测试网站在移动设备上的表现

```markdown
# AI 指令示例

请测试网站在移动设备上的表现：
1. 设置模拟 iPhone 14 Pro
2. 启用移动视口和触摸模拟
3. 刷新页面
4. 截图
5. 检查是否有水平滚动
6. 测试所有可点击元素的触摸区域是否足够大
```

**执行流程**:
```javascript
// AI 调用工具序列
1. chrome_setEmulation({
     device: "iPhone 14 Pro",
     mobile: true,
     touch: true
   })
2. chrome_reload()
3. chrome_screenshot()
4. chrome_evaluate("document.documentElement.scrollWidth > window.innerWidth")
5. 分析触摸元素尺寸
```

---

### 案例 7: 内存泄漏检测

**场景**: 检测页面是否存在内存泄漏

```markdown
# AI 指令示例

帮我检测内存泄漏：
1. 获取初始堆快照
2. 执行一些操作（打开/关闭模态框 10 次）
3. 获取第二次堆快照
4. 对比两次快照，找出新增的对象
5. 识别可疑的分离 DOM 节点
```

**执行流程**:
```javascript
// AI 调用工具序列
1. chrome_takeHeapSnapshot()
2. chrome_evaluate("repeat(10, () => { openModal(); closeModal(); })")
3. chrome_takeHeapSnapshot()
4. 分析快照差异
5. 生成内存泄漏报告
```

---

## 常见问题

### ❓ 浏览器连接失败

**问题**: AI 无法连接到浏览器

**解决方案**:
1. 确认浏览器是否以远程调试模式启动
2. 检查端口 9222 是否被占用
3. 访问 `http://localhost:9222/json` 验证连接
4. 防火墙是否阻止了本地连接
5. **Edge 特有**: 确保关闭所有 Edge 实例后再启动调试模式

### ❓ Edge 的 --remote-debugging-port 无效

**问题**: 添加参数后远程调试没有启用

**解决方案**:
1. **完全关闭**所有 Edge 进程（包括后台进程）
2. 使用任务管理器检查是否有 `msedge.exe` 进程
3. 使用独立的用户数据目录
4. 尝试使用不同的端口（如 9223）

```powershell
# 强制关闭所有 Edge 进程 (Windows)
taskkill /F /IM msedge.exe /T

# 然后重新启动
msedge.exe --remote-debugging-port=9222 --user-data-dir="C:\edge-debug-profile"
```

### ❓ 元素未找到

**问题**: `querySelector` 返回 null

**解决方案**:
1. 确认元素是否在 iframe 中（需要切换上下文）
2. 元素可能是动态加载的，需要等待
3. 使用更通用的选择器
4. 先获取 document 确认 DOM 结构

### ❓ 权限错误

**问题**: 某些操作被拒绝

**解决方案**:
1. 检查浏览器启动参数
2. 确认网站是否有 CORS 限制
3. 某些 API 需要 HTTPS 环境

### ❓ 同时使用 Chrome 和 Edge 端口冲突

**问题**: 两个浏览器都想使用 9222 端口

**解决方案**:
```powershell
# Edge 使用 9222
msedge.exe --remote-debugging-port=9222 --user-data-dir="C:\edge-debug"

# Chrome 使用 9223
chrome.exe --remote-debugging-port=9223 --user-data-dir="C:\chrome-debug"
```

然后在 MCP 配置中分别指定：
- Edge: `http://localhost:9222`
- Chrome: `http://localhost:9223`

---

## 高级技巧

### 💡 多标签页管理

```javascript
// 获取所有标签页
chrome_evaluate(`
  fetch('http://localhost:9222/json')
    .then(r => r.json())
    .then(tabs => tabs.map(t => t.id))
`)
```

### 💡 等待元素出现

```javascript
// 等待元素出现的辅助函数
chrome_evaluate(`
  async function waitForElement(selector, timeout = 5000) {
    const start = Date.now();
    while (Date.now() - start < timeout) {
      const el = document.querySelector(selector);
      if (el) return el;
      await new Promise(r => setTimeout(r, 100));
    }
    throw new Error('Element not found');
  }
  waitForElement('.my-element')
`)
```

### 💡 批量操作优化

```markdown
# AI 指令示例

一次性获取多个元素的信息：
1. 获取所有按钮元素
2. 同时获取它们的尺寸、位置和可见性
3. 生成一个包含所有信息的表格
```

### 💡 Edge 特有功能测试

```javascript
// 检测 Edge 特有 API
chrome_evaluate(`
  // 检查 Edge 特定功能
  const isEdge = /Edg/.test(navigator.userAgent);
  const hasCollectionsAPI = !!window.Collections;
  const hasWebNoteAPI = !!window.chrome?.webNote;

  { isEdge, hasCollectionsAPI, hasWebNoteAPI }
`)
```

---

## 创建快捷启动脚本

### Windows 快捷脚本

**Edge 调试模式启动脚本** (`start-edge-debug.bat`):
```batch
@echo off
echo Starting Microsoft Edge in remote debugging mode...
echo Debugging port: 9222
echo Press Ctrl+C to stop

taskkill /F /IM msedge.exe /T 2>nul

start "" "C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe" --remote-debugging-port=9222 --user-data-dir="C:\edge-debug-profile"

echo Microsoft Edge started with remote debugging on http://localhost:9222
timeout /t 3
```

**Chrome 调试模式启动脚本** (`start-chrome-debug.bat`):
```batch
@echo off
echo Starting Google Chrome in remote debugging mode...
echo Debugging port: 9223
echo Press Ctrl+C to stop

taskkill /F /IM chrome.exe /T 2>nul

start "" "C:\Program Files\Google\Chrome\Application\chrome.exe" --remote-debugging-port=9223 --user-data-dir="C:\chrome-debug-profile"

echo Google Chrome started with remote debugging on http://localhost:9223
timeout /t 3
```

### macOS 快捷脚本

**Edge 调试模式启动脚本** (`start-edge-debug.sh`):
```bash
#!/bin/bash
echo "Starting Microsoft Edge in remote debugging mode..."
killall "Microsoft Edge" 2>/dev/null

/Applications/Microsoft\ Edge.app/Contents/MacOS/Microsoft\ Edge \
  --remote-debugging-port=9222 \
  --user-data-dir="/tmp/edge-debug-profile" &

echo "Microsoft Edge started with remote debugging on http://localhost:9222"
```

**使用方法**:
```bash
chmod +x start-edge-debug.sh
./start-edge-debug.sh
```

---

## 参考资料

### 官方文档

- [Chrome DevTools MCP GitHub 仓库](https://github.com/ChromeDevTools/chrome-devtools-mcp)
- [Chrome DevTools MCP 官方博客](https://developer.chrome.com/blog/chrome-devtools-mcp)
- [工具参考文档](https://github.com/ChromeDevTools/chrome-devtools-mcp/blob/main/docs/tool-reference.md)
- [MCP 协议规范](https://modelcontextprotocol.io/)

### Edge 相关资源

- [Microsoft Edge DevTools Protocol](https://learn.microsoft.com/en-us/microsoft-edge/devtools/protocol/)
- [Microsoft Edge DevTools Protocol (中文)](https://learn.microsoft.com/zh-cn/microsoft-edge/devtools/protocol/)
- [Edge DevTools MCP (LobeHub)](https://lobehub.com/mcp/syunnrai123123-edge-devtools-mcp)

### 社区教程

- [Claude Code 配置指南](https://raf.dev/blog/chrome-debugging-profile-mcp/)
- [Cursor IDE 配置指南](https://lobehub.com/mcp/ochapple-chrome-devtools-mcp-cursor-guide)
- [完整指南 2025](https://vladimirsiddykh.com/blog/chrome-devtools-mcp-ai-browser-debugging-complete-guide-2025)
- [性能调试指南](https://www.debugbear.com/blog/chrome-devtools-mcp-performance-debugging)
- [Edge 远程调试问题解决](https://blog.csdn.net/qq_30576521/article/details/142370538)

---

## 总结

Chrome DevTools MCP 支持 Chrome 和 Edge 浏览器，为 AI 助手提供了强大的浏览器自动化和调试能力。通过合理配置和使用，可以显著提高开发和调试效率。

**推荐工作流**:
1. 根据需求选择 Chrome 或 Edge 作为调试浏览器
2. 创建快捷启动脚本方便使用
3. 使用 AI 快速诊断问题
4. 结合人工分析进行深度调试
5. 将常见调试操作保存为模板指令

**浏览器选择建议**:
- **Chrome**: 适用于大多数 Web 开发场景，插件生态丰富
- **Edge**: 适用于测试 IE 兼容模式、Edge 特定功能、或作为首选浏览器的用户

---

> 💾 **保存建议**: 将此文档保存在你的 Obsidian vault 中，并创建 `#devtools` `#mcp` `#chrome` `#edge` 标签以便快速检索。
