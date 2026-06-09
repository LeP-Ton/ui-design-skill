---
name: ui-design
description: 通用前端 UI&UX 设计规范 skill。当用户提到"生成页面"、"创建列表页"、"构建表单"、"设计界面"、"落地页"、"营销页"、"移动端界面"、"游戏UI"、"C端页面"、"产品首页"、"按主题生成"、"UI设计"、"页面设计"、"管理页面"、"数据看板"等场景时触发此 skill。基于 Theme 插件化架构，Theme 为可插拔的设计资产包，AI 负责组装。
license: Complete terms in LICENSE.txt
---

# UI Design Skill — Theme 插件化架构

## 核心理念

- **Theme 可以独立演进**：每个 theme 自带完整的设计资产
- **设计师负责 Theme**，**AI 负责组装**
- **统一主题仓库**：所有主题集中在一个 Git 仓库（`ui-theme-hub`）中管理，AI 从该仓库发现和安装主题，规范靠 SKILL.md 定义，质量靠 AI 安装时校验

## 工作流（6 阶段）

### Phase 1: 场景识别

根据用户输入中的关键词识别场景：

| 场景 | 关键词 | 默认 Theme |
|------|--------|-----------|
| `b-end` | 管理、后台、列表、表格、表单、仪表盘、数据看板 | boss-theme |
| `game` | 游戏、HUD、背包、技能栏、角色面板 | game-theme |
| `c-end` | 首页、产品、个人中心、社交、电商 | apple-theme |
| `marketing` | 落地页、营销、推广、Landing、Banner | apple-theme |
| `mobile` | 移动端、H5、小程序、App、手机 | apple-theme |

识别优先级：用户指定场景 > 用户指定 theme > 关键词匹配 > 简短询问一次

### Phase 2: 主题加载

根据场景查找已安装 theme：

1. 扫描 `themes/` 目录下的子目录
2. 读取每个子目录的 `theme.json`（市场介绍 + tokens）
3. 匹配场景对应的默认 theme
4. 若用户指定 theme 名，直接加载
5. 若该场景的 theme 未安装，从统一主题仓库安装到 `themes/`

加载 theme 后，按需读取：
- `theme.json` 中的 `tokens`：Design Token
- `patterns/`：页面/区块模板
- `standards/`：设计标准规范
- `assets/`：物料素材

**安装校验**：安装 theme 后，校验 theme.json 格式是否合规：
- 必需字段完整：`name`、`version`、`scene`、`tokens`
- tokens 6 维度完整：`color`、`spacing`、`typography`、`border`、`shadow`、`motion`
- color 基础字段完整：`primary`、`background.base`、`text.primary`、`border.default`
- 若校验不通过，提示用户修复或更换 theme

### Phase 2.5: 主题搜索与安装

统一主题仓库：`https://github.com/example/ui-theme-hub`

当用户提到"搜索主题"、"安装主题"、"有哪些主题"时触发：

- **搜索**：从统一主题仓库列出所有可用主题（读取各主题目录的 theme.json，展示 name/description/scene/tags/preview）
- **安装**：从统一主题仓库复制需要的主题目录到本地 `themes/`，然后校验 theme.json 格式合规性
- **卸载**：删除 `themes/{theme-name}/` 目录
- **查询**：先列本地已安装主题，再从统一主题仓库列出可安装主题

### Phase 3: 技术体系选择

根据 theme.json 中 `requires` 的依赖推断技术栈和代码风格：

| requires 中的依赖 | 推断代码风格 | 典型场景 |
|-------------------|------------|----------|
| 含 `antd` | antd 组件 + ConfigProvider 注入 | B端管理 |
| 含 `tailwind` | Tailwind CSS class + CSS 变量 | C端/营销 |
| 无 UI 框架依赖 | 原生 CSS 变量（`var(--xxx)`） | 游戏 |

`requires` 格式规则：对象数组，每项包含 `name`（依赖名称）和 `source`（开源地址）
- `name`：依赖标识，AI 据此推断代码风格（如 `antd` → antd 组件写法，`tailwind` → Tailwind 写法）
- `source`：仓库地址或文档地址，AI 不熟悉该依赖时从中获取 API 信息
- `requires` 只放需要安装/引入的外部依赖，浏览器原生能力（如 CSS Variables）不列入

优先级：用户指定 > 项目已有 > theme 声明

### Phase 4: 模板匹配

从 theme 的 `patterns/` 中选择模板：
- `patterns/pages/`：页面级模板
- `patterns/blocks/`：区块级模板

若 theme 中无匹配模板，基于 token 和 standards 生成自定义页面。

### Phase 5: 物料匹配

从 theme 的 `assets/` 目录引用物料（icons/illustrations/fragments）。

### Phase 6: 代码生成

