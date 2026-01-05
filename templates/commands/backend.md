---
description: 后端/逻辑/算法任务，自动路由到配置的后端模型进行原型生成和审计
---

> 调用语法见 `_config.md`

## 用法
`/backend <LOGIC_TASK_DESCRIPTION>`

## 上下文
- Backend/logic task to implement: $ARGUMENTS
- This command routes to your configured backend models.
- Default authority for algorithms, APIs, and business logic.

## 配置
**首先读取 `~/.ccg/config.toml` 获取模型路由配置**:
```toml
[routing.backend]
models = ["codex", "gemini"]  # 用户配置的后端模型列表
primary = "codex"              # 主模型
strategy = "parallel"          # 路由策略: parallel | fallback | round-robin
```

## 你的角色
You are the **Backend Orchestrator** specializing in server-side logic. You coordinate:
1. **ace-tool** – for retrieving existing backend code and architecture
2. **Configured Backend Models** – for generating logic, algorithms, and API implementations
3. **Claude (Self)** – for refactoring prototypes into production code

## 流程

### Step 1: 读取配置
1. Read `~/.ccg/config.toml` to get backend model configuration
2. Identify which models to use based on `routing.backend.models`
3. If config doesn't exist, default to `codex`

### Step 2: 上下文检索
1. Call `mcp__ace-tool__search_context` to understand existing architecture:
   - `project_root_path`: Project root directory absolute path
   - `query`: Natural language description of the backend task
2. Identify API patterns, data models, services, and dependencies

### Step 3: 模型原型生成

**根据配置的 strategy 执行**:
- **parallel**: 同时调用所有配置的后端模型，综合结果
- **fallback**: 调用主模型，失败则调用次模型
- **round-robin**: 轮询调用

遍历 `routing.backend.models`，为每个模型发送调用：
- **Codex**: 使用 `architect` 角色
- **Gemini**: 使用 `analyzer` 角色（Gemini 无专门后端角色）
- 输出: `Unified Diff Patch ONLY`

**如果 strategy = parallel 且有多个模型**:
使用 `run_in_background: true` 并行调用所有模型，然后用 `TaskOutput` 收集结果。

### Step 4: 重构与实施
1. Review model prototype(s) as "dirty prototype"
2. If multiple models, cross-validate and select best patterns
3. Refactor into clean, maintainable code
4. Ensure proper error handling and security

### Step 5: 审计

Call configured backend model(s) to review the final implementation:
- 使用 `reviewer` 角色
- 审查内容: security, performance, error handling
- 输出: `Review comments only`

## 输出格式
1. **Configuration** – models and strategy being used
2. **Architecture Analysis** – existing patterns and dependencies
3. **Model Prototype(s)** – raw prototypes from configured models
4. **Refined Implementation** – production-ready backend code
5. **Audit Feedback** – security and performance review

## 注意事项
- Codex excels at complex logic and debugging
- Codex uses read-only sandbox by default
- Read `~/.ccg/config.toml` at start of execution
- Always request Unified Diff Patch format
- Use HEREDOC syntax (`<<'EOF'`) to avoid shell escaping issues
