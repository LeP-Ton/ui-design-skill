<!--
方案：视觉验证闭环正式版
状态：当前正式版
来源：会话-3
映射目标：SKILL.md
derivedFrom：SKILL_V2.0
特点：沿用项目优先的六阶段结构，加入运行、截图、评估、修改与再评估闭环。
主要收益：以真实渲染结果验证 UI，并通过问题分级、三轮上限和能力降级约束自动修正范围。
主要取舍：完整视觉验证需要可运行界面与截图能力；不具备条件时只能降级验证，不能宣称视觉通过。
用途：SKILL.md 的 V2.1 版本档案，不是可直接发现的 Skill；正式入口始终为根目录 SKILL.md。
-->

---
name: ui-design
description: Theme-based UI implementation workflow. Use only when the user explicitly asks to use the ui-design skill or explicitly asks the agent to read this SKILL.md. Do not trigger from an ordinary UI or frontend request.
---

<!-- 本 SKILL.md 是骨架文件，遵循渐进式披露（progressive disclosure）原则。 -->
<!-- 详细规则按需从 references/ 加载，避免一次性消耗大量 token。 -->
<!-- 用户需要产出符合标准的 UI 时，手动让 AI 读取本 SKILL.md 即可触发完整工作流。 -->

# UI Design Skill — Theme 驱动的 UI 实现与视觉闭环工作流

## 核心原则

- **先理解项目，再选择 Theme**：Theme 必须适配目标项目，不得反向强迫项目迁移技术栈
- **设计师维护 Theme，AI 负责组装、实现与验证**
- **Theme 是设计输入，不是可直接覆盖项目的代码模板**
- **只使用已确认存在的 tokens、patterns、standards 和 assets**，不得虚构 Theme 内容
- **视觉结果必须验证**：可运行且具备截图能力时，必须完成“运行 → 截图 → 评估 → 修改 → 再评估”闭环
- **渐进式加载**：只读取当前任务需要的 Theme 文件和 reference

## 六阶段主流程

> **REQUIRED:** 生产 UI 时必须依次完成六个阶段的判定。阶段可以在无适用资源时快速通过，但不得省略判断。
> 仅在需要用户选择、发现冲突、需要额外授权、存在风险或最终交付时沟通；不要机械输出每阶段进度。

### Phase 1：需求与项目分析

先明确交付目标和实现环境：

1. 识别交付范围：页面、区块、组件、原型或现有界面改造
2. 提取场景、目标用户、核心任务、内容层级、交互状态和响应式要求
3. 检查目标项目已有的框架、组件库、样式方案、Design Token 机制、依赖版本、目录约定和运行方式
4. 记录用户明确指定的 Theme、技术栈和不可变约束

技术决策优先级：用户明确指定 > 目标项目已有 > Theme 推荐。

缺少的信息不会改变总体方案时，采用保守假设继续；只有会显著改变结果时才询问用户。

### Phase 2：Theme 决策与就绪

#### 2.1 选择 Theme

Theme 的机器唯一标识是 `dir`，`name` 仅用于展示：

1. 用户指定有效 `dir` 或唯一 `name` 时直接选择
2. 否则读取统一主题仓库的 `docs/themes-summary.json`
3. 根据需求与各 Theme 的 `scene`、`tags`、`description` 做语义匹配
4. 只有一个合理候选时直接选择；有多个时展示 `name` + `dir`，简短询问一次
5. 没有合理候选时说明未匹配，并提供主题市场入口；不得擅自选择默认 Theme

选择优先级：用户明确指定 Theme > 用户指定场景 > 语义匹配。

#### 2.2 确保 Theme 可用

1. 读取与本文件同目录的 `theme-config.json`，获得绝对路径 `themeDir`
2. 检查 `{themeDir}/{dir}/theme.json`
3. 未安装时才执行下载安装子流程
4. 校验 Theme；校验失败时停止使用并报告具体问题

> 主题搜索、安装、卸载、访问市场和校验规则见 `references/theme-management.md`。

### Phase 3：设计规则解析

