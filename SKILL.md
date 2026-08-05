---
name: ui-design
description: Use when creating any web page, UI component, form, landing page, dashboard, mobile interface, or game UI - before writing frontend code. Triggers on 生成页面/创建列表页/设计界面/落地页/营销页/移动端界面/游戏UI/C端页面/管理页面/数据看板/按主题生成. Ensures a theme-based design system (tokens + patterns + standards) is applied instead of hardcoded values.
license: Complete terms in LICENSE.txt
---

<!-- 本 SKILL.md 是骨架文件，遵循渐进式披露（progressive disclosure）原则。 -->
<!-- 详细规则按需从 references/ 加载，避免一次性消耗大量 token。 -->
<!-- 用户需要产出符合标准的 UI 时，手动让 AI 读取本 SKILL.md 即可触发完整工作流。 -->

# UI Design Skill — Theme 插件化架构

## 核心理念

- **Theme 可以独立演进**：每个 theme 自带完整的设计资产
- **设计师负责 Theme**，**AI 负责组装**
- **统一主题仓库**：所有主题集中在一个 Git 仓库（`ui-theme-hub`，`https://github.com/LeP-Ton/ui-theme-hub/`）中管理，AI 从该仓库发现和安装主题
- 规范靠 SKILL.md 定义，质量靠 AI 安装时校验

## 工作流（6 阶段）

> **REQUIRED:** 必须依次完成 6 个阶段的判定，不得跳过。某阶段没有可用资源时，明确记录“无匹配资源”并继续，不得虚构 Theme 内容。
> **主题搜索/安装/卸载/访问/校验的详细命令与规则**：见 `references/theme-management.md`

### Phase 1: 场景识别

读取统一主题仓库的 `docs/themes-summary.json`，根据 `scene` + `tags` + `description` 语义匹配用户意图。主题的机器唯一标识是 `dir`，`name` 仅用于展示：

1. 用户明确指定 Theme 的 `dir` 或唯一 `name` 时，直接选择该 Theme
2. 否则，从 themes-summary.json 提取各 Theme 的 `dir`、`name`、`scene`、`tags`、`description`
3. 将用户输入与 `scene`、`tags`、`description` 做语义匹配
4. 匹配结果唯一时直接使用；有多个合理候选时，列出 `name` + `dir` 简短询问一次
5. 未匹配到任何 Theme 时，说明没有匹配项，并给出主题市场入口，不得擅自选择“默认主题”

识别优先级：用户明确指定 Theme > 用户指定场景 > 语义匹配。

**NEXT:** 场景识别完成后，进入 Phase 2 加载主题。

### Phase 2: 主题加载

加载 Phase 1 选定的 Theme：

1. 先读取 `theme-config.json`（与 SKILL.md 同目录）获取 `themeDir`（绝对路径，若未配置则提示用户设置）
2. 使用 Theme 的 `dir` 检查 `{themeDir}/{dir}/theme.json`，不得用显示名称猜测目录
3. 若未安装，从 GitHub Pages 下载 `{dir}.zip` 并解压到 `{themeDir}`
4. 按 `references/theme-management.md` 校验安装结果；失败时停止使用该 Theme，并报告具体缺失项

加载 Theme 后，仅按当前任务需要读取，避免无差别加载全部上下文：
- `theme.json` 中的 `tokens`：Design Token
- `patterns/`：页面/组件模板
- `standards/`：设计标准规范
- `assets/`：物料素材

**NEXT:** 主题加载完成后，进入 Phase 3 确定设计标准。

### Phase 3: 实现上下文与标准解析

在匹配模板前确定实现边界：

1. 检查目标项目现有的框架、组件库、样式方案和版本约束
2. 读取 `theme.json.requires`，判断 Theme 依赖与目标项目是否兼容
3. 技术体系优先级：用户明确指定 > 目标项目已有 > Theme 推荐；存在冲突时先说明影响，不得静默改造项目技术栈
4. 按任务场景读取 `standards/` 中相关规范作为设计总纲；没有匹配规范时记录为空并继续

> **requires 字段格式与技术体系推断规则**：见 `references/theme-structure.md`

**NEXT:** 实现边界与设计标准确定后，进入 Phase 4 匹配模板。

### Phase 4: 模板匹配

从 theme 的 `patterns/` 中选择模板：
- `patterns/pages/`：页面级模板
- `patterns/components/`：组件级模板

优先选择与需求语义和目标技术体系兼容的 Pattern。Pattern 是设计与结构参考，不得不经检查就覆盖目标项目代码。

若 Theme 中无匹配 Pattern，记录为空，并基于 tokens + standards 设计；不得虚构不存在的模板。

**NEXT:** 模板选定后，进入 Phase 5 匹配物料。

### Phase 5: 物料匹配

从 Theme 的 `assets/` 目录匹配物料（icons / illustrations / fragments）。

- 只使用已确认存在的资产路径，并遵循 `standards/` 中的物料说明
- 模型无法识图时，依据文件名和 standards 判断；信息不足则报告风险，不得猜测图片内容
- 没有匹配物料时记录为空并继续，不得用占位路径伪装真实资产

**NEXT:** 物料齐备后，进入 Phase 6 生成代码。

### Phase 6: 代码生成

结合目标项目上下文与已确认的 tokens + patterns + assets + standards 生成代码。

- 遵循 Phase 3 确定的技术体系，不因 Theme 推荐依赖而擅自引入或升级包
- 将 Theme tokens 映射为目标项目可复用的变量或配置；优先复用项目现有 Token 机制
- 遵循已加载的 standards，并以选定 Pattern 为结构和视觉参考
- 只引用 Phase 5 确认存在的资产
- **禁止硬编码 Theme 已定义的颜色、间距、字体、圆角、阴影和动效值；必须引用对应 Token**
- 修改完成后运行与变更范围相称的检查；无法运行时明确说明未验证项

> **Token 语义字典（生成代码时必须按语义使用 token）**：见 `references/token-dictionary.md`

## 子流程交叉引用

| 需求 | 读取 |
|------|------|
| 主题搜索/安装/卸载/访问/校验 | `references/theme-management.md` |
| theme.json 格式与 Theme 结构约定 | `references/theme-structure.md` |
| Token 语义字典（6 维度详细表） | `references/token-dictionary.md` |

## 主题询问规则

| 条件 | 行为 |
|------|------|
| 用户指定有效的 Theme `dir` 或唯一 `name` | 直接使用，不询问 |
| 场景只有一个合理候选 | 直接使用，不询问 |
| 场景有多个合理候选 | 展示 `name` + `dir`，简短询问选择哪个 |
| 没有合理候选 | 说明未匹配，并提供主题市场入口 |

**仅询问一次**，不得反复追问。
