---
type: skill
name: agent-browser
category: 浏览器自动化
tier: 基础能力层
source: 社区
status: installed
install_date: 2026-03-05
last_used: 2026-04-21
tags:
  - type/skill
  - skill/browser
  - tier/basic
  - source/community
---

# Agent Browser

> [!note|style-green] 🌐 Agent Browser
> **梯队**：基础能力层 | **来源**：社区 | **状态**：✅ 已安装
>
> **一句话**：赋予 Claude 操作浏览器的能力

## 核心能力

- 自动打开网页、点击、填表、截图
- 支持 Playwright 底层，可与 E2E 测试复用基础设施
- 可实现"打开 URL → 抓取数据 → 写入文件"的完整链路

## 适用场景

- 自动化 Web 测试流程
- 抓取网页数据并生成报告
- 竞品页面截图对比
- UI 视觉验证

## 安装

```bash
/install-skill agent-browser
```

## 关联 Skills

- **互补**：[[frontend-design]] — 生成 UI 后用浏览器截图验证
- **互补**：[[tavily]] — Tavily 搜索找到目标，Browser 打开操作
- **协同**：E2E 测试相关 Skills

## 实测评价

> [!quote] 社区评价
> 安装后 Claude 的"执行力"质变。以前只能生成代码让你自己跑，现在能直接打开浏览器帮你操作。
