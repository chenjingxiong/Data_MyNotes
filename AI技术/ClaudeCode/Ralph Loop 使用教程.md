# Ralph Loop 使用教程

## 什么是 Ralph Loop？

Ralph Loop 是基于 **Ralph Wiggum 技术** 的迭代开发方法论，由 Geoffrey Huntley 发明。它通过持续的 AI 循环来实现自动化迭代开发。

### 核心理念

```bash
while :; do
  cat PROMPT.md | claude-code --continue
done
```

**简单来说：** 将同一个提示词反复喂给 Claude，让它在每次迭代中看到自己之前的工作成果，不断改进直到完成。

### 工作原理

每次迭代时：
1. Claude 收到**相同的提示词**
2. 处理任务，修改文件
3. 尝试退出
4. Stop hook 拦截退出，再次喂入相同的提示词
5. Claude 在文件中看到自己之前的工作
6. 迭代改进直到完成

这个技术被描述为"在不确定世界中确定性失败"——失败是可预测的，可以通过提示词调优来实现系统性改进。

---

## 可用命令

### `/ralph-loop:ralph-loop` - 启动循环

启动当前会话中的 Ralph 循环。

**语法：**
```
/ralph-loop:ralph-loop "你的提示词" [选项]
```

**选项：**
- `--max-iterations <n>` - 最大迭代次数，达到后自动停止
- `--completion-promise <text>` - 完成承诺短语，检测到此内容时停止

**示例：**

```bash
# 基本用法
/ralph-loop:ralph-loop "重构缓存层"

# 设置最大迭代次数
/ralph-loop:ralph-loop "添加测试" --max-iterations 20

# 使用完成承诺
/ralph-loop:ralph-loop "修复认证模块" --completion-promise "修复完成"
```

**工作流程：**
1. 创建 `.claude/.ralph-loop.local.md` 状态文件
2. 你处理任务
3. 尝试退出时，stop hook 拦截
4. 相同提示词再次被喂入
5. 你看到之前的工作成果
6. 继续迭代直到检测到承诺或达到最大迭代次数

---

### `/ralph-loop:cancel-ralph` - 取消循环

取消当前激活的 Ralph 循环（删除循环状态文件）。

**语法：**
```
/ralph-loop:cancel-ralph
```

**效果：**
- 检查活动的循环状态文件
- 删除 `.claude/.ralph-loop.local.md`
- 报告取消状态和迭代计数

---

## 关键概念

### 完成承诺（Completion Promise）

要标记任务完成，Claude 需要输出 `<promise>` 标签：

```xml
<promise>任务完成</promise>
```

stop hook 会查找这个特定标签。如果没有这个标签（或没有设置 `--max-iterations`），Ralph 会无限运行。

### 自引用机制

"循环"并不意味着 Claude 与自己对话。它的意思是：
- 相同的提示词被重复
- Claude 的工作结果保存在文件中
- 每次迭代都能看到之前的尝试
- 逐步向目标构建

---

## 使用示例

### 示例 1：交互式 Bug 修复

```bash
/ralph-loop:ralph-loop "修复 auth.ts 中的 token 刷新逻辑。当所有测试通过时输出 <promise>FIXED</promise>" --completion-promise "FIXED" --max-iterations 10
```

**Ralph 会：**
1. 尝试修复
2. 运行测试
3. 发现失败
4. 迭代解决方案
5. 直到测试通过或达到最大迭代次数

### 示例 2：添加测试覆盖

```bash
/ralph-loop:ralph-loop "为 src/calculator.ts 添加完整的单元测试，确保覆盖所有函数。完成后输出 <promise>测试完成</promise>" --completion-promise "测试完成" --max-iterations 15
```

### 示例 3：重构代码

```bash
/ralph-loop:ralph-loop "重构 user.service.ts，提取重复代码到辅助函数，保持功能不变。完成后输出 <promise>重构完成</promise>" --completion-promise "重构完成"
```

---

## 何时使用 Ralph Loop

### ✅ 适合场景

- **明确定义的任务**，有清晰的成功标准
- 需要迭代和精炼的任务
- 需要自我纠正的迭代开发
- 绿地项目（全新项目）
- 自动化重复性工作

### ❌ 不适合场景

- 需要人类判断或设计决策的任务
- 一次性操作
- 成功标准不明确的任务
- 生产环境调试（应使用有针对性的调试）
- 需要创造性设计的任务

---

## 最佳实践

### 1. 编写清晰的提示词

```bash
# 好的提示词
/ralph-loop:ralph-loop "修复 API 调用中的错误处理。要求：1) 添加 try-catch 2) 记录错误 3) 返回友好消息。完成后输出 <promise>完成</promise>"

# 不好的提示词
/ralph-loop:ralph-loop "改进代码"
```

### 2. 设置合理的安全限制

```bash
# 总是设置最大迭代次数
/ralph-loop:ralph-loop "任务描述" --max-iterations 20

# 同时使用完成承诺和最大迭代
/ralph-loop:ralph-loop "任务描述" --completion-promise "完成" --max-iterations 50
```

### 3. 使用有意义的完成承诺

```bash
# 清晰的承诺
--completion-promise "所有测试通过"
--completion-promise "API 已完全重构"

# 避免模糊的承诺
--completion-promise "完成"  # 可能意外触发
```

### 4. 监控迭代进度

- 观察每次迭代的变化
- 如果发现错误模式，及时使用 `/ralph-loop:cancel-ralph` 取消
- 根据中间结果调整提示词

---

## 故障排查

### 循环没有停止

**原因：** 没有检测到完成承诺且没有设置最大迭代次数

**解决：**
```bash
# 取消当前循环
/ralph-loop:cancel-ralph

# 重新启动时添加限制
/ralph-loop:ralph-loop "任务" --max-iterations 20 --completion-promise "完成"
```

### 循环过早停止

**原因：** 完成承诺过于宽泛，意外触发

**解决：** 使用更具体、更独特的完成承诺短语

### 每次迭代没有改进

**原因：** 提示词可能不够明确，或者任务没有明确的成功标准

**解决：** 取消循环，重新编写更详细的提示词

---

## 相关资源

- **原始技术介绍：** https://ghuntley.com/ralph/
- **Ralph Orchestrator：** https://github.com/mikeyobrien/ralph-orchestrator

---

## 快速参考

| 命令 | 用途 |
|------|------|
| `/ralph-loop:ralph-loop "提示词"` | 启动基本循环 |
| `/ralph-loop:ralph-loop "提示" --max-iterations N` | 限制迭代次数 |
| `/ralph-loop:ralph-loop "提示" --completion-promise "文本"` | 使用完成承诺 |
| `/ralph-loop:cancel-ralph` | 取消活动循环 |

**记住：** Ralph Loop 是一个强大的迭代工具，但需要明确定义的任务和适当的限制才能发挥最大效用！