结合 token + pattern + standard 生成代码。
- 根据 `requires` 中的依赖推断代码风格（如含 `antd` → antd 组件 + ConfigProvider，含 `tailwind` → Tailwind class，无 UI 框架 → 原生 CSS 变量 `var(--xxx)`，不熟悉的依赖 → 从 source 获取 API）
- 遵循 theme 的 standards/（如果有的话）
- 禁止硬编码颜色/间距/圆角，必须使用 token

## 主题询问规则

| 条件 | 行为 |
|------|------|
| 用户指定 theme | 直接使用，不询问 |
| 场景有唯一 theme | 直接使用，不询问 |
| 场景有多个 theme | 简短询问选择哪个 |
| 场景无 theme | 提示安装默认 theme |

**仅询问一次**，不得反复追问。

## Theme 结构约定

每个 theme 是 `themes/` 下的一个子目录，必须包含 `theme.json`：

```
{theme-name}/
├── theme.json              # 必需：市场介绍 + tokens（单文件）
├── patterns/               # 可选：页面/区块模板
│   ├── pages/
│   └── blocks/
├── standards/              # 可选：场景专属设计规范
├── assets/                 # 可选：物料素材
│   ├── icons/
│   ├── illustrations/
│   └── fragments/
└── examples/               # 可选：市场预览截图（.png/.webp）
```

### theme.json 格式

```json
{
  "name": "boss-theme",
  "version": "1.0.0",
  "description": "B端管理系统主题",
  "author": "设计团队",
  "scene": "b-end",
  "tags": ["b-end", "management"],
  "requires": [
    { "name": "antd", "source": "https://github.com/ant-design/ant-design" }
  ],
  "preview": ["examples/dashboard.png", "examples/list-page.png"],
  "repository": "https://github.com/example/boss-theme.git",
  "license": "MIT",
  "tokens": {
    "color": {
      "primary": "#1677ff",
      "secondary": "#597ef7",
      "accent": "#13c2c2",
      "success": "#52c41a",
      "warning": "#faad14",
      "error": "#ff4d4f",
      "info": "#1677ff",
      "background": { "base": "#ffffff", "elevated": "#ffffff", "muted": "#f5f5f5" },
      "text": { "primary": "rgba(0,0,0,0.88)", "secondary": "rgba(0,0,0,0.65)", "muted": "rgba(0,0,0,0.45)", "inverse": "#ffffff" },
      "border": { "default": "#d9d9d9", "muted": "#f0f0f0" }
    },
    "spacing": { "unit": 4, "scale": [0, 4, 8, 12, 16, 20, 24, 32, 40, 48, 64, 80, 96] },
    "typography": {
      "fontFamily": { "base": "...", "heading": "...", "code": "..." },
      "fontSize": { "xs": 12, "sm": 13, "base": 14, "lg": 16, "xl": 20, "2xl": 24, "3xl": 30, "4xl": 36 },
      "fontWeight": { "normal": 400, "medium": 500, "semibold": 600, "bold": 700 },
      "lineHeight": { "tight": 1.25, "base": 1.5, "relaxed": 1.75 }
    },
    "border": { "radius": { "none": 0, "sm": 4, "base": 6, "lg": 8, "xl": 12, "full": 9999 }, "width": { "thin": 1, "base": 1, "thick": 2 } },
    "shadow": { "sm": "...", "base": "...", "lg": "...", "xl": "..." },
    "motion": { "enabled": true, "duration": { "fast": "0.1s", "base": "0.2s", "slow": "0.3s" }, "easing": { "standard": "cubic-bezier(0.4,0,0.2,1)", "decelerate": "cubic-bezier(0,0,0.2,1)", "accelerate": "cubic-bezier(0.4,0,1,1)" } }
  }
}
```

`theme.json` 必需字段：`name`、`version`、`scene`、`tokens`。当某场景只有一个 theme 时，该 theme 自动为该场景的默认主题，无需显式声明。

### Token 语义字典

生成代码时，必须按以下语义使用 token 值，不可凭猜测：

#### color（色彩）

| Token | 语义 | 用在哪里 |
|-------|------|----------|
| `color.primary` | 品牌主色 | 主按钮、链接、选中态、进度条 |
| `color.secondary` | 次要色 | 辅助按钮、次要图标 |
| `color.accent` | 强调色 | 少量点缀高亮 |
| `color.success` | 成功色 | 成功状态标签、成功提示 |
| `color.warning` | 警告色 | 警告状态标签、警告提示 |
| `color.error` | 错误色 | 错误状态标签、错误提示、删除按钮 |
| `color.info` | 信息色 | 信息提示 |
| `color.background.base` | 页面最底层背景 | body、layout 背景 |
| `color.background.elevated` | 浮层背景 | 卡片、弹窗、下拉面板 |
| `color.background.muted` | 次要背景 | 表头、分割区域、代码块背景 |
| `color.text.primary` | 主要文字 | 标题、正文 |
| `color.text.secondary` | 次要文字 | 描述、标签、辅助信息 |
| `color.text.muted` | 最弱文字 | 占位符、禁用文字、注释 |
| `color.text.inverse` | 反色文字 | 主色背景上的文字（按钮内文字） |
| `color.border.default` | 默认边框 | 输入框、卡片边框 |
| `color.border.muted` | 淡边框 | 分割线、表格内边框 |
| `color.extended.*` | 场景专属扩展 | 由 theme 自定义，如 tableHeaderBg、sidebarBg 等 |

