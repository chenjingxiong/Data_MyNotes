---
title: Higress AI Gateway 介绍与开源 AI 网关对比指南
date: 2026-05-05
tags:
  - AI
  - gateway
  - higress
  - envoy
  - rust
  - go
  - cloud-native
  - CNCF
---

# Higress AI Gateway 介绍与开源 AI 网关对比指南

## 一、Higress 概述

**Higress** 是一款基于 **Istio + Envoy** 的云原生 API 网关，由阿里巴巴开源并已进入 **CNCF Sandbox** 孵化。它定位为 **AI Native API Gateway**，不仅能作为传统的 Kubernetes Ingress Controller 和微服务网关使用，更在 AI 场景下提供了开箱即用的能力。

> **GitHub**: [higress-group/higress](https://github.com/higress-group/higress)
> **官网**: [higress.io](https://higress.io)
> **Stars**: 8.3k ⭐ | **Forks**: 1.1k | **Contributors**: 175+
> **License**: Apache-2.0
> **语言分布**: Go 79.1% | C++ 14.3% | Rust 2.2% | Shell 1.2% | Python 1.0%

---

## 二、Higress 核心特性

### 2.1 AI Gateway 能力

| 能力 | 说明 |
|------|------|
| **多模型代理** | 支持 OpenAI、Anthropic、Azure、Gemini、通义千问、DeepSeek、Moonshot 等国内外主流 LLM 提供商 |
| **OpenAI 兼容 API** | 暴露 OpenAI 兼容接口，使用 OpenAI SDK 的应用可无缝切换后端 |
| **SSE 流式传输** | 原生支持 Server-Sent Events，完整流式处理请求/响应体 |
| **Token 限流** | 基于 Token 消耗的精细限流，支持按用户/Key 配额管理 |
| **多模型负载均衡** | 智能分配请求到多个模型端点，支持健康检查和自动故障转移 |
| **语义缓存** | 基于查询语义相似度缓存 AI 响应，降低延迟和成本 |
| **AI 内容安全** | 插件式内容审核，过滤请求/响应中的有害内容 |
| **MCP Server 托管** | 通过插件机制托管 MCP (Model Context Protocol) Server，支持 AI Agent 调用工具和服务 |
| **AI 可观测性** | 内置 Token 使用量、延迟、错误率等指标监控 |
| **openapi-to-mcp** | 一键将 OpenAPI 规范转换为远程 MCP Server |

### 2.2 网关通用能力

- **Kubernetes Ingress Controller**: 兼容 K8s Nginx Ingress 注解，支持 Ingress API 到 Gateway API 平滑迁移
- **微服务网关**: 支持从 Nacos、ZooKeeper、Consul、Eureka 等注册中心发现服务
- **安全网关**: WAF 防护、key-auth / JWT / OAuth2 等认证策略
- **Wasm 插件扩展**: 支持 Go / Rust / JS 编写插件，沙箱隔离，独立版本升级

### 2.3 核心优势

```
┌─────────────────────────────────────────────────────┐
│                   Higress 核心优势                    │
├──────────────┬──────────────────────────────────────┤
│ 生产级验证    │ 阿里巴巴内部 2 年+生产验证，          │
│              │ 支持数十万 QPS                        │
├──────────────┼──────────────────────────────────────┤
│ 毫秒级配置生效 │ 消除 Nginx reload 导致的流量抖动，   │
│              │ 配置毫秒级生效，对长连接友好           │
├──────────────┼──────────────────────────────────────┤
│ 完整流式处理  │ 真正的请求/响应体流式处理，           │
│              │ 高带宽场景显著降低内存开销              │
├──────────────┼──────────────────────────────────────┤
│ Wasm 扩展    │ 内存安全沙箱隔离，多语言支持，         │
│              │ 插件热更新零流量损失                   │
├──────────────┼──────────────────────────────────────┤
│ MCP 托管     │ 统一管理 LLM API 和 MCP API，        │
│              │ AI Agent 一站式工具调用                │
└──────────────┴──────────────────────────────────────┘
```

---

## 三、Higress 快速上手

### 3.1 Docker 一键启动

```bash
# 创建工作目录
mkdir higress && cd higress

# 启动 Higress AI 网关
docker run -d --rm --name higress-ai -v ${PWD}:/data \
    -p 8001:8001 -p 8080:8080 -p 8443:8443 \
    higress-registry.cn-hangzhou.cr.aliyuncs.com/higress/all-in-one:latest
```

| 端口 | 用途 |
|------|------|
| 8001 | Higress UI 控制台 |
| 8080 | HTTP 协议入口 |
| 8443 | HTTPS 协议入口 |

### 3.2 Kubernetes Helm 部署

```bash
# 使用北美镜像仓库（国内默认用杭州仓库）
helm install higress -n higress-system higress.io/higress \
    --set global.hub=higress-registry.us-west-1.cr.aliyuncs.com \
    --create-namespace
```

### 3.3 配置 AI 路由示例

通过 Higress Console (http://localhost:8001) 配置 AI 路由：
1. 添加 AI 服务来源（如 OpenAI、Anthropic 等）
2. 配置路由规则，将请求转发到对应的 LLM 提供商
3. 设置限流、认证等策略
4. 使用 OpenAI 兼容 API 调用

### 3.4 MCP Server 托管

```bash
# 使用 openapi-to-mcp 工具转换 OpenAPI 规范
# 然后通过 Higress 插件托管 MCP Server
# 体验平台: https://mcp.higress.ai/
```

---

## 四、开源 AI Gateway 全景对比

### 4.1 项目总览

| 项目 | 语言 | Stars | 定位 | License |
|------|------|-------|------|---------|
| **Higress** | Go / C++ (Envoy) | 8.3k ⭐ | AI Native API Gateway, CNCF Sandbox | Apache-2.0 |
| **Apache APISIX** | Lua / C (Nginx) | 16.6k ⭐ | 通用 API Gateway + AI 插件 | Apache-2.0 |
| **One API** | Go | 33k ⭐ | 多模型统一 API 管理 | MIT |
| **LiteLLM** | Python | 45.9k ⭐ | 100+ LLM 统一代理 | MIT |
| **Portkey Gateway** | TypeScript | 11.6k ⭐ | 轻量 AI Gateway | MIT |
| **TensorZero** | Rust | 11.3k ⭐ | LLMOps 平台 + Gateway | Apache-2.0 |
| **Glide** | Rust | 159 ⭐ | 云原生 LLM Gateway | Apache-2.0 |

> ⭐ Stars 数据截至 2026 年 5 月

### 4.2 重点对比：Go 语言 AI Gateway

| 特性 | Higress | One API | Apache APISIX |
|------|---------|---------|---------------|
| **核心语言** | Go + C++ (Envoy) | Go | Lua + C (Nginx) |
| **架构基础** | Istio + Envoy | 自研 | Nginx + etcd |
| **AI 代理** | ✅ 多模型统一路由 | ✅ 30+ 提供商 | ✅ 通过 AI 插件 |
| **SSE 流式** | ✅ 完整流式 | ✅ stream 模式 | ✅ 通过插件 |
| **负载均衡** | ✅ 多模型 + 健康检查 | ✅ 多渠道负载均衡 | ✅ 丰富策略 |
| **Token 限流** | ✅ 按 Token 消耗 | ✅ 额度管理 | ✅ Token 限流插件 |
| **语义缓存** | ✅ | ❌ | ✅ 通过插件 |
| **内容安全** | ✅ AI 内容审核 | ❌ | ✅ 通过插件 |
| **MCP 托管** | ✅ 原生支持 | ❌ | ✅ mcp-bridge 插件 |
| **K8s Ingress** | ✅ 原生支持 | ❌ | ✅ Ingress Controller |
| **服务发现** | ✅ Nacos/ZK/Consul | ❌ | ✅ etcd/Consul/Nacos |
| **管理控制台** | ✅ 开箱即用 | ✅ 完整 UI | ✅ Dashboard |
| **用户管理** | ✅ JWT/OAuth2 等 | ✅ 多种登录方式 | ✅ 通过插件 |
| **计费/额度** | ✅ Token 用量追踪 | ✅ 额度+兑换码 | ⚠️ 需自行实现 |
| **WAF** | ✅ 内置 | ❌ | ✅ 通过插件 |
| **Wasm 插件** | ✅ Go/Rust/JS | ❌ | ✅ 通过插件 |
| **生产验证** | 阿里巴巴大规模生产 | 社区广泛使用 | API7/多家企业 |

### 4.3 重点对比：Rust 语言 AI Gateway

| 特性 | TensorZero | Glide |
|------|------------|-------|
| **核心语言** | Rust | Rust |
| **定位** | LLMOps 全平台 (Gateway + 可观测 + 优化 + 评测) | 轻量 LLM Gateway |
| **Gateway 延迟** | <1ms p99 (10k+ QPS) | 未公布 |
| **AI 代理** | ✅ 20+ 提供商 | ✅ OpenAI/Anthropic/Azure/Bedrock/Cohere/Ollama |
| **SSE 流式** | ✅ | ✅ (部分) |
| **负载均衡** | ✅ 路由+回退+负载均衡 | ✅ 故障转移+重试 |
| **可观测性** | ✅ 内置 UI + OpenTelemetry | ✅ OpenTelemetry |
| **LLM 优化** | ✅ SFT/RLHF/Prompt 优化 | ❌ |
| **A/B 测试** | ✅ 内置 | ❌ |
| **Prompt 模板** | ✅ Schema 强制约束 | ❌ |
| **成本追踪** | ✅ 按标签范围 | ❌ |
| **OpenAI 兼容** | ✅ 通过 OpenAI SDK | ❌ 自有 API |
| **成熟度** | 高 (Fortune 10 使用，占全球 LLM API 支出 ~1%) | 早期 (开发中) |
| **MCP 支持** | ❌ | ❌ |

### 4.4 跨语言综合对比

| 维度          | Higress (Go/C++) | One API (Go) | LiteLLM (Python) | Portkey (TS) | TensorZero (Rust) | APISIX (Lua/C) |
| ----------- | ---------------- | ------------ | ---------------- | ------------ | ----------------- | -------------- |
| **网关延迟开销**  | ~1ms             | ~5-10ms      | ~8ms (P95)       | <1ms         | <1ms (p99)        | ~1ms           |
| **吞吐量**     | 极高 (数十万 QPS)     | 中等           | 中等               | 高            | 极高 (10k+ QPS)     | 极高             |
| **内存效率**    | 中等               | 中等           | 较高               | 低 (122kb)    | 极低                | 中等             |
| **GC 停顿**   | 有 (Go GC)        | 有 (Go GC)    | 有 (Python GC)    | 无 (V8 GC)    | 无                 | 有 (Lua GC)     |
| **流式性能**    | ★★★★★            | ★★★☆☆        | ★★★☆☆            | ★★★★☆        | ★★★★★             | ★★★★☆          |
| **生态丰富度**   | ★★★★☆            | ★★★★☆        | ★★★★★            | ★★★★☆        | ★★★☆☆             | ★★★★★          |
| **易用性**     | ★★★★☆            | ★★★★★        | ★★★★★            | ★★★★★        | ★★★☆☆             | ★★★☆☆          |
| **企业级特性**   | ★★★★★            | ★★★☆☆        | ★★★★☆            | ★★★★★        | ★★★★☆             | ★★★★★          |
| **AI 专项能力** | ★★★★★            | ★★★☆☆        | ★★★★★            | ★★★★☆        | ★★★★☆             | ★★★☆☆          |

---

## 五、Go vs Rust：AI Gateway 语言选择

### 5.1 Go 的优势与劣势

**优势**：
- **Goroutine 并发模型**：天生适合处理数千并发 LLM 代理请求
- **快速编译和部署**：迭代周期短，单二进制文件部署简单
- **成熟生态**：HTTP/gRPC 库丰富，容器化友好
- **开发者生产力**：学习曲线平缓，人才池大
- **代表项目**：Higress、One API、Traefik、Tyk

**劣势**：
- 内存占用相对较高
- GC 停顿（通常很小但存在）
- 底层性能优化空间有限

### 5.2 Rust 的优势与劣势

**优势**：
- **零成本抽象**：可匹配 C/C++ 性能
- **无 GC 停顿**：确定性延迟，对流式 AI 响应至关重要
- **极致内存效率**：适合边缘部署和资源受限场景
- **Tokio 异步运行时**：高性能网络 I/O
- **代表项目**：TensorZero、Glide

**劣势**：
- 学习曲线陡峭，开发速度较慢
- 编译时间较长
- 人才池较小
- Web 生态相对年轻

### 5.3 选型建议

```
┌────────────────────────────────────────────────────────┐
│              AI Gateway 语言选型决策树                    │
├────────────────────────────────────────────────────────┤
│                                                        │
│  需要极致吞吐和确定性延迟？                               │
│  ├─ 是 → Rust (TensorZero / Glide)                    │
│  └─ 否 ↓                                              │
│                                                        │
│  需要快速上手 + 大模型支持？                              │
│  ├─ 是 → Python (LiteLLM)                             │
│  └─ 否 ↓                                              │
│                                                        │
│  需要 K8s 原生 + 微服务 + AI 一体化网关？                 │
│  ├─ 是 → Go (Higress / APISIX)                        │
│  └─ 否 ↓                                              │
│                                                        │
│  只需要简单的多模型 API 管理？                            │
│  ├─ 是 → Go (One API)                                 │
│  └─ 否 ↓                                              │
│                                                        │
│  需要轻量级 + LLMOps 全链路？                            │
│  └─ 是 → Rust (TensorZero)                            │
│                                                        │
└────────────────────────────────────────────────────────┘
```

---

## 六、各项目适用场景

### 6.1 Higress — 最佳选择当

- ✅ 需要 K8s 原生 AI Gateway + Ingress Controller
- ✅ 需要同时管理 LLM API 和 MCP API
- ✅ 已在使用 Nacos / Dubbo 微服务技术栈
- ✅ 需要阿里巴巴级别的大规模生产验证
- ✅ 需要流式处理场景下的低内存开销
- ✅ 中国企业，需要国内云服务生态集成

### 6.2 One API — 最佳选择当

- ✅ 快速搭建多模型 API 管理，开箱即用
- ✅ 需要用户管理、额度管理、兑换码等运营功能
- ✅ 团队/小企业共享 API Key 的场景
- ✅ 不需要复杂的流量管理和微服务能力

### 6.3 LiteLLM — 最佳选择当

- ✅ Python 技术栈团队
- ✅ 需要 100+ LLM 提供商的广泛支持
- ✅ 需要与 LangChain / LlamaIndex 等框架集成
- ✅ 需要虚拟 Key、花费追踪、Guardrails 等内置功能

### 6.4 TensorZero — 最佳选择当

- ✅ 需要极致性能 (<1ms p99)
- ✅ 需要 LLMOps 全链路 (可观测 + 优化 + 评测 + A/B 测试)
- ✅ Rust 技术栈团队
- ✅ 需要从生产数据驱动 Prompt 和模型优化

### 6.5 Portkey Gateway — 最佳选择当

- ✅ 需要极致轻量 (122kb)
- ✅ Node.js / TypeScript 技术栈
- ✅ 需要 MCP Gateway 管理
- ✅ 快速原型和中小规模部署

### 6.6 Apache APISIX — 最佳选择当

- ✅ 已有 APISIX 基础设施，需要添加 AI 能力
- ✅ 需要丰富的通用 API Gateway 特性 + AI 插件
- ✅ 需要 MCP Bridge 将 stdio MCP 转为 HTTP SSE

---

## 七、性能参考数据

> ⚠️ 以下数据来自各项目官方文档/README，测试条件可能不同，仅供参考。

| 项目 | 延迟开销 | 吞吐量 | 内存占用 | 备注 |
|------|---------|--------|---------|------|
| **Higress** | ~1ms | 数十万 QPS | 中等 | Envoy C++ 数据面，Go 控制面 |
| **TensorZero** | <1ms p99 | 10k+ QPS | 极低 | Rust 原生，零 GC |
| **Portkey** | <1ms | 高 | 122kb | TypeScript/Worker |
| **LiteLLM** | ~8ms P95 | 1k RPS | 较高 | Python, 适合中小规模 |
| **One API** | ~5-10ms | 中等 | 中等 | Go, 带 Web 管理界面 |
| **APISIX** | ~1ms | 极高 | 中等 | Nginx C 核心 |
| **Glide** | 未公布 | 未公布 | 极低 | Rust, 早期开发中 |

---

## 八、总结

2025-2026 年的 AI Gateway 赛道呈现以下趋势：

1. **MCP 支持成为标配**：Higress、Portkey、APISIX 都已支持 MCP Server 托管/桥接
2. **流式性能是核心竞争力**：SSE/WebSocket 流式传输下，Rust 和 C++ 方案的零 GC 优势凸显
3. **Go 主导新建项目**：开发效率和大生态使得 Go 成为大多数 AI Gateway 的首选
4. **Rust 在高性能场景崛起**：TensorZero 等项目证明了 Rust 在延迟敏感场景的价值
5. **全链路 LLMOps**：从单纯的代理网关向可观测性、优化、评测一体化平台演进
6. **多语言融合**：Higress (Go + C++ + Rust + Wasm)、APISIX (Lua + C + Wasm) 等项目采用混合架构

**如果你是中国企业开发者**，推荐从 **Higress** 开始——它提供了最完整的 AI Gateway + K8s Ingress + 微服务网关 + MCP 托管一体化能力，且有阿里巴巴的生产验证背书。

**如果你追求极致性能**，关注 **TensorZero**——Rust 原生，<1ms p99 延迟，且提供 LLMOps 全链路能力。

**如果你只需要快速管理多模型 API**，**One API** 是最简单直接的选择。

---

## 参考链接

- [Higress GitHub](https://github.com/higress-group/higress)
- [Higress 官网](https://higress.io)
- [Higress MCP 平台](https://mcp.higress.ai/)
- [Apache APISIX GitHub](https://github.com/apache/apisix)
- [One API GitHub](https://github.com/songquanpeng/one-api)
- [LiteLLM GitHub](https://github.com/BerriAI/litellm)
- [Portkey AI Gateway GitHub](https://github.com/Portkey-ai/gateway)
- [TensorZero GitHub](https://github.com/TensorZero/tensorzero)
- [Glide GitHub](https://github.com/EinStack/glide)
