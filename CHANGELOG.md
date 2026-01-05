# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [1.1.1] - 2026-01-05

### 文档更新

- 更新 README 添加智能更新功能详细说明
- 新增"更新到最新版本"独立章节
- 优化交互式菜单说明，分离首次安装和更新流程
- 在"最新更新"部分新增 v1.1.0 智能更新系统介绍

---

## [1.1.0] - 2026-01-05

### 新增功能

- **智能更新系统**：一键更新命令模板和提示词，无需卸载重装
  - 自动检测 npm 最新版本并对比当前版本
  - 增量更新，仅更新命令和提示词文件
  - 保留用户配置（`~/.ccg/config.toml`）
  - 支持强制重装，修复损坏的文件
  - 无需 sudo 权限

### 核心实现

- 新增 `src/utils/version.ts` - 版本管理工具
  - `getCurrentVersion()` - 获取当前安装版本
  - `getLatestVersion()` - 查询 npm 最新版本
  - `compareVersions()` - 语义化版本对比
  - `checkForUpdates()` - 检查是否有可用更新

- 新增 `src/commands/update.ts` - 更新命令实现
  - 交互式更新流程
  - 版本检测和对比
  - 强制重装选项

- 更新 `src/commands/menu.ts` - 菜单集成
  - 新增"更新工作流"选项
  - 移除复杂的备份管理功能

### 用户体验

- 运行 `npx ccg-workflow` 选择"更新工作流"即可更新
- 显示当前版本 vs 最新版本对比
- 自动更新所有文件并保留配置
- 提供友好的进度提示和错误处理

---

## [1.0.6] - 2026-01-05

### 修复

- 修复命令模板中的 MCP 工具参数缺失问题
- 在所有命令模板中添加 `mcp__ace-tool__search_context` 完整参数说明
- 在 enhance/dev 模板中添加 `mcp__ace-tool__enhance_prompt` 参数说明
- 更新 `_config.md` 中的提示词路径引用

---

## [1.0.5] - 2026-01-05

### 修复

- 修复安装时复制 CLAUDE.md 到用户目录的问题
- 斜杠命令已自包含完整工作流指令
- 避免覆盖用户已有的 `~/.claude/CLAUDE.md` 配置

---

## [1.0.4] - 2026-01-05

### 新增

- 补充 init-project 命令所需的两个 subagent
  - `init-architect.md` - 架构师子智能体
  - `planner.md` - 任务规划师

---

## [1.0.3] - 2026-01-05

### 新增

- 为所有多模型命令添加 codeagent-wrapper 调用示例
- 优化命令模板，明确使用方式

---

## [1.0.2] - 2026-01-05

### 优化

- 优化 token 消耗，改用子进程读取角色提示词文件
- 减少内存占用

---

## [1.0.1] - 2026-01-05

### 修复

- 修复命令模板调用方式
- 明确使用 codeagent-wrapper 的标准语法

---

## [1.0.0] - 2026-01-05

### 重大更新：npm 首次发布

#### 安装方式革命性升级

- ✅ 从 Python 脚本重构为 **TypeScript + unbuild** 构建系统
- ✅ 发布到 npm: `npx ccg-workflow` 一键安装
- ✅ 交互式配置菜单（初始化/卸载）
- ✅ 更好的跨平台兼容性

#### 三模型协作时代

- ✅ 从双模型 (Codex + Gemini) 扩展到 **三模型 (Claude + Codex + Gemini)**
- ✅ 新增 6 个 Claude 角色提示词（architect, analyzer, debugger, optimizer, reviewer, tester）
- ✅ 专家提示词从 12 个扩展到 **18 个**

#### 配置系统升级

- ✅ 配置文件从 `config.json` 迁移到 `~/.ccg/config.toml`
- ✅ 支持 **smart/parallel/sequential** 三种协作模式
- ✅ 可配置前端/后端模型优先级

#### 核心功能

**开发工作流（12个命令）**
- `/ccg:dev` - 完整6阶段三模型工作流
- `/ccg:code` - 三模型代码生成（智能路由）
- `/ccg:debug` - UltraThink 三模型调试
- `/ccg:test` - 三模型测试生成
- `/ccg:bugfix` - 质量门控修复（90%+ 通过）
- `/ccg:think` - 深度分析
- `/ccg:optimize` - 性能优化
- `/ccg:frontend` - 前端任务 → Gemini
- `/ccg:backend` - 后端任务 → Codex
- `/ccg:review` - 三模型代码审查
- `/ccg:analyze` - 三模型技术分析
- `/ccg:enhance` - Prompt 增强（ace-tool MCP）

**智能规划（2个命令）**
- `/ccg:scan` - 智能仓库扫描
- `/ccg:feat` - 智能功能开发

**Git 工具（4个命令）**
- `/ccg:commit` - 智能 commit（支持 emoji）
- `/ccg:rollback` - 交互式回滚
- `/ccg:clean-branches` - 清理已合并分支
- `/ccg:worktree` - Worktree 管理

**项目初始化（1个命令）**
- `/ccg:init` - 初始化项目 AI 上下文

#### 专家提示词系统

**18个角色文件**，动态角色注入：
- **Codex 角色**（6个）：architect, analyzer, debugger, tester, reviewer, optimizer
- **Gemini 角色**（6个）：frontend, analyzer, debugger, tester, reviewer, optimizer
- **Claude 角色**（6个）：architect, analyzer, debugger, tester, reviewer, optimizer

#### 技术栈

- **构建工具**: unbuild
- **编程语言**: TypeScript
- **CLI 框架**: cac
- **交互界面**: inquirer
- **配置格式**: TOML
- **国际化**: i18next

#### 依赖项

```json
{
  "ansis": "^4.1.0",
  "cac": "^6.7.14",
  "fs-extra": "^11.3.2",
  "i18next": "^25.5.2",
  "inquirer": "^12.9.6",
  "ora": "^9.0.0",
  "pathe": "^2.0.3",
  "smol-toml": "^1.4.2"
}
```

---

## [Pre-1.0.0] - Python 版本

### Python 安装脚本时代（已弃用）

使用 `python3 install.py` 进行安装，支持双模型协作（Codex + Gemini）。

**主要限制**：
- 需要手动 clone 仓库
- Python 环境依赖
- 配置不够灵活
- 更新需要重新安装

---

## 链接

- [GitHub Repository](https://github.com/fengshao1227/ccg-workflow)
- [npm Package](https://www.npmjs.com/package/ccg-workflow)
- [README](https://github.com/fengshao1227/ccg-workflow/blob/main/README.md)
