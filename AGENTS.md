# UI Design Skill - 项目认知

## 项目定位
通用前端 UI&UX 设计规范 skill，Theme 插件化架构。**开源项目，平台无关**——任何能读取文件的 AI agent 都可使用，不依赖特定平台的触发机制（hook/plugin/instructions file）。用户有 UI 需求时手动让 AI 读取 SKILL.md 触发完整工作流。

## 设计哲学
- **手动调用而非自动触发**：用户明确要做 UI 时主动让 AI 读取 SKILL.md，是有意识的设计决策。不需要 superpowers 式的 SessionStart hook 注入 bootstrap
- **平台无关**：skill 内容不写死任何平台专属变量（如 `${CLAUDE_PLUGIN_ROOT}`）或工具名（如 `Skill` 工具），因此无需为每个 agent 单独适配。各 agent 的差异主要在"自动触发机制"，去掉触发机制后只剩"一份能被读取的 skill 内容"，跨平台兼容问题自然消失
- **渐进式披露**：SKILL.md 只保留骨架，详细规则按需从 references/ 加载

## 架构
- **SKILL.md**：包含 Codex 识别所需的最小 frontmatter（`name` + `description`），description 明确限制为用户主动点名 skill 或要求读取文件时才触发；正文承载 V2.1 六阶段闭环（需求与项目分析 + Theme 决策与就绪 + 设计规则解析 + UI 方案组装 + 代码实现 + 视觉验证闭环与交付）
- **references/**：按需加载的详细规则
  - `theme-management.md`：主题搜索/安装/卸载/校验
  - `theme-structure.md`：theme.json 格式与结构约定
  - `token-dictionary.md`：Token 语义字典（6 维度）
  - `visual-validation.md`：需求驱动截图矩阵、视觉评估、问题分级、三轮停止条件与能力降级
- **explorations/skill-workflows/**：根目录 `SKILL.md` 的版本档案，使用 `SKILL_V{major}.{minor}.md → SKILL.md` 映射；`README.md` 维护版本来源和 `derivedFrom`。目录内文件不可直接注册，根目录 `SKILL.md` 是唯一正式入口
- **统一主题仓库（ui-theme-hub）**：集中管理所有主题的 Git 仓库（`https://github.com/LeP-Ton/ui-theme-hub/`），AI 从该仓库发现和安装主题
- **theme-config.json**：主题目录配置文件（与 SKILL.md 同目录），`themeDir` 字段指定主题存放目录（绝对路径）
- **themes/**：本地已安装 theme 目录（默认），每个 theme 从统一仓库安装，自带 theme.json + patterns/standards/assets/examples

## 关于 superpowers 机制（已评估，不采用）
曾借鉴 obra/superpowers 的自动触发机制（SessionStart hook 注入 bootstrap + description SDO + plugin 结构），但评估后回退：
- **不采用 hook 自动触发**：本项目是开源、平台无关的，用户主动调用即可；hook 是平台专属机制（Claude Code/Cursor 的 shell hook、Gemini 的 instructions file、OpenCode/pi 的 in-process 插件），采用它会引入"大量 agent 兼容问题"
- **保留的收益**：SKILL.md 骨架化，详细规则按需加载，降低调用时的上下文消耗
- **何时重新考虑 hook**：若未来发现用户频繁"想不起来用 skill"成为真实痛点，或要为特定平台做硬自动触发发行版，再按平台单独接入

## Theme 结构
theme.json 包含介绍信息和运行时配置：
- name/version/description/author/scene/tags：介绍信息
- requires：对象数组，每项含 name（依赖标识，推断代码风格）和 source（开源地址，AI 不熟悉时从中获取 API）。只放外部依赖，浏览器原生能力（CSS Variables）不列入
- tokens：6 维度（color/spacing/typography/border/shadow/motion）+ extended 扩展
- examples/：预览截图（.png/.webp），AI 扫描目录自动发现
- repository/license/preview：不需要在 theme.json 中声明，由统一主题仓库（ui-theme-hub）管理

## 已安装 Theme

**不维护静态清单**——本地已安装主题以 `theme-config.json` 的 `themeDir` 目录为准，AI 用 `theme-management.md` 的探测命令即时读取（避免派生数据不一致）。skill 仓库 `themes/` 下的目录仅为示例，不代表用户实际已安装。

## 协作模型
- 设计师：维护 Theme（PR 到统一主题仓库）
- AI：从统一主题仓库发现、安装并加载 Theme，完成 UI 组装、代码实现与视觉验证闭环

## 关键设计决策
- 引擎逻辑全在 SKILL.md 骨架中，详细规则分散到 references/（Claude 只自动读取 SKILL.md 骨架，按需加载 references）
- tokens 合并为单个 theme.json，一次读取全部 token（原来 8 个文件→1 个文件）
- 删除了 registry 字段：技术栈由 requires 依赖推断，自定义组件信息由 patterns/ 提供，registry 整体冗余
- 删除了 core/ 目录：adapters 对 AI 无增量价值，standards 的内容 AI 本身就知道，core 层已无存在必要
- 统一主题仓库（ui-theme-hub）：所有主题集中在一个 Git 仓库中管理，AI 从该仓库发现和安装主题，新主题通过 PR 贡献。仓库位于 ui-design-skill 同级目录
- 删除了硬编码场景表：场景识别改为动态语义匹配（读取 themes-summary.json 的 scene/tags/description），新主题发布无需更新 SKILL.md
- themes-index.json 拆为两层：themes-summary.json（轻量索引，供列表页和 AI 场景识别）+ themes/{id}.json（单主题全量，供详情页按需加载）。原全量索引已废弃
- 主题安装方式改为 zip 下载（curl + unzip），避免 AI 手动复制遗漏文件。安装源：GitHub Pages packages/{theme-id}.zip
- **SKILL.md 拆骨架 + references/ 渐进式披露**：从 14065 字节降到 5018 字节，调用时省 token；详细规则按需加载
- **平台无关化**：skill 内容不写死平台专属变量/工具名，任何 agent 能读文件即可用，避免多平台适配工程
- **最小识别 frontmatter**：根 `SKILL.md` 只声明 Codex 必需的 `name` 与 `description`，不放 `license`；description 明确排除普通 UI/前端请求，兼顾可识别与仅主动调用
- **V2.1 视觉闭环**：正式工作流采用“运行 → 截图 → 评估 → 修改 → 再评估”，最多执行 3 个完整验证轮次；无阻断和主要问题时通过，缺少预览或截图能力时降级检查并明确标记视觉验证未完成
- **本地主题检索不建索引文件**：本地（`{themeDir}`）不维护 `installed-summary.json` 之类的派生索引——派生数据有一致性维护成本（安装/卸载需同步，用户手动删目录会脏数据），且本地主题数量少索引收益不显著。改为在 `references/theme-management.md` 固化两条零维护探测命令：① for 循环读各 `theme.json` 的 name/scene/tags；② `find` 提取目录名拼 `?installed=`。直接读源文件，即时正确。zsh 兼容：for 循环加 `setopt nullglob`，列目录用 `find -mindepth 1 -maxdepth 1`（避免 zsh `nomatch` 空目录报错）

## Token 设计思路
- **格式标准**：参考 W3C Design Tokens Community Group（https://design-tokens.github.io/community-group/format/）的分组思路（color/spacing/typography/border/shadow/motion），但做了简化——W3C 标准需要 `$value`、`$type` 嵌套，对 Claude 读取不友好，所以直接用扁平 JSON
- **语义字典**：仅靠字段名（如 `color.background.elevated`）AI 无法推断用途，所以在 references/token-dictionary.md 中为每个 token 字段提供了语义 + 使用场景的说明（"elevated → 浮层背景 → 卡片、弹窗、下拉面板"），AI 读后可直接正确使用
- **extended 扩展**：每个 token 维度可添加 `extended` 字段存放场景专属值（如 boss-theme 的 `tableHeaderBg`、game-theme 的 `rarityRare`），这些自定义字段需在 theme 的 `standards/` 中说明用途

## 检索方法
- 在对项目检索时，无需把 `.agentdocs/workflow` 中的变更文档直接纳入检索，可以先检索 `.agentdocs/index.md` 根索引文档，再按需检索详细变更文档
- 比较 SKILL 工作流方案时，先读 `explorations/skill-workflows/README.md`，再按映射读取具体候选；不要把 `.history/` 自动快照作为正式方案来源

## 使用方式
- **任意 agent**：把仓库放在 agent 可访问目录，需要时让 AI 读取 `SKILL.md`
- **Claude Code（可选）**：软链到 `~/.claude/skills/ui-design`，可用 `/ui-design` 或对话中调用
