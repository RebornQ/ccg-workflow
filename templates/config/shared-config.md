---
description: CCG 多模型协作系统 - 共享配置和调用规范
---

## 配置文件
路径: `~/.claude/.ccg/config.toml`

```toml
[routing]
mode = "smart"  # smart | parallel | sequential

[routing.frontend]
models = ["gemini", "claude", "codex"]  # 三模型并行
primary = "gemini"
strategy = "parallel"

[routing.backend]
models = ["codex", "claude", "gemini"]  # 三模型并行
primary = "codex"
strategy = "parallel"

[routing.review]
models = ["codex", "gemini", "claude"]  # 三模型交叉验证
strategy = "parallel"

[routing.prototype]
models = ["codex", "gemini", "claude"]  # 三模型原型生成
strategy = "parallel"
```

默认值: frontend=`gemini`, backend=`codex`, prototype=`["codex","gemini","claude"]`

---

## 调用语法模板

**⚠️ 重要**: 使用 `Bash` 工具调用 `codeagent-wrapper`，**不要**使用 `/collaborating-with-codex` 或 `/collaborating-with-gemini` 等旧的 Skill 方式！

### 基础模式
```bash
codeagent-wrapper --backend <MODEL> - $PROJECT_DIR <<'EOF'
ROLE_FILE: ~/.claude/.ccg/prompts/<model>/<role>.md

<TASK>
{{任务描述}}

Context:
{{相关代码}}
</TASK>

OUTPUT: {{输出格式}}
EOF
```

**说明**: 使用 `ROLE_FILE:` 指定提示词文件路径，让子进程自己读取，避免消耗主会话 token。

### 角色映射
| 任务类型 | Codex 角色 | Gemini 角色 | Claude 角色 |
|---------|-----------|-------------|-------------|
| 架构/后端 | `codex/architect.md` | `gemini/analyzer.md` | `claude/architect.md` |
| 前端/UI | `codex/architect.md` | `gemini/frontend.md` | `claude/architect.md` |
| 分析 | `codex/analyzer.md` | `gemini/analyzer.md` | `claude/analyzer.md` |
| 审查 | `codex/reviewer.md` | `gemini/reviewer.md` | `claude/reviewer.md` |
| 调试 | `codex/debugger.md` | `gemini/debugger.md` | `claude/debugger.md` |
| 测试 | `codex/tester.md` | `gemini/tester.md` | `claude/tester.md` |
| 优化 | `codex/optimizer.md` | `gemini/optimizer.md` | `claude/optimizer.md` |

### 三模型差异化定位
| 模型 | 专长领域 | 独特价值 |
|------|---------|----------|
| **Codex** | 后端逻辑、算法、调试 | 深度后端专业知识 |
| **Gemini** | 前端 UI、CSS、组件设计 | 视觉设计和用户体验 |
| **Claude** | 全栈整合、契约设计、跨层问题 | 桥接前后端视角 |

### 输出格式
| 任务类型 | OUTPUT 值 |
|---------|----------|
| 原型生成 | `Unified Diff Patch ONLY. Strictly prohibit any actual modifications.` |
| 代码审查 | `Review comments only. No code modifications.` |
| 分析诊断 | `Structured analysis/diagnostic report.` |

### 执行策略
| 策略 | 说明 |
|------|------|
| `parallel` | `run_in_background: true` 并行调用，`TaskOutput` 收集结果 |
| `fallback` | 主模型失败则调用次模型 |
| `round-robin` | 轮询调用 |
