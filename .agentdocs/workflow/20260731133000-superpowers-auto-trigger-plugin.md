# 借鉴 superpowers 自动触发机制：升级为 Plugin + SessionStart hook 注入

## 背景与目标
- 用户希望让 ui-design-skill 能在用户与大模型对话时**自动触发并使用主题**，无需手动 `/ui-design` 调用
- 研究 obra/superpowers 的自动触发机制后发现：superpowers 靠「SessionStart hook 注入 bootstrap（硬保证）+ description 语义匹配（软匹配）」双机制耦合实现可靠自动触发
- 当前 ui-design-skill 是纯 skill 软链注册，**无法触发 SessionStart hook**（hook 是 plugin 或 settings.json 的能力），且 SKILL.md 14065 字节一次性全载、description 混入了"what it does"
- 目标：升级为 Claude Code Plugin，补齐 hook 注入机制，按 SDO 规则重写 description，SKILL.md 拆为骨架 + references 渐进式披露

## 约束与原则
- 借鉴 superpowers 但不照搬：bootstrap 注入精简元规则（<150 词），不注入整个 SKILL.md
- 保留现有 6 阶段工作流和 Theme 插件化架构不变
- 详细规则迁移到 references/，SKILL.md 只保留骨架 + 交叉引用
- 跨平台兼容（Windows/Unix），hook 脚本借鉴 superpowers 的 polyglot 设计

## 阶段与 TODO
- [x] 研究 superpowers 自动触发机制（SessionStart hook + description SDO 规则）
- [x] 查证 Claude Code hook 配置位置、plugin hooks 自动发现、matcher 合法值、additionalContext 机制
- [x] 创建 `.claude-plugin/plugin.json` Plugin 清单
- [x] 创建 `hooks/` 三件套（hooks.json + session-start + run-hook.cmd）
- [x] 创建 `skills/ui-design/bootstrap.md` 精简元规则（含 unique-marker）
- [x] 拆分 SKILL.md 为骨架 + references/（theme-management / theme-structure / token-dictionary）
- [x] 按 SDO 规则重写 description
- [x] 删除根目录旧 SKILL.md，迁移到 skills/ui-design/
- [x] 更新 README.md、AGENTS.md 反映 Plugin 架构
- [x] 验证 plugin 结构（claude plugin validate . 通过）
- [x] 验证 token 成本（bootstrap ~113 tok/会话，SKILL.md ~1.5k on-invoke）

## 关键风险
- **注册方式变更**：从 skill 软链改为 plugin，用户需改用 `--plugin-dir` 或 marketplace 安装，旧的 `~/.claude/skills/ui-design` 软链需处理
- **theme-config.json 路径**：SKILL.md 从根目录迁移到 skills/ui-design/ 后，theme-config.json 的相对位置变了，已用 `${CLAUDE_PLUGIN_ROOT}` 表达
- **hook 注入重复**：Claude Code 会同时读 additional_context 和 hookSpecificOutput 且不去重，session-start 脚本用环境变量分支精确输出一种，避免重复注入
- **compaction 失忆**：matcher 必须含 `compact`，否则长会话压缩后 bootstrap 丢失

## 当前进展
- Plugin 结构已建立并通过 `claude plugin validate .` 验证
- `claude --plugin-dir . plugin details ui-design` 确认：Skills(1) ui-design + Hooks(1) SessionStart，Always-on ~113 tok，on-invoke ~1.5k
- 旧根目录 SKILL.md 已删除，新 SKILL.md 在 skills/ui-design/（14065→4920 字节，降 65%）
- 待用户处理：旧的 `~/.claude/skills/ui-design` 软链注册方式需切换为 plugin 加载

## superpowers 自动触发机制研究结论

### 双机制耦合
| 机制 | 性质 | 作用 |
|------|------|------|
| 机制 A：SessionStart hook 注入 bootstrap | 硬保证 | 每次会话启动（含 /clear、/compact）把"必须用 skill"元规则强制灌入上下文 |
| 机制 B：description 语义匹配 | 软匹配 | 模型读 description 字段，自行判断何时调用 Skill 工具 |