#### spacing（间距）

| Token | 语义 | 用在哪里 |
|-------|------|----------|
| `spacing.unit` | 基础单位（4px） | 所有间距值的计算基础 |
| `spacing.scale` | 间距阶梯数组 | 索引对应：[0]=0, [1]=4, [2]=8, [3]=12, [4]=16, [5]=20, [6]=24, [7]=32, [8]=40, [9]=48, [10]=64, [11]=80, [12]=96 |
| `spacing.extended.*` | 场景专属扩展 | 如 headerHeight、sidebarWidth 等 |

**间距层级规则**：
- 页面/场景间距 → scale[10-12]（64-96px）
- 区域/区块间距 → scale[7-8]（32-40px）
- 卡片/组件间距 → scale[4]（16px）
- 元素组间距 → scale[2]（8px）
- 同组元素间距 → scale[1]（4px）

#### typography（字体）

| Token | 语义 | 用在哪里 |
|-------|------|----------|
| `typography.fontFamily.base` | 正文字体 | body、段落、输入框 |
| `typography.fontFamily.heading` | 标题字体 | h1-h6、页面标题 |
| `typography.fontFamily.code` | 代码字体 | 代码块、行内代码 |
| `typography.fontSize.xs` | 12px | 辅助文字、标签 |
| `typography.fontSize.sm` | 13px | 小文字、表格辅助信息 |
| `typography.fontSize.base` | 14px | 正文（默认） |
| `typography.fontSize.lg` | 16px | 大正文、小标题 |
| `typography.fontSize.xl` | 20px | 区块标题 |
| `typography.fontSize.2xl` | 24px | 页面标题 |
| `typography.fontSize.3xl` | 30px | 大标题 |
| `typography.fontSize.4xl` | 36px | Hero 标题 |
| `typography.fontWeight.normal` | 400 | 正文 |
| `typography.fontWeight.medium` | 500 | 强调文字 |
| `typography.fontWeight.semibold` | 600 | 小标题 |
| `typography.fontWeight.bold` | 700 | 大标题 |
| `typography.lineHeight.tight` | 1.25 | 标题行高 |
| `typography.lineHeight.base` | 1.5 | 正文行高 |
| `typography.lineHeight.relaxed` | 1.75 | 长文行高 |

#### border（边框/圆角）

| Token | 语义 | 用在哪里 |
|-------|------|----------|
| `border.radius.none` | 0 | 无圆角 |
| `border.radius.sm` | 4px | 小圆角：标签、小按钮 |
| `border.radius.base` | 6px | 基础圆角：输入框、按钮、卡片 |
| `border.radius.lg` | 8px | 大圆角：弹窗、大卡片 |
| `border.radius.xl` | 12px | 超大圆角：C端卡片 |
| `border.radius.full` | 9999px | 完全圆角：头像、圆形按钮 |
| `border.width.thin` | 1px | 细边框：分割线 |
| `border.width.base` | 1px | 默认边框：输入框、卡片 |
| `border.width.thick` | 2px | 粗边框：焦点态 |

#### shadow（阴影）

| Token | 语义 | 用在哪里 |
|-------|------|----------|
| `shadow.sm` | 微阴影 | 卡片默认态 |
| `shadow.base` | 基础阴影 | 按钮、下拉 |
| `shadow.lg` | 大阴影 | 悬浮态卡片、弹窗 |
| `shadow.xl` | 超大阴影 | 全屏弹窗 |

#### motion（动效）

| Token | 语义 | 用在哪里 |
|-------|------|----------|
| `motion.enabled` | 是否启用动效 | false 时跳过所有动画 |
| `motion.duration.fast` | 0.1s | hover、焦点变化 |
| `motion.duration.base` | 0.2s | 展开/收起、显隐切换 |
| `motion.duration.slow` | 0.3s | 页面切换、弹窗出现 |
| `motion.easing.standard` | 标准缓动 | 大部分场景 |
| `motion.easing.decelerate` | 减速缓动 | 进入动画 |
| `motion.easing.accelerate` | 加速缓动 | 退出动画 |

每个 token 维度都可添加 `extended` 字段存放场景专属扩展值，`extended` 中的字段由 theme 自定义，需在 theme 的 `standards/` 中说明其用途。

## Theme 安装与卸载

- **安装**：从统一主题仓库复制主题目录到 `themes/`，或手动放入 `themes/`。安装后自动校验 theme.json 格式合规性
- **卸载**：删除 `themes/{theme-name}/` 目录即可
- **查询**：先列本地已安装主题，再按场景表列出可安装主题

## 引用路径

- Theme 内容：`themes/{theme-name}/` 下各子目录
