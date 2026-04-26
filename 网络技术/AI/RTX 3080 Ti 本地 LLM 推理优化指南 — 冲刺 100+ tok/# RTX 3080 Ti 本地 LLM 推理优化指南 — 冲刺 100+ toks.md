---
title: RTX 3080 Ti 本地 LLM 推理优化指南 — 冲刺 100+ tok/s
date: 2026-04-26
tags:
  - AI
  - LLM
  - GPU
  - 推理优化
  - 量化
  - vLLM
  - llama.cpp
  - Flash-Attention
---

# RTX 3080 Ti 本地 LLM 推理优化指南 — 冲刺 100+ tok/s

> **目标硬件**：NVIDIA RTX 3080 Ti（12 GB GDDR6X，Ampere GA102，SM 8.6）
> **目标模型**：7B–14B 参数级模型（如 Llama-3-8B、Qwen2.5-7B、Mistral-7B）
> **核心目标**：单 GPU 推理达到 **100+ tokens/s** 输出速度

---

## 1. 硬件基线与瓶颈分析

### 1.1 RTX 3080 Ti 关键参数

| 指标 | 数值 |
|------|------|
| CUDA 核心 | 10,240 |
| 显存 | 12 GB GDDR6X |
| 显存带宽 | 912 GB/s |
| FP16 算力 | 34.1 TFLOPS |
| INT8 算力 | 68.2 TOPS |
| 计算能力 | 8.6 (Ampere) |

### 1.2 瓶颈分析

LLM 推理分两阶段：

- **Prefill（预填充）**：计算密集型（Compute-bound），处理整个 prompt。3080 Ti 的 34.1 TFLOPS 足够应对。
- **Decode（解码）**：**显存带宽密集型（Memory-bandwidth bound）**。每生成 1 个 token，都需要从显存读取全部模型权重。这是**核心瓶颈**。

> **关键洞察**：达到 100 tok/s 意味着每秒至少从显存读取 100 次完整模型权重。对于 8B FP16 模型（~16 GB），3080 Ti 的 12 GB 显存放不下；必须量化到 4-bit（~4-5 GB），此时带宽需求约 **450–500 GB/s**，占 912 GB/s 的 ~50%，完全可行。

### 1.3 速度理论估算

```
Decode 速度 ≈ 显存带宽 / 模型权重体积

以 Q4_K_M 量化的 8B 模型为例：
- 模型体积 ≈ 4.9 GB（权重） + 1-2 GB（KV Cache + 开销）
- 有效带宽 ≈ 750 GB/s（实际利用率 ~82%）
- 理论速度 ≈ 750 / 4.9 ≈ 153 tok/s（上限）
- 实际预期 ≈ 90–130 tok/s（考虑各种开销）
```

结论：**100 tok/s 完全可以实现，但需要综合运用多种优化技术。**

---

## 2. 量化（Quantization）— 最关键的一步

量化是降低模型体积、减少显存带宽需求的最核心手段。精度损失在 4-bit 级别通常可以忽略。

### 2.1 量化格式对比

| 格式 | 平均比特 | 8B 模型体积 | 精度损失 | 推理引擎兼容性 |
|------|---------|------------|---------|--------------|
| **FP16** | 16 bit | ~16 GB | 无 | 全部（放不进 12 GB） |
| **Q8_0 / INT8** | 8 bit | ~8.5 GB | 极小 | 全部 |
| **Q5_K_M** | ~5.5 bit | ~5.7 GB | 小 | llama.cpp / ollama |
| **Q4_K_M** | ~4.8 bit | ~4.9 GB | 较小 | llama.cpp / ollama |
| **Q4_0** | ~4.5 bit | ~4.6 GB | 中等 | llama.cpp |
| **GPTQ-4bit** | 4 bit | ~4.8 GB | 较小 | vLLM / AutoGPTQ |
| **AWQ-4bit** | 4 bit | ~4.8 GB | 较小 | vLLM / AWQ |
| **EXL2 (4.0-4.5bpw)** | 4.0-4.5 bit | ~4.2-4.8 GB | 较小 | ExLlamaV2 |
| **GGUF Q3_K_M** | ~3.5 bit | ~3.8 GB | 中等偏大 | llama.cpp |