superpowers 官方原话：**Bootstrap is the entire integration. Without it, the skill files are inert — present on disk, never invoked.**

### description 编写规则（SDO）
- 只写"何时用"（when），绝不写"做什么"（what）——否则模型跳过读正文
- 以 `Use when...` 开头，第三人称
- 含任务阶段词和时机锚点（`before writing frontend code`）
- frontmatter ≤1024 字符

### hook 配置要点
- matcher: `startup|clear|compact`（compact 必须保留）
- async: false（同步执行，保证回复前注入）
- 输出 `hookSpecificOutput.additionalContext`（Claude Code 格式）
- 用 `<EXTREMELY_IMPORTANT>` 标签包裹（Anthropic 模型高优先级指令强信号）

## 代码变更

### 新增文件

#### `.claude-plugin/plugin.json`（新增）
```diff
+ {
+   "name": "ui-design",
+   "description": "通用前端 UI&UX 设计规范 plugin：基于 Theme 插件化架构，自动触发于前端 UI 生成场景，提供设计系统、主题管理与代码组装能力",
+   "version": "1.0.0",
+   "author": { "name": "liaopengtaolpt_v" },
+   "homepage": "https://github.com/LeP-Ton/ui-design-skill",
+   "repository": "https://github.com/LeP-Ton/ui-design-skill",
+   "license": "MIT",
+   "keywords": ["ui", "design", "theme", "design-system", "frontend", "design-tokens"]
+ }
```

#### `hooks/hooks.json`（新增）
```diff
+ {
+   "description": "ui-design plugin 的 SessionStart hook：会话启动时注入 bootstrap 元规则，强制模型在做前端 UI 任务前调用 ui-design skill",
+   "hooks": {
+     "SessionStart": [
+       {
+         "matcher": "startup|clear|compact",
+         "hooks": [
+           {
+             "type": "command",
+             "command": "\"${CLAUDE_PLUGIN_ROOT}/hooks/run-hook.cmd\" session-start",
+             "shell": "bash",
+             "async": false
+           }
+         ]
+       }
+     ]
+   }
+ }
```

#### `hooks/session-start`（新增，核心注入逻辑）
```diff
+ #!/usr/bin/env bash
+ # ui-design plugin 的 SessionStart hook
+ # 作用：会话启动时把 bootstrap 元规则注入模型上下文，强制模型在执行前端 UI 任务前先调用 ui-design skill
+ # 设计借鉴自 obra/superpowers，但注入内容为精简元规则（而非整个 SKILL.md）
+ # unique-marker：UI-DESIGN-BOOTSTRAP-ACTIVE-v1，用于验证注入是否真的生效
+
+ set -euo pipefail
+ SCRIPT_DIR="$(cd "$(dirname "$0")" && pwd)"
+ PLUGIN_ROOT="$(cd "${SCRIPT_DIR}/.." && pwd)"
+
+ bootstrap_content=$(cat "${PLUGIN_ROOT}/skills/ui-design/bootstrap.md" 2>&1 || echo "Error reading ui-design bootstrap")
+
+ escape_for_json() {
+     local s="$1"
+     s="${s//\\/\\\\}"; s="${s//\"/\\\"}"; s="${s//$'\n'/\\n}"; s="${s//$'\r'/\\r}"; s="${s//$'\t'/\\t}"
+     printf '%s' "$s"
+ }
+
+ bootstrap_escaped=$(escape_for_json "$bootstrap_content")
+ session_context="<EXTREMELY_IMPORTANT>\nYou have the ui-design skill (Theme 插件化架构的前端 UI&UX 设计规范).\n\n${bootstrap_escaped}\n</EXTREMELY_IMPORTANT>"
+
+ # 按平台输出不同字段，避免 Claude Code 重复注入
+ if [ -n "${CURSOR_PLUGIN_ROOT:-}" ]; then
+   printf '{\n  "additional_context": "%s"\n}\n' "$session_context" | cat
+ elif [ -n "${CLAUDE_PLUGIN_ROOT:-}" ] && [ -z "${COPILOT_CLI:-}" ]; then
+   printf '{\n  "hookSpecificOutput": {\n    "hookEventName": "SessionStart",\n    "additionalContext": "%s"\n  }\n}\n' "$session_context" | cat
+ else
+   printf '{\n  "additionalContext": "%s"\n}\n' "$session_context" | cat
+ fi
+ exit 0
```

