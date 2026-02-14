# Chrome DevTools MCP 完整教程

> Chrome DevTools MCP 是一个 Model Context Protocol (MCP) 服务器，让 AI 助手能够直接与 Chrome 浏览器和 DevTools 进行交互，实现自动化调试、性能分析、网络监控等功能。

---

## 📋 目录

- [什么是 Chrome DevTools MCP](#什么是-chrome-devtools-mcp)
- [系统要求](#系统要求)
- [安装步骤](#安装步骤)
- [配置方法](#配置方法)
- [核心功能](#核心功能)
- [使用案例](#使用案例)
- [常见问题](#常见问题)
- [参考资料](#参考资料)

---

## 什么是 Chrome DevTools MCP

**Chrome DevTools MCP** 是由 Chrome DevTools 团队开发的官方 MCP 服务器，它将 Chrome 浏览器的调试能力暴露给 MCP 客户端（如 Claude Code、Cursor IDE），使 AI 能够：

- 🔍 **检查页面元素** - 实时查看 DOM 结构和样式
- 🐛 **调试代码** - 设置断点、检查变量、单步执行
- 📊 **性能分析** - 分析页面加载性能、运行时性能
- 🌐 **网络监控** - 拦截和检查网络请求/响应
- 🎭 **设备模拟** - 模拟各种设备和用户代理
- 🤖 **自动化操作** - 模拟用户输入、点击、导航

---

## 系统要求

| 组件 | 要求 |
|------|------|
| **Node.js** | v18 或更高版本 |
| **Chrome 浏览器** | 最新稳定版 |
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

### 2️⃣ 启动 Chrome 远程调试端口

在启动 Chrome 时添加远程调试参数：

**Windows:**
```powershell
# 方法一：命令行启动
"C:\Program Files\Google\Chrome\Application\chrome.exe" --remote-debugging-port=9222

# 方法二：创建快捷方式，在目标后添加参数
# chrome.exe --remote-debugging-port=9222 --user-data-dir="C:\chrome-debug-profile"
```

**macOS:**
```bash
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --remote-debugging-port=9222 --user-data-dir="/tmp/chrome-debug-profile"
```

**Linux:**
```bash
google-chrome --remote-debugging-port=9222 --user-data-dir="/tmp/chrome-debug-profile"
```

> ⚠️ **注意**: 建议使用独立的用户数据目录，避免影响日常使用的 Chrome 配置。

### 3️⃣ 验证连接

访问 `http://localhost:9222/json`，如果看到 JSON 响应包含页面列表，说明 Chrome 远程调试已成功启用。

---

## 配置方法

### Claude Code 配置

1. 打开 Claude Code 设置
2. 找到 **MCP Servers** 配置项
3. 添加以下配置：

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

### Cursor IDE 配置

1. 打开 Cursor 设置 (Settings)
2. 搜索 "MCP" 或 "Model Context Protocol"
3. 添加服务器配置：

```json
{
  "mcpServers": [
    {
      "name": "chrome-devtools",
      "command": "npx",
      "args": ["@chromedevtools/mcp-server"],
      "env": {
        "CHROME_REMOTE_DEBUGGING_URL": "http://localhost:9222"
      }
    }
  ]
}
```

### Cline (VS Code 扩展) 配置

在 VS Code 设置中添加：

```json
{
  "cline.mcpServers": {
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

### 案例 2: 性能分析与优化

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

### 案例 3: 网络请求调试

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

### 案例 4: CSS 样式调试

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

### 案例 5: 设备模拟测试

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

### 案例 6: 内存泄漏检测

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

### ❓ Chrome 连接失败

**问题**: AI 无法连接到 Chrome

**解决方案**:
1. 确认 Chrome 是否以远程调试模式启动
2. 检查端口 9222 是否被占用
3. 访问 `http://localhost:9222/json` 验证连接
4. 防火墙是否阻止了本地连接

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
1. 检查 Chrome 启动参数
2. 确认网站是否有 CORS 限制
3. 某些 API 需要 HTTPS 环境

### ❓ 性能追踪无数据

**问题**: `stopPerformanceTrace` 返回空数据

**解决方案**:
1. 确保调用了 `startPerformanceTrace`
2. 在追踪期间执行一些操作
3. 检查页面是否有实际活动

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

---

## 参考资料

- [Chrome DevTools MCP GitHub 仓库](https://github.com/ChromeDevTools/chrome-devtools-mcp)
- [Chrome DevTools MCP 官方博客](https://developer.chrome.com/blog/chrome-devtools-mcp)
- [工具参考文档](https://github.com/ChromeDevTools/chrome-devtools-mcp/blob/main/docs/tool-reference.md)
- [MCP 协议规范](https://modelcontextprotocol.io/)
- [Claude Code 配置指南](https://raf.dev/blog/chrome-debugging-profile-mcp/)
- [完整配置指南 2025](https://vladimirsiddykh.com/blog/chrome-devtools-mcp-ai-browser-debugging-complete-guide-2025)
- [性能调试指南](https://www.debugbear.com/blog/chrome-devtools-mcp-performance-debugging)

---

## 总结

Chrome DevTools MCP 为 AI 助手提供了强大的浏览器自动化和调试能力。通过合理配置和使用，可以显著提高开发和调试效率。

**推荐工作流**:
1. 开发时保持一个专用调试 Chrome 实例
2. 使用 AI 快速诊断问题
3. 结合人工分析进行深度调试
4. 将常见调试操作保存为模板指令

---

> 💾 **保存建议**: 将此文档保存在你的 Obsidian vault 中，并创建 `#devtools` `#mcp` `#chrome` 标签以便快速检索。
