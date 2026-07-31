# 回退 plugin/hook，改为平台无关手动调用 skill

## 背景与目标
- 上一变更（20260731133000）借鉴 superpowers 把项目升级为 Claude Code Plugin + SessionStart hook 自动触发
- 用户澄清：本项目是**开源项目，平台无关**，目标是"让 AI 产出符合标准的 UI"，用户有需求时**手动调用**即可，不需要自动触发
- 重新评估后发现：superpowers 的 hook 机制是平台专属的（Claude Code/Cursor 的 shell hook、Gemini 的 instructions file、OpenCode/pi 的 in-process 插件），采用它会引入"大量 agent 兼容问题"；而手动调用模式下，触发机制不需要了，各平台差异恰恰全在触发机制上——去掉它后只剩"一份能被读取的 skill 内容"，跨平台兼容问题自然消失
- 目标：回退 plugin + hook 结构，保留骨架化 + SDO 这两项纯收益，恢复平台无关的通用 skill 形态

## 约束与原则
- 保留 SKILL.md 骨架化 + references/ 渐进式披露（与触发机制无关，纯收益）
- 保留 description 按 SDO 规则优化（手动调用时在 skill 列表里更清晰）
- 删除所有平台耦合：`${CLAUDE_PLUGIN_ROOT}`、`Skill 工具` 写死、plugin 根目录表述
- 恢复 SKILL.md 和 references/ 到项目根目录（与 theme-config.json 同目录）

## 阶段与 TODO
- [x] 评估：开源 + 平台无关 + 手动调用 → hook 自动触发不必要
- [x] 移动 skills/ui-design/SKILL.md → 根目录 SKILL.md
- [x] 移动 skills/ui-design/references/ → 根目录 references/
- [x] 删除 .claude-plugin/、hooks/、skills/、bootstrap.md
- [x] 清除 SKILL.md 残留平台耦合（`${CLAUDE_PLUGIN_ROOT}`、plugin 根目录表述、bootstrap 相关注释）
- [x] 清除 references/theme-management.md 残留平台耦合
- [x] 更新 README.md 为平台无关形态
- [x] 更新 AGENTS.md 项目认知
- [x] 更新 index.md 关键记忆

## 关键决策：为什么不采用 superpowers 自动触发

### 场景不匹配
- superpowers 的自动触发解决"用户不知道有 skill / 模型想不起来用"
- 本项目用户明确要做 UI → 会主动调用，不存在"想不起来"痛点
- 不做 UI 的会话不该被打扰（省掉每次 113 token 注入）

### 平台兼容代价
- hook 是平台专属机制，全平台接入需 6 套 manifest + 3 种 Shape + 每平台工具映射，工程巨大
- 手动调用模式下，任何 agent 能读文件即可用，零平台适配成本

### 何时重新考虑 hook
- 若未来发现用户频繁"想不起来用 skill"成为真实痛点
- 或要为特定平台做硬自动触发发行版
- 再按平台单独接入（一次一个平台）

## 保留的收益（与触发机制无关）
| 改造 | 价值 | 保留 |
|------|------|------|
| SKILL.md 拆骨架 + references/ | 14065→5018 字节，调用省 token | ✅ |
| description SDO 优化 | 避免"自以为懂了"跳过读正文 | ✅ |
| SessionStart hook | 硬触发保证 | ❌ 删除 |
| plugin 结构 | hook 载体 | ❌ 删除 |
| bootstrap.md | 自动触发元规则 | ❌ 删除 |

## 代码变更

### 文件移动
```diff
- skills/ui-design/SKILL.md → SKILL.md（移回根目录）
- skills/ui-design/references/ → references/（移回根目录）
```

### 删除文件
```diff
- .claude-plugin/plugin.json
- hooks/hooks.json
- hooks/session-start
- hooks/run-hook.cmd
- skills/ui-design/bootstrap.md
- skills/（空目录）
```

### 修改文件

#### `SKILL.md`
清除平台耦合注释和路径：
```diff
- <!-- 自动触发的元规则见 bootstrap.md（由 SessionStart hook 注入）。 -->
+ <!-- 用户需要产出符合标准的 UI 时，手动让 AI 读取本 SKILL.md 即可触发完整工作流。 -->
```
```diff
- 1. 先读取 `theme-config.json`（位于 plugin 根目录）获取 `themeDir`...plugin 环境下路径为 `${CLAUDE_PLUGIN_ROOT}/theme-config.json`
+ 1. 先读取 `theme-config.json`（与 SKILL.md 同目录）获取 `themeDir`（绝对路径，若未配置则提示用户设置）
```

#### `references/theme-management.md`
```diff
- 主题安装目标由 `theme-config.json`（位于 plugin 根目录，即 `skills/ui-design/` 的上级）中的 `themeDir` 字段决定。
- - 定位 `theme-config.json`：若在 plugin 环境下，路径为 `${CLAUDE_PLUGIN_ROOT}/theme-config.json`；若通过 skill 软链使用，为 skill 目录的上级
+ 主题安装目标由 `theme-config.json`（与 SKILL.md 同目录）中的 `themeDir` 字段决定。
```

#### `README.md`（重写为平台无关形态）
```diff
- 通用前端 UI&UX 设计规范 Claude Code **Plugin**...借鉴 obra/superpowers 的自动触发机制
+ 通用前端 UI&UX 设计规范 skill...平台无关——任何能读取文件的 AI agent 都能使用。
+ ## 定位：让 AI 产出符合设计标准的 UI。用户有 UI 需求时，手动让 AI 读取 SKILL.md 即可触发完整 6 阶段工作流。
```
新增「在任意 agent 中使用」「在 Claude Code 中（可选）」章节。

#### `AGENTS.md`（更新项目认知）
- 项目定位改为"开源项目，平台无关"
- 新增「设计哲学：手动调用而非自动触发」
- 新增「关于 superpowers 机制（已评估，不采用）」记录决策
- 架构改回 SKILL.md 骨架 + references/

#### `.agentdocs/index.md`
- 关键记忆改为平台无关 + 手动调用
- 新增本次回退变更文档指针

## 测试用例

### TC-001 平台无关性验证
- 类型：结构验证
- 优先级：高
- 操作步骤：
  1. `grep -rn "CLAUDE_PLUGIN_ROOT\|plugin 根目录\|Skill 工具" SKILL.md references/`
- 预期结果：无匹配（已清除所有平台耦合）
- 是否通过：✅ 通过

### TC-002 结构完整性
- 类型：结构验证
- 优先级：高
- 操作步骤：
  1. 确认根目录有 SKILL.md、references/、theme-config.json
  2. 确认 .claude-plugin/、hooks/、skills/ 已删除
- 预期结果：根目录结构干净，仅保留平台无关文件
- 是否通过：✅ 通过

### TC-003 任意 agent 可用性
- 类型：功能测试
- 优先级：中
- 操作步骤：
  1. 在任意 agent 中让 AI 读取 SKILL.md
  2. 发送"帮我生成一个用户管理列表页"
- 预期结果：AI 按 6 阶段工作流产出符合主题规范的代码
- 是否通过：待用户验证
