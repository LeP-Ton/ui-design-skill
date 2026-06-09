# UI Design Skill - 项目认知

## 项目定位
通用前端 UI&UX 设计规范 Claude Code Skill，Theme 插件化架构。所有主题集中在一个统一主题仓库中管理，AI 从该仓库发现和安装主题。

## 架构
- **SKILL.md**：完整引擎逻辑（场景识别+主题加载+主题搜索安装+技术选择+模板匹配+物料匹配+代码生成），Claude 触发时直接读取
- **统一主题仓库（ui-theme-hub）**：集中管理所有主题的 Git 仓库（`https://github.com/example/ui-theme-hub`），AI 从中发现和安装主题
- **themes/**：本地已安装 theme 目录，每个 theme 从统一仓库安装，自带 theme.json + patterns/standards/assets/examples

## Theme 结构
theme.json 兼作市场介绍页和运行时配置：
- name/version/description/author/scene/tags：市场展示信息
- requires：对象数组，每项含 name（依赖标识，推断代码风格）和 source（开源地址，AI 不熟悉时从中获取 API）。只放外部依赖，浏览器原生能力（CSS Variables）不列入
- preview：预览截图路径数组，引用 examples/ 中的图片
- repository：统一主题仓库地址（所有主题指向同一个 ui-theme-hub 仓库）
- license：开源协议
- tokens：6 维度（color/spacing/typography/border/shadow/motion）+ extended 扩展
- examples/：市场预览截图（.png/.webp），不是代码示例

## 已安装 Theme
| 名称 | 场景 | 技术栈 |
|------|------|--------|
| boss-theme | B端管理（默认） | antd |
| game-theme | 游戏UI（默认） | 原生 CSS 变量 + poem |
| apple-theme | C端/营销/移动（默认） | Tailwind CSS |

## 协作模型
- 设计师：维护 Theme（PR 到统一主题仓库）
- AI：从统一主题仓库发现+安装 Theme + 校验 + 加载 + 组装代码

## 关键设计决策
- 引擎逻辑全在 SKILL.md 中，不分散到子文件（Claude 只自动读取 SKILL.md）
- tokens 合并为单个 theme.json，一次读取全部 token（原来 8 个文件→1 个文件）
- 删除了 registry 字段：技术栈由 requires 依赖推断，自定义组件信息由 patterns/ 提供，registry 整体冗余
- 删除了 core/ 目录：adapters 对 AI 无增量价值，standards 的内容 AI 本身就知道，core 层已无存在必要
- 统一主题仓库（ui-theme-hub）：所有主题集中在一个 Git 仓库中管理，AI 从该仓库发现和安装主题，新主题通过 PR 贡献。仓库位于 ui-design-skill 同级目录

## Token 设计思路
- **格式标准**：参考 W3C Design Tokens Community Group（https://design-tokens.github.io/community-group/format/）的分组思路（color/spacing/typography/border/shadow/motion），但做了简化——W3C 标准需要 `$value`、`$type` 嵌套，对 Claude 读取不友好，所以直接用扁平 JSON
- **语义字典**：仅靠字段名（如 `color.background.elevated`）AI 无法推断用途，所以在 SKILL.md 中为每个 token 字段提供了语义 + 使用场景的说明（"elevated → 浮层背景 → 卡片、弹窗、下拉面板"），AI 读后可直接正确使用
- **extended 扩展**：每个 token 维度可添加 `extended` 字段存放场景专属值（如 boss-theme 的 `tableHeaderBg`、game-theme 的 `rarityRare`），这些自定义字段需在 theme 的 `standards/` 中说明用途

## Skill 注册
- 路径：~/.claude/skills/ui-design → /Users/didi/Documents/program/ui-design-skill
- 触发：/ui-design 或关键词
