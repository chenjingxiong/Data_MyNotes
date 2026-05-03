---
title: VirtualBrowser 指纹浏览器使用指南
date: 2026-05-03
tags:
  - 指纹浏览器
  - 反检测
  - 隐私保护
  - 自动化
  - Playwright
  - Chromium
  - 开源
source: https://github.com/Virtual-Browser/VirtualBrowser
---

# VirtualBrowser 指纹浏览器使用指南

> [!info] 项目信息
> - **项目地址**：[GitHub - Virtual-Browser/VirtualBrowser](https://github.com/Virtual-Browser/VirtualBrowser)
> - **官方网站**：[virtualbrowser.cc](http://virtualbrowser.cc/)
> - **最新版本**：2.2.15（截至 2026-05）
> - **开源协议**：BSD
> - **平台支持**：Windows 10+（Mac / Android / Linux 计划中）
> - **GitHub Stars**：2.8k+ | Forks：464+

---

## 1. 什么是 VirtualBrowser？

VirtualBrowser 是一款基于 [Chromium](https://dev.chromium.org/) 的**免费开源指纹浏览器**，专为多账户管理和隐私保护设计。它允许在单台机器上创建多个独立的浏览器指纹环境，每个环境拥有完全不同的浏览器特征，从而有效防止账号关联检测。

### 1.1 什么是浏览器指纹？

浏览器指纹识别是通过收集浏览器的各种特征（如 User-Agent、语言、屏幕分辨率、插件版本、字体、时区设置等），综合分析生成一个唯一标识符的过程。由于每个人的浏览器配置不同，网站可以利用指纹来：

- 追踪用户行为
- 识别用户身份
- 监视在线活动
- 进行跨站关联分析

### 1.2 VirtualBrowser 的核心优势

| 优势 | 说明 |
|------|------|
| **完全免费** | 免登录即可使用所有功能，无环境数量和打开次数限制 |
| **30+ 指纹参数** | 覆盖 Canvas、WebGL、字体、Audio 等全部主流指纹维度 |
| **多内核支持** | 支持 Chromium 138 / 142 / 144 / 146 内核 |
| **自动化 API** | 提供 RESTful API，兼容 Playwright、Selenium 等框架 |
| **窗口同步器** | 多窗口鼠标键盘同步操作，提升批量运营效率 |
| **本地存储** | 所有数据本地存储，不上传云端 |
| **开源透明** | BSD 协议开源，代码可审计 |

---

## 2. 安装与配置

### 2.1 下载安装

从以下任一渠道下载最新安装包：

- **GitHub Releases**：[https://github.com/Virtual-Browser/VirtualBrowser/releases](https://github.com/Virtual-Browser/VirtualBrowser/releases)
- **官方网站**：[http://virtualbrowser.cc/](http://virtualbrowser.cc/)

> [!tip] 版本选择
> 最新版 2.2.15 新增了 CLI 模式、多内核支持（142/144/146）、密码管理器、暗黑模式等功能，建议始终使用最新版。

### 2.2 安装步骤

1. 下载对应平台的安装包（Windows / Mac）
2. 运行安装程序，选择安装路径
3. 启动 VirtualBrowser

> [!note]
> VirtualBrowser 支持免登录直接使用，无需注册账号即可创建浏览器环境。

---

## 3. 快速入门

### 3.1 创建浏览器环境

1. 打开 VirtualBrowser 主界面
2. 点击 **"创建浏览器"** 按钮
3. 在弹出的配置对话框中，修改或使用默认指纹配置
4. 确认创建

### 3.2 启动浏览器环境

1. 在已创建的环境列表中，点击对应环境的 **"启动"** 按钮
2. 新启动的浏览器即为独立的指纹环境
3. 每个环境完全隔离，互不关联

### 3.3 环境管理

VirtualBrowser 提供以下管理功能：

- **分组管理**：将环境按用途分组
- **批量操作**：支持批量启动、关闭、删除
- **导入导出**：环境配置的导入导出
- **代理管理**：批量检测和配置代理

---

## 4. 指纹修改参数详解

VirtualBrowser 支持修改以下指纹维度，均经过实际测试验证有效：

### 4.1 基础指纹

| 参数 | 修改内容 | 说明 |
|------|---------|------|
| **User-Agent** | `navigator.userAgent` | 包含操作系统和浏览器版本信息 |
| **操作系统** | UA 中的 OS 部分 | 可伪装为不同操作系统 |
| **浏览器版本** | UA 中的版本号 | 可指定不同 Chromium 版本 |
| **语言** | `navigator.language` / `navigator.languages` | 支持根据 IP 自动匹配 |
| **时区** | `new Date()` 时区 | 支持根据 IP 自动匹配 |
| **分辨率** | `screen.width` / `screen.height` | 修改屏幕分辨率 |

### 4.2 硬件指纹

| 参数 | 修改内容 | 说明 |
|------|---------|------|
| **CPU** | `navigator.hardwareConcurrency` | 修改 CPU 核心数 |
| **内存** | `navigator.deviceMemory` | 修改设备内存 |
| **设备名称** | 设备标识信息 | 修改设备名称 |
| **MAC 地址** | 网络接口 MAC | 随机化 MAC 地址 |
| **硬件加速** | GPU 加速状态 | 控制硬件加速开关 |

### 4.3 图形指纹

| 参数 | 修改内容 | 说明 |
|------|---------|------|
| **Canvas** | Canvas 2D 绘制差分像素 | 随机化 Canvas 指纹 |
| **WebGL 图像** | WebGL 绘制差分像素 | 随机化 WebGL 渲染结果 |
| **WebGL 元数据** | WebGL 厂商 / 渲染器信息 | 修改显卡标识信息 |

### 4.4 多媒体指纹

| 参数 | 修改内容 | 说明 |
|------|---------|------|
| **AudioContext** | `getChannelData` / `getFloatFrequencyData` | 随机化音频处理差异 |
| **Speech Voices** | 语音合成声音列表 | 修改可用语音列表 |
| **字体** | 可用字体列表 | 随机化支持的字体集合 |

### 4.5 网络与位置

| 参数 | 修改内容 | 说明 |
|------|---------|------|
| **代理设置** | HTTP / SOCKS 代理 | 支持默认 / 不使用 / 自定义 |
| **WebRTC** | WebRTC 泄露防护 | 防止真实 IP 泄露 |
| **地理位置** | `navigator.geolocation` | 修改经纬度，支持 IP 自动匹配 |
| **端口扫描保护** | 本地端口扫描 | 防止端口指纹识别 |

### 4.6 其他

| 参数 | 修改内容 |
|------|---------|
| **ClientRects** | 元素布局矩形信息 |
| **Do Not Track** | DNT 设置 |
| **SSL** | SSL/TLS 指纹 |

### 4.7 指纹测试工具

推荐使用以下在线工具验证指纹修改效果：

- [FingerprintJS](https://fingerprintjs.github.io/fingerprintjs/) — 开源浏览器指纹检测
- [BrowserLeaks](https://browserleaks.com/) — 全面的浏览器泄露测试

---

## 5. 代理配置

VirtualBrowser 支持三种代理模式：

| 模式 | 说明 |
|------|------|
| **默认** | 使用系统代理设置 |
| **不使用代理** | 直连，不走任何代理 |
| **自定义** | 支持 HTTP / HTTPS / SOCKS4 / SOCKS5 代理 |

### 代理配置建议

1. **语言、时区、地理位置建议开启"根据 IP 自动匹配"**，确保指纹一致性
2. 使用代理时，务必测试代理可用性（内置批量检测功能）
3. 不同环境建议使用不同代理 IP，避免 IP 关联

---

## 6. 自动化开发

VirtualBrowser 基于 Chromium 内核，完全兼容 Playwright 等 Chromium 自动化工具。

### 6.1 Playwright 自动化示例

```javascript
// 需要先在 VirtualBrowser 中手动创建环境，获取 worker-id
const { chromium } = require('playwright');

const workerId = 1; // 替换为实际的 worker-id

(async () => {
  const browser = await chromium.launchPersistentContext(
    `${process.env.localappdata}\\VirtualBrowser\\Workers\\${workerId}`,
    {
      // 配置 VirtualBrowser 安装路径
      executablePath: 'D:\\VirtualBrowser\\Chrome-bin\\VirtualBrowser.exe',
      args: [`--worker-id=${workerId}`],
      headless: false,
      defaultViewport: null,
    }
  );

  const page = await browser.newPage();
  await page.goto('https://fingerprintjs.github.io/fingerprintjs/');

  // 在此编写自动化逻辑
  // ...

  // await browser.close();
})();
```

### 6.2 关键参数说明

| 参数 | 说明 |
|------|------|
| `launchPersistentContext` | 使用持久化上下文，保持指纹环境一致 |
| `executablePath` | VirtualBrowser 可执行文件路径 |
| `--worker-id` | 指定使用哪个指纹环境（需预先创建） |
| `headless` | 是否无头模式运行 |

### 6.3 RESTful API

VirtualBrowser 还提供完整的 RESTful API 接口，支持：

- 程序化创建 / 启动 / 关闭浏览器环境
- 批量管理环境配置
- 环境运行状态查询
- 清除缓存操作
- Cookie 导入导出

> [!info] API 文档
> 详细 API 文档请访问 [VirtualBrowser 官方网站](http://virtualbrowser.cc/) 的"API 文档"页面。

---

## 7. 窗口同步器

VirtualBrowser 内置**智能窗口同步器**功能，这是多账号运营的利器：

- **一键排列**：将多个浏览器窗口自动排列到屏幕上
- **主控窗口**：指定一个窗口为主控
- **操作同步**：在主控窗口中的所有鼠标和键盘操作会同步到所有被控窗口
- **适用场景**：批量注册、批量发布、批量互动等重复操作

---

## 8. 版本更新记录（近期）

| 版本 | 主要更新 |
|------|---------|
| **2.2.15** | CLI 模式、142/144/146 内核支持、密码管理器、暗黑模式、云同步增强 |
| **2.2.14** | Bug 修复 |
| **2.2.12** | 内核更新至 138 版本，新增多内核功能 |
| **2.2.11** | 代理备注列、无头模式修复、代理特殊字符修复 |
| **2.2.9** | API 调用优化、环境指纹优化 |
| **2.2.6** | 同步器窗口排列和文本输入、系统代理开关、代理批量检测 |
| **2.2.1** | 新增同步器功能 |
| **2.1.15** | 批量代理删除、免登录使用、SSH 断连修复 |
| **2.1.12** | 免登录使用、API 认证密钥、新增 API 接口 |

---

## 9. 常见问题

### Q: VirtualBrowser 真的完全免费吗？
是的，VirtualBrowser 完全免费，无需登录即可使用全部功能，没有环境数量和打开次数限制。

### Q: 支持哪些操作系统？
目前支持 Windows 10 及以上版本，Mac 版本已提供下载，Android 和 Linux 版本计划中。

### Q: 指纹修改是否可靠？
VirtualBrowser 的指纹修改经过 FingerprintJS 和 BrowserLeaks 等主流检测工具验证，能够通过多家网站的严格检测。指纹参数基于真实浏览器特征生成。

### Q: 如何配合代理使用？
建议在每个环境中配置独立的代理 IP，并开启语言、时区、地理位置的"根据 IP 自动匹配"功能，确保指纹与 IP 地理位置一致。

### Q: 可以用于自动化测试吗？
可以。VirtualBrowser 兼容 Playwright、Selenium 等 Chromium 自动化框架，同时提供 RESTful API 用于程序化管理。

### Q: 数据是否安全？
所有数据本地存储，不上传云端。支持配置文件加密保护。

---

## 10. 相关资源

- [GitHub 仓库](https://github.com/Virtual-Browser/VirtualBrowser)
- [官方网站](http://virtualbrowser.cc/)
- [自动化示例代码](https://github.com/Virtual-Browser/VirtualBrowser/tree/main/automation)
- [FingerprintJS 在线检测](https://fingerprintjs.github.io/fingerprintjs/)
- [BrowserLeaks 在线检测](https://browserleaks.com/)

### 联系方式

| 渠道 | 信息 |
|------|------|
| Email | virtual.browser.2020@gmail.com |
| 官网 | http://virtualbrowser.cc |
| QQ 群 | 564142956 |

---

> [!warning] 免责声明
> VirtualBrowser 项目仅供技术交流、学习和研究之用。使用者须承诺不利用该技术实施非法活动、侵犯他人权益或对系统进行攻击。因使用本项目技术所导致的任何损失或法律责任，与项目作者无关。
