---
title: ZenMux 使用指南
date: 2026-04-26
tags:
  - AI
  - API
  - LLM
  - 开发工具
  - 指南
description: ZenMux 企业级大模型聚合平台完整使用指南，涵盖注册、API调用、智能路由、保险赔付、订阅与按量付费等核心功能。
---

# ZenMux 使用指南

> **以禅者之心，驭 AI 之力 —— 万千模型归一处，至简体验达至优。**

## 什么是 ZenMux

**ZenMux** 是全球首个支持**保险赔付机制**的企业级大模型聚合平台。名称由 **Zen（禅）** 和 **Mux（Multiplexer，多路复用器）** 组合而成，核心理念是将复杂的多模型选择、风险管理、API 调用等繁琐流程，通过智能化手段简化为"一个 API、一套 SDK、一个平台"的极简体验。

通过 ZenMux，你可以用一个 API Key 调用 OpenAI、Anthropic、Google、xAI、DeepSeek 等全球 150+ 大语言模型，并享受智能路由、自动故障转移、AI 模型保险等企业级保障。

- **官网**：[https://zenmux.ai](https://zenmux.ai)
- **文档**：[https://docs.zenmux.ai](https://docs.zenmux.ai)
- **GitHub**：[https://github.com/ZenMux/zenmux-doc](https://github.com/ZenMux/zenmux-doc)
- **Discord**：[https://discord.gg/vHZZzj84Bm](https://discord.gg/vHZZzj84Bm)

---

## 核心特性一览

| 特性 | 说明 |
|------|------|
| **统一 API** | 一个 API Key 调用全球 150+ 大模型 |
| **双协议支持** | 完整兼容 OpenAI 和 Anthropic 两大协议标准 |
| **智能路由** | 自动选择最优模型和供应商，平衡效果与成本 |
| **AI 模型保险** | 全球首创，对幻觉、延迟等质量问题自动赔付 |
| **多供应商架构** | 主流模型配备 2-3 家供应商，自动故障转移 |
| **全球边缘节点** | 依托 Cloudflare 基础设施，全球低延迟访问 |
| **全面可观测性** | 日志分析、成本统计、用量监控、性能追踪 |

---

## 一、快速开始

### 1. 注册登录

访问 [ZenMux 登录页面](https://zenmux.ai/login)，支持以下方式：

- 邮箱登录
- GitHub 账号登录
- Google 账号登录

### 2. 选择计费方式

ZenMux 提供两种使用方式：

| 对比维度 | Pay As You Go（按量付费） | Builder Plan（订阅制） |
|---------|--------------------------|----------------------|
| **适用场景** | 生产环境、商业化产品 | 个人开发、学习探索、Vibe Coding |
| **计费方式** | 按实际消耗精准计费 | 固定月费 |
| **Rate Limit** | 无限制 | 10-15 RPM |
| **并发限制** | 无限制 | 有 Weekly Limit |
| **API Key 格式** | `sk-ai-v1-xxx` | `sk-ss-v1-xxx` |

> [!warning] 重要提示
> 如果你的项目已经上线或即将商业化，**必须使用 Pay As You Go 方案**。订阅制仅限个人开发和学习场景。

### 3. 获取 API Key

- **按量付费**：前往 [Pay As You Go 管理页面](https://zenmux.ai/platform/pay-as-you-go) 充值并创建 API Key
- **订阅制**：前往 [Pricing 页面](https://zenmux.ai/pricing/subscription) 选择套餐，然后在订阅管理页面创建 API Key

> [!tip] 充值优惠
> 平台目前提供充值额外赠送 20% Credits 优惠，支持 Stripe 信用卡和支付宝充值。

---

## 二、支持的 API 协议

ZenMux 支持四种主流 API 协议，可以使用最熟悉的 SDK 调用所有模型：

| 协议 | Base URL | 兼容 SDK |
|------|----------|----------|
| **OpenAI Chat Completions** | `https://zenmux.ai/api/v1` | OpenAI SDK |
| **OpenAI Responses** | `https://zenmux.ai/api/v1` | OpenAI SDK |
| **Anthropic Messages** | `https://zenmux.ai/api/anthropic` | Anthropic SDK |
| **Google Gemini** | `https://zenmux.ai/api/vertex-ai` | Google GenAI SDK |

> [!tip] 跨协议调用
> ZenMux 的核心优势之一是**协议无关性**——你可以通过任意支持的协议调用任意模型。例如，用 OpenAI SDK 调用 Claude 模型，或用 Anthropic SDK 调用 Gemini 模型。

---

## 三、API 调用示例

### OpenAI Chat Completions 协议

**Python：**

```python
from openai import OpenAI

client = OpenAI(
    base_url="https://zenmux.ai/api/v1",
    api_key="<你的 ZENMUX_API_KEY>",
)

completion = client.chat.completions.create(
    model="openai/gpt-5",  # 格式："供应商/模型名称"
    messages=[
        {"role": "user", "content": "生命的意义是什么？"}
    ]
)

print(completion.choices[0].message.content)
```

**TypeScript：**

```typescript
import OpenAI from "openai";

const openai = new OpenAI({
  baseURL: "https://zenmux.ai/api/v1",
  apiKey: "<你的 ZENMUX_API_KEY>",
});

const completion = await openai.chat.completions.create({
  model: "openai/gpt-5",
  messages: [{ role: "user", content: "生命的意义是什么？" }],
});

console.log(completion.choices[0].message.content);
```

**cURL：**

```bash
curl https://zenmux.ai/api/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $ZENMUX_API_KEY" \
  -d '{
    "model": "openai/gpt-5",
    "messages": [{"role": "user", "content": "生命的意义是什么？"}]
  }'
```

### Anthropic Messages 协议

```python
from anthropic import Anthropic

client = Anthropic(
    base_url="https://zenmux.ai/api/anthropic",
    api_key="<你的 ZENMUX_API_KEY>",
)

message = client.messages.create(
    model="google/gemini-3.1-pro-preview",  # 用 Anthropic SDK 调用 Gemini
    max_tokens=1024,
    messages=[{"role": "user", "content": "生命的意义是什么？"}]
)

print(message.content[0].text)
```

### OpenAI Responses 协议

```python
from openai import OpenAI

client = OpenAI(
    base_url="https://zenmux.ai/api/v1",
    api_key="<你的 ZENMUX_API_KEY>",
)

response = client.responses.create(
    model="openai/gpt-5",
    input="生命的意义是什么？"
)

print(response.output_text)
```

---

## 四、模型列表

ZenMux 聚合了 150+ 全球顶尖模型，覆盖文本生成、图片生成、视频生成、文本嵌入等全场景。

### 代表性模型

| 厂商 | 代表模型 |
|------|---------|
| **Anthropic** | Claude Opus 4.7、Claude Sonnet 4.6、Claude Haiku 4.5 |
| **OpenAI** | GPT-5.4、GPT-5、GPT-4.1、o4-mini |
| **Google** | Gemini 3.1 Pro Preview、Gemini 3 Flash、Gemini 2.5 Pro |
| **xAI** | Grok 4、Grok 4 Fast |
| **DeepSeek** | DeepSeek V3.2（含 Thinking Mode） |
| **其他** | Qwen3、GLM 4.7、MiniMax M2.1、StepFun Step-3 等 |

> [!tip] 模型 Slug
> ZenMux 使用 `供应商/模型名` 格式的唯一 Slug 标识模型。你可以在 [Models 页面](https://zenmux.ai/models) 查看所有模型及其 Slug。

---

## 五、智能路由

### 5.1 供应商路由

ZenMux 采用**多供应商架构**，同一模型配备多家供应商（如 Anthropic 原厂、AWS Bedrock、Google Vertex AI），通过智能路由选择最优供应商。

**指定供应商（后缀语法）：**

```json
{
  "model": "anthropic/claude-sonnet-4.6:amazon-bedrock",
  "messages": [{"role": "user", "content": "Hello!"}]
}
```

**按性能指标路由：**

```json
{
  "model": "anthropic/claude-sonnet-4",
  "messages": [{"role": "user", "content": "Hello!"}],
  "provider": {
    "routing": {
      "type": "priority",
      "primary_factor": "latency"
    }
  }
}
```

支持的路由维度：
- `latency` — 按首 Token 延迟排序
- `price` — 按综合价格排序
- `throughput` — 按吞吐量排序

### 5.2 模型路由（zenmux/auto）

使用 `zenmux/auto` 作为模型名，系统自动在候选模型池中选择最优模型：

```python
response = client.chat.completions.create(
    model="zenmux/auto",
    extra_body={
        "model_routing_config": {
            "available_models": [
                "anthropic/claude-sonnet-4.6",
                "openai/gpt-5",
                "google/gemini-2.5-flash-lite"
            ],
            "preference": "balanced"
        }
    },
    messages=[{"role": "user", "content": "解释量子计算"}]
)

print(f"选中的模型: {response.model}")
```

**偏好策略：**

| 策略 | 说明 |
|------|------|
| `balanced` | 均衡模式（默认），在性能和成本间寻求最佳平衡 |
| `performance` | 性能优先，优先选择最强模型 |
| `price` | 价格优先，优先选择经济型模型 |

### 5.3 模型兜底机制

当主要模型失败时，自动切换到备用模型：

```json
{
  "model": "anthropic/claude-sonnet-4",
  "messages": [{"role": "user", "content": "Hello!"}],
  "provider": {
    "fallback": "google/gemini-2.5-flash-lite"
  }
}
```

也可在控制台 [策略设置页面](https://zenmux.ai/settings/strategy) 设置全局兜底模型，无需修改代码。

---

## 六、AI 模型保险服务

ZenMux 是全球首个提供 AI 模型保险服务的平台，为模型调用效果提供兜底保障。

### 保险覆盖范围

- **内容质量不达标**：生成内容质量不符合预期
- **幻觉输出**：模型产生虚假或错误信息
- **高延迟**：响应时间超出合理范围

### 保险机制

1. 每天对平台调用数据自动进行保险检测
2. 触发保险条件的请求自动获得赔付
3. 赔付金额隔天自动到账（以 Credits 形式）

### 数据飞轮价值

保险算法挖掘出的数据都是高质量的 bad case，可用于：
- 优化你的 AI 产品
- 建立产品数据飞轮
- 持续改进应用效果

---

## 七、计费方案详解

### 7.1 Builder Plan（订阅制）

订阅制使用 **Flow** 作为复合计费单位，综合了 token 消耗量和 API 调用开销。

**当前汇率**：1 Flow ≈ $0.03283（约 30 Flows = $1）

| 套餐 | 价格 | 5h 配额 | 月最大 Flows | 等价 USD 价值 | 价值杠杆 |
|------|------|---------|-------------|--------------|---------|
| **Free** | $0/月 | 5 | 165.6 | $5.44 | - |
| **Pro** | $20/月 | 50 | 914 | $30.03 | 1.50x |
| **Max** | $100/月 | 300 | 5,487 | $180.15 | 1.80x |
| **Ultra** | $200/月 | 800 | 14,631 | $480.40 | 2.40x |

> [!tip] Insider 会员福利
> 保持活跃连续订阅的早期 Insider 会员，每个 Flow 将拥有更高的 USD 等价价值作为忠诚度奖励。

### 7.2 Pay As You Go（按量付费）

- **预充值 + 按实际消耗扣费**
- 无 Rate Limit 限制，支持高并发调用
- 按 Token 精确计费，成本透明可控
- 充值赠送 20% 额外 Credits

---

## 八、可观测性功能

ZenMux 提供全面的可观测性支持，帮助开发者深入了解 AI 模型使用情况：

| 功能 | 说明 |
|------|------|
| **请求日志** | 完整记录每次 API 调用的请求和响应详情 |
| **成本分析** | 从项目、模型、时间等维度分析成本分布 |
| **用量统计** | 实时监控 token 用量和调用频次 |
| **性能监控** | 追踪响应时间、并发数等关键指标 |
| **保险补偿** | 查看各类补偿数据、统计图表和明细记录 |

所有功能均可在 [Platform 控制台](https://zenmux.ai/platform) 中查看。

---

## 九、高级功能

### 9.1 模型别名

为常用模型设置别名，简化调用：

```json
{
  "model": "my-favorite-model",
  "messages": [{"role": "user", "content": "Hello!"}]
}
```

### 9.2 多模态支持

支持图片、视频等多模态输入输出，包括图片生成、视频生成等功能。

### 9.3 结构化输出

支持 JSON Schema 约束的结构化输出，确保模型返回符合预期格式的数据。

### 9.4 工具调用

支持 Function Calling / Tool Use，让模型调用外部工具和 API。

### 9.5 推理模型

支持 DeepSeek R1、o4-mini 等推理模型的 Thinking Mode。

### 9.6 提示词缓存

利用 Prompt Caching 降低成本、提升响应速度。

### 9.7 长上下文

支持 1M token 级别的长上下文窗口。

### 9.8 网络搜索

集成网络搜索能力，让模型获取实时信息。

---

## 十、主流工具集成

ZenMux 可无缝集成到各类开发工具中，以下是部分集成指南：

| 工具 | 配置方式 |
|------|---------|
| **Claude Code** | 设置 `ANTHROPIC_BASE_URL=https://zenmux.ai/api/anthropic` 和 `ANTHROPIC_AUTH_TOKEN` |
| **Cursor** | 设置 API Base URL 为 ZenMux 端点 |
| **CodeX** | 配置 OpenAI 兼容端点 |
| **Cline** | 配置 API Base URL 和 Key |
| **Cherry Studio** | 配置 OpenAI 兼容端点 |
| **Obsidian** | 通过插件配置 API 端点 |
| **GitHub Copilot** | 配置自定义 API 端点 |
| **Dify** | 配置模型供应商 |
| **Open WebUI** | 配置 OpenAI 兼容端点 |

### Claude Code 配置示例

```bash
# 添加到 ~/.bashrc 或 ~/.zshrc
export ANTHROPIC_BASE_URL="https://zenmux.ai/api/anthropic"
export ANTHROPIC_AUTH_TOKEN="sk-ss-v1-xxx"  # 替换为你的 ZenMux API Key
export ANTHROPIC_API_KEY=""  # 避免冲突
```

配置后即可在 Claude Code 中使用 GPT-5、Gemini 3、Grok 4 等非 Claude 模型。

---

## 十一、联系方式

| 渠道 | 信息 |
|------|------|
| **官网** | [https://zenmux.ai](https://zenmux.ai) |
| **文档** | [https://docs.zenmux.ai](https://docs.zenmux.ai) |
| **技术支持** | [support@zenmux.ai](mailto:support@zenmux.ai) |
| **商务合作** | [bd@zenmux.ai](mailto:bd@zenmux.ai) |
| **Twitter** | [@ZenMuxAI](https://twitter.com/ZenMuxAI) |
| **Discord** | [https://discord.gg/vHZZzj84Bm](https://discord.gg/vHZZzj84Bm) |
| **GitHub** | [https://github.com/ZenMux/zenmux-doc](https://github.com/ZenMux/zenmux-doc) |

---

## 参考链接

- [ZenMux 官方文档（中文）](https://docs.zenmux.ai/zh/)
- [快速开始指南](https://docs.zenmux.ai/zh/guide/quickstart)
- [模型列表](https://zenmux.ai/models)
- [价格页面](https://zenmux.ai/pricing/overview)
- [供应商路由文档](https://docs.zenmux.ai/zh/guide/advanced/provider-routing)
- [模型路由文档](https://docs.zenmux.ai/zh/guide/advanced/model-routing)
- [保险补偿文档](https://docs.zenmux.ai/zh/guide/observability/insurance)
- [订阅制套餐指南](https://docs.zenmux.ai/zh/guide/subscription)
- [Claude Code 集成指南](https://docs.zenmux.ai/zh/best-practices/claude-code)
