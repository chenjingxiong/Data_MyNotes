# Everything Claude Code (ECC) 资料库

欢迎来到 ECC 资料库！这里包含了 Everything Claude Code 系统的使用指南和参考资料。

---

## 📚 文档导航

### 📘 [完整使用指南](Everything-Claude-Code使用指南.md)
**推荐新手首先阅读！**

包含内容：
- ECC 系统简介
- 核心概念（Skills、Instincts）
- Skills 详细分类
- 工作流程说明
- 最佳实践
- 故障排查

---

### 📗 [Skills 完整索引](Skills索引.md)
快速查找所有已安装的 ECC Skills

包含内容：
- 按功能分类的 Skills
- 按编程语言分类
- 按框架分类
- 快速查找表

---

### 📙 [快速参考卡片](快速参考卡片.md)
**推荐打印！**

包含内容：
- 核心工作流
- 最常用 Skills
- 快捷命令示例
- 问题解决方案

---

## 🚀 快速开始

### 1. 了解核心概念

**Skills** - 预定义的专业能力模块
- 每个技能针对特定编程任务或领域
- 例如：`python-review`、`go-test`、`django-patterns`

**Instincts** - 从项目中学习的编码模式
- 自动提取可重用的最佳实践
- 可以导出、导入和分享

### 2. 基础工作流

```
需求 → plan → tdd → review → verify → learn
  ↓      ↓      ↓       ↓        ↓        ↓
 计划   测试驱动  实现    质量检查   测试验证   持续改进
```

### 3. 常用命令

```bash
# 查看已学习的模式
instinct-status

# 制定计划
plan: 实现用户认证功能

# TDD 开发
tdd: 实现 UserService

# 代码审查
python-review: 审查 main.py
```

---

## 🎯 按需查找

### 我想...

**开始新项目**
→ 阅读 [完整使用指南](Everything-Claude-Code使用指南.md) 的 "工作流程" 章节

**修复构建错误**
→ 使用 `go-build` 或 `kotlin-build` skill

**审查代码质量**
→ 使用 `{语言}-review` skill（如 `python-review`）

**学习新框架**
→ 查阅 [Skills 索引](Skills索引.md) 的 "Web 框架" 章节

**提高测试覆盖率**
→ 使用 `{语言}-testing` 或 `tdd` skill

**解决安全问题**
→ 使用 `security-review` skill

**优化数据库查询**
→ 使用 `postgres-patterns` 或 `jpa-patterns` skill

---

## 📖 推荐阅读顺序

### 初学者
1. [快速参考卡片](快速参考卡片.md) - 了解核心命令
2. [完整使用指南](Everything-Claude-Code使用指南.md) - 深入理解系统
3. [Skills 索引](Skills索引.md) - 查找所需技能

### 有经验用户
1. [Skills 索引](Skills索引.md) - 快速定位 skill
2. [完整使用指南](Everything-Claude-Code使用指南.md) - 查阅特定章节
3. [快速参考卡片](快速参考卡片.md) - 桌边速查

---

## 🔗 外部资源

- **GitHub 仓库**: https://github.com/affaan-m/everything-claude-code
- **Claude Code 官方文档**: https://docs.anthropic.com/claude-code

---

## 📝 更新日志

| 日期 | 更新内容 |
|------|---------|
| 2026-03-28 | 创建初始文档（使用指南、Skills 索引、快速参考） |

---

## 💡 贡献

如果你发现文档有错误或想补充内容，请：
1. 编辑相应的 `.md` 文件
2. 更新本 README 的更新日志
3. 提交更改

---

**祝编码愉快！** 🎉
