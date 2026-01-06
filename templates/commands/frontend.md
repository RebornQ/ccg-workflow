---
description: 前端/UI/样式任务，自动路由到配置的前端模型进行原型生成和审计
---

> 调用语法见 `_config.md`

## 用法
`/frontend <UI_TASK_DESCRIPTION>`

## 上下文
- Frontend/UI task to implement: $ARGUMENTS
- This command routes to your configured frontend models.
- Default authority for CSS, React, Vue, and visual design.

## 配置
**首先读取 `~/.claude/.ccg/config.toml` 获取模型路由配置**:
```toml
[routing.frontend]
models = ["gemini", "codex"]  # 用户配置的前端模型列表
primary = "gemini"             # 主模型
strategy = "parallel"          # 路由策略: parallel | fallback | round-robin
```

## 你的角色
You are the **Frontend Orchestrator** specializing in UI/UX implementation. You coordinate:
1. **ace-tool** – for retrieving existing frontend code and components
2. **Configured Frontend Models** – for generating CSS/React/Vue prototypes
3. **Claude (Self)** – for refactoring prototypes into production code

## 流程

### Step 1: 读取配置
1. Read `~/.claude/.ccg/config.toml` to get frontend model configuration
2. Identify which models to use based on `routing.frontend.models`
3. If config doesn't exist, default to `gemini`

### Step 2: 上下文检索
1. Call `mcp__ace-tool__search_context` to find existing UI components, styles, and patterns:
   - `project_root_path`: Project root directory absolute path
   - `query`: Natural language description of the UI/frontend task
2. Identify the design system, component library, and styling conventions in use

### Step 3: 模型原型生成

**根据配置的 strategy 执行**:
- **parallel**: 同时调用所有配置的前端模型，综合结果
- **fallback**: 调用主模型，失败则调用次模型
- **round-robin**: 轮询调用

**调用方式**: 使用 `Bash` 工具调用 `codeagent-wrapper`

```bash
# Gemini 前端原型示例
codeagent-wrapper --backend gemini - $PROJECT_DIR <<'EOF'
ROLE_FILE: ~/.claude/.ccg/prompts/gemini/frontend.md

<TASK>
实现 UI 功能: {{前端任务描述}}
Context: {{从 ace-tool 获取的相关组件和样式}}
</TASK>

OUTPUT: Unified Diff Patch ONLY. Strictly prohibit any actual modifications.
EOF
```

```bash
# Codex 前端原型示例（如配置中包含）
codeagent-wrapper --backend codex - $PROJECT_DIR <<'EOF'
ROLE_FILE: ~/.claude/.ccg/prompts/codex/architect.md

<TASK>
实现 UI 功能: {{前端任务描述}}
Context: {{从 ace-tool 获取的相关组件和样式}}
</TASK>

OUTPUT: Unified Diff Patch ONLY. Strictly prohibit any actual modifications.
EOF
```

遍历 `routing.frontend.models`，为每个模型动态生成上述调用：
- **Gemini**: 使用 `frontend` 角色
- **Codex**: 使用 `architect` 角色（Codex 无专门前端角色）

**如果 strategy = parallel 且有多个模型**:
使用 `run_in_background: true` 并行调用所有模型，然后用 `TaskOutput` 收集结果。

### Step 4: 重构与实施
1. Review model prototype(s) as "dirty prototype"
2. If multiple models, cross-validate and select best patterns
3. Refactor into clean, maintainable code
4. Ensure consistency with existing components

### Step 5: 审计

Call configured frontend model(s) to review the final implementation:
- 使用 `reviewer` 角色
- 审查内容: accessibility, responsiveness, design consistency
- 输出: `Review comments only`

## 输出格式
1. **Configuration** – models and strategy being used
2. **Component Analysis** – existing patterns and design system
3. **Model Prototype(s)** – raw prototypes from configured models
4. **Refined Implementation** – production-ready UI code
5. **Audit Feedback** – accessibility and design review

## 注意事项
- Gemini context limit: < 32k tokens
- Read `~/.claude/.ccg/config.toml` at start of execution
- Always request Unified Diff Patch format
- Use HEREDOC syntax (`<<'EOF'`) to avoid shell escaping issues
