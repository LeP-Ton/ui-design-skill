<!--
方案：标准解析优先版
状态：候选方案
来源：原 SKILL_NEW1.md；对应 Git commit 23fa40f 的 SKILL.md
映射目标：SKILL.md
基线：original@4b513f2
derivedFrom：SKILL_V1.0
特点：将 Phase 3 从“技术体系选择”改为“标准解析”，强调 standards 作为设计总纲，并补充 assets 的文字说明降级策略。
主要收益：强化 Theme 设计规范与物料语义在生成过程中的作用。
主要取舍：目标项目技术栈直到代码生成阶段才处理，容易较晚发现 Theme 依赖冲突。
用途：SKILL.md 的 V1.1 设计档案，不是可直接发现的 Skill；试用时从下方 YAML frontmatter 开始复制到根目录 SKILL.md。
-->

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

> **REQUIRED:** 完整执行 6 阶段，不得跳过。识别完场景后用「下一步」指引进入下一阶段。
> **主题搜索/安装/卸载/访问/校验的详细命令与规则**：见 `references/theme-management.md`

### Phase 1: 场景识别

读取统一主题仓库的 `docs/themes-summary.json`，根据 `scene` + `tags` + `description` 语义匹配用户意图：

1. 从 themes-summary.json 中提取各 theme 的 `scene`、`tags`、`description`
2. 将用户输入与各 theme 的 `scene`、`tags`、`description` 语义信息做匹配
3. 匹配到主题后，若匹配结果只有一个 theme 则直接使用，多个则询问选择
4. 未匹配到任何主题时，提示没有匹配的主题

识别优先级：用户指定场景 > 用户指定 theme > 语义匹配

**NEXT:** 场景识别完成后，进入 Phase 2 加载主题。

### Phase 2: 主题加载

加载 Phase 1 选择的 theme：

1. 先读取 `theme-config.json`（与 SKILL.md 同目录）获取 `themeDir`（绝对路径，若未配置则提示用户设置）
2. 若用户指定 theme 名，直接加载 `{themeDir}` 目录已安装的 theme
3. 若识别到的 theme 未安装，从 GitHub Pages 下载主题 zip 包解压到 `{themeDir}`

加载 theme 后，按需读取：
- `theme.json` 中的 `tokens`：Design Token
- `patterns/`：页面/组件模板
- `standards/`：设计标准规范
- `assets/`：物料素材

**NEXT:** 主题加载完成后，进入 Phase 3 选择技术体系。

### Phase 3: 标准解析

从 theme 的 `standards/` 目录解析标准规范作为设计总纲。

**NEXT:** 设计标准规范确定后，进入 Phase 4 匹配模板。

### Phase 4: 模板匹配

从 theme 的 `patterns/` 中选择模板：
- `patterns/pages/`：页面级模板
- `patterns/blocks/`：区块级模板

若 theme 中无匹配模板，基于 token 和 standards 生成自定义页面。

**NEXT:** 模板选定后，进入 Phase 5 匹配物料。

### Phase 5: 物料匹配

从 theme 的 `assets/` 目录引用物料（icons / illustrations / fragments）。

- 参考 `standards/` 中的物料介绍和使用方法
- 当大模型不支持识图能力时：可根据 `standards/` 中的物料介绍掌握物料信息和使用方法，若物料信息不足，需向用户报告风险。

**NEXT:** 物料齐备后，进入 Phase 6 生成代码。

### Phase 6: 代码生成

结合 tokens + patterns + assets + standards 生成代码。

- 根据 `requires` 中的依赖推断代码风格（含 `antd` → antd 组件 + ConfigProvider，含 `tailwind` → Tailwind class，无 UI 框架 → 原生 CSS 变量 `var(--xxx)`，不熟悉的依赖 → 从 source 获取 API）
- 遵循 theme 的 standards/（如果有的话）
- **禁止硬编码颜色/间距/圆角，必须使用 token**

> **requires 字段格式与技术栈推断详细规则**：见 `references/theme-structure.md`
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
| 用户指定 theme | 直接使用，不询问 |
| 场景有唯一 theme | 直接使用，不询问 |
| 场景有多个 theme | 简短询问选择哪个 |
| 场景无 theme | 提示安装默认 theme |

**仅询问一次**，不得反复追问。
