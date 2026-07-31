# Theme 结构约定

> 本文件由 SKILL.md 的「Theme 结构约定」部分抽离，按需加载。
> 创建/校验 theme 或理解 theme.json 格式时读取本文件。

## 目录结构

每个 theme 是 `themes/`（或 `{themeDir}`）下的一个子目录，必须包含 `theme.json`：

```
{theme-name}/
├── theme.json              # 必需：介绍页 + tokens（单文件）
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

## theme.json 格式

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

## 字段说明

### 必需字段

`theme.json` 必需字段：`name`、`version`、`scene`、`tokens`。当某场景只有一个 theme 时，该 theme 自动为该场景的默认主题，无需显式声明。

### 介绍信息字段

- `name` / `version` / `description` / `author` / `scene` / `tags`：介绍信息

### requires 字段

对象数组，每项包含 `name`（依赖名称）和 `source`（开源地址）：
- `name`：依赖标识，AI 据此推断代码风格（如 `antd` → antd 组件写法，`tailwind` → Tailwind 写法）
- `source`：仓库地址或文档地址，AI 不熟悉该依赖时从中获取 API 信息
- `requires` 只放需要安装/引入的外部依赖，浏览器原生能力（如 CSS Variables）不列入

### 不需要的字段

`repository` / `license` / `preview` 不需要在 theme.json 中声明，由统一主题仓库（ui-theme-hub）管理。

## 技术体系选择（Phase 3）

根据 theme.json 中 `requires` 的依赖推断技术栈和代码风格：

| requires 中的依赖 | 推断代码风格 | 典型场景 |
|-------------------|------------|----------|
| 含 `antd` | antd 组件 + ConfigProvider 注入 | B端管理 |
| 含 `tailwind` | Tailwind CSS class + CSS 变量 | C端/营销 |
| 无 UI 框架依赖 | 原生 CSS 变量（`var(--xxx)`） | 游戏 |

`requires` 格式规则：对象数组，每项包含 `name`（依赖名称）和 `source`（开源地址）。

优先级：用户指定 > 项目已有 > theme 声明。