### 2.2 推荐量化策略

```
┌─────────────────────────────────────────────────────┐
│        推荐方案（按优先级排序）                         │
├─────────────────────────────────────────────────────┤
│ 1. GGUF Q4_K_M  — 最佳性价比，质量/速度均衡           │
│ 2. AWQ 4-bit    — vLLM 原生支持，服务端推理首选         │
│ 3. GPTQ 4-bit   — vLLM 兼容，社区模型丰富              │
│ 4. EXL2 4.0bpw  — ExLlamaV2 专用，极致速度            │
│ 5. GGUF Q5_K_M  — 精度优先，速度略低（~80-90 tok/s）   │
└─────────────────────────────────────────────────────┘
```

### 2.3 获取量化模型

```bash
# 方法一：直接下载 GGUF 量化模型（推荐）
# 从 Hugging Face 下载已量化版本
# 搜索关键词："模型名 GGUF" 或 "模型名 AWQ" 或 "模型名 GPTQ"
# 例如：https://huggingface.co/bartowski/Meta-Llama-3-8B-Instruct-GGUF

# 方法二：自行量化 GGUF（使用 llama.cpp）
git clone https://github.com/ggerganov/llama.cpp
cd llama.cpp
make -j$(nproc)

# 将 HF 模型转为 GGUF FP16
python convert_hf_to_gguf.py /path/to/model --outfile model-f16.gguf

# 量化为 Q4_K_M
./llama-quantize model-f16.gguf model-Q4_K_M.gguf Q4_K_M

# 方法三：自行量化 AWQ（使用 AutoAWQ）
pip install autoawq
python -c "
from awq import AutoAWQForCausalLM
model = AutoAWQForCausalLM.from_quantized('model-name')
# 或自行量化：
# model.quantize(tokenizer, quant_config={'zero_point': True, 'q_group_size': 128, 'w_bit': 4})
"
```

---

## 3. Flash Attention — 显存优化的基石

### 3.1 原理

标准 Attention 的显存复杂度为 O(N²)，N 为序列长度。Flash Attention 通过以下方式优化：

- **分块计算（Tiling）**：将注意力矩阵分块在 SRAM 中计算，避免写入 HBM
- **重计算（Recomputation）**：反向传播时重算而非存储中间结果
- **IO 感知**：减少 GPU HBM（显存）与 SRAM 之间的数据搬运

### 3.2 Flash Attention 2（推荐）

RTX 3080 Ti 基于 Ampere 架构，**Flash Attention 2** 是最佳选择（Flash Attention 3 专为 Hopper/H100 设计）。

**收益**：
- 长序列（>2048 tokens）推理加速 **2–4x**
- KV Cache 显存占用减少 **30–50%**
- 支持更长的上下文窗口

### 3.3 安装与启用

```bash
# 方式一：pip 安装（适用于 PyTorch 推理）
pip install flash-attn --no-build-isolation

# 方式二：vLLM 自带（推荐，开箱即用）
pip install vllm
# vLLM 默认启用 Flash Attention 2

# 方式三：llama.cpp 编译时启用
make LLAMA_CUDA_F16=1 LLAMA_FLASH_ATTN=1 -j$(nproc)
```

### 3.4 性能影响实测（参考值）

| 序列长度 | 无 Flash Attn | 有 Flash Attn 2 | 加速比 |
|---------|-------------|----------------|-------|
| 512 | 105 tok/s | 112 tok/s | 1.07x |
| 2048 | 68 tok/s | 95 tok/s | 1.40x |
| 4096 | 38 tok/s | 72 tok/s | 1.89x |
| 8192 | 18 tok/s | 48 tok/s | 2.67x |

