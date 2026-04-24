---
type: skill
name: context7
category: 文档查询
tier: 效率倍增层
source: 社区
status: installed
install_date: 2026-03-20
last_used: 2026-04-22
tags:
  - type/skill
  - skill/docs
  - tier/efficiency
  - source/community
---

# Context7

> [!tip|style-purple] 📚 Context7
> **梯队**：效率倍增层 | **来源**：社区 | **状态**：✅ 已安装
>
> **一句话**：实时拉取最新文档，彻底消除 API 幻觉

## 核心能力

- 实时拉取最新版本的库/框架文档
- 支持 Next.js 15、React 19、Tailwind CSS 4 等最新框架
- 通过 MCP 或 Skill 两种模式接入
- 彻底解决"API 已废弃但 Claude 还在用"的问题

## 适用场景

- 使用最新版框架 API 编写代码
- 确保生成的代码与当前库版本兼容
- 查阅不熟悉库的实时文档

## 安装

```bash
# CLI + Skills 模式
npx context7 --claude

# 或 MCP 模式
# 在 Claude Code 中配置 Context7 MCP Server
```

## 关联 Skills

- **互补**：[[tdd-workflow]] — 用最新 API 文档写测试，零幻觉
- **前置**：[[frontend-design]] — 确保使用最新版本的 Tailwind/shadcn
- **协同**：[[deep-research]] — Context7 提供实时文档，Deep Research 聚合分析

## 实测评价

> [!quote] 社区评价
> 解决 Claude API 幻觉的终极方案。以前 Claude 经常生成已经废弃的 API 调用，装了 Context7 后，它会先查文档再写代码，准确率大幅提升。 — Addy Osmani (Google)
