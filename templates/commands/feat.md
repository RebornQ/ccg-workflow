---
name: ccg:feat
description: 🚀 智能功能开发 - 自动规划、设计、实施（支持需求规划/讨论迭代/执行实施三种模式）
tools: Task, mcp__ace-tool__search_context, Read, Write, Bash, AskUserQuestion, Glob
---

# CCG 智能功能开发

智能识别用户输入类型，自动选择工作流程：
- **需求规划**：完整的需求分析 → 任务分解 → 生成计划文档
- **讨论迭代**：基于现有计划，调整优化
- **执行实施**：按照计划文档，调用多模型实施开发

---

## 核心能力

1. **智能输入识别**：自动判断用户意图（规划/迭代/实施）
2. **多模型路由**：前端→Gemini，后端→Codex，全栈→并行
3. **计划版本控制**：`.claude/plan/功能名.md` → `.claude/plan/功能名-1.md` → ...
4. **自动 Agent 调用**：根据任务类型调用 planner / ui-ux-designer

---

## 执行流程

### 步骤 1：输入类型识别

分析用户输入，识别关键词并分类：

#### 1.1 需求规划模式（默认）

**关键词**：
- 实现、开发、新增、添加功能、构建
- 需求、设计、规划
- "我想要..."、"帮我实现..."

**示例**：
- "实现用户登录功能"
- "开发一个商品管理模块"
- "添加实时聊天功能"

→ 执行完整规划流程（步骤 2）

#### 1.2 讨论迭代模式

**关键词**：
- 调整、修改、优化、改进、更新
- 重新规划、重新设计
- "计划需要..."、"方案需要..."
- 包含计划文件路径（`.claude/plan/xxx.md`）

**示例**：
- "调整用户登录功能的方案"
- "优化商品管理模块的计划"
- "修改 .claude/plan/登录功能.md 的任务拆分"

→ 读取现有计划，调用 planner 重新规划（步骤 2.2）

#### 1.3 执行实施模式

**关键词**：
- 开始实施、执行计划、按照计划
- 实施、开发、编码
- "根据计划..."、"按照 .claude/plan/xxx.md"

**示例**：
- "开始实施用户登录功能"
- "按照计划执行开发"
- "根据 .claude/plan/登录功能.md 开始编码"

→ 直接进入实施阶段（步骤 3）

---

### 步骤 2：需求规划流程（仅"需求规划"模式）

#### 2.1 检索项目上下文

##### 2.1.1 检查是否已有 repo-context.md

```bash
# 检查文件是否存在
if [ -f ".claude/repo-context.md" ]; then
  echo "✓ 检测到项目上下文报告"
  Read(".claude/repo-context.md")
else
  echo "⚠ 未检测到项目上下文，建议先运行 /ccg:scan"
  # 询问用户是否立即扫描
  AskUserQuestion {
    question: "是否立即扫描项目以获取上下文？",
    options: [
      { label: "是，立即扫描（推荐）", description: "运行 /ccg:scan 生成上下文报告" },
      { label: "跳过，直接规划", description: "基于现有知识进行规划" }
    ]
  }
fi
```

##### 2.1.2 使用 ace-tool 检索相关代码

```bash
mcp__ace-tool__search_context {
  "project_root_path": "$PROJECT_DIR",
  "query": "{{用户需求关键词}} + 相关的代码实现、API 接口、组件、数据模型"
}
```

**示例查询**：
- 用户需求："实现用户登录功能"
- ace-tool 查询："用户认证、登录、JWT token、Session 管理、认证中间件"

**期望结果**：
- 现有认证代码位置
- 可复用的组件
- 数据库 User 模型
- API 路由结构

---

#### 2.2 任务类型判断

基于用户需求和检索结果，判断任务类型：

| 任务类型 | 判断依据 | 调用流程 |
|----------|----------|----------|
| **前端任务** | 关键词：页面、组件、UI、样式、布局 | ui-ux-designer → planner |
| **后端任务** | 关键词：API、接口、数据库、逻辑、算法 | planner |
| **全栈任务** | 同时包含前后端需求 | ui-ux-designer → planner |

---

#### 2.3 调用 ui-ux-designer agent（前端/全栈任务）

如果是前端或全栈任务，先调用 UI/UX 设计师：