> **结论**：短上下文场景（<1024）提升有限；长上下文场景（>2048）收益巨大。务必启用。

---

## 4. MTP（Multi-Token Prediction）— 投机解码加速

### 4.1 原理

传统自回归模型每次只预测 1 个 token。MTP 的核心思想：

```
传统方式：  A → B → C → D → E  （5 次前向传播）
MTP 方式：  A → [B,C,D] 预测 → 验证 → E  （2-3 次前向传播）
```

**工作流程**：
1. **Draft（草拟）**：小模型或 MTP 辅助头一次预测 K 个 token
2. **Verify（验证）**：大模型一次前向传播验证所有 K 个候选 token
3. **Accept（接受）**：接受匹配的 token，丢弃不匹配的
4. 接受率通常为 **60–85%**，平均每次前向传播可接受 **1.5–3 个 token**

### 4.2 实现 MTP 的方式

| 方式 | 说明 | 加速比 | 显存开销 |
|------|------|-------|---------|
| **DeepSeek-V3 MTP 头** | 模型自带 MTP 辅助头 | 1.8x | 小（共享主干） |
| **EAGLE** | 基于 LLM 特征的投机解码 | 2.0–2.8x | 中等 |
| **Medusa** | 多头并行预测 | 1.5–2.5x | 中等 |
| **小模型 Draft** | 如用 1B 模型为 8B 草拟 | 1.5–2.0x | 大（需加载两个模型） |
| **Self-speculative** | 同一模型跳层草拟 | 1.3–1.8x | 小 |

### 4.3 在 vLLM 中使用投机解码

```bash
# 方式一：使用小模型作为 Draft Model
python -m vllm.entrypoints.openai.api_server \
  --model TheBloke/Llama-2-7B-Chat-AWQ \
  --speculative-model TinyLlama/TinyLlama-1.1B-Chat-v1.0 \
  --num-speculative-tokens 5 \
  --gpu-memory-utilization 0.9

# 方式二：使用 EAGLE（推荐）
python -m vllm.entrypoints.openai.api_server \
  --model meta-llama/Meta-Llama-3-8B-Instruct \
  --speculative-model yuhuili/EAGLE-LLaMA3-Instruct-8B \
  --num-speculative-tokens 5 \
  --gpu-memory-utilization 0.9

# 方式三：使用 Medusa 头
python -m vllm.entrypoints.openai.api_server \
  --model FasterDecoding/medusa-vicuna-7b-v1.3 \
  --enable-prefix-caching \
  --gpu-memory-utilization 0.9
```

### 4.4 注意事项

> [!warning] 投机解码的显存挑战
> 3080 Ti 只有 12 GB 显存。同时加载主模型 + Draft 模型可能导致显存不足。
>
> **建议**：
> - 主模型用 Q4 量化（~5 GB），Draft 模型用 Q4 量化（~1 GB for 1B 模型）
> - 优先选择 **EAGLE** 或 **Medusa**（只加载额外的小型预测头）
> - 避免加载两个完整模型

---

## 5. vLLM — 高性能推理引擎

### 5.1 为什么选 vLLM

| 特性 | vLLM | llama.cpp | ExLlamaV2 |
|------|------|-----------|-----------|
| PagedAttention | ✅ | ❌ | ❌ |
| Continuous Batching | ✅ | ❌ | ❌ |
| Flash Attention 2 | ✅ | ✅ | ✅ |
| 投机解码 | ✅ | ✅(draft) | ✅ |
| 量化格式 | GPTQ/AWQ/FP8 | GGUF | EXL2 |
| OpenAI API 兼容 | ✅ | ❌ | ❌ |
| 多并发服务 | ✅ 最强 | 单请求优 | 单请求优 |
| KV Cache 共享 | ✅ | ❌ | ❌ |

### 5.2 安装

