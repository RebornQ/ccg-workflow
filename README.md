# CCG - Claude + Codex + Gemini 多模型协作系统

<div align="center">

**Claude Code 编排 Codex + Gemini 双模型协作的智能开发工作流系统**

[![npm version](https://img.shields.io/npm/v/ccg-workflow.svg)](https://www.npmjs.com/package/ccg-workflow)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Compatible-green.svg)](https://claude.ai/code)
[![Codex CLI](https://img.shields.io/badge/Codex%20CLI-Supported-orange.svg)](https://github.com/openai/openai-python)
[![Gemini CLI](https://img.shields.io/badge/Gemini%20CLI-Supported-purple.svg)](https://ai.google.dev/)

> **最新版本 v1.4.2** - 🎉 **Windows MCP 自动修复** - 彻底解决 Windows 用户 MCP 安装问题！

[快速开始](#-快速开始) • [命令参考](#-命令参考) • [常见问题](#-常见问题) • [更新日志](CHANGELOG.md)

</div>

---

## 🎯 v1.4.2 重大改进

### ✨ Windows 用户福音

**问题**：Windows 用户安装 MCP 后无法正常工作，需要手动设置环境变量

**解决**：从 [ZCF 项目](https://github.com/UfoMiao/zcf) 移植跨平台 MCP 配置逻辑

- ✅ **自动命令包装** - Windows 环境下 `npx` 自动包装为 `cmd /c npx`
- ✅ **零手动操作** - 用户无需设置环境变量或修改配置

### 技术细节

```json
// 修复前（Windows 不工作）
{
  "mcpServers": {
    "ace-tool": {
      "command": "npx",
      "args": ["-y", "ace-tool@latest"]
    }
  }
}

// 修复后（自动应用）
{
  "mcpServers": {
    "ace-tool": {
      "command": "cmd",
      "args": ["/c", "npx", "-y", "ace-tool@latest"]
    }
  }
}
```

---

## 📖 目录

- [核心理念](#-核心理念)
- [架构说明](#-架构说明)
- [核心特性](#-核心特性)
- [快速开始](#-快速开始)
- [命令参考](#-命令参考)
- [专家角色系统](#-专家角色系统)
- [配置文件](#-配置文件)
- [常见问题](#-常见问题)
- [致谢](#-致谢)

---

## 💡 核心理念

CCG = **Claude Code** (主导编排) + **Codex CLI** (后端原型) + **Gemini CLI** (前端原型)

### 设计哲学

让 Claude Code 专注于编排决策和代码实施，把具体的代码生成交给专业模型：
- **前端任务** → Gemini（视觉设计、组件原型）
- **后端任务** → Codex（逻辑运算、算法调试）
- **全栈整合** → Claude（工作流控制、代码主权）

### 核心优势

| 优势 | 说明 |
|-----|------|
| **智能路由** | 根据任务类型自动选择最合适的模型 |
| **交叉验证** | 双模型并行生成，相互验证减少错误 |
| **零写入权限** | 外部模型只能返回 Patch，Claude 保持代码主权 |
| **跨平台支持** | macOS、Linux、**Windows 自动修复** ✨ |
| **Token 优化** | ROLE_FILE 动态注入，专家提示词零 token 消耗 |

---

## 🏗️ 架构说明

```
┌─────────────────────────────────────────────────┐
│          Claude Code CLI (主导编排)              │
│        决策、编排、代码实施、质量把控             │
└──────────────┬──────────────────────────────────┘
               │
       ┌───────┴────────┐
       │                │
       ↓                ↓
┌─────────────┐  ┌─────────────┐
│  Codex CLI  │  │ Gemini CLI  │
│  后端原型   │  │  前端原型   │
│  逻辑算法   │  │  UI 组件    │
└─────────────┘  └─────────────┘
       │                │
       └────────┬───────┘
                ↓
      Unified Diff Patch
    (只读，不能直接修改文件)
```

### 安全机制

- **零写入权限**：Codex/Gemini 对文件系统无写入权限
- **Unified Diff**：所有外部模型输出必须为 Patch 格式
- **脏原型处理**：外部模型输出视为"脏原型"，需经 Claude 重构

---

## ✨ 核心特性

| 特性 | 描述 |
|------|------|
| **智能路由** | 前端任务→Gemini，后端任务→Codex，全栈整合→Claude |
| **双模型并行** | Codex ∥ Gemini 同时调用，交叉验证结果 |
| **MCP 自动配置** | **Windows 自动修复** + ace-tool/auggie 动态选择 |
| **6阶段工作流** | Prompt增强 → 代码检索 → 分析 → 原型 → 实施 → 审计 |
| **18个专家提示词** | Codex 6个 + Gemini 6个 + Claude 6个 |
| **Git 自动化** | 智能 commit、交互式回滚、分支清理、Worktree 管理 |
| **npx 一键安装** | 无需全局安装，交互式配置菜单 |
| **诊断工具** | **新增** `npx ccg diagnose-mcp` 和 `npx ccg fix-mcp` |

---

## 🚀 快速开始

### 前置要求

1. **必需**：
   - [Claude Code CLI](https://claude.ai/code) - 主导编排
   - Node.js 18+

2. **可选**（根据需求）：
   - [Codex CLI](https://github.com/openai/openai-codeinterpreter) - 后端任务
   - [Gemini CLI](https://github.com/google/generative-ai-cli) - 前端任务

### 一键安装

```bash
# 交互式安装
npx ccg-workflow

# 选择 "初始化工作流"
# 选择语言（中文 / English）
# 选择 MCP 工具（推荐 ace-tool）
# 等待安装完成（约 1-2 分钟）
# 重启终端
```

### 验证安装

```bash
# 检查 codeagent-wrapper
codeagent-wrapper --version

# 检查配置文件
cat ~/.claude/.ccg/config.toml

# 诊断 MCP 配置（v1.4.2 新增）
npx ccg diagnose-mcp
```

### 第一个命令

```bash
# 在 Claude Code 中执行
/ccg:dev 实现用户登录功能

# 系统会自动执行 6 阶段工作流：
# Phase 0: Prompt 增强 (MCP)
# Phase 1: 代码检索 (MCP)
# Phase 2: 多模型分析 (Codex ∥ Gemini 并行)
# Phase 3: 原型生成 (前端→Gemini / 后端→Codex)
# Phase 4: 代码实施 (Claude 重构为生产级)
# Phase 5: 审计交付 (Codex ∥ Gemini 交叉验证)
```

---

## 📚 命令参考

### 开发工作流命令

| 命令 | 用途 | 模型路由 |
|-----|------|---------|
| `/ccg:dev` | 完整6阶段开发工作流 | MCP + Codex + Gemini |
| `/ccg:code` | 智能代码生成（自动路由）| 前端→Gemini / 后端→Codex |
| `/ccg:frontend` | 前端/UI/样式任务 | Gemini |
| `/ccg:backend` | 后端/逻辑/算法任务 | Codex |
| `/ccg:debug` | UltraThink 多模型调试 | Codex + Gemini 并行 |
| `/ccg:test` | 多模型测试生成 | Codex + Gemini 并行 |
| `/ccg:bugfix` | 质量门控修复（90%+ 通过）| Codex + Gemini 交叉验证 |
| `/ccg:optimize` | 性能优化 | Codex + Gemini 并行 |
| `/ccg:review` | 代码审查（无参数自动审查 git diff）| Codex + Gemini 并行 |
| `/ccg:analyze` | 技术分析 | Codex + Gemini 并行 |
| `/ccg:think` | 深度分析 | Codex + Gemini 并行 |
| `/ccg:enhance` | Prompt 增强 | ace-tool MCP |
| `/ccg:scan` | 智能仓库扫描 | 分析项目结构 |
| `/ccg:feat` | 智能功能开发（规划→实施→审查）| 多模型协作 |

### Git 工具命令

| 命令 | 用途 |
|-----|------|
| `/ccg:commit` | 智能 commit：分析改动，生成 conventional commit 信息 |
| `/ccg:rollback` | 交互式回滚：列分支、列版本、二次确认 |
| `/ccg:clean-branches` | 分支清理：安全查找并清理已合并分支 |
| `/ccg:worktree` | Worktree 管理：在 `../.ccg/项目名/` 下创建 |

### CLI 诊断工具（v1.4.2 新增）

| 命令 | 用途 |
|-----|------|
| `npx ccg diagnose-mcp` | 诊断 MCP 配置问题 |
| `npx ccg fix-mcp` | 修复 Windows MCP 配置（Windows 用户）|

---

## 🎭 专家角色系统

### 核心机制：ROLE_FILE 动态注入

18个专家提示词（Codex 6个 + Gemini 6个 + Claude 6个），采用 **零 token 消耗** 的 ROLE_FILE 动态注入机制：

- ✅ 每个命令自动注入对应角色提示词
- ✅ 不占用主会话 token
- ✅ 无需手动配置全局提示词

### 角色映射表

| 命令 | Codex 角色 | Gemini 角色 |
|------|-----------|------------|
| `/ccg:code`, `/ccg:backend` | architect.md（后端架构师）| - |
| `/ccg:frontend` | - | frontend.md（前端架构师）|
| `/ccg:analyze`, `/ccg:think` | analyzer.md | analyzer.md |
| `/ccg:debug` | debugger.md | debugger.md |
| `/ccg:test` | tester.md | tester.md |
| `/ccg:review`, `/ccg:bugfix` | reviewer.md | reviewer.md |
| `/ccg:optimize` | optimizer.md | optimizer.md |

### 提示词文件结构

```
~/.claude/.ccg/prompts/          # v1.4.0+ 新位置
├── codex/         # Codex CLI 后端专家（6个）
│   ├── architect.md
│   ├── analyzer.md
│   ├── debugger.md
│   ├── optimizer.md
│   ├── reviewer.md
│   └── tester.md
├── gemini/        # Gemini CLI 前端专家（6个）
│   ├── frontend.md
│   ├── analyzer.md
│   ├── debugger.md
│   ├── optimizer.md
│   ├── reviewer.md
│   └── tester.md
└── claude/        # Claude CLI 全栈专家（6个）
    ├── architect.md
    ├── analyzer.md
    ├── debugger.md
    ├── optimizer.md
    ├── reviewer.md
    └── tester.md
```

---

## ⚙️ 配置文件

配置文件位于 `~/.claude/.ccg/config.toml`：

```toml
[general]
version = "1.4.2"
language = "zh-CN"

[mcp]
provider = "ace-tool"  # ace-tool | auggie | none

[routing]
mode = "smart"  # smart | parallel | sequential

[routing.frontend]
models = ["gemini"]
primary = "gemini"
strategy = "fallback"

[routing.backend]
models = ["codex"]
primary = "codex"
strategy = "fallback"
```

---

## 🗂️ 安装目录结构

```
~/.claude/
├── commands/ccg/           # ✅ Claude Code 读取的 slash commands
│   ├── dev.md, code.md, frontend.md, backend.md
│   ├── debug.md, test.md, bugfix.md, review.md
│   ├── optimize.md, analyze.md, think.md, enhance.md
│   ├── scan.md, feat.md, commit.md, rollback.md
│   ├── clean-branches.md, worktree.md, init.md
├── agents/ccg/             # ✅ Claude Code 读取的 subagents
│   ├── planner.md, ui-ux-designer.md
│   ├── init-architect.md, get-current-datetime.md
├── bin/                    # ✅ 二进制文件
│   └── codeagent-wrapper   # (Windows 自动包装 MCP 命令)
└── .ccg/                   # ✅ CCG 配置目录（v1.4.0+）
    ├── config.toml         # 主配置文件
    ├── shared-config.md    # 共享配置
    ├── backup/             # ✨ v1.4.2 新增：自动备份
    └── prompts/            # 专家提示词
        ├── codex/, gemini/, claude/
```

---

## ❓ 常见问题

<details>
<summary><strong>Q1: Windows 用户 MCP 安装后不工作怎么办？</strong></summary>

**v1.4.2 已自动修复！**

新用户：
```bash
npx ccg-workflow@latest init
# 安装时自动应用 Windows 修复
```

现有用户：
```bash
# 诊断问题
npx ccg diagnose-mcp

# 一键修复
npx ccg fix-mcp
```

手动验证：
```bash
# 检查配置是否正确
cat ~/.claude.json

# 应该看到 "command": "cmd", "args": ["/c", "npx", ...]
```

</details>

<details>
<summary><strong>Q2: 如何更新到 v1.4.2？</strong></summary>

一键更新，无需卸载：
```bash
npx ccg-workflow
# 选择 "更新工作流"
```

更新会自动：
- 检测 npm 最新版本
- 增量更新命令模板和提示词
- 保留用户配置和 MCP 设置
- 自动迁移旧版本目录结构（v1.3.x → v1.4.x）
- **应用 Windows MCP 修复**（v1.4.2）

</details>

<details>
<summary><strong>Q3: v1.4.0 目录迁移会影响我吗？</strong></summary>

**不会影响**，系统会自动迁移：

旧位置 → 新位置：
```
~/.ccg/                    → ~/.claude/.ccg/
~/.claude/prompts/ccg/     → ~/.claude/.ccg/prompts/
~/.claude/commands/ccg/    → 保持不变
```

安装/更新时会自动：
1. 检测旧版本目录
2. 迁移所有文件到新位置
3. 清理旧目录（安全检查后）
4. 显示迁移报告

</details>

<details>
<summary><strong>Q4: MCP 工具如何选择？</strong></summary>

**ace-tool**（推荐）：
- ✅ 开箱即用
- ✅ Prompt 增强 + 代码检索
- ✅ 无需额外配置
- 📖 获取 Token: https://augmentcode.com/

**auggie**（官方）：
- ✅ 代码检索（开箱即用）
- ⚠️ Prompt 增强需额外配置
- 📖 配置教程: https://linux.do/t/topic/1280612

切换方法：
```toml
# 编辑 ~/.claude/.ccg/config.toml
[mcp]
provider = "ace-tool"  # 或 "auggie"
```

</details>

<details>
<summary><strong>Q5: codeagent-wrapper 是什么？</strong></summary>

来自 [cexll/myclaude](https://github.com/cexll/myclaude) 的 Go 工具，封装了多 CLI 调用：
- 支持 `--backend codex/gemini/claude` 切换
- 会话管理（SESSION_ID）
- ROLE_FILE 动态注入
- **v1.4.2 新增**：Windows 命令自动包装

调用语法：
```bash
codeagent-wrapper --backend <codex|gemini|claude> - [工作目录] <<'EOF'
<任务内容>
EOF
```

</details>

<details>
<summary><strong>Q6: 安装后提示 "codeagent-wrapper: command not found"？</strong></summary>

**原因**：PATH 未生效。

**解决方案**：

Mac/Linux：
```bash
# 重启终端或执行
source ~/.zshrc
# 或
source ~/.bashrc
```

Windows：
```powershell
# 重新打开 PowerShell
# 或手动验证环境变量：
# %USERPROFILE%\.claude\bin
```

</details>

<details>
<summary><strong>Q7: 如何卸载 CCG 系统？</strong></summary>

```bash
npx ccg-workflow
# 选择 "卸载工作流"
```

卸载会：
- 删除 `~/.claude/commands/ccg/` 命令文件
- 删除 `~/.claude/agents/ccg/` 子智能体
- 删除 `~/.claude/bin/codeagent-wrapper` 二进制
- 删除 `~/.claude/.ccg/` 配置目录（可选保留）

</details>

---

## 🙏 致谢

感谢以下开源项目的贡献：

- **[cexll/myclaude](https://github.com/cexll/myclaude)** - `codeagent-wrapper` 多后端调用工具
- **[UfoMiao/zcf](https://github.com/UfoMiao/zcf)** - Git 工具 + **MCP 跨平台配置逻辑**（v1.4.2）
- **[GudaStudio/skills](https://github.com/GuDaStudio/skills)** - 智能路由设计理念
- **[ace-tool MCP](https://linux.do/t/topic/1344562)** - 轻量级代码检索和 Prompt 增强方案
- **[linux.do 社区](https://linux.do/)** - 活跃的 Vibe Coding 大型技术交流社区

---

## 💬 支持与反馈

- **GitHub Issues**: [提交问题](https://github.com/fengshao1227/ccg-workflow/issues)
- **讨论社区**: [linux.do - CCG 讨论帖](https://linux.do/t/topic/1405588)
- **完整文档**: [README.md](https://github.com/fengshao1227/ccg-workflow)

---

## 📄 许可证

本项目采用 [MIT License](LICENSE) 开源协议。

Copyright (c) 2025 fengshao1227

---

<div align="center">

**最后更新**: 2026-01-06 | **版本**: v1.4.2

Made with ❤️ by the CCG Community

</div>