```bash
Task(subagent_type="general-purpose", description="调用 ui-ux-designer agent") {
  prompt: "
执行以下 agent：
~/.claude/commands/ccg/agents/ui-ux-designer.md

项目上下文：
{{ace-tool 检索结果}}

用户需求：
{{原始需求}}

技术栈信息：
{{从 repo-context.md 提取的技术栈}}

输出：UI/UX 设计方案文档（Markdown 格式）
"
}
```

**输出示例**：
```markdown
# UI/UX 设计方案：用户登录页面

## 1. 页面结构
...

## 2. 组件拆分
- LoginPage.tsx
- LoginForm.tsx
- FormInput.tsx
...

## 3. 交互流程
...
```

---

#### 2.4 调用 planner agent

将所有上下文信息传递给 planner：

```bash
Task(subagent_type="general-purpose", description="调用 planner agent") {
  prompt: "
执行以下 agent：
~/.claude/commands/ccg/agents/planner.md

项目上下文：
{{ace-tool 检索结果}}

{{如果是前端/全栈任务，附加 UI/UX 设计方案}}
{{UI/UX 设计方案内容}}

用户需求：
{{原始需求}}

技术栈信息：
{{从 repo-context.md 提取}}

输出：
请生成详细的功能规划文档，并按照以下路径保存：
.claude/plan/{{功能名}}.md
"
}
```

**planner 输出格式**：
```markdown
# 功能规划：用户登录功能

## 1. 功能概述
...

## 2. WBS 任务分解
...

## 3. 依赖关系
...

## 4. 实施建议
...
```

---

#### 2.5 计划版本控制

##### 首次规划

```bash
# 生成计划文件名（基于用户需求）
FEATURE_NAME="{{从用户需求提取的功能名}}"
PLAN_FILE=".claude/plan/${FEATURE_NAME}.md"

# 检查是否已存在
if [ -f "$PLAN_FILE" ]; then
  echo "⚠ 计划文件已存在：$PLAN_FILE"
  echo "将创建新版本：${FEATURE_NAME}-1.md"
  PLAN_FILE=".claude/plan/${FEATURE_NAME}-1.md"
fi

# 保存计划
Write("$PLAN_FILE", "{{planner 输出}}")
```

##### 讨论迭代（调整现有计划）

```bash
# 用户输入："调整用户登录功能的方案"
# 识别现有计划文件
EXISTING_PLAN=$(find .claude/plan -name "*登录*.md" -o -name "*login*.md" | head -n 1)

if [ -n "$EXISTING_PLAN" ]; then
  echo "✓ 找到现有计划：$EXISTING_PLAN"

  # 读取现有计划
  EXISTING_CONTENT=$(Read("$EXISTING_PLAN"))

  # 生成新版本号
  BASE_NAME=$(basename "$EXISTING_PLAN" .md)
  if [[ $BASE_NAME =~ -([0-9]+)$ ]]; then
    VERSION=$((${BASH_REMATCH[1]} + 1))
    NEW_PLAN=".claude/plan/${BASE_NAME%-*}-${VERSION}.md"
  else
    NEW_PLAN=".claude/plan/${BASE_NAME}-1.md"
  fi

  # 调用 planner 重新规划
  Task(subagent_type="general-purpose") {
    prompt: "
基于现有计划进行调整：

【现有计划】
$EXISTING_CONTENT

【用户调整要求】
{{用户的调整需求}}

【任务】
请根据用户要求，重新规划并生成新的计划文档。
保持合理的部分，修改需要调整的部分。
输出到：$NEW_PLAN
"
  }
else
  echo "⚠ 未找到相关计划文件，将创建新计划"
  # 回退到首次规划流程
fi
```

---

#### 2.6 交互确认

计划生成后，询问用户下一步操作：

```bash
AskUserQuestion {
  question: "功能规划已完成，请选择下一步操作：",
  header: "下一步",
  options: [
    {
      label: "开始实施（推荐）",
      description: "根据计划调用多模型开始开发"
    },
    {
      label: "讨论调整",
      description: "我想修改计划中的某些部分"
    },
    {
      label: "重新规划",
      description: "推翻当前方案，重新开始"
    },
    {
      label: "仅保存计划",
      description: "暂时不实施，稍后手动执行"
    }
  ]
}
```

**用户选择处理**：
- **开始实施** → 进入步骤 3
- **讨论调整** → 提示用户描述调整需求，重新执行步骤 2.4
- **重新规划** → 删除当前计划，重新执行步骤 2
- **仅保存计划** → 退出，提示计划文件路径

