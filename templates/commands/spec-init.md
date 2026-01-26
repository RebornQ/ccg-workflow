---
description: '初始化 OpenSpec (OPSX) 环境 + 验证多模型 MCP 工具'
---
<!-- CCG:SPEC:INIT:START -->
**Core Philosophy**
- OPSX provides the specification framework; CCG adds multi-model collaboration.
- This phase ensures all tools are ready before any development work begins.
- Fail fast: detect missing dependencies early rather than mid-workflow.

**Guardrails**
- Detect OS (Linux/macOS/Windows) and adapt commands accordingly.
- Do not proceed to next step until current step completes successfully.
- Provide clear, actionable error messages when a step fails.
- Respect user's existing configurations; avoid overwriting without confirmation.

**Steps**
1. **Detect Operating System**
   - Identify OS using `uname -s` (Unix) or environment variables (Windows).
   - Inform user which OS was detected.

2. **Check and Install OpenSpec**
   - Verify if `openspec` CLI is installed: `openspec --version`
   - If not installed or not found:
     ```bash
     npm install -g @fission-ai/openspec@latest
     ```
   - **Path Verification**:
     - If `openspec` command is not found after installation:
       1. Find npm global bin: `npm bin -g`
       2. Add to PATH temporarily for this session: `export PATH=$PATH:$(npm bin -g)`
       3. **CRITICAL**: Instruct user to add this path to their shell profile (e.g., `~/.zshrc`, `~/.bashrc`, or Windows Environment Variables) to make it permanent.
   - Confirm installation success by running `openspec --version` again.

3. **Initialize OpenSpec (OPSX) for Current Project**
   - Run:
     ```bash
     openspec init --tools claude
     ```
   - Verify `openspec/` directory structure is created.
   - Verify `.claude/skills/` contains `openspec-*` skills.
   - Report any errors with remediation steps.

4. **Validate Multi-Model MCP Tools**
   - Check `codeagent-wrapper` availability: `~/.claude/bin/codeagent-wrapper --version`
   - Test Codex backend:
     ```bash
     ~/.claude/bin/codeagent-wrapper --backend codex - "$PWD" <<< "echo test"
     ```
   - Test Gemini backend:
     ```bash
     ~/.claude/bin/codeagent-wrapper --backend gemini - "$PWD" <<< "echo test"
     ```
   - For each unavailable tool, display warning with installation instructions.

5. **Validate Context Retrieval MCP** (Optional)
   - **Check Active Tool**: Is `mcp__ace-tool__search_context` available in the current session?
   - **Check Configuration**: If tool is missing, check `~/.claude.json` (or `%APPDATA%\Claude\claude.json` on Windows) for `"ace-tool"` or `"ace-tool-rs"` in `mcpServers`.
   - **Diagnosis**:
     - If tool available: Mark as "✓ Active".
     - If config exists but tool missing: Mark as "⚠️ Configured but inactive (Try restarting Claude)".
     - If neither: Mark as "○ Not installed (Optional)".
   - If not installed/configured, suggest: "Run `npx ccg-workflow` and select ace-tool MCP option."

6. **Summary Report**
   Display status table:
   ```
   Component                 Status
   ─────────────────────────────────
   OpenSpec CLI              ✓/✗
   Project initialized       ✓/✗
   OPSX Skills               ✓/✗
   codeagent-wrapper         ✓/✗
   Codex backend             ✓/✗
   Gemini backend            ✓/✗
   ace-tool MCP              ✓/✗ (optional)
   ```

   **Next Steps (Use CCG Encapsulated Commands)**
   1. Start Research: `/ccg:spec-research "description"`
   2. Plan & Design: `/ccg:spec-plan`
   3. Implement: `/ccg:spec-impl` (Includes auto-review & archive)

   **Standalone Tools (Available Anytime)**
   - Code Review: `/ccg:spec-review` (Independent dual-model review)

**Reference**
- OpenSpec CLI: `openspec --help`
- CCG Workflow: `npx ccg-workflow`
- Codex/Gemini MCP: Bundled with codeagent-wrapper
- Node.js >= 18.x required for OpenSpec
<!-- CCG:SPEC:INIT:END -->