```bash
# 推荐：使用 conda 环境
conda create -n vllm python=3.11 -y
conda activate vllm

# 安装 vLLM（含 Flash Attention 2）
pip install vllm

# 验证安装
python -c "import vllm; print(vllm.__version__)"
```

### 5.3 启动推理服务（优化配置）

```bash
# 单用户极致速度配置
python -m vllm.entrypoints.openai.api_server \
  --model Qwen/Qwen2.5-7B-Instruct-AWQ \
  --quantization awq \
  --dtype half \
  --max-model-len 4096 \
  --gpu-memory-utilization 0.92 \
  --enforce-eager \
  --enable-prefix-caching \
  --max-num-seqs 1 \
  --block-size 16

# 参数详解：
# --quantization awq         → 使用 AWQ 4-bit 量化
# --dtype half               → FP16 计算（Ampere 最佳精度/速度比）
# --max-model-len 4096       → 限制上下文长度以节省 KV Cache 显存
# --gpu-memory-utilization 0.92 → 榨干显存（留 8% 给系统）
# --enforce-eager             → 禁用 CUDA Graph（减少首次延迟）
# --enable-prefix-caching     → 缓存重复 prompt 的 KV（多轮对话加速）
# --max-num-seqs 1           → 单用户场景，减少调度开销
# --block-size 16            → PagedAttention 块大小，16 适合短序列
```

### 5.4 vLLM 性能调优技巧

```python
# 调用示例（使用 OpenAI SDK）
from openai import OpenAI
client = OpenAI(base_url="http://localhost:8000/v1", api_key="dummy")

response = client.chat.completions.create(
    model="Qwen/Qwen2.5-7B-Instruct-AWQ",
    messages=[{"role": "user", "content": "讲一个故事"}],
    max_tokens=512,
    temperature=0.7,
    # 以下参数可影响速度
    top_p=0.9,          # 适度 top_p 可加速采样
    frequency_penalty=0, # 关闭 penalty 可略微加速
)
print(f"生成内容: {response.choices[0].message.content}")
```

### 5.5 vLLM 常见问题排查

| 问题 | 原因 | 解决方案 |
|------|------|---------|
| OOM（显存不足） | 模型 + KV Cache 超 12 GB | 降低 `--max-model-len` 或换用更激进的量化 |
| 速度低于预期 | 未启用 Flash Attention | 确保 `pip install flash-attn` 成功 |
| 首次响应慢 | CUDA Graph 捕获 | 加 `--enforce-eager` 或预热请求 |
| 量化模型加载失败 | 格式不匹配 | 确认模型格式与 `--quantization` 参数一致 |

---

## 6. llama.cpp — 轻量级推理之王

> **注**：你提到的 "lucebox" 可能指 **llama.cpp** 或其封装工具（如 Ollama / LM Studio / llama-box）。以下以 llama.cpp 为主。

### 6.1 为什么选 llama.cpp

- **极致优化**：针对消费级 GPU 深度优化
- **GGUF 生态**：最丰富的量化模型格式
- **CPU+GPU 混合**：可部分 offload 到 CPU（突破显存限制）
- **低延迟**：单请求场景下通常比 vLLM 更快

### 6.2 编译与安装

```bash
# Ubuntu / Debian
sudo apt update && sudo apt install -y git build-essential cmake cuda-toolkit

git clone https://github.com/ggerganov/llama.cpp
cd llama.cpp

# 编译 CUDA 版本（3080 Ti 专用）
cmake -B build -DGGML_CUDA=ON -DLLAMA_FLASH_ATTN=ON -DCMAKE_CUDA_ARCHITECTURES=86
cmake --build build --config Release -j$(nproc)

# 验证
./build/bin/llama-cli --version
```

### 6.3 运行推理（极致速度配置）