---

### 步骤 3：执行实施（用户确认后）

#### 3.1 读取计划文档

```bash
# 确定计划文件路径
if [[ "$USER_INPUT" =~ \.claude/plan/.*\.md ]]; then
  # 用户明确指定了计划文件
  PLAN_FILE="{{用户指定的路径}}"
else
  # 查找最新的计划文件
  PLAN_FILE=$(ls -t .claude/plan/*.md | head -n 1)
fi

# 读取计划内容
PLAN_CONTENT=$(Read("$PLAN_FILE"))

echo "✓ 读取计划：$PLAN_FILE"
```

---

#### 3.2 任务类型分析

从计划文档中提取任务分类：

```bash
# 从 WBS 任务分解中提取模块类型
FRONTEND_TASKS=$(echo "$PLAN_CONTENT" | grep -E "#### 模块.*：前端" || echo "")
BACKEND_TASKS=$(echo "$PLAN_CONTENT" | grep -E "#### 模块.*：后端" || echo "")

if [ -n "$FRONTEND_TASKS" ] && [ -n "$BACKEND_TASKS" ]; then
  TASK_TYPE="fullstack"
elif [ -n "$FRONTEND_TASKS" ]; then
  TASK_TYPE="frontend"
elif [ -n "$BACKEND_TASKS" ]; then
  TASK_TYPE="backend"
else
  # 无法判断，询问用户
  AskUserQuestion {
    question: "请选择任务类型：",
    options: [
      { label: "前端任务", description: "主要涉及 UI/组件/样式" },
      { label: "后端任务", description: "主要涉及 API/逻辑/数据库" },
      { label: "全栈任务", description: "同时涉及前后端" }
    ]
  }
fi
```

---

#### 3.3 多模型路由实施

##### 3.3.1 前端任务 → Gemini

```bash
if [ "$TASK_TYPE" == "frontend" ] || [ "$TASK_TYPE" == "fullstack" ]; then
  echo "🎨 前端任务 → 路由到 Gemini"

  # 提取前端相关任务
  FRONTEND_PLAN=$(echo "$PLAN_CONTENT" | sed -n '/#### 模块.*：前端/,/#### 模块.*：/p')

  # 调用 codeagent-wrapper
  codeagent-wrapper --backend gemini - "$PROJECT_DIR" <<'EOF'
ROLE_FILE: ~/.claude/.ccg/prompts/gemini/frontend.md

<PROJECT_CONTEXT>
{{从 .claude/repo-context.md 提取的前端技术栈}}
</PROJECT_CONTEXT>

<TASK>
根据以下计划，实施前端功能：

$FRONTEND_PLAN

要求：
1. 创建/修改所有需要的组件文件
2. 确保组件符合项目现有的代码风格
3. 实现响应式设计（Mobile / Tablet / Desktop）
4. 添加必要的类型定义（TypeScript）
5. 处理 Loading / Success / Error 状态
6. 遵循无障碍访问（A11y）最佳实践
</TASK>

<OUTPUT>
完成后请提供：
1. 创建/修改的文件清单
2. 关键代码片段说明
3. 使用方法和示例
</OUTPUT>
EOF

  # 捕获 SESSION_ID
  GEMINI_SESSION=$(echo "$OUTPUT" | grep "SESSION_ID:" | awk '{print $2}')
  echo "✓ Gemini 前端实施完成 (Session: $GEMINI_SESSION)"
fi
```

##### 3.3.2 后端任务 → Codex

