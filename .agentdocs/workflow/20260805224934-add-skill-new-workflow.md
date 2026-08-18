# 新增候选版六阶段 UI 工作流

## 背景与目标

在不替换当前 `SKILL.md` 的前提下创建 `SKILL_NEW.md`，验证重新划分后的六阶段流程是否更符合 AI 实际完成 UI 任务的推理和执行顺序。

## 约束与原则

- 保留 Theme 插件化和渐进式披露架构。
- 先分析目标项目，再选择和解析 Theme。
- 主题安装等管理动作作为条件子流程，不占据独立主阶段。
- 合并 Pattern 与 assets 匹配，形成统一的 UI 方案组装阶段。
- 将实现与验证拆开，建立明确质量闭环。
- 不修改或替换现有 `SKILL.md`、`SKILL_OLD.md`。

## 阶段与 TODO

- [x] 编写候选新版 SKILL。
- [x] 检查六阶段顺序和职责边界。
- [x] 核对与当前 Theme 协议的一致性。
- [x] 验证 frontmatter 和交叉引用。

## 关键风险

- `SKILL_NEW.md` 与 `SKILL.md` 使用相同的 frontmatter `name`，两者不应同时安装为可发现 Skill；当前文件仅用于候选对比。
- 新版要求先检查目标项目，首次执行会增加少量检索，但可减少 Theme 与技术栈冲突导致的返工。

## 当前进展

候选版已按“需求与项目分析 → Theme 决策与就绪 → 设计规则解析 → UI 方案组装 → 代码实现 → 验证与交付”完成编写。

## 代码变更（见 4.3，使用 git diff）

- 新增文件：`SKILL_NEW.md`。包含新版 frontmatter、核心原则、六阶段主流程和按需子流程索引。
- 新增文件：`.agentdocs/workflow/20260805224934-add-skill-new-workflow.md`。记录候选方案的设计原因、风险和测试。
- `.agentdocs/index.md`：追加本变更文档指针。

## 测试用例

1. 检查 YAML frontmatter：预期包含 `name`、`description`、`license` 且边界闭合。
2. 检索 `Phase 1` 至 `Phase 6`：预期每个阶段恰好出现一次且顺序连续。
3. 检索 Theme 协议：预期使用 `dir`、`patterns/components`，不出现旧的 `theme-id`、`patterns/blocks`、`scene=enterprise`。
4. 检查 references 路径：预期三个交叉引用文件均真实存在。
