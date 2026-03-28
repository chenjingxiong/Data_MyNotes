# Everything Claude Code (ECC) 使用指南

## 📖 目录

- [系统简介](#系统简介)
- [核心概念](#核心概念)
- [Skills 分类](#skills-分类)
- [常用 Skills 详解](#常用-skills-详解)
- [工作流程](#工作流程)
- [最佳实践](#最佳实践)
- [故障排查](#故障排查)

---

## 系统简介

**Everything Claude Code (ECC)** 是一个强大的 Claude Code 扩展系统，通过 **Skills** 和 **Instincts** 两个核心概念，大幅提升了 AI 辅助编程的能力。

### 主要特点

- 🧠 **学习机制**: 从会话中提取可重用模式，保存为 instincts
- 🎯 **专用技能**: 针对特定编程语言、框架和任务的专业化技能
- 🔄 **持续改进**: 通过评估和反馈不断优化代码质量
- 🚀 **TDD 驱动**: 强制测试驱动开发流程
- 📦 **模块化设计**: 每个技能专注于特定领域

---

## 核心概念

### 1. Skills（技能）

Skills 是预定义的专业能力模块，每个技能针对特定的编程任务或领域。

**示例：**
- `python-review`: Python 代码审查
- `go-test`: Go TDD 工作流
- `django-patterns`: Django 架构模式
- `kotlin-review`: Kotlin 代码审查

### 2. Instincts（直觉/模式）

Instincts 是从你的项目中学习到的编码模式和最佳实践。

**生命周期：**
```
编码会话 → 提取模式 → 保存为 instinct → 应用到新代码 → 持续改进
```

### 3. 评估驱动开发

ECC 强调先写测试，再实现功能，确保代码质量。

---

## Skills 分类

根据你已安装的 skills，可以将其分为以下几类：

### 🛠️ 通用工作流

| Skill | 功能描述 |
|-------|---------|
| `plan` | 创建分步实施计划 |
| `tdd` | 测试驱动开发工作流 |
| `tdd-workflow` | 通用 TDD 流程 |
| `verification-loop` | 综合验证系统 |
| `blueprint` | 将目标转化为多会话施工计划 |
| `continuous-learning-v2` | 基于本能的学习系统 |

### 🔧 语言特定

#### Python
- `python-review`: PEP 8、类型提示、安全审查
- `python-patterns`: Pythonic 惯用法和最佳实践
- `python-testing`: pytest、TDD、fixtures、mocking

#### Go
- `go-review`: 惯用法、并发、错误处理审查
- `go-patterns`: 习惯用法、并发模式
- `go-testing`: 表驱动测试、子测试、基准测试
- `go-build`: 构建错误修复
- `go-test`: Go TDD 工作流

#### Kotlin
- `kotlin-review`: 惯用法、空安全、协程审查
- `kotlin-patterns`: 惯用法、最佳实践
- `kotlin-testing`: Kotest、MockK、协程测试
- `kotlin-build`: 构建错误修复
- `kotlin-test`: Kotlin TDD 工作流

#### Swift
- `swiftui-patterns`: SwiftUI 架构模式
- `swift-concurrency-6-2`: Swift 6.2 并发
- `swift-actor-persistence`: Actor 线程安全持久化
- `swift-protocol-di-testing`: 协议依赖注入

#### C++
- `cpp-coding-standards`: C++ 核心准则
- `cpp-testing`: GoogleTest/CTest 配置

### 🌐 框架特定

#### Django
- `django-patterns`: 架构模式、DRF、ORM
- `django-security`: 认证、授权、CSRF
- `django-tdd`: pytest-django、TDD
- `django-verification`: 迁移、linting、测试验证

#### Spring Boot
- `springboot-patterns`: REST API、分层服务
- `springboot-security`: Spring Security 最佳实践
- `springboot-tdd`: JUnit 5、Mockito、MockMvc
- `springboot-verification`: 构建验证循环

#### Compose
- `compose-multiplatform-patterns`: KMP Compose 状态管理
- `android-clean-architecture`: Android/KMP 清洁架构

### 🏗️ 架构与设计

- `backend-patterns`: 后端架构、API 设计
- `frontend-patterns`: React、Next.js、状态管理
- `api-design`: REST API 设计模式
- `database-migrations`: 数据库迁移最佳实践
- `docker-patterns`: Docker 容器化模式

### 🧪 测试相关

- `e2e-testing`: Playwright E2E 测试
- `plankton-code-quality`: 写时代码质量强制
- `testing-strategies`: 通用测试策略

### 🔒 安全与审查

- `security-review`: 认证、输入处理、SSRF 防护
- `security-scan`: .claude/ 目录安全扫描
- `coding-standards`: 通用编码标准

### 🤖 AI 与代理

- `agentic-engineering`: 代理工程实践
- `autonomous-loops`: 自主代理循环模式
- `continuous-agent-loop`: 持续代理循环与质量门
- `enterprise-agent-ops`: 长期代理工作负载运营
- `dmux-workflows`: 多代理编排

### 📊 数据与存储

- `postgres-patterns`: 查询优化、模式设计
- `clickhouse-io`: ClickHouse 分析引擎
- `jpa-patterns`: JPA/Hibernate 模式
- `kotlin-exposed-patterns`: Exposed ORM 模式

### 📦 部署与运维

- `deployment-patterns`: CI/CD、Docker、健康检查
- `configure-ecc`: ECC 交互式安装程序

### 🎨 前端与设计

- `frontend-design`: 高设计质量的前端界面
- `frontend-slides`: 动画丰富的 HTML 演示文稿
- `liquid-glass-design`: iOS 26 液态玻璃设计系统

### 📝 内容与媒体

- `article-writing`: 文章、指南、博客写作
- `content-engine`: 多平台内容系统
- `video-editing`: AI 辅助视频编辑
- `videodb`: 视频音频理解与处理

### 📋 业务领域

- `carrier-relationship-management`: 承运商组合管理
- `customs-trade-compliance`: 海关文档和关税分类
- `energy-procurement`: 电力和天然气采购
- `inventory-demand-planning`: 需求预测和库存优化
- `logistics-exception-management`: 货运异常处理
- `market-research`: 市场研究和竞争分析
- `production-scheduling`: 生产调度和作业排序
- `quality-nonconformance`: 质量控制和不符合项调查
- `returns-reverse-logistics`: 退货授权和处置

---

## 常用 Skills 详解

### 1. `plan` - 创建实施计划

**使用场景：** 开始任何复杂任务之前

**调用方式：**
```
请为添加用户认证功能制定计划
```

**输出：**
- 需求重述
- 风险评估
- 分步实施计划
- 关键文件识别

### 2. `tdd` / `tdd-workflow` - 测试驱动开发

**使用场景：** 编写新功能或修复 bug

**工作流程：**
1. 生成接口/契约
2. 编写失败测试
3. 实现最小可行代码
4. 运行测试直到通过
5. 重构优化

**调用示例：**
```
使用 TDD 实现用户注册功能
```

### 3. 代码审查 Skills

#### `python-review`
- PEP 8 合规性
- 类型提示检查
- 安全漏洞检测
- 性能优化建议

#### `go-review`
- Go 惯用法检查
- 并发安全分析
- 错误处理模式
- 性能优化

#### `kotlin-review`
- Kotlin 惯用法
- 空安全验证
- 协程使用检查
- 架构违规检测

**使用方式：**
在编写或修改代码后自动触发，或手动调用：
```
审查 src/main.py 文件
```

### 4. 框架特定 Skills

#### `django-patterns`
- Django 项目架构
- DRF API 设计
- ORM 最佳实践
- 管理命令定制

#### `springboot-patterns`
- 分层架构
- RESTful API
- 依赖注入
- 数据访问模式

### 5. `continuous-learning-v2` - 持续学习

**功能：**
- 从会话中观察模式
- 创建原子化 instincts
- 自动保存可重用知识

**相关命令：**
- `instinct-status`: 查看已学习的 instincts
- `instinct-export`: 导出 instincts 到文件
- `instinct-import`: 从文件导入 instincts
- `promote`: 将项目 instincts 提升为全局

---

## 工作流程

### 典型开发流程

```
┌─────────────────┐
│  1. Plan        │  使用 plan skill 制定计划
└────────┬────────┘
         ▼
┌─────────────────┐
│  2. Design      │  设计接口和数据模型
└────────┬────────┘
         ▼
┌─────────────────┐
│  3. TDD         │  编写测试 → 实现 → 重构
└────────┬────────┘
         ▼
┌─────────────────┐
│  4. Review      │  使用语言特定 review skill
└────────┬────────┘
         ▼
┌─────────────────┐
│  5. Verify      │  运行 verification-loop
└────────┬────────┘
         ▼
┌─────────────────┐
│  6. Learn       │  continuous-learning 提取模式
└─────────────────┘
```

### 快速修复流程

```
Bug 报告
    ↓
go-build / kotlin-build (修复构建错误)
    ↓
语言特定 review (代码质量)
    ↓
验证测试通过
```

### 新功能开发流程

```
功能需求
    ↓
plan (制定计划)
    ↓
tdd (TDD 实现)
    ↓
框架特定 patterns (应用框架模式)
    ↓
security-review (安全审查)
    ↓
verification-loop (最终验证)
```

---

## 最佳实践

### 1. 始终使用 Plan

对于任何非平凡的任务（>2-3 步），先使用 `plan` skill：

```
Plan: 实现 JWT 认证
- 评估安全需求
- 选择认证策略
- 设计中间件结构
- 分步实施
```

### 2. TDD 优先

ECC 强制 TDD 方法。先写测试，再写代码：

```
使用 tdd 实现 PaymentService
- 生成接口
- 编写失败测试
- 实现功能
- 验证通过
```

### 3. 使用专用 Review Skills

不同语言使用不同的 review skills：

```python
# Python 项目
使用 python-review 审查代码

# Go 项目
使用 go-review 审查代码

# Kotlin 项目
使用 kotlin-review 审查代码
```

### 4. 框架模式

使用框架特定的 patterns：

```django
Django 项目
- django-patterns: 架构和模式
- django-security: 安全最佳实践
- django-tdd: 测试策略
```

### 5. 持续学习

让 `continuous-learning-v2` 自动学习你的编码模式：

```
查看学习状态: instinct-status
导出 instincts: instinct-export
提升到全局: promote skill-name
```

### 6. 安全第一

对涉及用户输入、认证的功能使用 `security-review`：

```
添加登录功能后：
1. 实现功能
2. 运行 security-review
3. 修复发现的问题
```

### 7. 验证循环

使用 `verification-loop` 或框架特定的验证：

```python
# Django 项目
django-verification: 迁移 → linting → 测试

# Spring Boot 项目
springboot-verification: 构建 → 静态分析 → 测试
```

---

## 故障排查

### 构建错误

**症状：** 代码无法编译

**解决方案：**
```bash
# Go 项目
go-build: 修复构建错误

# Kotlin/Gradle 项目
kotlin-build: 修复编译错误
gradle-build: 修复 Gradle 错误
```

### 测试失败

**症状：** 测试不通过

**解决方案：**
1. 确保使用 TDD workflow
2. 检查测试覆盖：`python-testing` / `go-testing` / `kotlin-testing`
3. 运行验证：`verification-loop`

### 代码质量问题

**症状：** Code review 提出问题

**解决方案：**
1. 运行语言特定的 review
2. 检查 `coding-standards`
3. 使用 `plankton-code-quality` 强制质量

### 性能问题

**症状：** 代码运行缓慢

**解决方案：**
- 数据库: `postgres-patterns` / `clickhouse-io`
- 缓存: `content-hash-cache-pattern`
- API: `api-design`

---

## 高级主题

### Instincts 管理

**查看已学习的模式：**
```
instinct-status
```

**导出项目 instincts：**
```
instinct-export ./my-instincts.md
```

**导入 instincts：**
```
instinct-import ./my-instincts.md
```

**提升到全局：**
```
promote python-patterns
```

### 多代理编排

使用 `dmux-workflows` 或 `autonomous-loops` 进行复杂的并行任务：

```
使用 autonomous-loops 处理批量数据处理
- 设置质量门
- 配置监控
- 自动重试
```

### 搜索优先开发

使用 `search-first` 在编码前搜索现有解决方案：

```
任务: 实现 JWT 认证
1. search-first: 搜索现有库
2. plan: 评估方案
3. tdd: 实现选择
```

---

## 参考资源

- **GitHub 仓库**: https://github.com/affaan-m/everything-claude-code
- **Skills 完整列表**: 查看系统提示中的所有可用 skills
- **项目特定技能**: 使用 `skill-stocktake` 审计已安装 skills

---

## 快速参考

### 常用命令速查表

| 任务 | Skill | 命令示例 |
|-----|-------|---------|
| 制定计划 | `plan` | "为 X 功能制定计划" |
| TDD 开发 | `tdd` | "使用 TDD 实现 Y" |
| Python 审查 | `python-review` | "审查 main.py" |
| Go 审查 | `go-review` | "审查 main.go" |
| Kotlin 审查 | `kotlin-review` | "审查 Main.kt" |
| Django 安全 | `django-security` | "检查安全问题" |
| 修复构建 | `go-build` / `kotlin-build` | "修复构建错误" |
| 查看状态 | `instinct-status` | "显示已学习的模式" |
| 安全扫描 | `security-scan` | "扫描配置" |
| 搜索代码 | `search-first` | "搜索 Z 库" |

---

## 结语

Everything Claude Code 是一个不断演进的系统。通过合理使用 Skills 和 Instincts，你可以：

1. **提高代码质量**: 专用审查技能确保最佳实践
2. **加快开发速度**: TDD 和自动化流程减少返工
3. **持续学习**: Instincts 系统积累团队知识
4. **降低风险**: 安全审查和验证循环捕获问题

**建议：** 从简单的任务开始，逐步熟悉各个 skills，然后将其整合到你的日常开发流程中。

---

*最后更新: 2026-03-28*
*基于已安装的 ECC Skills (v2.x)*
