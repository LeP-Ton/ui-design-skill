<!--
方案：项目优先端到端版
状态：候选方案
来源：原 SKILL_NEW2.md
映射目标：SKILL.md
基线：original@4b513f2
derivedFrom：SKILL_V1.1
特点：重组为需求与项目分析、Theme 决策、规则解析、UI 方案组装、代码实现、验证与交付。
主要收益：先理解目标项目再选择 Theme，合并 Pattern 与 assets 组装，并建立独立验证闭环。
主要取舍：主流程更长、约束更完整，简单 UI 任务的首次分析成本略高。
用途：SKILL.md 的 V2.0 设计档案，不是可直接发现的 Skill；试用时从下方 YAML frontmatter 开始复制到根目录 SKILL.md。
-->

---
name: ui-design
description: Use when creating or modifying any web page, UI component, form, landing page, dashboard, mobile interface, presentation, or game UI - before planning or writing frontend code. Triggers on 生成页面/创建列表页/设计界面/落地页/营销页/移动端界面/游戏UI/C端页面/管理页面/数据看板/按主题生成. Ensures the target project's constraints and a selected Theme's tokens, patterns, standards, and assets guide implementation.
license: Complete terms in LICENSE.txt
---

<!-- 本文件是候选新版，不替换当前 SKILL.md。 -->
<!-- 遵循渐进式披露：这里只保留主流程，主题管理、结构和 Token 语义按需读取 references/。 -->

# UI Design Skill — Theme 驱动的 UI 实现工作流

## 核心原则

- **先理解项目，再选择 Theme**：Theme 必须适配目标项目，而不是反向强迫项目迁移技术栈
- **设计师维护 Theme，AI 负责组装与实现**
- **Theme 是设计输入，不是可直接覆盖项目的代码模板**
- **只使用已确认存在的 tokens、patterns、standards 和 assets，不得虚构 Theme 内容**
- **渐进式加载**：只读取当前任务需要的 Theme 文件，控制上下文消耗

## 六阶段主流程

> **REQUIRED:** 必须依次完成六个阶段的判定。阶段可以在无适用资源时快速通过，但不得省略判断。
> 仅在需要用户选择、发现冲突、存在风险或最终交付时沟通；不要机械输出每阶段进度。

### Phase 1：需求与项目分析

先明确“要做什么”和“要在什么环境中实现”：

1. 识别交付范围：页面、区块、组件、原型或现有界面改造
2. 提取场景、目标用户、核心任务、内容层级、交互状态和响应式要求
3. 检查目标项目已有的框架、组件库、样式方案、Design Token 机制、依赖版本和目录约定
4. 记录用户明确指定的 Theme、技术栈和不可变约束

技术决策优先级：用户明确指定 > 目标项目已有 > Theme 推荐。

如果缺少的信息不会改变总体方案，采用保守假设继续；只有会显著改变结果时才询问用户。

### Phase 2：Theme 决策与就绪

#### 2.1 选择 Theme

Theme 的机器唯一标识是 `dir`，`name` 仅用于展示。

1. 用户指定有效 `dir` 或唯一 `name` 时直接选择
2. 否则读取统一主题仓库的 `docs/themes-summary.json`
3. 根据用户需求与各 Theme 的 `scene`、`tags`、`description` 做语义匹配
4. 只有一个合理候选时直接选择；有多个时展示 `name` + `dir`，简短询问一次
5. 没有合理候选时说明未匹配，并提供主题市场入口；不得擅自选择默认 Theme

选择优先级：用户明确指定 Theme > 用户指定场景 > 语义匹配。

#### 2.2 确保 Theme 可用

1. 读取与本文件同目录的 `theme-config.json`，获得绝对路径 `themeDir`
2. 检查 `{themeDir}/{dir}/theme.json`
3. 未安装时才执行下载安装子流程
4. 校验 Theme；校验失败时停止使用并报告具体问题

> 搜索、安装、卸载、访问市场和校验规则见 `references/theme-management.md`。

### Phase 3：设计规则解析

按当前任务需要读取 Theme，而不是一次加载全部内容：

1. 读取 `theme.json`，获得 `tokens` 和 `requires`
2. 判断 `requires` 与 Phase 1 的项目环境是否兼容
3. 只读取与当前页面和场景相关的 `standards/*.md`
4. 将通用 Token 与 `extended` Token 的语义映射到当前 UI 需求
5. 若 Theme 规则与用户要求或项目约束冲突，采用“用户要求 > 项目约束 > Theme 规则”，并说明取舍

本阶段产出一份内部设计约束：视觉语义、布局原则、组件风格、交互规则和技术边界。

> Theme 结构与 `requires` 规则见 `references/theme-structure.md`。
> Token 的使用语义见 `references/token-dictionary.md`。

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
- 实现必要的 loading、empty、error、disabled、hover、focus 等状态
- 保证基本响应式、键盘操作、语义结构和可读对比度
- 禁止硬编码 Theme 已定义的颜色、间距、字体、圆角、阴影和动效值

### Phase 6：验证与交付

#### 6.1 实现验证

运行与变更范围相称的检查：

- 构建、类型检查、Lint 和相关测试
- 页面或组件的关键状态与响应式表现
- 资源路径是否真实存在
- 新增依赖是否经过用户同意且确有必要

#### 6.2 Theme 一致性检查

- Theme 已定义的视觉值是否全部使用 Token
- Token 是否按语义使用，而非只因数值相近而误用
- 实现是否遵循相关 standards
- Pattern 的结构意图是否保留，同时适配了目标项目
- 不支持或缺失的 Theme 能力是否已明确说明

#### 6.3 交付说明

简要说明完成内容、采用的 Theme、关键取舍、验证方法和未验证风险。无法运行某项检查时必须明确说明，不得声称已经通过。

## 按需子流程

| 情况 | 读取 |
|------|------|
| 主题搜索、安装、卸载、访问市场、安装校验 | `references/theme-management.md` |
| 创建或理解 Theme、解析 `requires` | `references/theme-structure.md` |
| 生成代码或映射 Design Token | `references/token-dictionary.md` |

主题管理不是每次 UI 实现的固定主阶段：Theme 已安装且有效时，只需完成可用性检查后继续。
