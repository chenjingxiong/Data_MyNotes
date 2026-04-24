# Google Stitch × Claude Code 完整开发指南

> **副标题**：从设计到全栈应用 —— AI 驱动的完整开发工作流
> **创建日期**：2026-04-23
> **标签**：#AI #GoogleStitch #ClaudeCode #全栈开发 #设计到代码

---

## 目录

- [[#1. Google Stitch 简介]]
- [[#2. 核心功能与能力]]
- [[#3. 为什么 Stitch + Claude Code 是完美组合]]
- [[#4. 完整工作流概览]]
- [[#5. 实战示例 1 —— Web 应用：在线项目管理工具]]
- [[#6. 实战示例 2 —— 移动 App：健身追踪应用]]
- [[#7. 实战示例 3 —— 后端 + 前端全栈：电商 API 平台]]
- [[#8. 高级技巧与最佳实践]]
- [[#9. 通过 API 直接调用 Stitch 能力（Gemini API 集成）]]
- [[#10. 常见问题与故障排除]]

---

## 1. Google Stitch 简介

### 1.1 什么是 Google Stitch？

**Google Stitch** 是 Google 在 **Google I/O 2025**（2025 年 5 月）上发布的实验性 AI 设计工具，旨在弥合 **视觉设计与前端代码** 之间的鸿沟。

> **一句话概括**：用自然语言或图片描述你想要的 UI，Stitch 自动生成设计稿 + 可用代码。

| 维度 | 说明 |
|------|------|
| **开发者** | Google Labs |
| **发布时间** | 2025 年 5 月（Google I/O 2025） |
| **底层 AI** | Google Gemini 多模态模型 |
| **访问方式** | [stitch.withgoogle.com](https://stitch.withgoogle.com)（需 Google 账号） |
| **当前状态** | 实验性 / Preview 阶段 |
| **设计系统** | 基于 Material Design 3 |

### 1.2 设计理念

Stitch 的核心理念是 **"Design → Code in Seconds"**：

```
📝 自然语言描述  →  🎨 AI 生成 UI 设计  →  📦 导出生产级代码
🖼️ 图片/线框图  ↗️         ↓
                         ✏️ 迭代优化
```

传统开发流程：
```
产品需求 → UX 设计师出图 → UI 设计稿评审 → 前端手写代码 → 反复调整
（周期：数天 ~ 数周）
```

Stitch 流程：
```
自然语言描述 → AI 实时生成设计 + 代码 → 即时迭代 → 导出
（周期：分钟级）
```

### 1.3 适用人群

| 角色            | 用途                            |
| ------------- | ----------------------------- |
| **设计师**       | 快速生成设计原型，探索视觉方案               |
| **前端开发者**     | 获取高质量代码起点，避免从零搭建 UI           |
| **全栈开发者**     | 与 Claude Code 配合，实现设计到后端的完整链路 |
| **产品经理**      | 可视化产品概念，快速验证想法                |
| **创业者/独立开发者** | 一人完成设计 + 开发全流程                |

---

## 2. 核心功能与能力

### 2.1 Prompt-to-UI 生成

通过自然语言描述生成完整的 UI 界面：

```
示例 Prompt：
"A dashboard for a project management tool with:
- Sidebar navigation with icons
- Project cards with progress bars
- Team member avatars
- Dark theme with blue accents
- Responsive layout"
```

Stitch 会在数秒内生成：
- ✅ 完整的可视化 UI 预览
- ✅ 对应的前端代码（HTML/CSS/JS 或框架代码）
- ✅ 响应式布局方案

### 2.2 图片/截图转代码

上传参考图片，Stitch 自动解析并生成对应代码：

- 📐 支持上传线框图（Wireframe）
- 🖼️ 支持截图（Screenshot）
- 🎨 支持设计稿（Figma 导出图等）
- 📱 支持移动端 UI 截图

### 2.3 多框架代码导出

Stitch 支持将设计导出为多种前端框架代码：

| 框架 | 输出格式 | 适用场景 |
|------|----------|----------|
| **HTML/CSS/JS** | 原生 Web | 简单页面、原型演示 |
| **Angular** | Angular 组件 | Google 生态企业项目 |
| **React** | React 组件 | 主流 Web 应用 |
| **Flutter/Dart** | Flutter Widget | 跨平台移动应用 |

### 2.4 Material Design 3 原生支持

- 默认遵循 Material Design 3 (Material You) 设计规范
- 支持动态色彩（Dynamic Color）
- 遵循 Google 无障碍设计标准
- 内置 Material 组件库

### 2.5 迭代式优化

```
[初始 Prompt] → 生成 V1 → [反馈 Prompt] → 生成 V2 → ... → 满意 → 导出

示例迭代：
Prompt 1: "A login page"
→ 生成基础登录页

Prompt 2: "Add social login buttons and make it more modern"
→ 添加社交登录按钮

Prompt 3: "Change to a gradient background with glass morphism effect"
→ 应用毛玻璃渐变效果
```

---

## 3. 为什么 Stitch + Claude Code 是完美组合

### 3.1 能力互补

| 层面 | Google Stitch | Claude Code |
|------|---------------|-------------|
| **UI/UX 设计** | ⭐⭐⭐⭐⭐ AI 生成设计 | ⭐⭐⭐ 基础 UI 生成 |
| **前端代码** | ⭐⭐⭐⭐ 生成 UI 代码 | ⭐⭐⭐⭐⭐ 深度代码开发 |
| **后端逻辑** | ❌ 不涉及 | ⭐⭐⭐⭐⭐ 全栈能力 |
| **数据库设计** | ❌ 不涉及 | ⭐⭐⭐⭐⭐ 完整支持 |
| **API 开发** | ❌ 不涉及 | ⭐⭐⭐⭐⭐ REST/GraphQL |
| **测试编写** | ❌ 不涉及 | ⭐⭐⭐⭐⭐ 全类型测试 |
| **DevOps/部署** | ❌ 不涉及 | ⭐⭐⭐⭐ CI/CD 配置 |
| **迭代优化** | ⭐⭐⭐⭐⭐ 设计迭代 | ⭐⭐⭐⭐⭐ 代码迭代 |

### 3.2 完整的 AI 开发流水线

```
┌──────────────────────────────────────────────────────────────────┐
│                    AI 全栈开发流水线                               │
│                                                                  │
│  ┌─────────────┐     ┌──────────────┐     ┌──────────────────┐  │
│  │   Google     │     │  Claude Code  │     │   Claude Code    │  │
│  │   Stitch     │────▶│  前端深化     │────▶│   后端开发       │  │
│  │              │     │  组件优化     │     │   数据库设计     │  │
│  │  ① UI 设计   │     │  ② 前端代码   │     │   ③ API 接口     │  │
│  │  ① 基础代码   │     │  ② 状态管理   │     │   ④ 业务逻辑     │  │
│  └─────────────┘     └──────────────┘     │   ⑤ 测试部署     │  │
│                                            └──────────────────┘  │
└──────────────────────────────────────────────────────────────────┘
```

### 3.3 具体协作模式

**模式 A：Stitch 先行（推荐）**
```
Stitch 生成 UI 设计和基础代码
   ↓
复制代码到 Claude Code 工作区
   ↓
Claude Code 深度优化前端代码
   ↓
Claude Code 开发后端 API + 数据库
   ↓
Claude Code 编写测试 + 部署配置
```

**模式 B：Claude Code 驱动**
```
Claude Code 搭建项目框架和后端
   ↓
Stitch 生成关键页面的 UI 代码
   ↓
Claude Code 集成前后端
   ↓
Claude Code 完成测试和优化
```

**模式 C：并行协作**
```
Stitch：并行生成多个页面的 UI 设计     Claude Code：同时开发后端 API
   ↓                                       ↓
   └─────────────── 汇合 ───────────────────┘
                       ↓
              Claude Code 集成联调
```

---

## 4. 完整工作流概览

### 4.1 环境准备

```bash
# 1. 确保已安装 Claude Code
# 参见：[[Claude Code安装指南-GLM模型]]

# 2. 初始化项目
mkdir my-fullstack-app && cd my-fullstack-app
claude init

# 3. 准备 Stitch 输出目录
mkdir -p stitch-exports
```

### 4.2 标准工作流（6 步法）

```
Step 1: 在 Stitch 中设计 UI
    ↓
Step 2: 从 Stitch 导出代码
    ↓
Step 3: 在 Claude Code 中创建项目结构
    ↓
Step 4: 集成 Stitch 代码到项目
    ↓
Step 5: Claude Code 开发后端逻辑
    ↓
Step 6: 测试、优化、部署
```

---

## 5. 实战示例 1 —— Web 应用：在线项目管理工具

### 5.1 需求描述

构建一个类似 Trello 的项目管理 Web 应用：
- 仪表盘视图：项目卡片 + 进度统计
- 看板视图：拖拽式任务管理
- 团队协作：成员头像 + 在线状态
- 响应式设计：桌面 + 平板 + 手机

### 5.2 Step 1：在 Stitch 中生成 UI

在 [stitch.withgoogle.com](https://stitch.withgoogle.com) 输入：

```
Prompt:
"A project management dashboard with:
- Left sidebar: navigation with Dashboard, Projects, Team, Settings icons
- Main area: grid of project cards showing name, progress bar, due date, team avatars
- Top bar: search, notifications bell, user profile
- Stats row: total projects, active tasks, completed this week, team members
- Color scheme: dark sidebar (#1a1a2e), light main area, blue accent (#4361ee)
- Modern, clean design with subtle shadows and rounded corners"
```

**Stitch 生成结果**：
- 🎨 完整仪表盘 UI 设计稿
- 📦 HTML/CSS/Angular 代码

### 5.3 Step 2：导出并集成到 Claude Code

从 Stitch 复制导出的代码，然后让 Claude Code 创建完整项目：

```bash
# 在 Claude Code 中执行
claude "基于 Stitch 导出的 UI 代码，创建一个完整的 React + TypeScript 项目：
1. 使用 Next.js 14 + App Router
2. 集成 Tailwind CSS 样式
3. 使用 shadcn/ui 组件库
4. 添加响应式布局支持

以下是 Stitch 生成的 UI 代码：
<paste stitch code here>"
```

### 5.4 Step 3：Claude Code 开发完整前端

**提示 Claude Code**：

```
请完善这个项目管理工具的前端，包括：

1. 页面路由结构：
   - / → Dashboard 仪表盘
   - /projects → 项目列表
   - /projects/[id] → 看板视图
   - /team → 团队管理
   - /settings → 设置

2. 核心组件：
   - ProjectCard：项目卡片组件
   - KanbanBoard：可拖拽看板
   - TaskModal：任务详情弹窗
   - Sidebar：侧边导航栏
   - StatsRow：统计数据行

3. 状态管理：使用 Zustand
4. 数据模拟：使用 MSW（Mock Service Worker）
```

### 5.5 Step 4：Claude Code 开发后端

```
请为这个项目管理工具创建后端 API：

1. 技术栈：Node.js + Express + TypeScript
2. 数据库：PostgreSQL + Prisma ORM
3. 认证：JWT + bcrypt

4. API 端点：
   - POST   /api/auth/login
   - POST   /api/auth/register
   - GET    /api/projects
   - POST   /api/projects
   - GET    /api/projects/:id/tasks
   - POST   /api/projects/:id/tasks
   - PATCH  /api/tasks/:id
   - DELETE /api/tasks/:id
   - GET    /api/team
   - GET    /api/stats

5. Prisma Schema 设计：
   - User（用户）
   - Project（项目）
   - Task（任务，含 status: TODO/IN_PROGRESS/DONE）
   - Comment（评论）
   - ProjectMember（项目成员关联表）
```

### 5.6 Step 5：测试与部署

```
请为这个全栈应用配置：

1. 测试：
   - Vitest 单元测试（后端 API）
   - Playwright E2E 测试（前端流程）
   - 覆盖率目标 80%+

2. Docker 配置：
   - 前端 Dockerfile
   - 后端 Dockerfile
   - docker-compose.yml（含 PostgreSQL）

3. CI/CD：
   - GitHub Actions workflow
   - 自动测试 + 构建推送
```

### 5.7 最终项目结构

```
project-manager/
├── frontend/                    # Next.js 前端
│   ├── app/                     # App Router 页面
│   │   ├── page.tsx             # Dashboard（Stitch 设计）
│   │   ├── projects/
│   │   │   ├── page.tsx         # 项目列表
│   │   │   └── [id]/page.tsx    # 看板视图
│   │   ├── team/page.tsx
│   │   └── settings/page.tsx
│   ├── components/
│   │   ├── ui/                  # shadcn/ui 基础组件
│   │   ├── Sidebar.tsx          # 基于 Stitch 设计
│   │   ├── ProjectCard.tsx      # 基于 Stitch 设计
│   │   ├── KanbanBoard.tsx
│   │   ├── StatsRow.tsx         # 基于 Stitch 设计
│   │   └── TaskModal.tsx
│   ├── store/                   # Zustand 状态
│   └── lib/                     # 工具函数
├── backend/                     # Express 后端
│   ├── src/
│   │   ├── routes/              # API 路由
│   │   ├── middleware/          # 认证中间件
│   │   ├── services/            # 业务逻辑
│   │   └── index.ts
│   ├── prisma/
│   │   └── schema.prisma       # 数据模型
│   └── tests/                   # API 测试
├── docker-compose.yml
└── .github/workflows/ci.yml
```

---

## 6. 实战示例 2 —— 移动 App：健身追踪应用

### 6.1 需求描述

构建一个跨平台健身追踪 App（Flutter）：
- 首页：今日运动数据环形图 + 快速开始按钮
- 运动记录：GPS 轨迹 + 实时心率 + 卡路里
- 社交功能：好友排行榜 + 动态 Feed
- 个人中心：运动历史 + 成就徽章

### 6.2 Step 1：在 Stitch 中设计移动端 UI

在 Stitch 中依次生成各页面的设计：

**首页 Prompt**：
```
"A fitness app home screen with:
- Top: greeting with user name and profile picture
- Circular progress ring showing daily activity (steps, calories, minutes)
- Three stat cards: steps count, calories burned, active minutes
- Quick start buttons: Run, Cycle, Yoga, Strength
- Bottom navigation bar with Home, Activity, Social, Profile tabs
- Gradient blue-to-purple background for the progress ring area
- Clean white card design with subtle shadows
- Material Design 3 style"
```

**运动记录页 Prompt**：
```
"A workout tracking screen for a fitness app:
- Map area showing GPS route with gradient path
- Real-time stats overlay: distance, pace, heart rate, calories
- Start/Pause/Stop button cluster
- Elevation profile chart at bottom
- Elapsed time display prominently at top
- Dark theme with neon accent colors (green for active metrics)"
```

**社交页 Prompt**：
```
"A social feed screen for a fitness app:
- Weekly leaderboard showing top friends with avatars and step counts
- Activity feed cards showing friend workouts
- Like and comment interactions
- Challenge banners
- Material Design 3 cards with rounded corners"
```

### 6.3 Step 2：导出 Flutter 代码

Stitch 可以导出 Flutter/Dart Widget 代码。将导出的代码交给 Claude Code：

```bash
claude "基于 Stitch 导出的 Flutter UI 代码，创建一个完整的 Flutter 健身追踪 App：

1. 项目结构采用 Clean Architecture：
   - lib/domain/（实体 + 用例）
   - lib/data/（数据源 + 仓库实现）
   - lib/presentation/（Bloc + 页面 + 组件）

2. 状态管理：flutter_bloc

3. 核心功能模块：
   - 运动追踪（GPS 定位 + 传感器）
   - 数据可视化（fl_chart 图表）
   - 社交功能（用户系统 + 排行榜）
   - 本地存储（Hive/Isar）

以下是 Stitch 生成的首页 Widget 代码：
<paste stitch flutter code>"
```

### 6.4 Step 3：Claude Code 开发完整 App

让 Claude Code 逐步完善：

```dart
// Claude Code 生成的项目结构
fitness_app/
├── lib/
│   ├── main.dart
│   ├── app.dart
│   ├── domain/
│   │   ├── entities/
│   │   │   ├── workout.dart
│   │   │   ├── user.dart
│   │   │   └── achievement.dart
│   │   └── usecases/
│   │       ├── start_workout.dart
│   │       ├── get_leaderboard.dart
│   │       └── save_workout.dart
│   ├── data/
│   │   ├── models/
│   │   ├── repositories/
│   │   ├── datasources/
│   │   │   ├── local/      # Hive 本地存储
│   │   │   └── remote/     # API 数据源
│   │   └── services/
│   │       ├── gps_service.dart
│   │       └── health_service.dart
│   ├── presentation/
│   │   ├── bloc/
│   │   │   ├── workout/
│   │   │   ├── social/
│   │   │   └── profile/
│   │   ├── pages/
│   │   │   ├── home_page.dart          # ← Stitch 设计转写
│   │   │   ├── workout_page.dart       # ← Stitch 设计转写
│   │   │   ├── social_page.dart        # ← Stitch 设计转写
│   │   │   └── profile_page.dart
│   │   └── widgets/
│   │       ├── activity_ring.dart       # 环形进度图
│   │       ├── stat_card.dart
│   │       ├── workout_map.dart         # GPS 地图
│   │       └── leaderboard_tile.dart
│   └── core/
│       ├── theme/
│       │   └── app_theme.dart          # 基于 Stitch 设计的配色
│       ├── constants/
│       └── utils/
├── test/
│   ├── domain/
│   ├── data/
│   └── presentation/
├── pubspec.yaml
└── analysis_options.yaml
```

### 6.5 Step 4：后端 API（Supabase + Claude Code）

```
请为这个健身 App 创建 Supabase 后端：

1. 数据库表设计：
   - users：用户信息
   - workouts：运动记录（GPS 轨迹 JSONB）
   - achievements：成就系统
   - friendships：好友关系
   - challenges：挑战赛

2. Edge Functions：
   - 计算排行榜
   - 处理运动数据聚合
   - 推送通知触发

3. Row Level Security 策略
```

### 6.6 Stitch → Claude Code 的 UI 转化示例

Stitch 导出的原始 Widget 可能是这样的简化版：

```dart
// Stitch 导出（简化版）
class HomePage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Column(
        children: [
          GreetingSection(),
          ActivityRing(),  // Stitch 生成的静态环形图
          StatCardsRow(),
          QuickStartButtons(),
        ],
      ),
    );
  }
}
```

Claude Code 优化后的完整版：

```dart
// Claude Code 深化后
class HomePage extends StatelessWidget {
  const HomePage({super.key});

  @override
  Widget build(BuildContext context) {
    return BlocBuilder<HomeBloc, HomeState>(
      builder: (context, state) {
        return state.when(
          initial: () => const Center(child: CircularProgressIndicator()),
          loading: () => const HomeShimmerLoading(),
          loaded: (data) => RefreshIndicator(
            onRefresh: () => context.read<HomeBloc>().loadData(),
            child: CustomScrollView(
              slivers: [
                SliverToBoxAdapter(child: GreetingSection(user: data.user)),
                SliverToBoxAdapter(
                  child: AnimatedActivityRing(       // 添加动画
                    progress: data.dailyProgress,
                    steps: data.steps,
                    calories: data.calories,
                    activeMinutes: data.activeMinutes,
                  ),
                ),
                SliverToBoxAdapter(
                  child: StatCardsRow(
                    steps: data.steps,
                    calories: data.calories,
                    minutes: data.activeMinutes,
                  ),
                ),
                const SliverToBoxAdapter(
                  child: QuickStartButtons(),        // 带 Hero 动画的按钮
                ),
                SliverList(
                  delegate: SliverChildBuilderDelegate(
                    (ctx, i) => RecentWorkoutCard(
                      workout: data.recentWorkouts[i],
                      onTap: () => context.go('/workout/${data.recentWorkouts[i].id}'),
                    ),
                    childCount: data.recentWorkouts.length,
                  ),
                ),
              ],
            ),
          ),
          error: (msg) => ErrorWidget(message: msg),
        );
      },
    );
  }
}
```

**关键改进点**：
| 维度 | Stitch 输出 | Claude Code 优化 |
|------|-------------|------------------|
| 状态管理 | 无状态 Widget | BlocBuilder 响应式 |
| 加载状态 | 无 | Shimmer 骨架屏 |
| 错误处理 | 无 | ErrorWidget |
| 动画 | 静态 | AnimatedActivityRing |
| 下拉刷新 | 无 | RefreshIndicator |
| 路由导航 | 无 | GoRouter 集成 |
| 列表性能 | Column | SliverList 懒加载 |

---

## 7. 实战示例 3 —— 后端 + 前端全栈：电商 API 平台

### 7.1 需求描述

构建一个完整的电商 Web 平台：
- 商品展示：分类浏览 + 搜索 + 筛选
- 购物车：增删改查 + 数量管理
- 结账流程：地址 + 支付 + 订单确认
- 用户系统：注册/登录 + 订单历史 + 收藏夹
- 管理后台：商品管理 + 订单管理 + 数据统计

### 7.2 Step 1：Stitch 批量生成 UI

为电商平台的每个核心页面分别生成 Stitch 设计：

| 页面 | Stitch Prompt 要点 |
|------|-------------------|
| **商品列表** | "E-commerce product grid with filters sidebar, search bar, sort dropdown, infinite scroll" |
| **商品详情** | "Product detail page with image gallery, size selector, add to cart button, reviews section" |
| **购物车** | "Shopping cart with quantity controls, price summary, coupon code input, checkout button" |
| **结账流程** | "Multi-step checkout: shipping address form → payment method → order review → confirmation" |
| **用户中心** | "User profile page with order history tabs, saved addresses, wishlist grid" |
| **管理后台** | "Admin dashboard with sales chart, order table with status filters, inventory alerts" |

### 7.3 Step 2：Claude Code 搭建全栈项目

```bash
# 在项目根目录启动 Claude Code
claude "创建一个全栈电商应用：

## 技术栈决策
- 前端：Next.js 14 (App Router) + Tailwind CSS + shadcn/ui
- 后端：Next.js API Routes (Route Handlers)
- 数据库：PostgreSQL + Prisma
- 认证：NextAuth.js
- 支付：Stripe 集成
- 图片：Uploadthing

## 我已有 Stitch 生成的 UI 设计代码，请先创建项目骨架。

然后我将逐页提供 Stitch 代码，你需要：
1. 将 Stitch 的 HTML/CSS 转化为 React/TSX 组件
2. 添加交互逻辑和状态管理
3. 连接 API 端点
4. 实现后端业务逻辑"
```

### 7.4 Step 3：数据库设计

```prisma
// prisma/schema.prisma — Claude Code 生成

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id            String    @id @default(cuid())
  name          String
  email         String    @unique
  passwordHash  String
  role          Role      @default(CUSTOMER)
  addresses     Address[]
  orders        Order[]
  reviews       Review[]
  wishlist      WishlistItem[]
  cart          CartItem[]
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt
}

enum Role {
  CUSTOMER
  ADMIN
}

model Product {
  id          String       @id @default(cuid())
  name        String
  slug        String       @unique
  description String
  price       Float
  comparePrice Float?
  images      String[]     // URL array
  category    Category     @relation(fields: [categoryId], references: [id])
  categoryId  String
  variants    ProductVariant[]
  reviews     Review[]
  orderItems  OrderItem[]
  wishlistedBy WishlistItem[]
  createdAt   DateTime     @default(now())
  updatedAt   DateTime     @updatedAt
}

model ProductVariant {
  id        String  @id @default(cuid())
  sku       String  @unique
  size      String?
  color     String?
  stock     Int     @default(0)
  product   Product @relation(fields: [productId], references: [id])
  productId String
  cartItems CartItem[]
  orderItems OrderItem[]
}

model Category {
  id       String    @id @default(cuid())
  name     String
  slug     String    @unique
  parent   Category? @relation("CategoryTree", fields: [parentId], references: [id])
  parentId String?
  children Category[] @relation("CategoryTree")
  products Product[]
}

model CartItem {
  id        String         @id @default(cuid())
  quantity  Int            @default(1)
  user      User           @relation(fields: [userId], references: [id])
  userId    String
  variant   ProductVariant @relation(fields: [variantId], references: [id])
  variantId String
}

model Order {
  id            String      @id @default(cuid())
  orderNumber   String      @unique
  status        OrderStatus @default(PENDING)
  total         Float
  shippingAddr  Json
  user          User        @relation(fields: [userId], references: [id])
  userId        String
  items         OrderItem[]
  payment       Payment?
  createdAt     DateTime    @default(now())
  updatedAt     DateTime    @updatedAt
}

enum OrderStatus {
  PENDING
  CONFIRMED
  SHIPPED
  DELIVERED
  CANCELLED
  REFUNDED
}

model OrderItem {
  id        String         @id @default(cuid())
  quantity  Int
  price     Float
  order     Order          @relation(fields: [orderId], references: [id])
  orderId   String
  variant   ProductVariant @relation(fields: [variantId], references: [id])
  variantId String
  product   Product        @relation(fields: [productId], references: [id])
  productId String
}

model Payment {
  id              String   @id @default(cuid())
  stripePaymentId String   @unique
  amount          Float
  status          String
  order           Order    @relation(fields: [orderId], references: [id])
  orderId         String   @unique
}

model Review {
  id        String   @id @default(cuid())
  rating    Int
  comment   String
  user      User     @relation(fields: [userId], references: [id])
  userId    String
  product   Product  @relation(fields: [productId], references: [id])
  productId String
  createdAt DateTime @default(now())
}

model WishlistItem {
  id        String  @id @default(cuid())
  user      User    @relation(fields: [userId], references: [id])
  userId    String
  product   Product @relation(fields: [productId], references: [id])
  productId String

  @@unique([userId, productId])
}

model Address {
  id       String @id @default(cuid())
  label    String
  street   String
  city     String
  state    String
  zipCode  String
  country  String
  isDefault Boolean @default(false)
  user     User   @relation(fields: [userId], references: [id])
  userId   String
}
```

### 7.5 Step 4：API 路由实现

```typescript
// app/api/products/route.ts — Claude Code 生成

import { prisma } from '@/lib/prisma';
import { NextRequest, NextResponse } from 'next/server';

export async function GET(request: NextRequest) {
  const searchParams = request.nextUrl.searchParams;

  // 分页参数
  const page = parseInt(searchParams.get('page') || '1');
  const limit = parseInt(searchParams.get('limit') || '20');
  const skip = (page - 1) * limit;

  // 筛选参数
  const category = searchParams.get('category');
  const search = searchParams.get('search');
  const minPrice = parseFloat(searchParams.get('minPrice') || '0');
  const maxPrice = parseFloat(searchParams.get('maxPrice') || '99999');
  const sortBy = searchParams.get('sortBy') || 'createdAt';
  const sortOrder = searchParams.get('sortOrder') === 'asc' ? 'asc' : 'desc';

  // 构建查询条件
  const where = {
    AND: [
      { price: { gte: minPrice } },
      { price: { lte: maxPrice } },
      ...(category ? [{
        category: { slug: category }
      }] : []),
      ...(search ? [{
        OR: [
          { name: { contains: search, mode: 'insensitive' } },
          { description: { contains: search, mode: 'insensitive' } },
        ]
      }] : []),
    ],
  };

  const [products, total] = await Promise.all([
    prisma.product.findMany({
      where,
      skip,
      take: limit,
      orderBy: { [sortBy]: sortOrder },
      include: {
        category: { select: { name: true, slug: true } },
        variants: true,
        reviews: {
          select: { rating: true },
        },
      },
    }),
    prisma.product.count({ where }),
  ]);

  // 计算平均评分
  const productsWithRating = products.map((p) => ({
    ...p,
    avgRating: p.reviews.length > 0
      ? p.reviews.reduce((sum, r) => sum + r.rating, 0) / p.reviews.length
      : 0,
    reviewCount: p.reviews.length,
  }));

  return NextResponse.json({
    products: productsWithRating,
    pagination: {
      page,
      limit,
      total,
      totalPages: Math.ceil(total / limit),
    },
  });
}
```

### 7.6 Step 5：前端页面集成 Stitch 设计

```typescript
// app/products/page.tsx — Stitch 设计 → React 组件

import { ProductGrid } from '@/components/products/ProductGrid';
import { FilterSidebar } from '@/components/products/FilterSidebar';
import { SearchBar } from '@/components/products/SearchBar';
import { Suspense } from 'react';

// Stitch 设计中的配色和布局被转化为 Tailwind 类
export default async function ProductsPage({
  searchParams,
}: {
  searchParams: Promise<{ [key: string]: string }>;
}) {
  const params = await searchParams;

  return (
    <div className="container mx-auto px-4 py-8">
      {/* 顶部搜索栏 — 基于 Stitch 设计 */}
      <div className="mb-8">
        <h1 className="text-3xl font-bold text-gray-900 mb-4">
          探索商品
        </h1>
        <SearchBar defaultValue={params.search} />
      </div>

      <div className="flex gap-8">
        {/* 左侧筛选栏 — 基于 Stitch 设计 */}
        <aside className="w-64 shrink-0 hidden lg:block">
          <FilterSidebar
            selectedCategory={params.category}
            priceRange={[
              parseFloat(params.minPrice || '0'),
              parseFloat(params.maxPrice || '99999'),
            ]}
          />
        </aside>

        {/* 商品网格 — 基于 Stitch 设计 */}
        <main className="flex-1">
          <Suspense fallback={<ProductGridSkeleton />}>
            <ProductGrid searchParams={params} />
          </Suspense>
        </main>
      </div>
    </div>
  );
}
```

### 7.7 完整架构图

```
┌─────────────────────────────────────────────────────────────────────┐
│                      电商平台架构                                     │
│                                                                     │
│  ┌──────────────────────────────────────────────┐                  │
│  │              Next.js 前端 (SSR/SSG)           │                  │
│  │  ┌──────────┐ ┌──────────┐ ┌───────────────┐ │                  │
│  │  │ 商品列表  │ │ 购物车   │ │ 用户中心       │ │                  │
│  │  │(Stitch)  │ │(Stitch)  │ │(Stitch)       │ │                  │
│  │  └────┬─────┘ └────┬─────┘ └──────┬────────┘ │                  │
│  └───────┼─────────────┼──────────────┼──────────┘                  │
│          │             │              │                              │
│  ┌───────▼─────────────▼──────────────▼──────────┐                  │
│  │          Next.js API Route Handlers            │                  │
│  │  /api/products  /api/cart  /api/orders         │                  │
│  │  /api/auth/*    /api/payments                  │                  │
│  └───────────────────┬───────────────────────────┘                  │
│                      │                                              │
│  ┌───────────────────▼───────────────────────────┐                  │
│  │              服务层 (Services)                  │                  │
│  │  ProductService │ CartService │ OrderService   │                  │
│  │  PaymentService │ AuthService │ ReviewService  │                  │
│  └───────────────────┬───────────────────────────┘                  │
│                      │                                              │
│  ┌───────┬───────────▼──────────────┬──────────────┐               │
│  │Prisma │      PostgreSQL          │  Redis Cache │               │
│  │  ORM  │      (主数据库)           │  (会话/缓存)  │               │
│  └───────┴──────────────────────────┴──────────────┘               │
│                                                                     │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐              │
│  │   Stripe     │  │ Uploadthing  │  │   Resend     │              │
│  │   (支付)     │  │  (图片存储)   │  │  (邮件通知)   │              │
│  └──────────────┘  └──────────────┘  └──────────────┘              │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 8. 高级技巧与最佳实践

### 8.1 Stitch Prompt 优化技巧

| 技巧 | 示例 |
|------|------|
| **指定设计系统** | "Use Material Design 3 components" |
| **定义配色方案** | "Color palette: primary #4361ee, background #f8f9fa" |
| **描述布局结构** | "Two-column layout: sidebar 240px + main content area" |
| **列出具体组件** | "Include: data table, search bar, pagination, action buttons" |
| **指定交互行为** | "Cards should have hover effect, click opens detail view" |
| **参考现有产品** | "Design similar to Notion's sidebar + Linear's issue list" |
| **设定响应式策略** | "Mobile: stack vertically, Tablet: 2 columns, Desktop: sidebar + 3 columns" |

### 8.2 Claude Code 集成 Stitch 代码的最佳实践

**1. 分层处理**
```bash
# 不要一次性粘贴所有 Stitch 代码
# 按页面/组件逐个处理

# Step 1: 先处理布局组件（Sidebar, Header, Footer）
claude "基于这段 Stitch 代码创建 Layout 组件：<paste>"

# Step 2: 再处理页面组件
claude "基于这段 Stitch 代码创建 Dashboard 页面：<paste>"

# Step 3: 最后处理交互逻辑
claude "为 Dashboard 页面添加数据获取和交互逻辑"
```

**2. 建立设计 Token**
```typescript
// lib/design-tokens.ts — 从 Stitch 设计中提取
export const designTokens = {
  colors: {
    primary: '#4361ee',
    secondary: '#3f37c9',
    accent: '#4cc9f0',
    background: '#f8f9fa',
    surface: '#ffffff',
    text: {
      primary: '#1a1a2e',
      secondary: '#6c757d',
      inverse: '#ffffff',
    },
  },
  spacing: {
    xs: '0.25rem',
    sm: '0.5rem',
    md: '1rem',
    lg: '1.5rem',
    xl: '2rem',
  },
  borderRadius: {
    sm: '0.375rem',
    md: '0.5rem',
    lg: '0.75rem',
    xl: '1rem',
  },
  shadows: {
    sm: '0 1px 2px rgba(0,0,0,0.05)',
    md: '0 4px 6px rgba(0,0,0,0.07)',
    lg: '0 10px 15px rgba(0,0,0,0.1)',
  },
} as const;
```

**3. 使用 CLAUDE.md 记录设计决策**
```markdown
# CLAUDE.md

## 项目：电商平台
## UI 来源：Google Stitch 设计导出

### 设计规范
- 配色：基于 Stitch 生成的 Material Design 3 主题
- 组件库：shadcn/ui + Radix UI
- 图标：Lucide Icons
- 字体：Inter (UI) + Plus Jakarta Sans (标题)

### Stitch → Code 映射
| Stitch 页面 | 代码文件 | 状态 |
|-------------|----------|------|
| 商品列表 | app/products/page.tsx | ✅ |
| 购物车 | app/cart/page.tsx | ✅ |
| 结账流程 | app/checkout/page.tsx | 🔄 |
```

### 8.3 代码质量保证流程

```
Stitch 导出代码
    ↓
Claude Code 转化 + 优化
    ↓
┌─ 代码审查 ─────────────────────┐
│ /code-review 检查：              │
│ ✅ TypeScript 类型安全           │
│ ✅ 组件可访问性 (a11y)           │
│ ✅ 响应式布局测试                │
│ ✅ 性能优化 (lazy load, memo)   │
│ ✅ 安全审查 (XSS, CSRF)         │
└─────────────────────────────┬───┘
                              ↓
┌─ 测试验证 ─────────────────────┐
│ /tdd 生成测试：                 │
│ ✅ 组件单元测试 (Vitest)        │
│ ✅ API 集成测试                 │
│ ✅ E2E 流程测试 (Playwright)    │
│ ✅ 视觉回归测试                 │
└─────────────────────────────┬───┘
                              ↓
                        合并部署 🚀
```

### 8.4 常用 Claude Code 命令速查

```bash
# 项目初始化
claude init                          # 初始化 Claude Code

# Stitch 代码集成
claude "将这段 Stitch HTML/CSS 转为 React 组件：\n<paste>"

# 前端开发
claude "创建 shadcn/ui 的 Button, Card, Dialog 组件"
claude "添加 Tailwind 动画和过渡效果"
claude "实现响应式布局：mobile-first"

# 后端开发
claude "设计 Prisma schema 包含 User, Product, Order"
claude "创建 RESTful API 路由：CRUD + 搜索 + 分页"
claude "实现 JWT 认证中间件"

# 测试
claude "为 /api/products 路由编写 Vitest 测试"
claude "创建 Playwright E2E 测试：完整购物流程"

# 代码审查
claude "/code-review"               # 审查所有更改
claude "/security-review"           # 安全审查

# 部署
claude "创建 Dockerfile 和 docker-compose.yml"
claude "配置 GitHub Actions CI/CD"
```

---

## 9. 通过 API 直接调用 Stitch 能力（Gemini API 集成）

> **核心思路**：Google Stitch 底层使用 **Gemini 多模态模型**驱动。通过获取 Google AI Studio 的 API Key，可以在 Claude Code 中直接调用 Gemini API，实现与 Stitch 类似的 UI 生成能力，**无需手动访问 Stitch 网页**。

### 9.1 原理说明

```
┌─────────────────────────────────────────────────────────────────┐
│            两种使用 Stitch 能力的方式                              │
│                                                                 │
│  方式 A（手动）：                                                 │
│  浏览器 → stitch.withgoogle.com → 手动输入 Prompt → 复制代码      │
│                                                                 │
│  方式 B（API 直接调用）✅ 推荐：                                   │
│  Claude Code → Gemini API → 自动生成 UI 代码 → 直接集成到项目     │
│                                                                 │
│  优势：自动化、可批量、可嵌入 CI/CD、无需离开终端                   │
└─────────────────────────────────────────────────────────────────┘
```

| 对比维度 | 手动使用 Stitch 网页 | API 直接调用 |
|----------|---------------------|-------------|
| **效率** | 每次需手动操作 | 一次配置，反复使用 |
| **批量生成** | 逐页操作 | 脚本批量生成 |
| **集成度** | 需手动复制粘贴 | 代码直接写入项目 |
| **自动化** | 无法自动化 | 可嵌入 CI/CD 流水线 |
| **费用** | 免费额度 | 按 Gemini API 计费 |

### 9.2 获取 Google AI Studio API Key

**Step 1：访问 Google AI Studio**

前往 [Google AI Studio](https://aistudio.google.com/)，使用 Google 账号登录。

**Step 2：创建 API Key**

```
Google AI Studio 控制台
    → 左侧菜单 "Get API Key"
    → "Create API Key"
    → 选择 Google Cloud 项目（或创建新项目）
    → 复制生成的 API Key
```

> ⚠️ **安全提示**：API Key 类似密码，切勿提交到 Git 仓库。建议存储在环境变量或密钥管理工具中。

**Step 3：确认 API Key 可用**

```bash
# 测试 API Key 是否有效（替换 YOUR_API_KEY）
curl "https://generativelanguage.googleapis.com/v1beta/models?key=YOUR_API_KEY" | head -20
```

### 9.3 方法一：环境变量配置 + Claude Code 直接调用

**Step 1：设置环境变量**

```bash
# macOS/Linux — 添加到 ~/.zshrc 或 ~/.bashrc
echo 'export GOOGLE_AI_API_KEY="your-api-key-here"' >> ~/.zshrc
source ~/.zshrc

# Windows PowerShell
[Environment]::SetEnvironmentVariable("GOOGLE_AI_API_KEY", "your-api-key-here", "User")

# Windows CMD
setx GOOGLE_AI_API_KEY "your-api-key-here"
```

**Step 2：在 Claude Code 中使用**

配置完成后，在 Claude Code 中可以直接让 Claude 调用 Gemini API：

```bash
# 在 Claude Code 中执行
claude "使用我的 GOOGLE_AI_API_KEY 环境变量，调用 Gemini API 生成以下 UI：
一个项目管理仪表盘，包含：
- 左侧边栏导航
- 项目卡片网格
- 统计数据行
- 深色主题，蓝色强调色
请生成完整的 HTML + CSS + JavaScript 代码，并保存到 dashboard.html"
```

Claude Code 会自动读取环境变量，构造 API 请求，生成代码并写入文件。

**Step 3：进阶 — 创建专用脚本**

在项目根目录创建 `scripts/generate-ui.sh`：

```bash
#!/bin/bash
# generate-ui.sh — 通过 Gemini API 生成 UI 代码

API_KEY="${GOOGLE_AI_API_KEY}"
MODEL="gemini-2.5-flash"  # 或 gemini-2.5-pro

if [ -z "$API_KEY" ]; then
  echo "❌ 错误：请先设置 GOOGLE_AI_API_KEY 环境变量"
  exit 1
fi

PROMPT="$1"
OUTPUT="${2:-generated-ui.html}"

if [ -z "$PROMPT" ]; then
  echo "用法: ./generate-ui.sh '<UI 描述>' [输出文件名]"
  echo "示例: ./generate-ui.sh 'A login page with social login buttons' login.html"
  exit 1
fi

echo "🎨 正在通过 Gemini API 生成 UI..."

# 构造系统提示词 — 模拟 Stitch 的行为
SYSTEM_PROMPT="You are an expert UI designer and frontend developer.
Generate production-quality HTML + CSS + JavaScript code based on the user's description.
Follow these guidelines:
- Use modern CSS (Flexbox/Grid, CSS Variables, animations)
- Responsive design (mobile-first)
- Clean semantic HTML5
- Include inline styles or a <style> block
- Output ONLY the complete HTML file, no explanations
- Use Material Design 3 principles
- Support both light and dark mode with CSS prefers-color-scheme"

# 调用 Gemini API
curl -s "https://generativelanguage.googleapis.com/v1beta/models/${MODEL}:generateContent?key=${API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "system_instruction": {
      "parts": [{"text": "'"$SYSTEM_PROMPT"'"}]
    },
    "contents": [{
      "parts": [{"text": "'"$PROMPT"'"}]
    }],
    "generationConfig": {
      "temperature": 0.7,
      "maxOutputTokens": 8192
    }
  }' | python3 -c "
import sys, json
data = json.load(sys.stdin)
try:
    text = data['candidates'][0]['content']['parts'][0]['text']
    # 移除 markdown 代码块标记
    text = text.replace('\`\`\`html', '').replace('\`\`\`', '')
    print(text.strip())
except (KeyError, IndexError):
    print('Error: Failed to parse API response', file=sys.stderr)
    print(json.dumps(data, indent=2), file=sys.stderr)
    sys.exit(1)
" > "$OUTPUT"

echo "✅ UI 代码已生成: $OUTPUT"
echo "📏 文件大小: $(wc -c < "$OUTPUT" | tr -d ' ') bytes"
```

赋予执行权限并使用：

```bash
chmod +x scripts/generate-ui.sh

# 生成单个页面
./scripts/generate-ui.sh "A modern login page with email/password fields, social login buttons (Google, GitHub), and a gradient background" login.html

# 生成仪表盘
./scripts/generate-ui.sh "Project management dashboard with sidebar, cards grid, stats row, dark theme" dashboard.html

# 批量生成多个页面
for page in "login:登录页" "dashboard:仪表盘" "profile:用户中心" "settings:设置页"; do
  IFS=':' read -r name desc <<< "$page"
  ./scripts/generate-ui.sh "A ${desc} with Material Design 3, responsive, dark mode support" "stitch-exports/${name}.html"
done
```

### 9.4 方法二：MCP Server 集成（推荐）

通过 MCP（Model Context Protocol）服务器，让 Claude Code **原生集成** Gemini API 调用能力。

**Step 1：配置 Claude Code 的 MCP 设置**

在项目根目录或用户目录下创建 MCP 配置文件：

```bash
# 项目级配置（仅当前项目生效）
# 创建 .mcp.json 到项目根目录

# 或全局配置（所有项目生效）
# 编辑 ~/.claude/settings.json
```

**`.mcp.json` 配置内容**：

```json
{
  "mcpServers": {
    "google-gemini": {
      "command": "npx",
      "args": ["-y", "@anthropic-ai/mcp-gemini"],
      "env": {
        "GEMINI_API_KEY": "your-api-key-here"
      }
    }
  }
}
```

> 💡 也可以将 API Key 存储在环境变量中，避免硬编码：
> ```json
> "env": {
>   "GEMINI_API_KEY": "${GOOGLE_AI_API_KEY}"
> }
> ```

**Step 2：在 Claude Code 中使用**

配置完成后重启 Claude Code，即可直接通过 MCP 调用 Gemini：

```bash
# 启动 Claude Code
claude

# 在 Claude Code 中直接请求 UI 生成
> "通过 Gemini 生成一个电商首页的 UI 代码，包含：
> - 顶部导航栏（Logo、搜索、购物车、用户头像）
> - Hero Banner 轮播
> - 商品分类网格
> - 热门商品卡片列表
> - 页脚
> 输出为 React + Tailwind CSS 组件"
```

Claude Code 会通过 MCP Server 自动调用 Gemini API，获取生成的代码，并直接写入你的项目文件。

**Step 3：进阶 — 自定义 MCP Server**

如果需要更定制化的行为，可以创建自己的 MCP Server：

```typescript
// mcp-servers/stitch-ui-server/index.ts
import { Server } from "@modelcontextprotocol/sdk/server";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio";
import { z } from "zod";

const server = new Server(
  { name: "stitch-ui-generator", version: "1.0.0" },
  { capabilities: { tools: {} } }
);

// 注册工具：生成 UI
server.setRequestHandler("tools/list", async () => ({
  tools: [
    {
      name: "generate_ui",
      description:
        "使用 Gemini API 生成 UI 代码（模拟 Google Stitch 功能）。输入 UI 描述，输出完整的 HTML/CSS/JS 或框架代码。",
      inputSchema: {
        type: "object",
        properties: {
          prompt: {
            type: "string",
            description: "UI 设计描述（中文或英文）",
          },
          framework: {
            type: "string",
            enum: ["html", "react", "vue", "angular", "flutter"],
            description: "输出框架（默认 html）",
          },
          theme: {
            type: "string",
            enum: ["light", "dark", "auto"],
            description: "主题模式（默认 auto）",
          },
          responsive: {
            type: "boolean",
            description: "是否响应式设计（默认 true）",
          },
        },
        required: ["prompt"],
      },
    },
    {
      name: "generate_ui_from_image",
      description: "上传图片/线框图，通过 Gemini API 生成对应的 UI 代码（类似 Stitch 的图片转代码功能）。",
      inputSchema: {
        type: "object",
        properties: {
          imagePath: {
            type: "string",
            description: "图片文件路径（本地路径）",
          },
          framework: {
            type: "string",
            enum: ["html", "react", "vue", "angular", "flutter"],
            description: "输出框架（默认 html）",
          },
        },
        required: ["imagePath"],
      },
    },
  ],
}));

// 处理工具调用
server.setRequestHandler("tools/call", async (request) => {
  const { name, arguments: args } = request.params;

  if (name === "generate_ui") {
    const apiKey = process.env.GEMINI_API_KEY;
    const model = "gemini-2.5-flash";

    const frameworkMap: Record<string, string> = {
      html: "HTML5 + CSS3 + Vanilla JavaScript",
      react: "React + TypeScript + Tailwind CSS",
      vue: "Vue 3 + TypeScript + Tailwind CSS",
      angular: "Angular + TypeScript + SCSS",
      flutter: "Flutter + Dart with Material Design 3",
    };

    const framework = (args.framework as string) || "html";
    const theme = (args.theme as string) || "auto";
    const responsive = args.responsive !== false;

    const systemPrompt = `You are Google Stitch, an expert UI designer.
Generate production-quality ${frameworkMap[framework]} code.
Material Design 3 principles. ${responsive ? "Responsive mobile-first." : "Fixed width."}
Theme: ${theme === "auto" ? "Support both light and dark mode via CSS prefers-color-scheme" : theme + " mode only"}.
Output ONLY the complete code, no markdown fences, no explanations.`;

    const response = await fetch(
      `https://generativelanguage.googleapis.com/v1beta/models/${model}:generateContent?key=${apiKey}`,
      {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          system_instruction: {
            parts: [{ text: systemPrompt }],
          },
          contents: {
            parts: [{ text: args.prompt as string }],
          },
          generationConfig: {
            temperature: 0.7,
            maxOutputTokens: 16384,
          },
        }),
      }
    );

    const data = await response.json();
    const code =
      data.candidates?.[0]?.content?.parts?.[0]?.text ||
      "Error: Failed to generate UI";

    return {
      content: [{ type: "text", text: code }],
    };
  }

  if (name === "generate_ui_from_image") {
    const apiKey = process.env.GEMINI_API_KEY;
    const model = "gemini-2.5-flash"; // 使用多模态模型

    // 读取图片并转为 base64
    const fs = await import("fs");
    const path = await import("path");
    const imagePath = args.imagePath as string;
    const imageBuffer = fs.readFileSync(imagePath);
    const base64Image = imageBuffer.toString("base64");
    const ext = path.extname(imagePath).toLowerCase();
    const mimeType =
      ext === ".png"
        ? "image/png"
        : ext === ".jpg" || ext === ".jpeg"
          ? "image/jpeg"
          : ext === ".webp"
            ? "image/webp"
            : "image/png";

    const framework = (args.framework as string) || "html";

    const response = await fetch(
      `https://generativelanguage.googleapis.com/v1beta/models/${model}:generateContent?key=${apiKey}`,
      {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          contents: {
            parts: [
              {
                text: `Analyze this UI design/wireframe and generate production-quality ${framework} code that faithfully reproduces it. Use Material Design 3. Output ONLY the code, no explanations.`,
              },
              {
                inlineData: {
                  mimeType: mimeType,
                  data: base64Image,
                },
              },
            ],
          },
          generationConfig: {
            temperature: 0.5,
            maxOutputTokens: 16384,
          },
        }),
      }
    );

    const data = await response.json();
    const code =
      data.candidates?.[0]?.content?.parts?.[0]?.text ||
      "Error: Failed to generate UI from image";

    return {
      content: [{ type: "text", text: code }],
    };
  }

  throw new Error(`Unknown tool: ${name}`);
});

// 启动服务器
const transport = new StdioServerTransport();
server.connect(transport);
```

**配置到 Claude Code**：

```json
// .mcp.json
{
  "mcpServers": {
    "stitch-ui": {
      "command": "node",
      "args": ["./mcp-servers/stitch-ui-server/index.js"],
      "env": {
        "GEMINI_API_KEY": "your-api-key-here"
      }
    }
  }
}
```

### 9.5 方法三：Python 封装工具

创建一个 Python 工具脚本，在 Claude Code 中通过 Bash 工具调用：

```python
# scripts/stitch_api.py
"""Stitch-like UI Generator — 通过 Gemini API 模拟 Stitch 功能"""

import os
import sys
import json
import argparse
import urllib.request
import urllib.error
from pathlib import Path

API_KEY = os.environ.get("GOOGLE_AI_API_KEY", "")
API_BASE = "https://generativelanguage.googleapis.com/v1beta/models"

FRAMEWORKS = {
    "html": "HTML5 + CSS3 + Vanilla JavaScript, inline styles in <style> block",
    "react": "React functional components + TypeScript + Tailwind CSS classes",
    "vue": "Vue 3 Single File Components + TypeScript + Tailwind CSS",
    "angular": "Angular standalone component + TypeScript + SCSS",
    "flutter": "Flutter + Dart widgets with Material Design 3",
}

def generate_ui(prompt: str, framework: str = "html", model: str = "gemini-2.5-flash") -> str:
    """调用 Gemini API 生成 UI 代码"""
    if not API_KEY:
        print("❌ 错误：请设置 GOOGLE_AI_API_KEY 环境变量", file=sys.stderr)
        sys.exit(1)

    system_prompt = f"""You are Google Stitch, an expert UI designer and frontend developer.

Generate production-quality code in: {FRAMEWORKS.get(framework, FRAMEWORKS["html"])}
Follow these strict rules:
1. Material Design 3 principles (components, spacing, elevation)
2. Fully responsive (mobile-first, breakpoints at 640px, 768px, 1024px, 1280px)
3. Support light and dark mode via CSS prefers-color-scheme
4. Clean, semantic structure with proper accessibility attributes
5. Smooth transitions and micro-animations
6. Output ONLY the complete code — no markdown fences, no explanations, no comments about what you're doing"""

    payload = {
        "system_instruction": {"parts": [{"text": system_prompt}]},
        "contents": {"parts": [{"text": prompt}]},
        "generationConfig": {
            "temperature": 0.7,
            "topP": 0.95,
            "topK": 40,
            "maxOutputTokens": 16384,
        },
    }

    url = f"{API_BASE}/{model}:generateContent?key={API_KEY}"
    data = json.dumps(payload).encode("utf-8")

    req = urllib.request.Request(url, data=data, headers={"Content-Type": "application/json"})

    try:
        with urllib.request.urlopen(req, timeout=60) as resp:
            result = json.loads(resp.read().decode("utf-8"))
            code = result["candidates"][0]["content"]["parts"][0]["text"]
            # 清理 markdown 代码块标记
            code = code.replace("```html", "").replace("```tsx", "").replace("```vue", "").replace("```dart", "").replace("```", "")
            return code.strip()
    except urllib.error.HTTPError as e:
        error_body = e.read().decode("utf-8")
        print(f"❌ API 错误 ({e.code}): {error_body}", file=sys.stderr)
        sys.exit(1)
    except (KeyError, IndexError) as e:
        print(f"❌ 解析错误: {e}", file=sys.stderr)
        print(json.dumps(result, indent=2, ensure_ascii=False), file=sys.stderr)
        sys.exit(1)


def generate_from_image(image_path: str, framework: str = "html", model: str = "gemini-2.5-flash") -> str:
    """上传图片/线框图生成 UI 代码"""
    import base64

    if not API_KEY:
        print("❌ 错误：请设置 GOOGLE_AI_API_KEY 环境变量", file=sys.stderr)
        sys.exit(1)

    path = Path(image_path)
    if not path.exists():
        print(f"❌ 文件不存在: {image_path}", file=sys.stderr)
        sys.exit(1)

    ext = path.suffix.lower()
    mime_map = {".png": "image/png", ".jpg": "image/jpeg", ".jpeg": "image/jpeg", ".webp": "image/webp"}
    mime_type = mime_map.get(ext, "image/png")

    with open(path, "rb") as f:
        b64_data = base64.b64encode(f.read()).decode("utf-8")

    payload = {
        "contents": {
            "parts": [
                {
                    "text": f"Analyze this UI design/wireframe/screenshot and reproduce it faithfully as production-quality {FRAMEWORKS.get(framework, FRAMEWORKS['html'])} code. Material Design 3. Responsive. Output ONLY the complete code."
                },
                {
                    "inlineData": {
                        "mimeType": mime_type,
                        "data": b64_data,
                    }
                },
            ]
        },
        "generationConfig": {
            "temperature": 0.5,
            "maxOutputTokens": 16384,
        },
    }

    url = f"{API_BASE}/{model}:generateContent?key={API_KEY}"
    data = json.dumps(payload).encode("utf-8")

    req = urllib.request.Request(url, data=data, headers={"Content-Type": "application/json"})

    try:
        with urllib.request.urlopen(req, timeout=120) as resp:
            result = json.loads(resp.read().decode("utf-8"))
            code = result["candidates"][0]["content"]["parts"][0]["text"]
            code = code.replace("```html", "").replace("```tsx", "").replace("```vue", "").replace("```dart", "").replace("```", "")
            return code.strip()
    except urllib.error.HTTPError as e:
        error_body = e.read().decode("utf-8")
        print(f"❌ API 错误 ({e.code}): {error_body}", file=sys.stderr)
        sys.exit(1)


if __name__ == "__main__":
    parser = argparse.ArgumentParser(description="Stitch-like UI Generator via Gemini API")
    subparsers = parser.add_subparsers(dest="command")

    # generate 命令
    gen_parser = subparsers.add_parser("generate", help="通过文字描述生成 UI")
    gen_parser.add_argument("prompt", help="UI 设计描述")
    gen_parser.add_argument("-f", "--framework", default="html", choices=FRAMEWORKS.keys())
    gen_parser.add_argument("-o", "--output", help="输出文件路径")
    gen_parser.add_argument("-m", "--model", default="gemini-2.5-flash")

    # image 命令
    img_parser = subparsers.add_parser("image", help="从图片/线框图生成 UI")
    img_parser.add_argument("image", help="图片文件路径")
    img_parser.add_argument("-f", "--framework", default="html", choices=FRAMEWORKS.keys())
    img_parser.add_argument("-o", "--output", help="输出文件路径")
    img_parser.add_argument("-m", "--model", default="gemini-2.5-flash")

    args = parser.parse_args()

    if args.command == "generate":
        print(f"🎨 正在生成 UI ({args.framework})...", file=sys.stderr)
        code = generate_ui(args.prompt, args.framework, args.model)
        if args.output:
            Path(args.output).parent.mkdir(parents=True, exist_ok=True)
            Path(args.output).write_text(code, encoding="utf-8")
            print(f"✅ 已保存到: {args.output}", file=sys.stderr)
        else:
            print(code)

    elif args.command == "image":
        print(f"🖼️ 正在从图片生成 UI ({args.framework})...", file=sys.stderr)
        code = generate_from_image(args.image, args.framework, args.model)
        if args.output:
            Path(args.output).parent.mkdir(parents=True, exist_ok=True)
            Path(args.output).write_text(code, encoding="utf-8")
            print(f"✅ 已保存到: {args.output}", file=sys.stderr)
        else:
            print(code)

    else:
        parser.print_help()
```

**使用方式**：

```bash
# 设置 API Key
export GOOGLE_AI_API_KEY="your-api-key"

# 文字生成 UI
python scripts/stitch_api.py generate "A modern e-commerce homepage with hero banner, product grid, category navigation" -f react -o components/HomePage.tsx

# 图片生成 UI
python scripts/stitch_api.py image ./wireframes/login.png -f react -o components/LoginPage.tsx

# 在 Claude Code 中调用
claude "运行 python scripts/stitch_api.py generate '项目管理仪表盘，侧边栏+卡片+统计行' -f react -o dashboard.tsx，然后基于生成的代码集成到项目中"
```

### 9.6 三种方法对比与选择建议

| 维度 | 方法一：环境变量 + 脚本 | 方法二：MCP Server | 方法三：Python 工具 |
|------|------------------------|-------------------|-------------------|
| **配置难度** | ⭐ 简单 | ⭐⭐ 中等 | ⭐⭐ 中等 |
| **集成深度** | ⭐⭐ 浅 | ⭐⭐⭐⭐⭐ 最深 | ⭐⭐⭐ 中等 |
| **使用体验** | 需手动运行脚本 | Claude Code 原生调用 | Claude Code 调用 Bash |
| **图片支持** | 需额外处理 | ✅ 原生支持 | ✅ 内置支持 |
| **定制性** | ⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| **适合场景** | 快速验证、简单任务 | 日常开发、深度集成 | 批量生成、CI/CD |

> 💡 **推荐方案**：日常开发使用 **方法二（MCP Server）**，需要批量生成或 CI/CD 集成时使用 **方法三（Python 工具）**。

### 9.7 API 调用成本估算

| Gemini 模型 | 免费额度 | 输入价格 | 输出价格 |
|-------------|---------|---------|---------|
| **gemini-2.5-flash** | 免费层有 RPM/RPD 限制 | $0.15/1M tokens | $0.60/1M tokens |
| **gemini-2.5-pro** | 免费层有 RPM/RPD 限制 | $1.25/1M tokens | $10.00/1M tokens |

> 💡 **成本建议**：日常 UI 生成使用 `gemini-2.5-flash` 即可，成本极低。复杂/精细的 UI 需求可升级到 `gemini-2.5-pro`。

### 9.8 最佳实践：Stitch API + Claude Code 联动

```bash
# 1. 通过 API 快速生成 UI 原型
claude "使用 stitch_api.py 生成登录页 UI: python scripts/stitch_api.py generate 'Modern login with social buttons' -f react -o stitch-exports/login.tsx"

# 2. 让 Claude Code 深度优化生成的代码
claude "查看 stitch-exports/login.tsx，然后：
1. 添加 TypeScript 类型定义
2. 集成 React Hook Form + Zod 验证
3. 连接到 /api/auth/login 端点
4. 添加错误处理和加载状态
5. 将组件移到 src/components/auth/LoginForm.tsx"

# 3. 批量生成 → 逐个优化
claude "批量为以下页面生成 UI：
- dashboard: 仪表盘
- projects: 项目列表
- settings: 设置页
使用 stitch_api.py 生成后，逐个优化为生产级组件"
```

---

## 10. 常见问题与故障排除

### Q1：Stitch 生成的代码质量如何？

> Stitch 生成的是**起点代码**，不是生产代码。它非常适合作为设计参考和快速原型，但通常需要 Claude Code 进行以下优化：
> - 添加 TypeScript 类型
> - 实现状态管理逻辑
> - 处理错误和加载状态
> - 优化性能（memo、lazy loading）
> - 添加无障碍属性

### Q2：Stitch 支持哪些设计风格？

> 默认 Material Design 3，但通过详细的 Prompt 可以生成各种风格：
> - Apple HIG 风格
> - 极简风格
> - 毛玻璃/新拟态
> - 暗色模式
> - 品牌定制风格

### Q3：如何保持 Stitch 设计与代码的一致性？

> 建议使用 Design Tokens 方式管理：
> 1. 从 Stitch 设计中提取颜色、间距、圆角等 Token
> 2. 在代码中统一定义（CSS Variables / Tailwind Config）
> 3. 所有组件引用 Token 而非硬编码值

### Q4：Stitch 代码导出格式选择建议？

| 场景 | 推荐格式 | 原因 |
|------|----------|------|
| Next.js 项目 | HTML/CSS | 方便 Claude Code 转化为 React |
| Angular 项目 | Angular | 直接可用 |
| Flutter App | Flutter/Dart | 直接可用 |
| 纯原型演示 | HTML/CSS/JS | 开箱即用 |

### Q5：如何处理 Stitch 不支持的复杂交互？

> Stitch 负责视觉设计和静态布局，复杂交互交给 Claude Code：
> - **拖拽排序** → Claude Code 集成 dnd-kit
> - **实时数据** → Claude Code 集成 WebSocket/SWR
> - **复杂表单** → Claude Code 集成 React Hook Form + Zod
> - **图表可视化** → Claude Code 集成 Recharts/Nivo
> - **地图** → Claude Code 集成 Google Maps / Mapbox

### Q6：多语言/国际化支持？

> Stitch 生成单一语言 UI，Claude Code 负责国际化：
> ```bash
> claude "将这个项目改造为支持 i18n（中/英/日），使用 next-intl"
> ```

### Q7：API 调用返回 429 错误怎么办？

> 这是速率限制（Rate Limit）错误。解决方案：
> 1. **降低调用频率**：添加请求间隔（如 `sleep 2`）
> 2. **升级计划**：在 Google AI Studio 中升级到付费计划
> 3. **换用 Flash 模型**：`gemini-2.5-flash` 的速率限制更宽松
> 4. **指数退避**：在脚本中实现自动重试逻辑

### Q8：Gemini API 生成的 UI 质量不如 Stitch 网页版？

> 这是正常的，因为 Stitch 网页版经过了专门的 fine-tuning 和后处理。优化建议：
> 1. **优化 System Prompt**：更详细地描述设计规范和约束
> 2. **使用 Pro 模型**：`gemini-2.5-pro` 生成质量更高
> 3. **提供参考图片**：使用图片输入比纯文字描述效果更好
> 4. **分步生成**：先布局，再组件，最后样式
> 5. **多次迭代**：对生成结果进行多轮优化提示

---

## 总结

### 🎯 核心结论

| 维度 | Google Stitch | Claude Code | 组合效果 |
|------|---------------|-------------|----------|
| **速度** | 分钟级生成设计 | 分钟级生成代码 | ⚡ 秒级设计 + 分钟级全栈 |
| **质量** | 中等（设计起点） | 高（生产级代码） | 🎨 设计专业 + 代码可靠 |
| **范围** | 仅前端 UI | 全栈全流程 | 🌐 覆盖完整开发链路 |
| **适用** | 快速原型 + UI 设计 | 完整应用开发 | 🚀 从想法到上线 |

### 📊 工作效率对比

| 任务 | 传统开发 | 仅 Claude Code | Stitch + Claude Code |
|------|----------|----------------|---------------------|
| UI 设计 | 2-3 天 | 30 分钟（文字描述） | **5 分钟（可视化）** |
| 前端开发 | 3-5 天 | 1-2 天 | **4-8 小时** |
| 后端开发 | 3-5 天 | 1-2 天 | **1-2 天** |
| 测试部署 | 2-3 天 | 4-8 小时 | **4-8 小时** |
| **总计** | **10-16 天** | **3-5 天** | **2-3 天** |

> 💡 **最佳实践**：Stitch 负责"看起来怎样"，Claude Code 负责"怎样工作"。两者结合，一个人就能完成一个完整产品团队的工作。

---

## 参考资源

- [Google Stitch 官方网站](https://stitch.withgoogle.com/)
- [Google AI Studio（获取 Gemini API Key）](https://aistudio.google.com/)
- [Gemini API 官方文档](https://ai.google.dev/gemini-api/docs)
- [Google I/O 2025 公告](https://io.google/2025/)
- [Material Design 3 文档](https://m3.material.io/)
- [MCP 协议文档](https://modelcontextprotocol.io/)
- [[Claude Code安装指南-GLM模型]] — Claude Code 安装
- [[Claude Code 必装 Skills 推荐指南]] — 推荐 Skills
- [[Claude Code Web-UI 详细指南]] — Web UI 使用
- [[ClaudeCode浏览器自动化指南]] — 浏览器自动化
