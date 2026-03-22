# TrendRadar 使用指南

> **最快30秒部署的热点助手** —— 告别无效刷屏，只看真正关心的新闻资讯

## 📋 目录

- [项目简介](#项目简介)
- [核心功能](#核心功能)
- [快速开始](#快速开始)
- [部署方式](#部署方式)
- [配置详解](#配置详解)
- [AI智能分析](#ai智能分析)
- [常见问题](#常见问题)
- [高级功能](#高级功能)

---

## 项目简介

**TrendRadar** 是一个轻量级、易部署的热点助手工具，能够从多个主流平台聚合热点新闻，通过关键词筛选和AI智能分析，只推送真正关心的资讯内容。

### 主要特点

- ⚡ **超快部署**：最快30秒即可完成部署
- 🎯 **精准筛选**：关键词过滤 + AI智能筛选双重机制
- 🤖 **AI驱动**：支持AI智能分析和趋势预测
- 📱 **多渠道推送**：支持10+种推送渠道
- 🌐 **多平台支持**：覆盖11+主流热榜平台
- 📊 **可视化报告**：自动生成精美的HTML报告
- 🔄 **灵活调度**：支持5种预设时间模板

### 版本信息

- **当前版本**：v6.5.1
- **开源协议**：GPL-3.0
- **官方仓库**：[https://github.com/sansan0/TrendRadar](https://github.com/sansan0/TrendRadar)

---

## 核心功能

### 1. 全网热点聚合

默认监控以下11个主流平台：

| 平台 | 说明 |
|------|------|
| 知乎 | 知识问答社区热点 |
| 抖音 | 短视频平台热点 |
| 哔哩哔哩 | 弹幕视频网站热点 |
| 华尔街见闻 | 财经资讯 |
| 贴吧 | 百度贴吧热点 |
| 百度热搜 | 百度搜索引擎热点 |
| 财联社 | 金融资讯 |
| 澎湃新闻 | 新闻资讯 |
| 凤凰网 | 综合新闻 |
| 今日头条 | 个性化资讯 |
| 微博 | 社交媒体热点 |

### 2. RSS订阅源支持

- 支持RSS/Atom订阅源抓取
- 按关键词分组统计
- 自动过滤旧文章，避免重复推送
- 与热榜使用相同的筛选机制

### 3. 可视化配置编辑器

提供基于Web的图形化配置界面：

- **在线体验**：[https://sansan0.github.io/TrendRadar/](https://sansan0.github.io/TrendRadar/)
- 无需手动编辑YAML文件
- 表单化配置操作
- 实时预览配置效果

### 4. 智能推送策略

支持三种推送模式：

#### 模式一：当日汇总（daily）
- **适用场景**：企业管理者、普通用户
- **推送特点**：按时推送当日所有匹配新闻
- **优点**：全面了解当天热点

#### 模式二：当前榜单（current）
- **适用场景**：自媒体人、内容创作者
- **推送特点**：推送当前榜单匹配新闻
- **优点**：持续追踪在榜内容

#### 模式三：增量监控（incremental）
- **适用场景**：投资者、交易员
- **推送特点**：仅推送新增内容，零重复
- **优点**：精准捕获新热点

### 5. AI智能分析

- **AI智能筛选**：用自然语言描述兴趣方向，AI自动提取标签并打分
- **趋势分析**：自动生成热点趋势概述
- **情感分析**：识别舆论的正负面、争议情绪
- **跨平台关联**：发现不同平台的相关话题
- **多AI支持**：DeepSeek、OpenAI、Google Gemini等

### 6. 多渠道推送

支持以下10+种推送渠道：

| 渠道 | 类型 | 说明 |
|------|------|------|
| 企业微信 | 企业通讯 | 支持群机器人推送 |
| 个人微信 | 社交通信 | 通过企业微信应用推送到个人 |
| Telegram | 即时通讯 | 支持Bot推送 |
| 钉钉 | 企业通讯 | 支持群机器人推送 |
| 飞书 | 企业协作 | 支持Webhook推送 |
| 邮件 | Email | 支持HTML格式精美邮件 |
| ntfy | 开源推送 | 支持自托管服务器 |
| Bark | iOS推送 | 基于APNs的iOS推送 |
| Slack | 团队协作 | 支持Incoming Webhooks |
| Webhook | 通用接口 | 支持自定义Webhook |

---

## 快速开始

### 方式一：GitHub Actions 部署（推荐新手）

这是最简单的部署方式，无需自己的服务器。

#### 步骤1：Fork 项目

1. 访问 [https://github.com/sansan0/TrendRadar](https://github.com/sansan0/TrendRadar)
2. 点击右上角的 **Fork** 按钮
3. 将项目Fork到你的GitHub账号

#### 步骤2：配置关键词

1. 在你Fork的仓库中，找到 `config/frequency_words.txt` 文件
2. 点击编辑，添加你感兴趣的关键词

**关键词配置示例**：

```
# 科技类
人工智能
AI
ChatGPT
深度学习

# 金融类
股票
基金
理财
A股

# 娱乐类
电影
音乐
游戏
```

**高级语法**：

```
# 必须词（同时包含多个词）
+人工智能
+应用场景

# 过滤词（排除不想看的内容）
!广告
!营销
!八卦

# 正则表达式
/\bai\b/ => AI相关

# 分组显示
[科技前沿]
人工智能
机器学习

[投资理财]
A股
基金
```

#### 步骤3：配置推送渠道

选择一个推送渠道进行配置：

##### 飞书推送配置

1. 在飞书中创建一个群组
2. 群设置 → 群机器人 → 添加机器人 → 自定义机器人
3. 复制Webhook URL
4. 在GitHub仓库中：Settings → Secrets and variables → Actions → New repository secret
5. Name: `FEISHU_WEBHOOK_URL`, Value: 粘贴你的Webhook URL

##### 微信推送配置（企业微信）

1. 登录企业微信管理后台
2. 应用管理 → 创建应用 → 设置可信域名
3. 获取Secret和AgentId
4. 在GitHub Secrets中添加：
   - `WEWORK_WEBHOOK_URL`: 你的企业微信Webhook URL
   - `WEWORK_MSG_TYPE`: `markdown` 或 `text`

##### Telegram推送配置

1. 在Telegram中创建Bot（通过@BotFather）
2. 获取Bot Token
3. 获取Chat ID（可以通过@userinfobot获取）
4. 在GitHub Secrets中添加：
   - `TELEGRAM_BOT_TOKEN`: 你的Bot Token
   - `TELEGRAM_CHAT_ID`: 你的Chat ID

##### 钉钉推送配置

1. 在钉钉群中添加自定义机器人
2. 获取Webhook URL
3. 在GitHub Secrets中添加：
   - `DINGTALK_WEBHOOK_URL`: 钉钉Webhook URL

#### 步骤4：启用GitHub Actions

1. 在你Fork的仓库中，进入 **Actions** 标签
2. 点击 **I understand my workflows, go ahead and enable them**
3. 进入 **.github/workflows/crawler.yml**
4. 修改 `cron` 字段设置运行时间（默认为每天8点、20点运行）

**cron格式说明**：

```yaml
on:
  schedule:
    # 每天8点和20点运行（UTC时间）
    - cron: '0 0,12 * * *'
```

**cron字段含义**：
```
* * * * *
│ │ │ │ │
│ │ │ │ └── 星期几 (0-6, 0=周日)
│ │ │ └──── 月份 (1-12)
│ │ └────── 日期 (1-31)
│ └──────── 小时 (0-23)
└────────── 分钟 (0-59)
```

#### 步骤5：测试运行

1. 在 **Actions** 标签中，选择 **Crawler** workflow
2. 点击 **Run workflow** → **Run workflow**
3. 等待执行完成，检查你的推送渠道是否收到消息

### 方式二：Docker 部署

适合有服务器的用户。

#### 步骤1：安装Docker

确保你的系统已安装Docker和Docker Compose。

#### 步骤2：拉取镜像

```bash
docker pull wantcat/trendradar:latest
```

#### 步骤3：准备配置文件

```bash
# 克隆配置文件
git clone https://github.com/sansan0/TrendRadar.git
cd TrendRadar

# 编辑配置文件
vim config/frequency_words.txt
vim config/config.yaml
```

#### 步骤4：运行容器

```bash
docker run -d \
  --name trendradar \
  -v $(pwd)/config:/app/config \
  -v $(pwd)/output:/app/output \
  -e FEISHU_WEBHOOK_URL=your_webhook_url \
  wantcat/trendradar:latest
```

#### 步骤5：设置定时任务

使用系统定时任务或Docker自带的调度功能：

```bash
# 使用crontab设置每天8点和20点运行
crontab -e

# 添加以下行
0 8,20 * * * docker restart trendradar
```

### 方式三：本地运行

适合开发者和需要自定义的用户。

#### 步骤1：克隆项目

```bash
git clone https://github.com/sansan0/TrendRadar.git
cd TrendRadar
```

#### 步骤2：安装依赖

```bash
pip install -r requirements.txt
```

#### 步骤3：配置文件

编辑 `config/config.yaml` 和 `config/frequency_words.txt`

#### 步骤4：运行程序

```bash
python main.py
```

#### 步骤5：设置定时任务

**Linux/Mac**：

```bash
crontab -e

# 添加定时任务
0 8,20 * * * cd /path/to/TrendRadar && python main.py
```

**Windows**：

使用任务计划程序创建定时任务。

---

## 部署方式

### 1. GitHub Actions 部署

**优点**：
- 完全免费
- 无需服务器
- 配置简单
- 自动运行

**缺点**：
- 依赖GitHub服务
- 运行时间受限

**适用场景**：个人用户、轻度使用者

### 2. Docker 部署

**优点**：
- 环境隔离
- 部署快速
- 易于维护
- 跨平台

**缺点**：
- 需要服务器
- 需要Docker知识

**适用场景**：有服务器的用户、团队使用

### 3. 本地运行

**优点**：
- 完全控制
- 可自定义开发
- 无网络延迟

**缺点**：
- 需要配置环境
- 需要手动维护
- 电脑必须开机

**适用场景**：开发者、高度定制需求

### 4. 部署方式对比

| 特性 | GitHub Actions | Docker | 本地运行 |
|------|---------------|--------|----------|
| 难度 | ⭐ | ⭐⭐ | ⭐⭐⭐ |
| 费用 | 免费 | 需服务器 | 免费 |
| 稳定性 | 高 | 高 | 中 |
| 可控性 | 中 | 高 | 高 |
| 推荐度 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ |

---

## 配置详解

### 核心配置文件

TrendRadar 有三个核心配置文件：

1. **config.yaml** - 主配置文件
2. **frequency_words.txt** - 关键词配置
3. **timeline.yaml** - 时间调度配置

### 1. config.yaml 配置

#### 应用配置

```yaml
app:
  name: "TrendRadar"
  version: "6.5.1"
  timezone: "Asia/Shanghai"
  language: "zh-CN"
```

#### 爬虫配置

```yaml
crawler:
  enable_crawler: true  # 是否启用爬虫
  timeout: 30  # 请求超时时间（秒）
  retry_times: 3  # 重试次数
  user_agent: "Mozilla/5.0..."  # 用户代理
```

#### 运行模式

```yaml
app:
  report_mode: "daily"  # 运行模式：daily/current/incremental
```

**模式说明**：

- **daily**: 当日汇总模式，推送当天所有匹配的新闻
- **current**: 当前榜单模式，推送当前榜单中的新闻
- **incremental**: 增量监控模式，只推送新增的新闻

#### 推送配置

```yaml
notification:
  # 推送时间窗口（可选）
  push_window:
    enabled: false
    start_time: "08:00"
    end_time: "22:00"
    push_mode: "once"  # once: 每时间段推送一次, multiple: 多次推送

  # 推送渠道配置
  channels:
    feishu:
      enabled: true
      webhook_url: "${FEISHU_WEBHOOK_URL}"

    telegram:
      enabled: true
      bot_token: "${TELEGRAM_BOT_TOKEN}"
      chat_id: "${TELEGRAM_CHAT_ID}"

    email:
      enabled: false
      smtp_server: "smtp.gmail.com"
      smtp_port: 587
      sender_email: "your@gmail.com"
      sender_password: "your_password"
      recipients: ["recipient1@example.com", "recipient2@example.com"]
```

#### 平台配置

```yaml
platforms:
  enabled:
    - zhihu
    - douyin
    - bilibili
    - baidu
    - weibo
    - tieba
    - toutiao
    - pengpai
    - fenghuang
    - cailian
    - wallstreet

  disabled: []  # 禁用的平台
```

#### RSS配置

```yaml
rss:
  enabled: true
  feeds:
    - name: "科技博客"
      url: "https://example.com/rss"
      max_age_days: 7  # 文章最大天数
    - name: "新闻源"
      url: "https://news.example.com/rss"
      max_age_days: 3
```

#### 显示配置

```yaml
display:
  max_news_per_keyword: 10  # 每个关键词最多显示多少条新闻
  sort_by_position_first: true  # 是否按排名排序优先
  region_order:  # 推送区域显示顺序
    - hot_news
    - new_added
    - rss
    - standalone
    - ai_analysis
  regions:  # 各区域是否显示
    hot_news: true
    new_added: true
    rss: true
    standalone: true
    ai_analysis: true
```

#### 存储配置

```yaml
storage:
  type: "sqlite"  # 存储类型：sqlite/s3
  sqlite:
    path: "output"
  s3:
    endpoint_url: "https://your-s3-endpoint.com"
    bucket_name: "trendradar"
    access_key: "${S3_ACCESS_KEY}"
    secret_key: "${S3_SECRET_KEY}"
```

### 2. frequency_words.txt 配置

#### 基础语法

```
# 单个关键词
人工智能

# 多个关键词（空格或换行分隔）
机器学习
深度学习
神经网络

# 注释（以#开头）
# 这是注释，会被忽略
```

#### 高级语法

##### 1. 必须词（+号）

同时包含多个词才会收录：

```
# 标题必须同时包含"人工智能"和"应用"
+人工智能
+应用
```

##### 2. 过滤词（!号）

排除包含特定词的内容：

```
# 排除包含"广告"的内容
!广告

# 排除多个词
!八卦
!营销
!推广
```

##### 3. 正则表达式

使用正则表达式进行精确匹配：

```
# 精确匹配"AI"（不匹配"training"等）
/\bai\b/

# 匹配邮箱格式
/\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b/

# 匹配URL
/https?:\/\/[^\s]+/
```

##### 4. 显示名称（=>）

给关键词或正则表达式起一个好记的名字：

```
# 正则表达式的显示名称
/\bai\b/ => AI相关

# 关键词的显示名称
人工智能 => 人工智能和机器学习
```

##### 5. 分组显示（[]）

将关键词分组，推送时按组显示：

```
[科技前沿]
人工智能
机器学习
深度学习
量子计算

[投资理财]
A股
港股
基金
理财
```

##### 6. 全局过滤（[GLOBAL_FILTER]）

全局过滤不想看的内容：

```
[GLOBAL_FILTER]
!广告
!营销
!推广
!八卦
```

#### 配置示例

```
# ====================
# 科技类
# ====================
[人工智能]
+人工智能
+AI
ChatGPT
GPT-4
深度学习
机器学习
/\bai\b/ => AI相关

# ====================
# 金融投资
# ====================
[投资理财]
A股
港股
美股
基金
股票
理财
牛市
熊市

# ====================
# 前沿技术
# ====================
[前沿技术]
量子计算
区块链
元宇宙
Web3
芯片
半导体

# ====================
# 全局过滤
# ====================
[GLOBAL_FILTER]
!广告
!营销
!推广
!八卦
!震惊
!速看
```

### 3. timeline.yaml 配置

时间调度配置，控制什么时候采集、推送、分析。

#### 预设模板

系统提供5种预设模板：

```yaml
schedule:
  preset: "always_on"  # 预设模板
```

**模板说明**：

1. **always_on**（全天候，默认）
   - 24小时运行
   - 适合需要持续监控的用户

2. **morning_evening**（早晚汇总）
   - 早上8点、晚上8点推送
   - 适合普通用户

3. **office_hours**（办公时间）
   - 工作日9点-18点
   - 适合上班族

4. **night_owl**（夜猫子）
   - 晚上10点-凌晨2点
   - 适合夜猫子

5. **custom**（自定义）
   - 完全自定义时间
   - 适合有特殊需求的用户

#### 自定义时间配置

```yaml
schedule:
  preset: "custom"

  periods:
    - name: "早晨推送"
      enabled: true
      days: [1, 2, 3, 4, 5]  # 周一到周五
      start_time: "08:00"
      end_time: "09:00"

      tasks:
        crawl: true  # 是否采集
        push: true   # 是否推送
        analyze: true  # 是否AI分析

      filter:
        mode: "keyword"  # 筛选模式：keyword/ai
        keywords_file: "config/custom/keyword/morning.txt"
        ai_interests_file: "config/custom/ai/morning.txt"

    - name: "晚间推送"
      enabled: true
      days: [0, 1, 2, 3, 4, 5, 6]  # 每天都推送
      start_time: "20:00"
      end_time: "21:00"

      tasks:
        crawl: true
        push: true
        analyze: true

      filter:
        mode: "ai"  # 使用AI筛选
        ai_interests_file: "config/custom/ai/evening.txt"
```

---

## AI智能分析

### 1. AI分析功能

TrendRadar 支持多种AI分析功能：

#### AI智能筛选

不用手动设关键词，用自然语言描述兴趣：

**创建 `config/custom/ai/interests.txt`**：

```
我想看人工智能和新能源相关的新闻，特别是技术突破和产业应用方面的
关注金融市场的波动，特别是A股和科技股的动向
对太空探索和量子计算等前沿科技感兴趣
```

AI会自动：
- 提取关键词标签
- 对每条新闻打分
- 只推送相关内容

#### AI分析推送

对推送内容进行深度分析：

**分析维度**：

1. **核心热点态势**：当前最重要的热点
2. **舆论风向争议**：正负面情绪分析
3. **异动与弱信号**：潜在趋势和异常
4. **研判策略建议**：基于趋势的建议

### 2. AI配置

#### 配置AI提供商

```yaml
ai:
  enabled: true  # 是否启用AI

  # 模型配置（使用LiteLLM格式）
  model: "deepseek/deepseek-chat"  # 或 openai/gpt-4o, gemini/gemini-2.5-flash

  # API配置
  api_key: "${AI_API_KEY}"
  api_base: "${AI_API_BASE}"  # 可选

  # 高级配置
  num_retries: 3  # 重试次数
  fallback_models: []  # 备用模型
```

**支持的AI提供商**：

- DeepSeek（推荐，性价比高）
- OpenAI（GPT-4、GPT-3.5）
- Google（Gemini）
- Anthropic（Claude）
- 任何OpenAI兼容的API

#### AI分析模式

```yaml
ai:
  analysis:
    enabled: true
    mode: "both"  # only_analysis: 仅分析, both: 分析+推送

    include_rank_timeline: true  # 是否包含排名时间线
    include_standalone: true  # 是否分析独立展示区

    # 分析时间窗口
    window:
      enabled: true
      max_runs_per_day: 4  # 每天最多分析次数
```

#### AI翻译配置

```yaml
ai:
  translation:
    enabled: true
    target_language: "en"  # 目标语言
    translate_regions:  # 翻译哪些区域
      hot_news: true
      rss: true
      standalone: false
```

### 3. 自定义AI提示词

编辑 `config/ai_analysis_prompt.txt`：

```
你是一个专业的新闻分析师，擅长从海量信息中提取价值。

你的任务：
1. 分析推送的新闻内容
2. 识别核心热点和趋势
3. 评估舆论情感倾向
4. 提供策略建议

输出要求：
- 使用JSON格式
- 简洁明了，重点突出
- 字数限制：每个字段不超过200字

输出格式：
{
  "core_trends": "核心热点概述",
  "sentiment": "情感倾向分析",
  "weak_signals": "潜在趋势信号",
  "suggestions": "策略建议"
}
```

### 4. AI使用场景

#### 场景1：投资者监控市场

```
# config/custom/ai/investor.txt
关注A股、港股、美股的波动
特别关注科技股、新能源、生物医药板块
留意政策变化和市场情绪
```

#### 场景2：技术从业者追踪前沿

```
# config/custom/ai/tech.txt
人工智能、机器学习、深度学习的最新进展
大模型、AIGC、工具链的发展
编程语言、框架、云服务的更新
```

#### 场景3：产品经理了解趋势

```
# config/custom/ai/pm.txt
互联网产品的创新和迭代
用户体验和交互设计的趋势
新产品的发布和用户反馈
行业竞争格局的变化
```

---

## 常见问题

### 1. 推送问题

#### Q: 没有收到推送消息？

**可能原因**：

1. Webhook配置错误
2. GitHub Actions未启用
3. 定时任务时间未到
4. 没有匹配的新闻

**解决方法**：

1. 检查Webhook URL是否正确
2. 手动触发Actions测试
3. 检查关键词配置
4. 查看Actions运行日志

#### Q: 推送消息不完整？

**原因**：部分平台有消息长度限制

**解决方法**：

- 启用分批推送功能
- 减少每个关键词的显示数量
- 使用企业微信或Telegram（支持分批）

#### Q: 推送时间不准确？

**原因**：时区配置问题

**解决方法**：

```yaml
app:
  timezone: "Asia/Shanghai"  # 设置为你的时区
```

### 2. 配置问题

#### Q: 关键词不生效？

**检查清单**：

1. 关键词格式是否正确
2. 是否有过滤词冲突
3. 正则表达式语法是否正确
4. 文件编码是否为UTF-8

#### Q: 如何调试配置？

**方法**：

1. 手动运行程序查看输出
2. 查看GitHub Actions日志
3. 添加测试关键词验证

### 3. 部署问题

#### Q: GitHub Actions运行失败？

**常见原因**：

1. Secrets配置错误
2. workflow文件格式错误
3. 依赖包安装失败

**解决方法**：

1. 检查Secrets名称和值
2. 验证YAML格式
3. 查看Actions详细日志

#### Q: Docker容器无法启动？

**检查清单**：

1. Docker是否正常运行
2. 端口是否被占用
3. 挂载路径是否正确
4. 环境变量是否设置

### 4. 数据问题

#### Q: 新闻数据重复？

**原因**：

1. 增量模式配置错误
2. 数据库文件损坏
3. 平台返回重复数据

**解决方法**：

1. 检查运行模式配置
2. 清理数据库文件
3. 启用URL标准化

#### Q: 某些平台没有数据？

**原因**：

1. 平台被禁用
2. 网络连接问题
3. 平台API变化

**解决方法**：

1. 检查平台配置
2. 检查网络连接
3. 更新程序版本

### 5. AI问题

#### Q: AI分析失败？

**可能原因**：

1. API Key配置错误
2. API额度不足
3. 网络连接问题

**解决方法**：

1. 检查API Key
2. 检查API余额
3. 检查网络连接

#### Q: AI分析结果不准确？

**优化方法**：

1. 调整AI提示词
2. 更换AI模型
3. 优化兴趣描述

---

## 高级功能

### 1. 推送到多个群/设备

所有推送渠道都支持多账号配置：

```yaml
notification:
  channels:
    feishu:
      enabled: true
      # 使用分号分隔多个Webhook
      webhook_url: "url1;url2;url3"

    telegram:
      enabled: true
      # 使用分号分隔多个账号
      bot_token: "token1;token2"
      chat_id: "chat_id1;chat_id2"
```

### 2. 自定义推送区域

控制推送内容的显示区域：

```yaml
display:
  regions:
    hot_news: true  # 热榜新闻
    new_added: true  # 本次新增
    rss: true  # RSS订阅
    standalone: true  # 独立展示区
    ai_analysis: true  # AI分析
```

### 3. 独立展示区

指定平台的完整热榜或RSS源，不受关键词过滤：

```yaml
standalone:
  platforms:
    - name: "知乎热榜"
      enabled: true
      max_items: 10

  rss:
    - name: "精选RSS"
      enabled: true
      max_items: 5
```

### 4. 远程存储

支持S3兼容的远程存储：

```yaml
storage:
  type: "s3"
  s3:
    endpoint_url: "https://your-endpoint.com"
    bucket_name: "trendradar"
    access_key: "${S3_ACCESS_KEY}"
    secret_key: "${S3_SECRET_KEY}"
    region: "auto"
```

### 5. 网页版报告

自动生成精美的HTML报告：

```yaml
report:
  html:
    enabled: true
    output_path: "output/index.html"
    auto_open: false  # 是否自动打开浏览器
```

**访问方式**：

1. GitHub Pages：开启仓库的Pages功能
2. 本地访问：直接打开 `output/index.html`
3. Web服务器：使用内置Web服务器（端口8080）

### 6. MCP客户端支持

TrendRadar 支持MCP（Model Context Protocol）客户端：

**支持的客户端**：

- Claude Desktop
- Cherry Studio
- Cursor
- Cline

**功能**：

- 17种智能分析工具
- 自然语言交互
- 跨平台数据查询
- 趋势预测和分析

---

## 项目相关

### 更新日志

最新版本：**v6.5.1** (2026/03/12)

**主要更新**：

- AI智能筛选系统
- 每个时段支持不同的筛选方式
- AI分析范围独立于推送
- AI筛选智能省钱
- 多文件配置与标签隔离
- AI翻译精准控制

[查看完整更新日志](https://github.com/sansan0/TrendRadar#-更新日志)

### 支持项目

如果觉得TrendRadar好用，可以考虑支持项目：

- **GitHub Star**：给项目点个星⭐
- **分享推荐**：分享给朋友和同事
- **资金支持**：通过微信或支付宝赞赏

### 交流与反馈

- **GitHub Issues**：提交Bug和功能建议
- **公众号**：硅基茶水间
- **讨论社区**：LinuxDo

### 数据来源

本项目使用 [newsnow](https://github.com/ourongxing/newsnow) 项目的API获取数据，感谢作者提供的服务。

### 开源协议

本项目采用 **GPL-3.0** 开源协议。

---

## 附录

### A. 完整配置示例

```yaml
# config.yaml 完整示例

app:
  name: "TrendRadar"
  version: "6.5.1"
  timezone: "Asia/Shanghai"
  language: "zh-CN"
  report_mode: "daily"

crawler:
  enable_crawler: true
  timeout: 30
  retry_times: 3

notification:
  push_window:
    enabled: false

  channels:
    feishu:
      enabled: true
      webhook_url: "${FEISHU_WEBHOOK_URL}"

    telegram:
      enabled: false
      bot_token: "${TELEGRAM_BOT_TOKEN}"
      chat_id: "${TELEGRAM_CHAT_ID}"

platforms:
  enabled:
    - zhihu
    - douyin
    - bilibili
    - baidu
    - weibo
  disabled: []

rss:
  enabled: true
  feeds:
    - name: "示例RSS"
      url: "https://example.com/rss"
      max_age_days: 7

display:
  max_news_per_keyword: 10
  sort_by_position_first: true
  region_order:
    - hot_news
    - new_added
    - ai_analysis
  regions:
    hot_news: true
    new_added: true
    rss: false
    standalone: false
    ai_analysis: true

storage:
  type: "sqlite"
  sqlite:
    path: "output"

ai:
  enabled: true
  model: "deepseek/deepseek-chat"
  api_key: "${AI_API_KEY}"

  analysis:
    enabled: true
    mode: "both"
    include_rank_timeline: true
    include_standalone: true

schedule:
  preset: "morning_evening"
```

### B. 推荐配置模板

#### 模板1：投资者配置

```
# frequency_words.txt
[宏观经济]
GDP
CPI
通胀
通缩
货币政策

[A股市场]
上证指数
深证成指
创业板
科创板

[行业板块]
新能源
半导体
生物医药
消费电子
```

#### 模板2：技术从业者配置

```
# frequency_words.txt
[人工智能]
ChatGPT
GPT-4
深度学习
机器学习
大模型

[开发工具]
GitHub
VSCode
Docker
Kubernetes

[编程语言]
Python
JavaScript
Go
Rust
```

#### 模板3：产品经理配置

```
# frequency_words.txt
[产品动态]
新产品
功能更新
用户体验
交互设计

[行业趋势]
SaaS
数字化转型
低代码
无代码

[竞品分析]
阿里
腾讯
字节
美团
```

### C. 快速参考卡片

#### 关键词语法速查

| 语法 | 说明 | 示例 |
|------|------|------|
| `关键词` | 基础匹配 | `人工智能` |
| `+必须词` | 必须包含 | `+AI` |
| `!过滤词` | 排除内容 | `!广告` |
| `/正则/` | 正则表达式 | `/\bai\b/` |
| `=> 显示名` | 自定义名称 | `AI => 人工智能` |
| `[组名]` | 分组显示 | `[科技]` |

#### 推送模式对比

| 模式 | 特点 | 适用场景 |
|------|------|----------|
| `daily` | 当日汇总 | 全面了解热点 |
| `current` | 当前榜单 | 追踪在榜内容 |
| `incremental` | 仅推送新增 | 捕获新热点 |

#### 部署方式选择

| 场景 | 推荐方式 | 难度 |
|------|----------|------|
| 个人使用 | GitHub Actions | ⭐ |
| 团队使用 | Docker | ⭐⭐ |
| 开发定制 | 本地运行 | ⭐⭐⭐ |

---

## 结语

TrendRadar 是一个强大而灵活的热点助手工具，无论是个人追踪热点，还是团队监控资讯，都能满足你的需求。

希望这份使用指南能帮助你快速上手。如果你在使用过程中遇到任何问题，欢迎：

1. 查看[官方文档](https://github.com/sansan0/TrendRadar)
2. 提交[GitHub Issues](https://github.com/sansan0/TrendRadar/issues)
3. 关注公众号：硅基茶水间

**祝你使用愉快！** 🎉

---

*文档最后更新：2026年3月22日*
*TrendRadar版本：v6.5.1*