#### `hooks/run-hook.cmd`（新增，跨平台 polyglot 包装器）
```diff
+ : << 'CMDBLOCK'
+ @echo off
+ REM 跨平台 polyglot 包装器。Windows: cmd.exe 执行 batch 部分；Unix: shell 作为脚本执行
+ REM Hook 脚本用无扩展名文件名，避免 Claude Code Windows 自动检测干扰
+ ...（Git for Windows bash 探测 + PATH bash 回退 + 无 bash 静默退出）
+ CMDBLOCK
+ # Unix: 直接执行指定脚本
+ SCRIPT_DIR="$(cd "$(dirname "$0")" && pwd)"
+ SCRIPT_NAME="$1"; shift
+ exec bash "${SCRIPT_DIR}/${SCRIPT_NAME}" "$@"
```

#### `skills/ui-design/bootstrap.md`（新增，被注入的精简元规则）
```diff
+ <!-- unique-marker: UI-DESIGN-BOOTSTRAP-ACTIVE-v1 -->
+ # UI Design Skill — 自动触发元规则
+ <EXTREMELY-IMPORTANT>
+ 当用户要做任何前端界面相关的工作时，你 ABSOLUTELY MUST 先调用 `ui-design` skill，再写代码。
+ 这没有商量余地。你不能用"我先看看代码"、"这只是个简单页面"、"我直接写就行"等理由跳过。
+ </EXTREMELY-IMPORTANT>
+ ## 触发条件
+ - 创建/生成任何 Web 页面、UI 组件、表单、落地页、数据看板、移动端界面、游戏 UI
+ - 中文触发词：生成页面、创建列表页、构建表单、设计界面、落地页、营销页...
+ ## 反合理化红旗
+ | "这只是个简单页面" | 越简单越该用设计系统 | ... |
```

#### `skills/ui-design/references/theme-management.md`（新增，从旧 SKILL.md 抽离）
```diff
+ # 主题管理详细规则
+ ## 主题目录配置（themeDir 绝对路径，${CLAUDE_PLUGIN_ROOT}/theme-config.json）
+ ## 主题搜索与安装（查询/安装 zip/卸载/访问主题仓库 URL 参数）
+ ## 主题询问规则（仅询问一次）
+ ## 安装校验（必需目录/字段/tokens 6 维度/color 基础字段）
```

#### `skills/ui-design/references/theme-structure.md`（新增，从旧 SKILL.md 抽离）
```diff
+ # Theme 结构约定
+ ## 目录结构（theme.json + patterns/standards/assets/examples）
+ ## theme.json 格式（完整 JSON 示例）
+ ## 字段说明（必需字段/介绍信息/requires/不需要的字段）
+ ## 技术体系选择 Phase 3（requires 推断技术栈表）
```

#### `skills/ui-design/references/token-dictionary.md`（新增，从旧 SKILL.md 抽离）
```diff
+ # Token 语义字典
+ ## color（17 个 token 语义表）
+ ## spacing（间距层级规则）
+ ## typography（字体/字号/字重/行高）
+ ## border（圆角/边框宽度）
+ ## shadow（4 级阴影）
+ ## motion（动效开关/时长/缓动）
+ ## extended 扩展说明
```

### 修改文件

#### `skills/ui-design/SKILL.md`（新增骨架，替代根目录旧 SKILL.md）
description 按 SDO 规则重写：
```diff
- description: 通用前端 UI&UX 设计规范 skill。当用户提到"生成页面"、"创建列表页"...等场景时触发此 skill。基于 Theme 插件化架构，Theme 为可插拔的设计资产包，AI 负责组装。
+ description: Use when creating any web page, UI component, form, landing page, dashboard, mobile interface, or game UI - before writing frontend code. Triggers on 生成页面/创建列表页/设计界面/落地页/营销页/移动端界面/游戏UI/C端页面/管理页面/数据看板/按主题生成. Ensures a theme-based design system (tokens + patterns + standards) is applied instead of hardcoded values.
```

