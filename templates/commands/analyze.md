---
description: 多模型技术分析（根据配置并行），交叉验证后综合见解
---

> 调用语法见 `_config.md`

## 用法
`/analyze <QUESTION_OR_TASK>`

## 上下文
- Question or analysis task: $ARGUMENTS
- This command triggers multi-model analysis without code changes.
- Configured models provide perspectives for cross-validation.

## 配置
**首先读取 `~/.claude/.ccg/config.toml` 获取模型配置**:
```toml
[routing.frontend]
models = ["gemini", "codex"]

[routing.backend]
models = ["codex", "gemini"]
```

分析任务使用 `routing.frontend.models` 和 `routing.backend.models` 的并集。

## 你的角色
You are the **Analysis Coordinator** orchestrating multi-model research. You direct:
1. **ace-tool** – for codebase context retrieval
2. **Configured Models** – for comprehensive multi-perspective analysis
3. **Claude (Self)** – for synthesizing insights

## 流程

### Step 1: 读取配置 + 上下文检索

1. **读取 `~/.claude/.ccg/config.toml`** 获取模型配置
2. 合并 `routing.frontend.models` 和 `routing.backend.models` 获取分析模型列表
3. 如果配置不存在，默认使用 `["codex", "gemini"]`
4. Call `mcp__ace-tool__search_context` to understand relevant code:
   - `project_root_path`: Project root directory absolute path
   - `query`: Natural language description of what to analyze
5. Identify key files, patterns, and architecture

### Step 2: 并行分析

**并行调用所有配置的分析模型**（使用 `run_in_background: true` 非阻塞执行）：

**调用方式**: 使用 `Bash` 工具调用 `codeagent-wrapper`

```bash
# Codex 技术分析
codeagent-wrapper --backend codex - $PROJECT_DIR <<'EOF'
ROLE_FILE: ~/.claude/.ccg/prompts/codex/analyzer.md

<TASK>
分析任务: {{分析问题或任务}}
Context: {{从 ace-tool 获取的相关代码}}
</TASK>

OUTPUT: Detailed analysis with recommendations.
EOF
```

```bash
# Gemini 技术分析
codeagent-wrapper --backend gemini - $PROJECT_DIR <<'EOF'
ROLE_FILE: ~/.claude/.ccg/prompts/gemini/analyzer.md

<TASK>
分析任务: {{分析问题或任务}}
Context: {{从 ace-tool 获取的相关代码}}
</TASK>

OUTPUT: Detailed analysis with recommendations.
EOF
```

遍历合并后的模型列表（去重），为每个模型动态生成上述调用，使用 `analyzer` 角色。

### Step 3: 交叉验证
使用 `TaskOutput` 获取所有任务的结果，然后：
1. Compare perspectives from all configured models
2. Identify agreements and disagreements
3. Evaluate trade-offs objectively
4. Weight opinions based on model strengths

### Step 4: 综合输出
Present unified analysis combining all perspectives.

## 输出格式
1. **Configuration** – models used for analysis
2. **Context Overview** – relevant codebase elements
3. **Model Perspectives** – analysis from each configured model
4. **Synthesis** – combined insights and trade-offs
5. **Recommendations** – actionable next steps

## 注意事项
- **首先读取 `~/.claude/.ccg/config.toml` 获取模型配置**
- This command is for analysis only, no code changes
- **Use `run_in_background: true` for parallel execution** to avoid blocking
- 多模型结果交叉验证，取长补短
- Use HEREDOC syntax (`<<'EOF'`) to avoid shell escaping issues
