---
description: 完整6阶段多模型协作工作流（Prompt增强 → 上下文检索 → 多模型分析 → 原型生成 → 代码实施 → 审计交付）
---

> 调用语法见 `_config.md`

## 用法
`/dev <功能描述>`

## 上下文
- 要实现的功能: $ARGUMENTS
- 此命令触发完整的 6 阶段多模型协作工作流
- 根据 `~/.ccg/config.toml` 配置路由模型

## 配置
**首先读取 `~/.ccg/config.toml` 获取模型路由配置**:
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

## 你的角色
你是**编排者**，协调三模型协作系统。你指挥:
1. **ace-tool** – 用于 Prompt 增强和代码库上下文检索
2. **Codex** – 后端逻辑、算法、调试专家
3. **Gemini** – 前端 UI/UX、视觉设计专家
4. **Claude (子进程)** – 全栈整合、契约设计、跨层问题
5. **Claude (自己)** – 编排、重构、最终交付

## 流程

### 阶段 0: 读取配置 + Prompt 增强
1. **读取 `~/.ccg/config.toml`** 获取模型路由配置
2. 如果配置不存在，使用默认值：frontend=gemini, backend=codex
3. 调用 `mcp__ace-tool__enhance_prompt` 优化原始需求:
   - `prompt`: 用户的原始需求 ($ARGUMENTS)
   - `conversation_history`: 最近的对话历史(5-10轮对话)
   - `project_root_path`: 当前项目根目录绝对路径
4. 向用户展示原始和增强后的 prompt:

```
📝 原始需求:
<原始需求>

✨ 增强后需求:
<增强后需求>

🔧 模型配置:
- 前端模型: <routing.frontend.models>
- 后端模型: <routing.backend.models>
- 协作模式: <routing.mode>

---
**使用增强后的需求继续？(Y/N)**
```

5. **强制停止**: 等待用户确认
   - 如果 Y: 后续阶段使用增强后的 prompt
   - 如果 N: 使用原始 prompt 或要求修改

### 阶段 1: 上下文检索
1. 调用 `mcp__ace-tool__search_context` 获取（增强后的）需求相关代码:
   - `project_root_path`: 项目根目录绝对路径
   - `query`: 增强后的需求描述
2. 识别所有相关文件、类、函数和依赖
3. 如需求仍不清晰，提出澄清问题

### 阶段 2: 多模型分析

**根据配置并行调用模型进行分析**（使用 `run_in_background: true` 非阻塞执行）：

根据 `routing.backend.models` 和 `routing.frontend.models` 动态生成调用：
- **后端模型**: 使用 `analyzer` 角色，输出实现方案
- **前端模型**: 使用 `analyzer` 角色，输出实现方案

然后使用 `TaskOutput` 获取所有任务的结果，交叉验证后综合方案。

**强制停止**: 询问用户 **"是否继续执行此方案？(Y/N)"** 并等待确认

### 阶段 3: 原型生成

**三模型并行生成原型**（使用 `run_in_background: true`）：

根据 `routing.prototype.models` 配置，同时调用三个模型：
- **Codex** + `architect` 角色 → 后端架构视角的原型
- **Gemini** + `frontend` 角色 → 前端 UI 视角的原型
- **Claude** + `architect` 角色 → 全栈整合视角的原型

输出: `Unified Diff Patch ONLY`

使用 `TaskOutput` 收集三个模型的结果。

**三模型差异化价值**：
| 模型 | 专注点 | 独特贡献 |
|------|--------|----------|
| Codex | 后端逻辑、算法 | 深度后端专业知识 |
| Gemini | 前端 UI、样式 | 视觉设计和用户体验 |
| Claude | 全栈整合、契约 | 桥接前后端视角 |

### 阶段 4: 代码实施
1. 将三个原型视为"脏原型" – 仅作参考
2. **交叉验证三模型结果，集各家所长**：
   - Codex 的后端逻辑优势
   - Gemini 的前端设计优势
   - Claude 的整合视角优势
3. 重构为干净的生产级代码
4. 验证变更不会引入副作用

### 阶段 5: 审计与交付

**三模型并行代码审查**（使用 `run_in_background: true`）：

根据 `routing.review.models` 配置调用所有模型：
- **Codex** + `reviewer` 角色 → 安全性、性能、错误处理
- **Gemini** + `reviewer` 角色 → 可访问性、响应式设计、设计一致性
- **Claude** + `reviewer` 角色 → 集成正确性、契约一致性、可维护性

输出: `Review comments only`

使用 `TaskOutput` 获取所有审查结果，整合三方反馈后修正并交付。

## 输出格式
1. **配置信息** – 使用的模型和路由策略
2. **增强后需求** – 优化后的 prompt (阶段 0)
3. **上下文摘要** – 识别的相关代码元素
4. **实施方案** – 含模型路由的逐步方案
5. **代码变更** – 生产级实现
6. **审计报告** – 审查反馈和修正
7. **后续步骤** – 部署或跟进操作

## 关键规则
- 未经用户批准不得跳过任何阶段
- **首先读取 `~/.ccg/config.toml` 获取模型配置**
- **阶段 0 的 prompt 增强是强制性的** – 必须先展示增强后的 prompt
- 始终要求外部模型输出 Unified Diff Patch
- 外部模型对文件系统**零写入权限**
- 实时向用户报告当前阶段和下一阶段
- 使用 HEREDOC 语法 (`<<'EOF'`) 避免 shell 转义问题
- **三模型并行调用使用 `run_in_background: true`** 避免阻塞
- **三模型结果需交叉验证，集各家所长**
