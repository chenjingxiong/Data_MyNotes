---
title: FP-Browser 移动端指纹浏览器使用指南
date: 2026-01-21
tags:
  - 指纹浏览器
  - 移动端
  - 反检测
  - Appium
  - 自动化测试
  - 开源
source: https://github.com/SyncMeIn/FP-Browser
---

# FP-Browser 移动端指纹浏览器使用指南

> [!info] 项目信息
> - **项目地址**：[GitHub - SyncMeIn/FP-Browser](https://github.com/SyncMeIn/FP-Browser)
> - **最新版本**：基于 Chrome 102
> - **开源协议**：MPL-2.0
> - **平台支持**：Android（ARM64 / x86_64）
> - **开发语言**：Python 100%

---

## 1. 什么是 FP-Browser？

FP-Browser 是一款**开源免费的移动端指纹浏览器**，专为移动端浏览器指纹伪装和自动化测试设计。通过底层动态注入技术，可以在模拟器或真机上创建具有不同浏览器指纹的测试环境，让模拟器的浏览器环境与真机保持一致。

### 1.1 什么是移动端浏览器指纹？

移动端浏览器指纹是通过收集移动设备浏览器的各种特征（如 User-Agent、设备内存、触控点数量、WebGL 渲染器、电池状态等），综合分析生成唯一标识符的过程。网站可以利用指纹来：

- 识别移动设备身份
- 检测模拟器环境
- 追踪用户行为
- 进行风险评估和反欺诈

### 1.2 FP-Browser 的核心优势

| 优势 | 说明 |
|------|------|
| **完全免费开源** | MPL-2.0 协议，代码完全开源 |
| **底层动态注入** | 通过代码注入浏览器运行时环境，配置永久生效 |
| **多版本支持** | 提供多版本兼容的指纹浏览器方案 |
| **多架构支持** | 支持 ARM、ARM64、x86_64 等平台 |
| **通过 JS 检测** | 底层修改手机配置，能通过任何 JS 检测 |
| **分布式测试** | 完整的分布式测试解决方案 |
| **客户端 SDK** | 提供注入参数 SDK，确保参数合法性 |

---

## 2. 环境要求

### 2.1 硬件要求

| 项目 | 最低配置 | 推荐配置 |
|------|----------|----------|
| CPU | 4 核 | 8 核及以上 |
| 内存 | 8 GB | 16 GB 及以上 |
| 磁盘 | 50 GB | 100 GB 及以上 |

### 2.2 软件要求

| 软件 | 版本要求 | 说明 |
|------|---------|------|
| **Python** | 3.8+ | 运行测试脚本 |
| **Appium** | 最新版 | 自动化框架 |
| **ADB** | 已配置 | Android 调试桥 |
| **模拟器/真机** | - | 雷电模拟器、夜神模拟器或 Android 真机 |

---

## 3. 安装与配置

### 3.1 克隆项目

```bash
git clone https://github.com/SyncMeIn/FP-Browser.git
cd FP-Browser
```

### 3.2 安装 Python 依赖

```bash
pip install -r requirements.txt
```

### 3.3 下载指纹浏览器 APK

> [!warning] 注意
> 完整的指纹浏览器 APK 需要联系作者获取。
> 微信: @kainyguo

根据设备架构选择对应的 APK：

| APK 文件 | 适用设备 |
|---------|---------|
| arm_64.apk | ARM 64 位设备（大多数真机） |
| x86_64.apk | x86_64 设备（模拟器） |

---

## 4. 快速入门

### 4.1 生成注入参数

创建 JSON 配置文件：

```json
{
    "webgl.vendor": "Qualcomm",
    "webgl.renderer": "Adreno (TM) 640",
    "navigator.platform": "Linux armv8l",
    "navigator.hardware-concurrency": "8",
    "navigator.device-memory": "4"
}
```

### 4.2 运行测试

```bash
python main.py
```

---

## 5. 指纹注入参数详解

### 5.1 Navigator 相关

| 参数名 | 验证状态 | 描述 | 示例 |
|--------|---------|------|------|
| navigator.user-agent | 通过 | User-Agent 字符串 | Mozilla/5.0... |
| navigator.platform | 通过 | 操作系统平台 | Linux armv8l |
| navigator.hardware-concurrency | 通过 | CPU 核心数 | 8 |
| navigator.device-memory | 通过 | 设备内存（GB） | 4 |

### 5.2 WebGL 相关

| 参数名 | 验证状态 | 描述 | 示例 |
|--------|---------|------|------|
| webgl.vendor | 通过 | WebGL 厂商 | Qualcomm |
| webgl.renderer | 通过 | WebGL 渲染器 | Adreno (TM) 640 |

---

## 6. 指纹测试工具

- [FP-Browser-Detect](http://tyua07.github.io/FP-Browser-Detect/)
- [BrowserLeaks](https://browserleaks.com/)

---

## 7. 相关开源项目

| 项目 | 说明 |
|------|------|
| **FP-Browser-Public** | 浏览器底层动态注入 |
| **FP-Browser-Detect** | 浏览器属性检测 |
| **FP-Browser-SDK** | 浏览器属性注入参数 SDK |
| **FP-Browser-Test** | 指纹浏览器全部选项的测试用例 |

---

## 8. 常见问题

### Q: 如何验证注入是否成功？

访问浏览器指纹检测网站进行验证。

### Q: 如何获取完整版 APK？

联系作者：微信 @kainyguo

---

## 9. 相关资源

- [GitHub 仓库](https://github.com/SyncMeIn/FP-Browser)
- [问题反馈](https://github.com/SyncMeIn/FP-Browser/issues)

---

> [!warning] 免责声明
> FP-Browser 项目仅供技术交流、学习和研究之用。使用者须承诺不利用该技术实施非法活动、侵犯他人权益或对系统进行攻击。因使用本项目技术所导致的任何损失或法律责任，与项目作者无关。
