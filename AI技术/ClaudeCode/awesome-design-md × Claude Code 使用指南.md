# awesome-design-md × Claude Code 使用指南

> **副标题**：一份 DESIGN.md，让 AI 生成像素级精准的 UI
> **创建日期**：2026-04-24
> **标签**：#AI #ClaudeCode #DESIGN.md #UI设计 #GoogleStitch

---

## 目录

- [[#1. 什么是 awesome-design-md]]
- [[#2. 什么是 DESIGN.md]]
- [[#3. DESIGN.md 的 9 大结构]]
- [[#4. 收录网站一览]]
- [[#5. Claude Code 如何使用 DESIGN.md]]
- [[#6. 实战：使用 DESIGN.md 构建项目]]
- [[#7. 高级用法与最佳实践]]
- [[#8. 自定义 DESIGN.md]]
- [[#9. 与 Google Stitch 的关系]]
- [[#10. 常见问题]]

---

## 1. 什么是 awesome-design-md

[**awesome-design-md**](https://github.com/VoltAgent/awesome-design-md) 是由 [VoltAgent](https://github.com/VoltAgent) 团队维护的开源项目，收录了 **69 份** 从真实网站提取的 DESIGN.md 文件。

> **一句话概括**：复制一份 DESIGN.md 到项目中，告诉 AI Agent "照着这个风格做"，就能获得像素级精准的 UI。

| 维度 | 说明 |
|------|------|
| **仓库地址** | [github.com/VoltAgent/awesome-design-md](https://github.com/VoltAgent/awesome-design-md) |
| **维护者** | VoltAgent 团队 |
| **收录数量** | 69 份 DESIGN.md（持续增长中） |
| **许可证** | MIT License |
| **核心价值** | 让 AI 编码助手生成风格一致的 UI |

**核心理念**：

```
传统方式：
  产品经理口头描述 → 开发者凭感觉写 UI → 反复调整风格（数天）

使用 DESIGN.md：
  复制 DESIGN.md 到项目 → AI Agent 读取设计规范 → 一次性生成精准 UI（分钟级）
```

---

## 2. 什么是 DESIGN.md

### 2.1 概念起源

**DESIGN.md** 是 [Google Stitch](https://stitch.withgoogle.com/) 引入的新概念 —— 一种纯文本设计系统文档，AI Agent 可以直接阅读并据此生成一致的 UI。

### 2.2 与其他规范文件的对比

| 文件 | 谁来读 | 定义什么 |
|------|--------|----------|
| `README.md` | 人类开发者 | 项目概述和入门指南 |
| `CLAUDE.md` / `AGENTS.md` | 编码 Agent | 如何构建项目（代码规范） |
| **`DESIGN.md`** | **设计 Agent** | **项目应该看起来什么样（视觉规范）** |

### 2.3 为什么是 Markdown？

- 🤖 **LLM 天生理解 Markdown**：无需额外解析器或配置
- 📝 **纯文本，零工具链**：不需要 Figma 导出、JSON Schema 或特殊工具
- 📂 **放入项目根目录即可**：AI 编码助手自动识别
- 🔧 **易于版本控制**：Git 友好，diff 可追踪设计变更

---

## 3. DESIGN.md 的 9 大结构

每份 DESIGN.md 遵循 [Stitch DESIGN.md 格式规范](https://stitch.withgoogle.com/docs/design-md/format/)，包含以下 9 个标准章节：

| # | 章节 | 描述 | 示例内容 |
|---|------|------|----------|
| 1 | **Visual Theme & Atmosphere** | 视觉主题与氛围 | 情绪基调、设计密度、设计哲学 |
| 2 | **Color Palette & Roles** | 色彩系统 | 语义化颜色名称 + HEX 值 + 功能角色 |
| 3 | **Typography Rules** | 排版规则 | 字体族、完整字号层级表 |
| 4 | **Component Stylings** | 组件样式 | 按钮、卡片、输入框、导航栏及各状态 |
| 5 | **Layout Principles** | 布局原则 | 间距系统、网格、留白哲学 |
| 6 | **Depth & Elevation** | 层级与深度 | 阴影系统、表面层级 |
| 7 | **Do's and Don'ts** | 设计守则 | 设计护栏与反模式 |
| 8 | **Responsive Behavior** | 响应式行为 | 断点、触摸目标、折叠策略 |
| 9 | **Agent Prompt Guide** | Agent 提示词指南 | 快速颜色参考、可直接使用的提示词 |

### 每个网站包含的文件

| 文件 | 用途 |
|------|------|
| `DESIGN.md` | 设计系统文档（AI Agent 读取的主文件） |
| `preview.html` | 视觉预览目录：颜色色板、字号梯度、按钮、卡片 |
| `preview-dark.html` | 暗色模式下的视觉预览 |

---

## 4. 收录网站一览

### AI & LLM 平台

| 网站 | 风格特征 |
|------|----------|
| [**Claude**](https://getdesign.md/claude/design-md) | 温暖陶土色调、干净编辑式布局 |
| [**Cohere**](https://getdesign.md/cohere/design-md) | 鲜艳渐变、数据密集仪表盘美学 |
| [**ElevenLabs**](https://getdesign.md/elevenlabs/design-md) | 暗色电影级 UI、音频波形美学 |
| [**Mistral AI**](https://getdesign.md/mistral.ai/design-md) | 法式极简、紫色调 |
| [**Ollama**](https://getdesign.md/ollama/design-md) | 终端优先、单色简约 |
| [**Replicate**](https://getdesign.md/replicate/design-md) | 干净白色画布、代码优先 |
| [**RunwayML**](https://getdesign.md/runwayml/design-md) | 电影级暗色 UI、富媒体布局 |
| [**VoltAgent**](https://getdesign.md/voltagent/design-md) | 纯黑画布、翡翠绿强调、终端原生 |
| [**xAI**](https://getdesign.md/x.ai/design-md) | 冷峻单色、未来极简主义 |

### 开发者工具 & IDE

| 网站 | 风格特征 |
|------|----------|
| [**Cursor**](https://getdesign.md/cursor/design-md) | 流畅暗色界面、渐变强调 |
| [**Lovable**](https://getdesign.md/lovable/design-md) | 活泼渐变、友好开发美学 |
| [**Raycast**](https://getdesign.md/raycast/design-md) | 精致暗色、鲜艳渐变点缀 |
| [**Vercel**](https://getdesign.md/vercel/design-md) | 黑白精准、Geist 字体 |
| [**Warp**](https://getdesign.md/warp/design-md) | 现代 IDE 风终端、块状命令 UI |

### 后端、数据库 & DevOps

| 网站 | 风格特征 |
|------|----------|
| [**ClickHouse**](https://getdesign.md/clickhouse/design-md) | 黄色强调、技术文档风格 |
| [**MongoDB**](https://getdesign.md/mongodb/design-md) | 绿叶品牌、开发者文档风格 |
| [**PostHog**](https://getdesign.md/posthog/design-md) | 俏皮刺猬品牌、开发者友好暗色 UI |
| [**Sentry**](https://getdesign.md/sentry/design-md) | 暗色仪表盘、数据密集、粉紫强调 |
| [**Supabase**](https://getdesign.md/supabase/design-md) | 暗色翡翠绿主题、代码优先 |

### 效率 & SaaS

| 网站 | 风格特征 |
|------|----------|
| [**Linear**](https://getdesign.md/linear.app/design-md) | 超级极简、精准、紫色强调 |
| [**Notion**](https://getdesign.md/notion/design-md) | 温暖极简、衬线标题、柔和表面 |
| [**Resend**](https://getdesign.md/resend/design-md) | 极简暗色主题、等宽字体点缀 |
| [**Stripe**](https://getdesign.md/stripe/design-md) | 标志性紫色渐变、轻量优雅 |

### 设计 & 创意工具

| 网站 | 风格特征 |
|------|----------|
| [**Figma**](https://getdesign.md/figma/design-md) | 活力多彩、专业而有趣 |
| [**Framer**](https://getdesign.md/framer/design-md) | 大胆黑蓝、动画优先 |
| [**Miro**](https://getdesign.md/miro/design-md) | 亮黄强调、无限画布美学 |

### 金融科技 & 加密货币

| 网站 | 风格特征 |
|------|----------|
| [**Binance**](https://getdesign.md/binance/design-md) | 币安黄 + 单色、交易紧迫感 |
| [**Coinbase**](https://getdesign.md/coinbase/design-md) | 干净蓝色、机构级信任感 |
| [**Stripe**](https://getdesign.md/stripe/design-md) | 紫色渐变、极致优雅 |

### 电商 & 零售

| 网站 | 风格特征 |
|------|----------|
| [**Airbnb**](https://getdesign.md/airbnb/design-md) | 温暖珊瑚色、图片驱动、圆润 UI |
| [**Nike**](https://getdesign.md/nike/design-md) | 单色 UI、巨型大写 Futura、全出血图片 |
| [**Shopify**](https://getdesign.md/shopify/design-md) | 暗色优先电影感、霓虹绿强调 |
| [**Starbucks**](https://getdesign.md/starbucks/design-md) | 四级大地绿系统、暖奶油画布 |

### 媒体 & 消费科技

| 网站 | 风格特征 |
|------|----------|
| [**Apple**](https://getdesign.md/apple/design-md) | 极致留白、SF Pro 字体、电影级图片 |
| [**NVIDIA**](https://getdesign.md/nvidia/design-md) | 绿黑能量、技术力量美学 |
| [**Spotify**](https://getdesign.md/spotify/design-md) | 暗色活力绿、大胆排版、专辑封面驱动 |
| [**Tesla**](https://getdesign.md/tesla/design-md) | 极致减法、电影级全视口摄影 |
| [**SpaceX**](https://getdesign.md/spacex/design-md) | 冷峻黑白、全出血图片、未来感 |

### 汽车

| 网站 | 风格特征 |
|------|----------|
| [**BMW**](https://getdesign.md/bmw/design-md) | 暗色高级表面、精密德式工程美学 |
| [**Ferrari**](https://getdesign.md/ferrari/design-md) | 明暗对比黑白编辑风、法拉利红极致稀疏 |
| [**Lamborghini**](https://getdesign.md/lamborghini/design-md) | 纯黑教堂、金色强调、定制 Neo-Grotesk 字体 |
| [**Tesla**](https://getdesign.md/tesla/design-md) | 极致减法、电影级全视口摄影 |

---

## 5. Claude Code 如何使用 DESIGN.md

### 5.1 三步上手

```
Step 1: 选择一份 DESIGN.md → 复制到项目根目录
Step 2: 告诉 Claude Code "按照 DESIGN.md 的风格构建"
Step 3: Claude Code 自动读取并生成风格一致的 UI
```

### 5.2 Step 1：获取 DESIGN.md

**方式 A：从 GitHub 仓库下载**

```bash
# 克隆仓库
git clone https://github.com/VoltAgent/awesome-design-md.git

# 复制你想要的 DESIGN.md 到项目根目录
# 例如：使用 Linear 风格
cp awesome-design-md/design-md/linear.app/DESIGN.md ./my-project/DESIGN.md

# 或者使用 Vercel 风格
cp awesome-design-md/design-md/vercel/DESIGN.md ./my-project/DESIGN.md
```

**方式 B：从 getdesign.md 在线复制**

访问 [getdesign.md](https://getdesign.md/)，搜索目标网站，直接复制 DESIGN.md 内容。

**方式 C：在 Claude Code 中一键获取**

```bash
# 让 Claude Code 直接下载指定风格的 DESIGN.md
claude "从 https://github.com/VoltAgent/awesome-design-md 下载 linear.app 的 DESIGN.md，保存到项目根目录"
```

### 5.3 Step 2：项目结构

将 DESIGN.md 放入项目根目录，与其他 Agent 配置文件并列：

```
my-project/
├── CLAUDE.md          ← Claude Code 的编码规范
├── DESIGN.md          ← ✅ 设计系统规范（新增）
├── AGENTS.md          ← 其他 Agent 配置（可选）
├── README.md
├── package.json
└── src/
```

### 5.4 Step 3：让 Claude Code 读取并使用

**基本用法**：

```bash
claude "阅读 DESIGN.md，按照其中定义的设计系统，构建一个项目仪表盘页面"
```

**结合 CLAUDE.md 使用**：

在 `CLAUDE.md` 中添加引用：

```markdown
# CLAUDE.md

## 设计规范
- 本项目使用 DESIGN.md 中定义的设计系统
- 所有 UI 组件必须遵循 DESIGN.md 的色彩、排版、间距规范
- 生成页面时，优先参考 DESIGN.md 的 Component Stylings 章节
```

**指定特定风格的提示词**：

```bash
# Linear 风格：极简紫色
claude "严格按照 DESIGN.md（Linear 风格）的设计系统，创建一个任务管理页面"

# Apple 风格：极致留白
claude "按照 DESIGN.md 中定义的 Apple 设计语言，构建产品展示页"

# Stripe 风格：紫色渐变
claude "参考 DESIGN.md 的 Stripe 风格规范，设计一个支付设置页面"
```

---

## 6. 实战：使用 DESIGN.md 构建项目

### 6.1 实战 1：构建 Linear 风格的项目管理工具

**Step 1：初始化项目并添加 DESIGN.md**

```bash
mkdir linear-clone && cd linear-clone
npm create next-app@latest . -- --typescript --tailwind --app

# 下载 Linear 的 DESIGN.md
curl -sL "https://getdesign.md/linear.app/design-md" -o DESIGN.md
```

**Step 2：创建 CLAUDE.md**

```bash
claude "创建 CLAUDE.md，内容包含：
1. 本项目是 Linear 风格的项目管理工具
2. 严格遵循 DESIGN.md 中的设计系统
3. 技术栈：Next.js 14 + TypeScript + Tailwind CSS
4. 使用 shadcn/ui 组件库
5. 所有颜色、字体、间距必须来自 DESIGN.md 中定义的 Token"
```

**Step 3：让 Claude Code 构建页面**

```bash
claude "阅读 DESIGN.md，然后构建以下页面：

1. 侧边栏导航：
   - 使用 DESIGN.md 中定义的颜色和间距
   - 包含：Workspace、Projects、My Issues、Views 菜单项

2. 主内容区：
   - Issue 列表视图，包含状态图标、标题、标签、优先级、负责人
   - 符合 DESIGN.md 中 Component Stylings 的按钮和输入框样式

3. 顶栏：
   - 面包屑导航 + 搜索栏 + 过滤器 + 用户头像
   - 使用 DESIGN.md 定义的 Depth & Elevation 阴影

请严格按照 DESIGN.md 的排版规则和响应式行为实现"
```

**Step 4：迭代优化**

```bash
claude "检查生成的页面，确认：
1. 所有颜色值是否与 DESIGN.md Color Palette 一致
2. 字体层级是否符合 Typography Rules
3. 间距是否遵循 Layout Principles 的间距系统
4. 暗色模式是否正确应用
如有偏差，请修正"
```

### 6.2 实战 2：构建 Stripe 风格的支付页面

```bash
# 下载 Stripe 的 DESIGN.md
claude "从 awesome-design-md 仓库获取 Stripe 的 DESIGN.md，保存到项目根目录"

# 构建支付页面
claude "按照 DESIGN.md 中 Stripe 的设计系统，构建一个支付设置页面：
1. 紫色渐变 Hero 区域
2. 支付方式卡片（信用卡、Apple Pay、Google Pay）
3. 交易历史表格
4. 发票列表
5. 所有组件严格使用 DESIGN.md 中定义的 Design Token"
```

### 6.3 实战 3：构建 Apple 风格的产品展示页

```bash
# 获取 Apple 的 DESIGN.md
claude "获取 Apple 的 DESIGN.md 并保存到项目根目录"

# 构建产品页
claude "按照 DESIGN.md 的 Apple 设计语言，构建 iPhone 产品页：
1. 全视口电影级产品图片
2. SF Pro 字体层级（遵循 DESIGN.md Typography Rules）
3. 极致留白（遵循 Layout Principles 的间距哲学）
4. 滚动触发动画
5. 规格参数对比表格
6. CTA 购买按钮（遵循 Component Stylings 的按钮规范）"
```

---

## 7. 高级用法与最佳实践

### 7.1 多风格混搭

一个项目中可以放置多个 DESIGN.md，通过命名区分：

```
my-project/
├── DESIGN.md                  ← 主设计系统
├── DESIGN-admin.md            ← 管理后台风格（如 Linear）
├── DESIGN-marketing.md        ← 营销页面风格（如 Apple）
├── DESIGN-dashboard.md        ← 仪表盘风格（如 Vercel）
└── ...
```

```bash
# 指定使用哪个设计系统
claude "使用 DESIGN-admin.md（Linear 风格）构建后台管理页面"
claude "使用 DESIGN-marketing.md（Apple 风格）构建着陆页"
```

### 7.2 在 CLAUDE.md 中注册 Design Token

将 DESIGN.md 中的关键 Token 提取到 `CLAUDE.md`，让 Claude Code 更稳定地遵循：

```markdown
# CLAUDE.md

## Design System Source
- 主设计规范：`DESIGN.md`
- 所有 UI 生成必须以此为标准

## Quick Reference（从 DESIGN.md 提取）

### Colors
| Token | Value | Usage |
|-------|-------|-------|
| --color-bg | #0A0A0A | 页面背景 |
| --color-surface | #141414 | 卡片/面板背景 |
| --color-accent | #5E6AD2 | 主强调色（Linear 紫） |
| --color-text | #FFFFFF | 主文本 |
| --color-text-secondary | #8B8B8B | 次要文本 |

### Typography
| Level | Size | Weight | Font |
|-------|------|--------|------|
| H1 | 48px | 700 | Inter |
| H2 | 32px | 600 | Inter |
| Body | 14px | 400 | Inter |
| Mono | 13px | 400 | JetBrains Mono |

### Spacing
| Token | Value |
|--------|-------|
| xs | 4px |
| sm | 8px |
| md | 16px |
| lg | 24px |
| xl | 32px |
```

### 7.3 自动化 DESIGN.md 下载脚本

创建 `scripts/fetch-design.sh`：

```bash
#!/bin/bash
# fetch-design.sh — 从 awesome-design-md 下载 DESIGN.md

SITE="${1:?用法: ./fetch-design.sh <site-name> [output-dir]}"
OUTPUT_DIR="${2:-.}"

REPO="https://raw.githubusercontent.com/VoltAgent/awesome-design-md/main/design-md"

# 下载 DESIGN.md
echo "🎨 下载 ${SITE} 的 DESIGN.md..."
curl -sL "${REPO}/${SITE}/DESIGN.md" -o "${OUTPUT_DIR}/DESIGN.md"

if [ -s "${OUTPUT_DIR}/DESIGN.md" ]; then
  echo "✅ 已保存到: ${OUTPUT_DIR}/DESIGN.md"
  echo "📊 文件大小: $(wc -c < "${OUTPUT_DIR}/DESIGN.md" | tr -d ' ') bytes"
else
  echo "❌ 下载失败，请检查网站名称是否正确"
  echo "可用网站: linear.app, vercel, apple, stripe, spotify, notion..."
  rm -f "${OUTPUT_DIR}/DESIGN.md"
  exit 1
fi

# 可选：下载预览文件
echo "🖼️  下载预览文件..."
curl -sL "${REPO}/${SITE}/preview.html" -o "${OUTPUT_DIR}/design-preview.html"
curl -sL "${REPO}/${SITE}/preview-dark.html" -o "${OUTPUT_DIR}/design-preview-dark.html"

echo "✨ 完成！在浏览器中打开 design-preview.html 预览设计系统"
```

使用：

```bash
chmod +x scripts/fetch-design.sh

# 下载 Linear 风格
./scripts/fetch-design.sh linear.app .

# 下载 Apple 风格到指定目录
./scripts/fetch-design.sh apple ./my-apple-project

# 在 Claude Code 中使用
claude "运行 ./scripts/fetch-design.sh spotify . 下载 Spotify 风格的 DESIGN.md，然后按照该设计系统构建音乐播放器 UI"
```

### 7.4 结合 Tailwind CSS 使用

DESIGN.md 中的 Design Token 可以直接映射到 Tailwind 配置：

```bash
claude "阅读 DESIGN.md，然后：
1. 提取 Color Palette 中的所有颜色值
2. 提取 Typography Rules 中的字体和字号
3. 提取 Layout Principles 中的间距系统
4. 将这些 Token 映射到 tailwind.config.ts 的 theme 扩展中
5. 确保所有组件使用这些 Tailwind Token 而非硬编码值"
```

生成的 `tailwind.config.ts` 示例：

```typescript
import type { Config } from "tailwindcss";

const config: Config = {
  content: ["./src/**/*.{js,ts,jsx,tsx,mdx}"],
  darkMode: "class",
  theme: {
    extend: {
      colors: {
        // 从 DESIGN.md Color Palette 提取
        background: "#0A0A0A",
        surface: "#141414",
        accent: "#5E6AD2",
        "text-primary": "#FFFFFF",
        "text-secondary": "#8B8B8B",
        border: "#2B2B2B",
      },
      fontFamily: {
        sans: ["Inter", "sans-serif"],
        mono: ["JetBrains Mono", "monospace"],
      },
      spacing: {
        // 从 DESIGN.md Layout Principles 提取
        xs: "4px",
        sm: "8px",
        md: "16px",
        lg: "24px",
        xl: "32px",
        "2xl": "48px",
      },
    },
  },
  plugins: [],
};
export default config;
```

### 7.5 批量生成多风格页面

```bash
# 一次性为同一个项目生成不同风格的页面
claude "为我的 SaaS 项目构建以下页面，每个页面使用不同的 DESIGN.md：

1. 着陆页 → 使用 DESIGN-landing.md（Apple 风格）
2. 定价页 → 使用 DESIGN-pricing.md（Stripe 风格）
3. 仪表盘 → 使用 DESIGN-dashboard.md（Linear 风格）
4. 文档页 → 使用 DESIGN-docs.md（Vercel 风格）

请分别阅读对应的 DESIGN.md，确保每个页面严格遵循指定的设计系统"
```

---

## 8. 自定义 DESIGN.md

### 8.1 从零创建

如果收录的 69 份 DESIGN.md 不满足需求，可以从零创建：

```bash
claude "为我的项目创建一份 DESIGN.md，参考以下要求：

## 视觉风格
- 整体感觉：专业、现代、科技感
- 参考网站：Linear + Vercel 的混合风格
- 主色调：深蓝 #1a1b2e + 亮绿 #00d68f
- 暗色优先

## 字体
- 标题：Inter 700
- 正文：Inter 400
- 代码：JetBrains Mono

## 组件
- 按钮：圆角 8px、微阴影、hover 渐变色
- 卡片：暗色表面、1px 边框、悬浮微上移
- 输入框：暗色背景、聚焦时绿色边框

请按照 DESIGN.md 标准格式输出"
```

### 8.2 从现有网站提取

```bash
# 提供参考网站 URL，让 Claude Code 分析并生成 DESIGN.md
claude "访问 https://example.com，分析其视觉设计系统：
1. 提取颜色（主色、辅色、背景色、文本色、边框色）
2. 提取字体（标题、正文、代码）
3. 提取间距规律
4. 分析按钮、卡片、导航等组件样式
5. 按照 DESIGN.md 标准格式输出到 DESIGN.md"
```

### 8.3 混合多个 DESIGN.md

```bash
claude "读取 DESIGN-linear.md 和 DESIGN-apple.md，创建一个新的 DESIGN.md：
- 色彩系统：采用 Linear 的暗色系
- 排版规则：采用 Apple 的字号层级和留白哲学
- 组件样式：按钮用 Linear 风格，卡片用 Apple 风格
- 响应式：合并两者的断点策略
输出到 DESIGN.md"
```

---

## 9. 与 Google Stitch 的关系

### 9.1 DESIGN.md 规范的起源

DESIGN.md 格式由 **Google Stitch** 团队定义：

| 维度 | 说明 |
|------|------|
| **规范文档** | [stitch.withgoogle.com/docs/design-md/](https://stitch.withgoogle.com/docs/design-md/overview/) |
| **格式标准** | [stitch.withgoogle.com/docs/design-md/format/](https://stitch.withgoogle.com/docs/design-md/format/) |
| **核心作用** | 让 AI Agent 理解设计意图，生成一致的 UI |

### 9.2 Stitch 网页版 vs DESIGN.md + Claude Code

| 对比 | Stitch 网页版 | DESIGN.md + Claude Code |
|------|--------------|------------------------|
| **工作方式** | 在线输入 Prompt → 生成设计 | 本地 DESIGN.md → Claude Code 读取生成 |
| **设计一致性** | 依赖 Prompt 描述 | DESIGN.md 精确定义，高度一致 |
| **可复现性** | 相同 Prompt 可能不同结果 | DESIGN.md 固定规范，结果可复现 |
| **版本控制** | 无法 Git 管理 | DESIGN.md 纳入 Git，设计变更可追踪 |
| **团队协作** | 个人操作 | 团队共享同一份 DESIGN.md |
| **适用场景** | 快速原型、灵感探索 | 正式项目、团队协作、生产代码 |

### 9.3 最佳搭配：DESIGN.md + Stitch API + Claude Code

```
┌──────────────────────────────────────────────────────────────┐
│              三者协同工作流                                     │
│                                                              │
│  ① DESIGN.md（设计规范）                                      │
│     定义：颜色、字体、间距、组件样式                             │
│     ↓                                                        │
│  ② Stitch / Gemini API（UI 生成）                            │
│     输入：DESIGN.md + 页面描述                                 │
│     输出：基础 UI 代码                                        │
│     ↓                                                        │
│  ③ Claude Code（深度开发）                                    │
│     输入：Stitch 生成的基础代码 + DESIGN.md                    │
│     输出：生产级组件 + 后端 + 测试                              │
└──────────────────────────────────────────────────────────────┘
```

详细配置方法参见 → [[Google Stitch × Claude Code 完整开发指南#9. 通过 API 直接调用 Stitch 能力（Gemini API 集成）]]

---

## 10. 常见问题

### Q1：DESIGN.md 中的颜色/字体值准确吗？

> awesome-design-md 的 DESIGN.md 是从公开网站的 CSS 中提取的，代表公开可见的设计 Token。准确度较高，但可能与网站最新版本存在细微差异。建议用于风格参考，而非像素级还原。

### Q2：必须放在项目根目录吗？

> 不是必须的，但推荐放在根目录。Claude Code 会自动读取根目录的 `DESIGN.md`。如果放在其他位置，需要在提示词中指定路径：
> ```bash
> claude "阅读 ./designs/DESIGN.md，按照其中的设计规范构建页面"
> ```

### Q3：一个项目可以用多份 DESIGN.md 吗？

> 可以！建议通过命名区分用途：
> - `DESIGN.md` — 主设计系统
> - `DESIGN-admin.md` — 管理后台风格
> - `DESIGN-mobile.md` — 移动端风格
> 使用时在提示词中指定文件名即可。

### Q4：DESIGN.md 和 CLAUDE.md 的关系？

> 两者互补：
> - `CLAUDE.md` 定义**代码规范**（目录结构、命名、技术栈）
> - `DESIGN.md` 定义**视觉规范**（颜色、字体、布局、组件样式）
>
> 建议在 `CLAUDE.md` 中引用 `DESIGN.md`：
> ```markdown
> ## 设计规范
> - 本项目 UI 遵循 DESIGN.md 中定义的设计系统
> - 所有颜色使用 DESIGN.md Color Palette 中的 Token
> ```

### Q5：可以请求收录新的网站吗？

> 可以！访问 [getdesign.md/request](https://getdesign.md/request) 提交请求。也支持私有请求（仅交付给你）。

### Q6：如何验证 Claude Code 是否正确遵循了 DESIGN.md？

> ```bash
> claude "审查 src/ 目录下的所有组件，对比 DESIGN.md：
> 1. 列出所有使用了硬编码颜色（非 DESIGN.md Token）的地方
> 2. 检查字体是否符合 Typography Rules
> 3. 检查间距是否符合 Layout Principles
> 4. 检查响应式断点是否正确
> 输出偏差报告并修复"
> ```

---

## 总结

### 🎯 核心价值

| 传统 AI 生成 UI | DESIGN.md 驱动的 AI 生成 |
|----------------|------------------------|
| "做一个好看的登录页" → 随机风格 | "按 DESIGN.md 做" → 精准风格 |
| 每次 Prompt 描述不同 → 风格不一致 | DESIGN.md 固定规范 → 风格始终一致 |
| 无法在团队间共享设计意图 | DESIGN.md = 团队统一的设计语言 |
| 无法版本控制 | Git 追踪每一次设计变更 |

### 📊 工作流对比

| 步骤 | 无 DESIGN.md | 有 DESIGN.md |
|------|-------------|-------------|
| 设计定义 | 口头描述 / Figma | DESIGN.md 文件 |
| AI 理解度 | 模糊、不稳定 | 精确、可复现 |
| 风格一致性 | ⭐⭐ | ⭐⭐⭐⭐⭐ |
| 团队协作 | 困难 | 共享同一文件 |
| 版本追踪 | 无 | Git 管理 |

> 💡 **最佳实践**：将 `DESIGN.md` 纳入项目根目录，与 `CLAUDE.md` 并列。前者管"看起来怎样"，后者管"怎样构建"。两者结合，让 Claude Code 生成的 UI 既精准又一致。

---

## 参考资源

- [awesome-design-md GitHub 仓库](https://github.com/VoltAgent/awesome-design-md)
- [getdesign.md — 在线浏览 DESIGN.md](https://getdesign.md/)
- [Google Stitch DESIGN.md 格式规范](https://stitch.withgoogle.com/docs/design-md/overview/)
- [VoltAgent 官方网站](https://voltagent.dev/)
- [[Google Stitch × Claude Code 完整开发指南]] — Stitch + Claude Code 完整工作流
- [[Claude Code安装指南-GLM模型]] — Claude Code 安装
- [[Claude Code 必装 Skills 推荐指南]] — 推荐 Skills
