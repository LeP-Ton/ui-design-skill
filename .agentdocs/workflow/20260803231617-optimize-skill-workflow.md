# 优化面向 AI 的主题工作流

## 背景与目标

结合 `ui-theme-hub` 当前数据结构优化 UI Design Skill，使 AI 能稳定完成主题识别、安装、技术适配、资产选择和代码生成，避免流程断裂或虚构不存在的主题资源。

## 约束与原则

- 保持 6 阶段骨架和渐进式披露，不把 references 全量塞回 SKILL.md。
- 以 `dir` 作为机器唯一标识，`name` 仅用于用户展示。
- 优先适配用户要求和目标项目现有技术体系，不因 Theme 推荐依赖静默改造项目。
- Pattern、standards、assets 均允许无匹配资源，但每阶段必须完成判定。
- 与 `ui-theme-hub` 当前 `components`、`previews`、`b2b` 协议一致。

## 阶段与 TODO

- [x] 重构 SKILL.md 六阶段职责和阶段间衔接。
- [x] 修正 Theme 标识、Pattern 目录和场景参数协议。
- [x] 同步直接交叉引用的主题管理、结构规范。
- [x] 检查过期术语和 Markdown diff 格式。

## 关键风险

- 远程主题仓库未来若再次修改目录或标识协议，SKILL 与 references 仍需同步更新。
- Theme 推荐依赖可能与目标项目冲突，因此只能作为推荐，不能直接触发依赖升级。

## 当前进展

六阶段已改为“逐阶段判定”：Phase 3 在 Pattern 匹配前统一确认项目技术上下文、Theme 依赖和 standards；缺少 Pattern、standards 或 assets 时明确为空并继续。安装、路径与 URL 统一使用 `dir`。

## 代码变更（见 4.3，使用 git diff）

以下记录依据 `git diff HEAD -- SKILL.md references/theme-management.md references/theme-structure.md AGENTS.md` 生成：

- `SKILL.md`：重构场景选择优先级、安装校验、实现上下文解析、Pattern/资产匹配和 Token 生成约束。
- `references/theme-management.md`：将旧 `id/theme-name/enterprise` 协议替换为 `dir/b2b`，补充卸载确认约束。
- `references/theme-structure.md`：将 `blocks/examples` 更新为 `components/previews`，同步 `dir/name` 和 `b2b` 示例。
- `AGENTS.md`：覆盖更新六阶段工作流的核心项目认知。
- 新增文件：`.agentdocs/workflow/20260803231617-optimize-skill-workflow.md`，记录本次设计原因、风险与验证方式。

## 测试用例

1. `git diff --check HEAD`：预期无空白错误。
2. 检索 `patterns/blocks|examples/|theme-id|theme-name|scene=enterprise|"b-end"`：预期 SKILL 与 references 中无旧协议残留。
3. 对照 `ui-theme-hub/docs/themes-summary.json` 和主题 zip：预期使用 `dir`、`b2b`、`patterns/components`。
4. 检查所有 `NEXT`：预期 Phase 1—6 名称与跳转语义一致。