```bash
if [ "$TASK_TYPE" == "backend" ] || [ "$TASK_TYPE" == "fullstack" ]; then
  echo "⚙️ 后端任务 → 路由到 Codex"

  # 提取后端相关任务
  BACKEND_PLAN=$(echo "$PLAN_CONTENT" | sed -n '/#### 模块.*：后端/,/#### 模块.*：/p')

  # 调用 codeagent-wrapper
  codeagent-wrapper --backend codex - "$PROJECT_DIR" <<'EOF'
ROLE_FILE: ~/.claude/.ccg/prompts/codex/architect.md

<PROJECT_CONTEXT>
{{从 .claude/repo-context.md 提取的后端技术栈}}
</PROJECT_CONTEXT>

<TASK>
根据以下计划，实施后端功能：

$BACKEND_PLAN

要求：
1. 创建/修改所有需要的 API 端点
2. 实现数据验证（使用 Zod / Joi / 等）
3. 处理错误和异常（统一错误格式）
4. 添加必要的中间件（认证、日志、CORS）
5. 编写数据库查询（ORM / 原生 SQL）
6. 确保代码可测试性（依赖注入、接口抽象）
</TASK>

<OUTPUT>
完成后请提供：
1. 创建/修改的文件清单
2. API 接口文档（请求/响应格式）
3. 数据库 Schema 变更（如有）
4. 测试用例建议
</OUTPUT>
EOF

  # 捕获 SESSION_ID
  CODEX_SESSION=$(echo "$OUTPUT" | grep "SESSION_ID:" | awk '{print $2}')
  echo "✓ Codex 后端实施完成 (Session: $CODEX_SESSION)"
fi
```

##### 3.3.3 全栈任务 → 并行调用

```bash
if [ "$TASK_TYPE" == "fullstack" ]; then
  echo "🔗 全栈任务 → 并行调用 Gemini + Codex"

  # 前端和后端任务并行执行（使用后台进程）
  {
    # 前端任务（见 3.3.1）
    codeagent-wrapper --backend gemini - "$PROJECT_DIR" <<'EOF'
...
EOF
  } &

  {
    # 后端任务（见 3.3.2）
    codeagent-wrapper --backend codex - "$PROJECT_DIR" <<'EOF'
...
EOF
  } &

  # 等待两个任务完成
  wait

  echo "✓ 前后端并行实施完成"
fi
```

---

#### 3.4 实施后验证

##### 3.4.1 检查实施结果

```bash
# 读取 git 状态
git status --short

# 提示用户查看变更
echo ""
echo "📝 实施完成！变更文件："
git diff --name-status

# 询问用户是否需要审查
AskUserQuestion {
  question: "是否运行代码审查？",
  options: [
    { label: "是，运行 /ccg:review", description: "双模型代码审查" },
    { label: "跳过，手动检查", description: "稍后自行审查代码" }
  ]
}
```

##### 3.4.2 调用代码审查（可选）

```bash
if [ "$USER_CHOICE" == "是，运行 /ccg:review" ]; then
  echo "🔍 启动代码审查..."

  # 调用 ccg:review 命令
  /ccg:review
fi
```

---

### 步骤 4：后续优化建议

实施完成后，提供后续步骤建议：

```bash
echo ""
echo "✅ 功能开发完成！"
echo ""
echo "📋 后续步骤建议："
echo "  1. 运行测试：npm run test"
echo "  2. 启动开发服务器：npm run dev"
echo "  3. 验证功能是否正常工作"
echo "  4. 运行 /ccg:commit 提交代码"
echo ""
echo "📂 相关文件："
echo "  • 计划文档：$PLAN_FILE"
echo "  • 项目上下文：.claude/repo-context.md"
echo ""
```

---

## 命令参数（可选）

支持通过参数快速指定模式：

```bash
# 显式指定规划模式
/ccg:feat --mode=plan "实现用户登录功能"

# 显式指定实施模式
/ccg:feat --mode=implement ".claude/plan/用户登录功能.md"

# 显式指定迭代模式
/ccg:feat --mode=iterate ".claude/plan/用户登录功能.md" "调整密码验证逻辑"

# 指定任务类型（跳过自动判断）
/ccg:feat --type=frontend "实现登录页面"
/ccg:feat --type=backend "实现登录 API"
/ccg:feat --type=fullstack "实现完整登录功能"
```

---

## 使用示例

### 示例 1：从需求到实施（完整流程）

```bash
# 第 1 步：用户输入
/ccg:feat 实现用户登录功能

# 系统识别：需求规划模式

# 第 2 步：检索上下文
✓ 检测到项目上下文报告
✓ 使用 ace-tool 检索相关代码...
  - 找到现有 User 模型
  - 找到认证中间件模板

# 第 3 步：任务类型判断
✓ 识别为全栈任务

# 第 4 步：调用 ui-ux-designer
✓ UI/UX 设计方案已生成

# 第 5 步：调用 planner
✓ 功能规划已生成：.claude/plan/用户登录功能.md

# 第 6 步：交互确认
? 功能规划已完成，请选择下一步操作：
  > 开始实施（推荐）
    讨论调整
    重新规划
    仅保存计划

# 用户选择：开始实施

# 第 7 步：多模型实施
🎨 前端任务 → 路由到 Gemini
⚙️ 后端任务 → 路由到 Codex

✓ 前端实施完成 (Session: 019a7247-...)
✓ 后端实施完成 (Session: 019a7248-...)

# 第 8 步：实施后验证
📝 实施完成！变更文件：
  M  app/api/auth/login/route.ts
  A  app/login/page.tsx
  A  components/LoginForm.tsx
  M  lib/auth.ts

? 是否运行代码审查？
  > 是，运行 /ccg:review
    跳过，手动检查

# 完成！
✅ 功能开发完成！
```

