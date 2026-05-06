# LiteLLM 介绍与使用指南

> **项目地址**：[github.com/BerriAI/litellm](https://github.com/BerriAI/litellm)
> **官方文档**：[docs.litellm.ai](https://docs.litellm.ai)
> **许可证**：MIT
> **GitHub Stars**：20k+
> **语言**：Python

---

## 一、LiteLLM 是什么？

LiteLLM 是一个 Python 库和代理服务器，提供**统一的接口**来调用 100+ LLM 提供商（OpenAI、Anthropic、Google、Azure、AWS Bedrock、Ollama 等）。它的核心理念是：**用 OpenAI 的格式调用任何模型**。

简单说，你只需要改一行模型名称，就能在 Claude、GPT-4、Gemini、Llama 等模型之间无缝切换——代码几乎不用改。

### 核心定位

| 定位 | 说明 |
|------|------|
| **统一 API** | 用 OpenAI 格式调用 100+ LLM 提供商 |
| **Proxy 服务器** | 自托管的 OpenAI 兼容网关，统一管理 Key、预算、限速 |
| **零代码迁移** | 将 `import openai` 替换为 `import litellm`，即可支持所有模型 |
| **企业就绪** | 内置预算管理、RBAC、审计日志、SSO |

### 为什么选择 LiteLLM？

- **不再被锁定**：自由切换 OpenAI / Anthropic / Google / 本地模型，不绑定任何一家
- **统一格式**：所有模型的输入输出统一为 OpenAI 格式（`messages`、`tools`、`streaming`）
- **代理服务器**：一个 `config.yaml` 配置多模型，对外暴露标准 OpenAI API
- **内置健壮性**：自动 Fallback（降级）、重试、超时、负载均衡、速率限制
- **成本追踪**：实时统计 Token 用量和花费，支持预算上限
- **100+ 提供商**：从云端 API 到本地 Ollama/vLLM 全覆盖

---

## 二、支持的 LLM 提供商

### 云端提供商

| 提供商 | 示例模型 | 调用前缀 |
|--------|----------|----------|
| **OpenAI** | gpt-4o, gpt-4-turbo, o1 | `openai/` |
| **Anthropic** | claude-sonnet-4-20250514, claude-opus-4 | `anthropic/` |
| **Google** | gemini-2.5-pro, gemini-2.5-flash | `gemini/` |
| **Azure OpenAI** | Azure 部署的 GPT-4 | `azure/` |
| **AWS Bedrock** | Claude、Llama、Titan | `bedrock/` |
| **Mistral** | mistral-large, codestral | `mistral/` |
| **Cohere** | command-r-plus | `cohere/` |
| **DeepSeek** | deepseek-chat, deepseek-coder | `deepseek/` |
| **Groq** | llama-3, mixtral | `groq/` |
| **Together AI** | 开源模型推理 | `together_ai/` |
| **Anyscale** | 托管 Llama | `anyscale/` |
| **OpenRouter** | 聚合多模型 | `openrouter/` |

### 本地 / 自托管

| 提供商 | 说明 | 调用前缀 |
|--------|------|----------|
| **Ollama** | 本地运行 Llama、Mistral 等 | `ollama/` |
| **vLLM** | 高性能本地推理 | `openai/`（兼容接口） |
| **LM Studio** | 本地模型 GUI | `openai/` |
| **Hugging Face** | Inference API | `huggingface/` |

---

## 三、安装

### 基础安装（Python SDK）

```bash
pip install litellm
```

### 安装 Proxy 服务器

```bash
pip install 'litellm[proxy]'
```

### Docker 部署 Proxy

```bash
docker pull ghcr.io/berriai/litellm:main-latest
```

---

## 四、Python SDK 使用

### 4.1 基础调用

```python
import litellm
import os

# 设置 API Key（也可以用环境变量）
os.environ["OPENAI_API_KEY"] = "sk-xxx"
os.environ["ANTHROPIC_API_KEY"] = "sk-ant-xxx"

# 调用 OpenAI
response = litellm.completion(
    model="gpt-4o",
    messages=[{"role": "user", "content": "你好，介绍一下你自己"}]
)
print(response.choices[0].message.content)

# 调用 Claude —— 只需改模型名
response = litellm.completion(
    model="anthropic/claude-sonnet-4-20250514",
    messages=[{"role": "user", "content": "你好，介绍一下你自己"}]
)

# 调用 Gemini
response = litellm.completion(
    model="gemini/gemini-2.5-pro",
    messages=[{"role": "user", "content": "你好，介绍一下你自己"}]
)

# 调用本地 Ollama
response = litellm.completion(
    model="ollama/llama3",
    messages=[{"role": "user", "content": "你好"}],
    api_base="http://localhost:11434"
)
```

### 4.2 使用 OpenAI SDK 格式

LiteLLM 完全兼容 OpenAI SDK，只需替换 `base_url`：

```python
from openai import OpenAI
import litellm

# 方法一：用 litellm 的 completion（推荐）
response = litellm.completion(
    model="anthropic/claude-sonnet-4-20250514",
    messages=[{"role": "user", "content": "写一首诗"}]
)

# 方法二：用 OpenAI SDK + litellm router
client = OpenAI(
    api_key="anything",       # litellm proxy 不校验
    base_url="http://localhost:4000"  # 指向 litellm proxy
)
response = client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "你好"}]
)
```

### 4.3 流式输出（Streaming）

```python
response = litellm.completion(
    model="anthropic/claude-sonnet-4-20250514",
    messages=[{"role": "user", "content": "写一篇关于 AI 的文章"}],
    stream=True
)

for chunk in response:
    content = chunk.choices[0].delta.content or ""
    print(content, end="", flush=True)
```

### 4.4 Function Calling（工具调用）

```python
tools = [
    {
        "type": "function",
        "function": {
            "name": "get_weather",
            "description": "获取指定城市的天气",
            "parameters": {
                "type": "object",
                "properties": {
                    "city": {"type": "string", "description": "城市名称"}
                },
                "required": ["city"]
            }
        }
    }
]

response = litellm.completion(
    model="gpt-4o",
    messages=[{"role": "user", "content": "北京今天天气怎么样？"}],
    tools=tools
)

# 解析工具调用
tool_calls = response.choices[0].message.tool_calls
for call in tool_calls:
    print(f"调用函数: {call.function.name}")
    print(f"参数: {call.function.arguments}")
```

### 4.5 Embedding（向量化）

```python
response = litellm.embedding(
    model="openai/text-embedding-3-small",
    input=["这是一段文本", "这是另一段文本"]
)

for item in response.data:
    print(f"向量维度: {len(item.embedding)}")
```

### 4.6 异步调用

```python
import asyncio
import litellm

async def main():
    # 并发调用多个模型
    tasks = [
        litellm.acompletion(
            model="gpt-4o",
            messages=[{"role": "user", "content": "1+1=?"}]
        ),
        litellm.acompletion(
            model="anthropic/claude-sonnet-4-20250514",
            messages=[{"role": "user", "content": "1+1=?"}]
        ),
    ]
    results = await asyncio.gather(*tasks)

    for r in results:
        print(r.choices[0].message.content)

asyncio.run(main())
```

---

## 五、Proxy 服务器

LiteLLM Proxy 是一个**自托管的 OpenAI 兼容 API 网关**，提供统一的入口来管理多个 LLM 提供商。

### 5.1 核心功能

| 功能 | 说明 |
|------|------|
| **统一端点** | 所有模型通过 `/chat/completions` 调用 |
| **虚拟 Key** | 为团队成员生成独立的 API Key |
| **预算管理** | 按 Key / Team / User 设置花费上限 |
| **速率限制** | RPM / TPM 限制 |
| **Fallback** | 主模型失败自动切换备用模型 |
| **负载均衡** | 多个模型间分配流量 |
| **成本追踪** | 实时查看 Token 用量和花费 |
| **日志审计** | 记录所有请求和响应 |

### 5.2 配置文件 `config.yaml`

```yaml
model_list:
  # OpenAI GPT-4o
  - model_name: gpt-4o              # 对外暴露的模型名
    litellm_params:
      model: openai/gpt-4o          # 实际调用的模型
      api_key: os.environ/OPENAI_API_KEY

  # Anthropic Claude
  - model_name: claude-sonnet
    litellm_params:
      model: anthropic/claude-sonnet-4-20250514
      api_key: os.environ/ANTHROPIC_API_KEY

  # Google Gemini
  - model_name: gemini-pro
    litellm_params:
      model: gemini/gemini-2.5-pro
      api_key: os.environ/GEMINI_API_KEY

  # 本地 Ollama
  - model_name: llama-local
    litellm_params:
      model: ollama/llama3
      api_base: http://localhost:11434

# 通用设置
litellm_settings:
  drop_params: true                  # 自动丢弃不支持的参数
  set_verbose: false                 # 关闭详细日志

# 服务器设置
general_settings:
  master_key: sk-your-master-key     # 管理员密钥
```

### 5.3 启动 Proxy

```bash
# 直接启动
litellm --config config.yaml --port 4000

# 指定主机和端口
litellm --config config.yaml --host 0.0.0.0 --port 4000
```

### 5.4 Docker Compose 部署

```yaml
# docker-compose.yml
version: "3.8"
services:
  litellm:
    image: ghcr.io/berriai/litellm:main-latest
    ports:
      - "4000:4000"
    volumes:
      - ./config.yaml:/app/config.yaml
    environment:
      - OPENAI_API_KEY=${OPENAI_API_KEY}
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
      - GEMINI_API_KEY=${GEMINI_API_KEY}
    command: ["--config", "/app/config.yaml", "--port", "4000"]
    restart: unless-stopped

  # 可选：PostgreSQL 数据库（用于持久化日志和配置）
  db:
    image: postgres:16
    environment:
      POSTGRES_DB: litellm
      POSTGRES_USER: litellm
      POSTGRES_PASSWORD: litellm_password
    volumes:
      - litellm_db:/var/lib/postgresql/data
    restart: unless-stopped

volumes:
  litellm_db:
```

如果使用 PostgreSQL，在 `config.yaml` 中添加：

```yaml
general_settings:
  master_key: sk-your-master-key
  database_url: postgresql://litellm:litellm_password@db:5432/litellm
```

启动：

```bash
docker compose up -d
```

### 5.5 调用 Proxy

Proxy 启动后，任何兼容 OpenAI 的客户端都能直接使用：

```bash
# cURL
curl http://localhost:4000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer sk-your-master-key" \
  -d '{
    "model": "claude-sonnet",
    "messages": [{"role": "user", "content": "你好"}]
  }'
```

```python
# Python OpenAI SDK
from openai import OpenAI

client = OpenAI(
    api_key="sk-your-master-key",
    base_url="http://localhost:4000"
)

response = client.chat.completions.create(
    model="claude-sonnet",
    messages=[{"role": "user", "content": "你好"}]
)
print(response.choices[0].message.content)
```

### 5.6 管理 UI

LiteLLM Proxy 自带 Web 管理界面，访问：

```
http://localhost:4000/ui
```

在管理界面中可以：
- 查看 API 使用情况
- 管理虚拟 Key 和团队
- 配置模型和 Fallback
- 查看日志和成本报告

---

## 六、高级功能

### 6.1 Fallback（模型降级）

当主模型失败时自动切换到备用模型：

```python
# Python SDK
response = litellm.completion(
    model="gpt-4o",
    messages=[{"role": "user", "content": "你好"}],
    fallbacks=["anthropic/claude-sonnet-4-20250514", "gemini/gemini-2.5-pro"]
)
```

在 Proxy 的 `config.yaml` 中配置：

```yaml
router_settings:
  model_group_alias:
    "smart-model": ["gpt-4o", "anthropic/claude-sonnet-4-20250514"]

  fallbacks:
    - gpt-4o: ["claude-sonnet", "gemini-pro"]
    - claude-sonnet: ["gpt-4o"]
```

### 6.2 负载均衡

同一模型名配置多个实例，自动轮询：

```yaml
model_list:
  - model_name: gpt-4o
    litellm_params:
      model: openai/gpt-4o
      api_key: os.environ/OPENAI_API_KEY_1
  - model_name: gpt-4o
    litellm_params:
      model: azure/gpt-4o
      api_key: os.environ/AZURE_API_KEY
      api_base: https://your-azure.openai.azure.com
```

LiteLLM 会自动在两个 GPT-4o 端点之间分配请求。

### 6.3 预算与速率限制

```yaml
# 在 Proxy 中创建虚拟 Key 并设置限制
general_settings:
  master_key: sk-master-key

litellm_settings:
  max_budget: 100                    # 全局月预算（美元）
  budget_duration: 30d               # 预算周期
```

通过 API 创建受限制的 Key：

```bash
curl -X POST http://localhost:4000/key/generate \
  -H "Authorization: Bearer sk-master-key" \
  -H "Content-Type: application/json" \
  -d '{
    "max_budget": 10,
    "budget_duration": "7d",
    "max_parallel_requests": 5,
    "models": ["gpt-4o", "claude-sonnet"],
    "key_alias": "dev-team-key"
  }'
```

### 6.4 成本追踪

```python
# Python SDK 中追踪成本
response = litellm.completion(
    model="gpt-4o",
    messages=[{"role": "user", "content": "你好"}]
)

# 获取成本信息
print(f"模型: {response.model}")
print(f"输入 Token: {response.usage.prompt_tokens}")
print(f"输出 Token: {response.usage.completion_tokens}")
print(f"花费: ${litellm.completion_cost(response):.6f}")
```

### 6.5 结构化输出（JSON Mode）

```python
response = litellm.completion(
    model="gpt-4o",
    messages=[{"role": "user", "content": "列出三个编程语言"}],
    response_format={"type": "json_object"}
)
```

### 6.6 自定义 Prompt 模板

不同模型可能有不同的系统提示格式，LiteLLM 可以自定义：

```python
litellm.add_prompt_template(
    template="claude-template",
    messages=[
        {"role": "system", "content": "你是一个{role}"},
        {"role": "user", "content": "{question}"}
    ]
)
```

---

## 七、常见使用场景

### 场景一：多模型 A/B 测试

```python
import litellm
import json

models = ["gpt-4o", "anthropic/claude-sonnet-4-20250514", "gemini/gemini-2.5-pro"]
prompt = "解释量子计算的基本原理，用100字以内"

for model in models:
    response = litellm.completion(
        model=model,
        messages=[{"role": "user", "content": prompt}]
    )
    print(f"\n{'='*40}")
    print(f"模型: {model}")
    print(f"耗时: {response._hidden_params.get('response_time', 'N/A')}ms")
    print(f"内容: {response.choices[0].message.content}")
```

### 场景二：本地 + 云端混合

```python
import litellm

def smart_chat(message, use_local=True):
    """优先使用本地模型，失败则回退到云端"""
    try:
        if use_local:
            return litellm.completion(
                model="ollama/llama3",
                messages=[{"role": "user", "content": message}],
                api_base="http://localhost:11434",
                timeout=10
            )
    except Exception:
        print("本地模型不可用，切换到云端...")

    return litellm.completion(
        model="gpt-4o",
        messages=[{"role": "user", "content": message}],
        fallbacks=["anthropic/claude-sonnet-4-20250514"]
    )
```

### 场景三：为团队搭建统一 API 网关

```yaml
# config.yaml — 团队共享 API 网关
model_list:
  - model_name: fast
    litellm_params:
      model: groq/llama-3.1-70b-versatile
      api_key: os.environ/GROQ_API_KEY
  - model_name: smart
    litellm_params:
      model: anthropic/claude-sonnet-4-20250514
      api_key: os.environ/ANTHROPIC_API_KEY
  - model_name: cheap
    litellm_params:
      model: deepseek/deepseek-chat
      api_key: os.environ/DEEPSEEK_API_KEY

general_settings:
  master_key: sk-master-key
  database_url: postgresql://litellm:password@db:5432/litellm

litellm_settings:
  drop_params: true
```

团队成员用统一入口：

```python
from openai import OpenAI

# 开发者 A — 用 fast 模型
client = OpenAI(api_key="sk-team-key-a", base_url="http://gateway:4000")
client.chat.completions.create(model="fast", messages=[...])

# 开发者 B — 用 smart 模型
client.chat.completions.create(model="smart", messages=[...])
```

---

## 八、环境变量参考

| 变量 | 说明 | 示例 |
|------|------|------|
| `OPENAI_API_KEY` | OpenAI API 密钥 | `sk-...` |
| `ANTHROPIC_API_KEY` | Anthropic API 密钥 | `sk-ant-...` |
| `GEMINI_API_KEY` | Google AI API 密钥 | `AIza...` |
| `AWS_ACCESS_KEY_ID` | AWS Bedrock | `AKIA...` |
| `AZURE_API_KEY` | Azure OpenAI | `abc...` |
| `DEEPSEEK_API_KEY` | DeepSeek | `sk-...` |
| `GROQ_API_KEY` | Groq | `gsk_...` |
| `LITELLM_MASTER_KEY` | Proxy 管理密钥 | `sk-master-...` |
| `LITELLM_SALT_KEY` | Proxy 加密密钥 | `sk-salt-...` |

在 `config.yaml` 中用 `os.environ/VAR_NAME` 引用环境变量：

```yaml
litellm_params:
  model: openai/gpt-4o
  api_key: os.environ/OPENAI_API_KEY
```

---

## 九、常见问题

### Q1: 如何查看支持的模型完整列表？

```python
import litellm
print(litellm.model_list)
# 或查看模型成本
print(litellm.model_cost)
```

### Q2: 调用报错 "model not found"？

确保使用正确的模型前缀：
- OpenAI：`gpt-4o` 或 `openai/gpt-4o`
- Anthropic：`anthropic/claude-sonnet-4-20250514`（**必须加前缀**）
- Google：`gemini/gemini-2.5-pro`（**必须加前缀**）
- Ollama：`ollama/llama3`

### Q3: 如何处理不同模型的参数差异？

```python
litellm.modify_params = True  # 自动适配不同模型的参数

# 或者丢弃不支持的参数
litellm.drop_params = True
```

### Q4: 如何在 Proxy 中设置超时？

```yaml
litellm_settings:
  request_timeout: 60              # 全局超时（秒）

model_list:
  - model_name: gpt-4o
    litellm_params:
      model: openai/gpt-4o
      timeout: 30                  # 按模型设置超时
```

### Q5: 如何在 Cursor / Continue.dev 等工具中使用？

在工具的设置中，将 API Base URL 指向 LiteLLM Proxy，API Key 填 Proxy 的密钥：

```
API Base URL: http://localhost:4000
API Key: sk-your-master-key
```

这样就能让任何支持 OpenAI API 的工具使用 Claude、Gemini 等模型。

---

## 十、最佳实践

1. **使用 Proxy 管理生产环境**：不要在代码中硬编码 API Key，通过 Proxy 统一管理
2. **配置 Fallback**：关键业务至少配一个备用模型，避免单点故障
3. **设置预算上限**：给每个 Key / Team 设置预算，防止意外高额账单
4. **优先使用结构化输出**：用 `response_format` 确保 JSON 输出格式稳定
5. **本地模型做降级**：本地 Ollama 作为最后兜底，云端全挂也能响应
6. **定期检查成本**：利用 Proxy UI 或 `/spend` 端点监控花费
7. **使用 `drop_params: true`**：避免不同模型参数不兼容导致的报错

---

## 十一、相关资源

- [官方文档](https://docs.litellm.ai)
- [GitHub 仓库](https://github.com/BerriAI/litellm)
- [Proxy 部署指南](https://docs.litellm.ai/docs/proxy/deploy)
- [支持模型列表](https://docs.litellm.ai/docs/providers)
- [Cookbook 示例](https://github.com/BerriAI/litellm/tree/main/cookbook)