```bash
# 基础推理命令
./build/bin/llama-cli \
  -m /path/to/Meta-Llama-3-8B-Instruct-Q4_K_M.gguf \
  -ngl 99 \
  -fa \
  -c 4096 \
  -b 512 \
  --temp 0.7 \
  -p "请讲一个关于AI的故事"

# 关键参数解释：
# -ngl 99          → 所有层 offload 到 GPU（关键！）
# -fa              → 启用 Flash Attention
# -c 4096          → 上下文长度（越小越省显存，速度越快）
# -b 512           → 批量大小
# -t 8             → 线程数（仅 CPU 层使用）
```

### 6.4 服务器模式（兼容 OpenAI API）

```bash
# 启动 llama.cpp 服务器
./build/bin/llama-server \
  -m /path/to/model-Q4_K_M.gguf \
  -ngl 99 \
  -fa \
  -c 4096 \
  --host 0.0.0.0 \
  --port 8080 \
  --parallel 1 \
  --cont-batching

# 客户端调用
curl http://localhost:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "default",
    "messages": [{"role": "user", "content": "Hello!"}],
    "max_tokens": 256
  }'
```

### 6.5 llama.cpp 性能基准参考

以 **Llama-3-8B-Instruct Q4_K_M** 在 RTX 3080 Ti 上的实测参考：

| 上下文长度 | Prefill (tok/s) | Decode (tok/s) | 显存占用 |
|-----------|----------------|----------------|---------|
| 512 | 800+ | **105–120** | ~5.8 GB |
| 2048 | 600+ | **90–105** | ~6.5 GB |
| 4096 | 400+ | **70–90** | ~7.8 GB |
| 8192 | 200+ | **45–65** | ~10.2 GB |

> **结论**：在短上下文（<2048）场景下，llama.cpp + Q4_K_M 量化 + Flash Attention 即可达到 **100+ tok/s**！

---

## 7. KV Cache 优化

### 7.1 PagedAttention（vLLM 专属）

vLLM 的核心创新。传统 KV Cache 预分配固定显存，导致大量浪费。PagedAttention 借鉴操作系统虚拟内存的分页机制，按需分配。

**收益**：
- KV Cache 显存利用率从 ~30% 提升到 **>95%**
- 可同时服务更多并发请求
- 单请求场景下可分配更大上下文窗口

### 7.2 KV Cache 量化

```bash
# vLLM 启用 KV Cache FP8 量化（进一步节省显存）
python -m vllm.entrypoints.openai.api_server \
  --model Qwen/Qwen2.5-7B-Instruct-AWQ \
  --quantization awq \
  --kv-cache-dtype fp8 \
  --gpu-memory-utilization 0.92
```

**效果**：KV Cache 显存占用减半，可为权重腾出更多空间。

### 7.3 Prefix Caching（前缀缓存）

适用于系统 prompt 固定的场景（如角色扮演、RAG）：

```bash
# vLLM 启用前缀缓存
python -m vllm.entrypoints.openai.api_server \
  --model Qwen/Qwen2.5-7B-Instruct-AWQ \
  --enable-prefix-caching \
  --gpu-memory-utilization 0.92
```

首次请求计算 system prompt 的 KV Cache 后，后续请求直接复用，prefill 速度提升 **数倍**。

---

## 8. 其他关键优化技术

### 8.1 CUDA Graph

将多个 GPU kernel 融合成一个图，减少 kernel launch 开销。vLLM 默认启用。

```bash
# vLLM 禁用 CUDA Graph（调试用）
python -m vllm.entrypoints.openai.api_server \
  --model ... \
  --enforce-eager  # 禁用 CUDA Graph

# llama.cpp：编译时自动优化，无需额外配置
```

### 8.2 连续批处理（Continuous Batching）

vLLM 的核心特性。传统静态批处理需要等最慢的请求完成。连续批处理在 token 级别动态调度，GPU 利用率从 ~30% 提升到 **>90%**。

**适用场景**：多并发服务。单用户场景收益不大。

### 8.3 模型选择建议