正文从 294 行/14065 字节精简为 114 行/4920 字节（降 65%），6 阶段间用 `**NEXT:**` 交叉引用串联，详细规则指向 references/。

#### `README.md`（重写，反映 Plugin 架构）
```diff
- 通用前端 UI&UX 设计规范 skill，采用 Core + Theme 插件化架构。
+ 通用前端 UI&UX 设计规范 Claude Code **Plugin**，采用 Theme 插件化架构。
+ 借鉴 obra/superpowers 的自动触发机制：通过 SessionStart hook 注入 bootstrap 元规则，让模型在做前端 UI 任务时自动触发。
```
新增「自动触发机制」「安装方式（--plugin-dir / marketplace）」「验证自动触发」章节。

#### `AGENTS.md`（重写，更新项目认知）
新增「自动触发机制（核心创新）」「Plugin 注册与加载」「自动触发验证」章节，记录 SDO 规则、双机制耦合、关键认知。

### 删除文件
```diff
- SKILL.md（根目录旧版，14065 字节，已迁移到 skills/ui-design/SKILL.md 骨架 + references/）
```

## 测试用例

### TC-001 plugin 结构合法性
- 类型：结构验证
- 优先级：高
- 关联模块：.claude-plugin/plugin.json
- 前置条件：项目目录存在
- 操作步骤：
  1. 执行 `claude plugin validate .`
- 预期结果：
  - 输出 "✔ Validation passed"
- 是否通过：✅ 通过

### TC-002 plugin 组件清单与 token 成本
- 类型：结构验证
- 优先级：高
- 关联模块：hooks/ + skills/
- 前置条件：plugin 结构已建立
- 操作步骤：
  1. 执行 `claude --plugin-dir . plugin details ui-design`
- 预期结果：
  - Skills (1) ui-design
  - Hooks (1) SessionStart (harness-only — no model context cost)
  - Always-on: ~113 tok
  - on-invoke: ~1.5k
- 是否通过：✅ 通过

### TC-003 SessionStart hook 脚本执行
- 类型：功能测试
- 优先级：高
- 关联模块：hooks/session-start
- 前置条件：bootstrap.md 存在
- 操作步骤：
  1. 执行 `CLAUDE_PLUGIN_ROOT="$(pwd)" COPILOT_CLI="" bash hooks/session-start`
- 预期结果：
  - 输出合法 JSON，含 `hookSpecificOutput.additionalContext`
  - 内容含 `<EXTREMELY_IMPORTANT>` 包裹
  - 内容含 unique-marker `UI-DESIGN-BOOTSTRAP-ACTIVE-v1`
- 是否通过：✅ 通过

### TC-004 自动触发链路验证（需用户在真实会话验证）
- 类型：端到端测试
- 优先级：高
- 关联模块：SessionStart hook + skill 自动触发
- 前置条件：以 `claude --plugin-dir /Users/didi/Documents/program/ui-design-skill` 启动会话
- 操作步骤：
  1. 干净会话发送"帮我做一个后台管理首页"
  2. 观察模型是否主动调用 ui-design skill 并进入 Phase 1
- 预期结果：
  - 模型主动调用 ui-design skill（无需用户 /ui-design）
  - 走 6 阶段工作流
- 是否通过：待用户验证

### TC-005 compaction 后重新注入
- 类型：回归测试
- 优先级：中
- 关联模块：hooks/hooks.json matcher
- 前置条件：长会话中使用 ui-design skill
- 操作步骤：
  1. 会话中执行 `/compact`
  2. 继续发送 UI 任务
- 预期结果：
  - compact 后模型仍能自动触发 ui-design skill（因 matcher 含 compact）
- 是否通过：待用户验证