按当前任务需要读取 Theme，而不是一次加载全部内容：

1. 读取 `theme.json`，获得 `tokens` 和 `requires`
2. 判断 `requires` 与 Phase 1 的项目环境是否兼容
3. 只读取与当前页面和场景相关的 `standards/*.md`
4. 将通用 Token 与 `extended` Token 的语义映射到当前 UI 需求
5. 若 Theme 规则与用户要求或项目约束冲突，采用“用户要求 > 项目约束 > Theme 规则”，并说明取舍

本阶段形成视觉语义、布局原则、组件风格、交互规则和技术边界。

> Theme 结构与 `requires` 规则见 `references/theme-structure.md`。
> Token 使用语义见 `references/token-dictionary.md`。

### Phase 4：UI 方案组装

在写代码前形成完整方案：

1. 从 `patterns/pages/` 匹配页面级 Pattern
2. 从 `patterns/components/` 匹配组件级 Pattern
3. 从 `assets/` 匹配 icons、illustrations 和 fragments
4. 确定信息架构、页面区块、组件层级、关键状态和响应式行为
5. 确定 Theme Token 到项目变量、主题配置或组件 Token 的映射方式

组装规则：

- Pattern 是结构和视觉参考，必须先检查其技术体系与目标项目是否兼容
- 只引用已确认存在的资产路径
- 模型无法识图时，可依据文件名和 standards 判断；信息不足时报告风险，不得猜测图片内容
- 没有匹配 Pattern 或 asset 时记录为空，基于 tokens + standards 自行设计，不得虚构资源

### Phase 5：代码实现

按照 Phase 4 的方案修改目标项目：

- 遵循目标项目现有架构、命名、组件组织和注释风格
- 优先复用项目已有组件和依赖，不因 Theme 的 `requires` 擅自引入或升级包
- 将 Theme tokens 接入项目已有 Token 机制；没有现成机制时建立集中、可复用的变量或配置
- 遵循已加载的 standards，以选定 Pattern 为参考，并使用已确认的 assets
- 实现需求涉及的 loading、empty、error、disabled、hover、focus 等状态
- 保证与需求相称的响应式、键盘操作、语义结构和可读对比度
- 禁止硬编码 Theme 已定义的颜色、间距、字体、圆角、阴影和动效值

### Phase 6：视觉验证闭环与交付

实现完成后，读取 `references/visual-validation.md` 并执行与任务范围相称的验证：

1. 运行构建、类型检查、Lint 和相关测试，确认界面具备可靠的渲染基础
2. 建立需求驱动的截图矩阵，覆盖主视口以及需求涉及的响应式断点和关键状态
3. 运行或复用预览环境，等待界面稳定后采集真实截图
4. 对照用户要求、Theme tokens、standards、Pattern 和目标项目约束评估截图
5. 若存在阻断或主要问题，在剩余轮次内修改代码并重新检查、截图和评估

最多执行 3 个完整验证轮次。不存在阻断和主要问题时即为通过；次要问题可以保留，但必须在交付说明中列出。第 3 轮后仍未通过时停止自动修改并报告剩余问题，不得宣称已经通过。

若目标界面无法运行或当前环境没有截图能力，执行可完成的静态与运行检查，并明确标记“视觉验证未完成”及原因；不得用代码检查代替截图评估，也不得为完成截图擅自引入依赖或改变项目技术体系。

交付时说明采用的 Theme、实际验证命令、截图覆盖范围、迭代轮次、已修复问题、遗留次要问题和未验证风险。

## 按需子流程

| 情况 | 读取 |
|------|------|
| 主题搜索、安装、卸载、访问市场、安装校验 | `references/theme-management.md` |
| 创建或理解 Theme、解析 `requires` | `references/theme-structure.md` |
| 生成代码或映射 Design Token | `references/token-dictionary.md` |
| 运行界面、截图、视觉评估、修改和再评估 | `references/visual-validation.md` |

主题管理不是每次 UI 实现的固定主阶段：Theme 已安装且有效时，只需完成可用性检查后继续。