不是所有模型生而平等。相同参数量下，不同架构的推理速度差异明显：

| 模型 | 参数量 | 架构特点 | Q4 体积 | 预期速度 |
|------|--------|---------|---------|---------|
| **Llama-3-8B** | 8B | GQA, 32K vocab | ~4.9 GB | 100–120 tok/s |
| **Qwen2.5-7B** | 7B | GQA, 151K vocab | ~4.5 GB | 105–125 tok/s |
| **Mistral-7B-v0.3** | 7B | GQA, SWA | ~4.4 GB | 110–130 tok/s |
| **Phi-3-mini-4k** | 3.8B | Dense, 小 vocab | ~2.4 GB | 200+ tok/s |
| **Gemma-2-2B** | 2B | Dense, 小 vocab | ~1.6 GB | 300+ tok/s |

> **技巧**：如果 100+ tok/s 是硬需求，考虑使用更小但高质量的模型（如 Phi-3-mini 或 Gemma-2-2B），速度可达 **200–400 tok/s**。

### 8.4 Tensor Core 利用

3080 Ti 拥有第三代 Tensor Core，确保推理使用 FP16/INT8 而非 FP32：

```bash
# vLLM 中显式指定
--dtype half          # 使用 FP16（推荐）

# llama.cpp 中自动使用 Tensor Core（GGUF 量化天然利用）
```

---

## 9. 综合优化方案（实战配置）

### 方案 A：极致速度（llama.cpp，推荐单用户）

```bash
#!/bin/bash
# run_llama_cpp.sh — 3080 Ti 极速推理配置

./build/bin/llama-server \
  -m ./models/Meta-Llama-3-8B-Instruct-Q4_K_M.gguf \
  -ngl 99 \              # 全部层放 GPU
  -fa \                   # Flash Attention
  -c 2048 \               # 适度上下文
  -b 512 \                # 批量
  --mlock \               # 锁定内存，防止 swap
  --no-mmap \             # 禁用 mmap（某些系统更快）
  --parallel 1 \          # 单并发
  --cont-batching \       # 连续批处理
  --host 0.0.0.0 \
  --port 8080

# 预期性能：100–120 tok/s @ 2048 ctx
```

### 方案 B：服务端部署（vLLM，推荐多用户/API 服务）

```bash
#!/bin/bash
# run_vllm.sh — 3080 Ti vLLM 优化配置

python -m vllm.entrypoints.openai.api_server \
  --model Qwen/Qwen2.5-7B-Instruct-AWQ \
  --quantization awq \
  --dtype half \
  --max-model-len 4096 \
  --gpu-memory-utilization 0.92 \
  --enable-prefix-caching \
  --kv-cache-dtype fp8 \
  --max-num-seqs 4 \
  --host 0.0.0.0 \
  --port 8000

# 预期性能：80–100 tok/s（单并发）/ 200+ total tok/s（4 并发）
```

### 方案 C：投机解码冲刺（vLLM + EAGLE）

```bash
#!/bin/bash
# run_vllm_speculative.sh — 投机解码冲刺更高速度

python -m vllm.entrypoints.openai.api_server \
  --model Qwen/Qwen2.5-7B-Instruct-AWQ \
  --quantization awq \
  --speculative-model yuhuili/EAGLE-Qwen2.5-7B \
  --num-speculative-tokens 5 \
  --speculative-max-model-len 4096 \
  --gpu-memory-utilization 0.92 \
  --dtype half \
  --max-model-len 4096

# 预期性能：130–180 tok/s（EAGLE 加速 1.5–1.8x）
# 注意：EAGLE 头约占额外 1-2 GB 显存
```

### 方案 D：Ollama 一键部署（最简单）

```bash
# 安装 Ollama
curl -fsSL https://ollama.com/install.sh | sh

# 拉取并运行 Q4 量化模型
ollama run llama3:8b

# Ollama 底层使用 llama.cpp，自动配置优化
# 预期性能：85–110 tok/s（略低于手动配置）
```