### 示例 2：讨论迭代（调整计划）

```bash
# 用户输入
/ccg:feat 调整用户登录功能，增加"记住我"选项

# 系统识别：讨论迭代模式

✓ 找到现有计划：.claude/plan/用户登录功能.md
✓ 读取现有计划内容
✓ 调用 planner 重新规划...

✓ 新计划已生成：.claude/plan/用户登录功能-1.md

? 功能规划已完成，请选择下一步操作：
  > 开始实施（推荐）
    继续讨论调整
    重新规划
    仅保存计划
```

### 示例 3：直接实施（跳过规划）

```bash
# 用户输入
/ccg:feat 按照 .claude/plan/用户登录功能.md 开始实施

# 系统识别：执行实施模式

✓ 读取计划：.claude/plan/用户登录功能.md
✓ 分析任务类型：全栈任务

🎨 前端任务 → 路由到 Gemini
⚙️ 后端任务 → 路由到 Codex

✓ 实施完成！
```

---

## 注意事项

1. **计划文件管理**：
   - 所有计划保存在 `.claude/plan/` 目录
   - 建议将 `.claude/` 加入 `.gitignore`（或选择性提交计划文档）

2. **Session 恢复**：
   - 如果实施中断，可使用 SESSION_ID 恢复：
     ```bash
     codeagent-wrapper --backend gemini resume <SESSION_ID> - <<'EOF'
     继续上次的任务
     EOF
     ```

3. **多模型协作**：
   - Gemini：擅长 UI/UX、组件设计、样式实现
   - Codex：擅长业务逻辑、API 设计、数据库操作
   - Claude：负责整体编排、代码重构、质量保证

4. **代码审查**：
   - 强烈建议实施后运行 `/ccg:review` 进行双模型审查
   - 90 分以上视为高质量代码

5. **测试覆盖**：
   - planner 会建议测试策略，但需手动实施
   - 可运行 `/ccg:test` 生成测试用例

---

## 工作流程图

```mermaid
graph TD
    Start[用户输入需求] --> Identify{识别输入类型}

    Identify -->|需求规划| CheckContext[检查项目上下文]
    Identify -->|讨论迭代| LoadPlan[读取现有计划]
    Identify -->|执行实施| ReadPlan[读取计划文档]

    CheckContext --> AceTool[ace-tool 检索相关代码]
    AceTool --> JudgeType{判断任务类型}

    JudgeType -->|前端/全栈| UIUX[调用 ui-ux-designer]
    JudgeType -->|后端| Planner1[调用 planner]

    UIUX --> Planner2[调用 planner]
    Planner1 --> SavePlan[保存计划文档]
    Planner2 --> SavePlan

    SavePlan --> Confirm{用户确认}

    Confirm -->|开始实施| ReadPlan
    Confirm -->|讨论调整| LoadPlan
    Confirm -->|重新规划| CheckContext
    Confirm -->|仅保存| End1[结束]

    LoadPlan --> Planner3[调用 planner 重新规划]
    Planner3 --> SavePlan

    ReadPlan --> Analyze[分析任务类型]

    Analyze -->|前端| Gemini[codeagent-wrapper: Gemini]
    Analyze -->|后端| Codex[codeagent-wrapper: Codex]
    Analyze -->|全栈| Parallel[并行调用 Gemini + Codex]

    Gemini --> Verify[实施后验证]
    Codex --> Verify
    Parallel --> Verify

    Verify --> Review{运行代码审查?}

    Review -->|是| CCGReview[/ccg:review]
    Review -->|否| End2[结束]

    CCGReview --> End2

    End1[结束]
    End2[结束]
```

---

现在开始智能功能开发！
