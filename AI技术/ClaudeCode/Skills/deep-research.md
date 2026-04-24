---
type: skill
name: deep-research
category: 深度调研
tier: 专业场景层
source: ECC
status: installed
install_date: 2026-04-01
last_used: 2026-04-18
tags:
  - type/skill
  - skill/research
  - tier/professional
  - source/ecc
---

# Deep Research

> [!danger|style-blue] 🔬 Deep Research
> **梯队**：专业场景层 | **来源**：ECC | **状态**：✅ 已安装
>
> **一句话**：多源并行搜索与信息聚合，生成结构化研究报告

## 核心能力

- 多搜索引擎并行检索
- 自动提取、去重、整合信息
- 输出带引用的结构化报告
- 支持递进式检索（iterative-retrieval）
- 需要 Firecrawl / Exa MCP 配合

## 适用场景

- 技术选型调研
- 竞品分析
- 市场研究
- 论文综述

## 安装

```bash
/install-skill deep-research
# 需要配合 MCP 使用
# 需要配置 Firecrawl 和 Exa API Key
```

## 关联 Skills

- **互补**：[[context7]] — Context7 提供实时文档，Deep Research 聚合分析
- **后续**：[[docx]] — 调研报告转为 Word 文档交付
- **输入**：[[tavily]] + Firecrawl — 搜索引擎 + 网页抓取

## 实测评价

> [!quote] 社区评价
> 这个 Skill 把 Claude 从"问答机器"变成了"研究助手"。输出质量取决于搜索源质量，但整体结构化程度远超手动搜索+整理。