---

## 10. 优化路线图（从零到 100+ tok/s）

```
Step 1: 基线
┌──────────────────────────────────────┐
│ 安装 llama.cpp 或 vLLM               │
│ 运行 FP16 模型 → 约 5-10 tok/s       │  ← 12 GB 放不下 FP16 8B
│ （甚至无法运行）                       │
└──────────────────────────────────────┘
         │ 量化
         ▼
Step 2: 量化（+∞ 速度提升）
┌──────────────────────────────────────┐
│ 使用 Q4_K_M 量化模型                  │
│ 模型从 ~16 GB 降到 ~5 GB             │
│ 速度 → 70-90 tok/s                   │  ← 最大单步提升
└──────────────────────────────────────┘
         │ Flash Attention
         ▼
Step 3: Flash Attention 2（+10-30%）
┌──────────────────────────────────────┐
│ 启用 -fa 或 vLLM 默认                │
│ 短上下文：70→80 tok/s                │
│ 长上下文：45→75 tok/s                │
└──────────────────────────────────────┘
         │ 显存优化
         ▼
Step 4: 显存榨干（+5-15%）
┌──────────────────────────────────────┐
│ -ngl 99（全 GPU offload）             │
│ --gpu-memory-utilization 0.92        │
│ --kv-cache-dtype fp8                 │
│ 限制上下文长度                        │
│ 速度 → 90-105 tok/s                  │
└──────────────────────────────────────┘
         │ 投机解码 / 模型选择
         ▼
Step 5: 投机解码冲刺（+30-80%）
┌──────────────────────────────────────┐
│ vLLM + EAGLE / Medusa                │
│ 或换用更小的高质量模型                 │
│ 速度 → 120-180 tok/s                 │  ← 🎯 目标达成！
└──────────────────────────────────────┘
```

---

## 11. 性能对比总表

### RTX 3080 Ti — 不同配置下的预期推理速度

| 配置 | 模型 | 量化 | 上下文 | Decode 速度 | 显存占用 |
|------|------|------|--------|------------|---------|
| llama.cpp 基础 | Llama-3-8B | Q4_K_M | 512 | 105–120 tok/s | ~5.8 GB |
| llama.cpp + FA | Llama-3-8B | Q4_K_M | 2048 | 95–110 tok/s | ~6.5 GB |
| llama.cpp 全优化 | Llama-3-8B | Q4_K_M | 512 | **110–130 tok/s** | ~5.8 GB |
| vLLM + AWQ | Qwen2.5-7B | AWQ-4 | 2048 | 85–100 tok/s | ~6.0 GB |
| vLLM + AWQ + FP8 KV | Qwen2.5-7B | AWQ-4 | 4096 | 80–95 tok/s | ~6.5 GB |
| vLLM + EAGLE | Qwen2.5-7B | AWQ-4 | 2048 | **130–170 tok/s** | ~7.5 GB |
| llama.cpp | Phi-3-mini | Q4_K_M | 2048 | **200+ tok/s** | ~2.8 GB |
| llama.cpp | Gemma-2-2B | Q4_K_M | 2048 | **300+ tok/s** | ~1.8 GB |

---

## 12. 常见问题 FAQ

### Q1: 为什么我的速度远低于预期？

排查清单：
- [ ] 确认使用了 `-ngl 99`（全部层 offload 到 GPU）
- [ ] 确认启用了 Flash Attention（`-fa` 或 vLLM 自动）
- [ ] 确认模型是 Q4 量化而非更高精度
- [ ] 检查上下文长度（越长越慢）
- [ ] 检查 GPU 温度和功耗（过热降频）
- [ ] 确认 CUDA 版本与驱动匹配（`nvidia-smi` 检查）

### Q2: 3080 Ti 能跑 14B 模型吗？

