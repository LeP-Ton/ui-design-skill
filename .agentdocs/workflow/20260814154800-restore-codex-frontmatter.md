# 恢复 Codex 最小识别 Frontmatter

## 背景与目标

- 完全移除 YAML frontmatter 后，Codex 无法把项目识别为可用 Skill。
- 恢复识别能力的同时，继续遵守“不因普通 UI 请求自动介入，只响应用户主动调用”的产品理念。

## 约束与原则

- 只恢复 Codex 必需的 `name` 与 `description`。
- 不恢复与识别无关的 `license` 字段。
- description 同时写明能力与严格触发边界，并明确排除普通 UI/前端请求。
- 不修改六阶段正文和探索版本档案。

## 阶段与 TODO

- [x] 恢复合法 YAML frontmatter。
- [x] 将触发条件限制为用户点名 `ui-design` 或明确要求读取 `SKILL.md`。
- [x] 同步 AGENTS 与 AgentDocs 索引的核心认知。
- [x] 检查 YAML 字段、触发边界和正文完整性。

## 关键风险

- 是否触发最终仍由各 Agent 平台解释 description；当前通过明确的否定语句最大限度降低普通 UI 请求误触发概率。
- `license` 不再位于 frontmatter；仓库许可仍由实际许可证文件承担。

## 当前进展

根 `SKILL.md` 已恢复 Codex 可识别的最小 frontmatter，并保留“仅用户主动调用”的触发限制。

## 代码变更

- `SKILL.md`
```diff
+---
+name: ui-design
+description: Theme-based UI implementation workflow. Use only when the user explicitly asks to use the ui-design skill or explicitly asks the agent to read this SKILL.md. Do not trigger from an ordinary UI or frontend request.
+---
+
 <!-- 本 SKILL.md 是骨架文件，遵循渐进式披露（progressive disclosure）原则。 -->
```

- `AGENTS.md`
```diff
-- **SKILL.md**：无触发 frontmatter 的手动调用入口；6 阶段工作流骨架（场景识别 + 主题加载 + 主题解析 + 模板匹配 + 物料匹配 + 代码生成），以 `dir` 贯穿主题选择/安装，先适配目标项目技术体系，再按需组合 Theme 资产；含交叉引用指向 references/
+- **SKILL.md**：包含 Codex 识别所需的最小 frontmatter（`name` + `description`），description 明确限制为用户主动点名 skill 或要求读取文件时才触发；正文承载 6 阶段工作流骨架（场景识别 + 主题加载 + 主题解析 + 模板匹配 + 物料匹配 + 代码生成）
-- **无触发 frontmatter**：根 `SKILL.md` 不声明 `name`、`description`、`license`，避免平台按描述自动触发；用户主动要求 AI 读取文件后才启动工作流
+- **最小识别 frontmatter**：根 `SKILL.md` 只声明 Codex 必需的 `name` 与 `description`，不放 `license`；description 明确排除普通 UI/前端请求，兼顾可识别与仅主动调用
```

- `.agentdocs/index.md`
```diff
+`workflow/20260814154800-restore-codex-frontmatter.md` - 恢复 Codex 识别所需的最小 YAML frontmatter，并限制为用户主动调用时触发
-- 启动方式：根 `SKILL.md` 无 name/description/license frontmatter，只在用户主动要求读取文件时启动
+- 启动方式：根 `SKILL.md` 只保留 Codex 识别所需的 name/description；description 明确仅在用户主动点名 skill 或要求读取文件时触发
```

- 新增文件：`.agentdocs/workflow/20260814154800-restore-codex-frontmatter.md`，记录恢复原因、触发边界和验证方式。

## 测试用例

### TC-001 YAML 结构合法
- 类型：静态检查
- 优先级：高
- 操作步骤：解析 `SKILL.md` 开头 frontmatter。
- 预期结果：边界闭合，只包含 `name` 和 `description`，`name=ui-design`。
- 是否通过：通过；使用 Ruby 标准库 YAML 解析验证。

### TC-002 主动调用边界
- 类型：内容检查
- 优先级：高
- 操作步骤：检查 description 的正向与反向触发条件。
- 预期结果：包含“用户点名 skill 或读取文件时使用”，同时包含“普通 UI/前端请求不得触发”。
- 是否通过：通过

### TC-003 正文保持完整
- 类型：回归检查
- 优先级：中
- 操作步骤：检查六个 Phase 标题和“主题解析”。
- 预期结果：六阶段各出现一次，Phase 3 仍为“主题解析”。
- 是否通过：通过