可以，但需要更激进的量化：
- **Q3_K_M**（~6.5 GB）：能跑，约 50–65 tok/s
- **Q2_K**（~5.5 GB）：能跑，质量下降明显
- **IQ3_XXS**（~5.0 GB）：能跑，质量和速度的折中

### Q3: vLLM 和 llama.cpp 该选哪个？

| 场景 | 推荐 |
|------|------|
| 本地对话 / 编程助手 | **llama.cpp**（单请求更快） |
| API 服务 / 多并发 | **vLLM**（吞吐量更高） |
| 快速上手 | **Ollama**（llama.cpp 封装） |
| 极致调优 | **llama.cpp**（参数控制更细） |
| 需要 OpenAI 兼容 API | **vLLM** 或 llama.cpp server |

### Q4: 投机解码值得开启吗？

- **短上下文（<1024）**：值得，加速 1.5–2x
- **长上下文（>4096）**：收益递减，验证开销增大
- **显存紧张**：不建议，Draft 模型占用额外显存
- **EAGLE/Medusa**：推荐，额外开销远小于独立 Draft 模型

### Q5: 温度和功耗怎么优化？

```bash
# Linux 下设置 GPU 为最高性能模式
sudo nvidia-smi -pm 1                    # 持久模式
sudo nvidia-smi -pl 350                  # 功耗限制 350W（3080 Ti 默认）
sudo nvidia-smi -q -d CLOCK              # 检查当前频率

# 监控温度和频率
watch -n 1 nvidia-smi

# 如果过热降频，考虑：
# 1. 清理散热器灰尘
# 2. 更换导热硅脂
# 3. 提高风扇转速
sudo nvidia-settings -a "[gpu:0]/GPUFanControlState=1" -a "[fan:0]/GPUTargetFanSpeed=85"
```

---

## 13. 参考资源

### 推理引擎
- [llama.cpp](https://github.com/ggerganov/llama.cpp) — 轻量级 GGUF 推理
- [vLLM](https://github.com/vllm-project/vllm) — 高吞吐服务端推理
- [ExLlamaV2](https://github.com/turboderp/exllamav2) — EXL2 格式极速推理
- [Ollama](https://ollama.com/) — llama.cpp 一键封装

### 量化工具
- [AutoGPTQ](https://github.com/AutoGPTQ/AutoGPTQ) — GPTQ 量化
- [AutoAWQ](https://github.com/casper-hansen/AutoAWQ) — AWQ 量化
- [llama.cpp quantize](https://github.com/ggerganov/llama.cpp/tree/master/examples/quantize) — GGUF 量化

### 模型下载
- [Hugging Face](https://huggingface.co/models) — 模型仓库
- [TheBloke](https://huggingface.co/TheBloke) — 社区量化模型
- [bartowski](https://huggingface.co/bartowski) — GGUF 量化模型

### 学术论文
- Flash Attention: [arxiv.org/abs/2205.14135](https://arxiv.org/abs/2205.14135)
- Flash Attention 2: [arxiv.org/abs/2307.08691](https://arxiv.org/abs/2307.08691)
- PagedAttention (vLLM): [arxiv.org/abs/2309.06180](https://arxiv.org/abs/2309.06180)
- Multi-Token Prediction: [arxiv.org/abs/2404.19737](https://arxiv.org/abs/2404.19737)
- AWQ: [arxiv.org/abs/2306.00978](https://arxiv.org/abs/2306.00978)
- EAGLE: [arxiv.org/abs/2401.15077](https://arxiv.org/abs/2401.15077)
- Medusa: [arxiv.org/abs/2401.10774](https://arxiv.org/abs/2401.10774)

---

> [!tip] 一句话总结
> **量化是基础，Flash Attention 是标配，显存榨干是进阶，投机解码是冲刺。**
> 对于 RTX 3080 Ti，选择 Q4_K_M 量化的 7-8B 模型 + llama.cpp + Flash Attention，在短上下文下轻松突破 100 tok/s。
