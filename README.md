# UI Design Skill

通用前端 UI&UX 设计规范 skill，采用 Core + Theme 插件化架构。

## 架构

```
Core（引擎）+ Theme（可插拔设计资产包）
```

- **Core 永远很小**：SKILL.md 内含完整引擎逻辑 + core/ 只保留适配器和通用标准
- **Theme 可以独立演进**：每个 theme 自带 theme.json + patterns/standards/assets/examples
- **设计师负责 Theme**，**前端负责 Core**，**AI 负责组装**

## 目录结构

```
├── SKILL.md                    # Skill 主入口（含完整引擎逻辑）
├── core/
│   ├── adapters/               # 技术适配器
│   │   ├── antd.md
│   │   ├── tailwind.md
│   │   └── css-variables.md
│   └── standards/              # 通用标准（跨主题复用）
│       ├── accessibility.md
│       ├── responsive.md
│       └── interaction.md
└── themes/                     # Theme 目录（动态下载）
    ├── boss-theme/             # B端管理主题
    ├── game-theme/             # 游戏UI主题
    └── apple-theme/            # C端/营销/移动主题
```

## Theme 结构

```
{theme-name}/
├── theme.json          # manifest + tokens + registry（单文件）
├── patterns/           # 页面/区块模板
├── standards/          # 场景专属规范
├── assets/             # 物料素材
└── examples/           # 示例代码
```

theme.json 必需字段：`name`、`version`、`scene`、`tokens`、`registry`

## 已安装 Theme

| Theme | 场景 | 技术栈 |
|-------|------|--------|
| boss-theme | B端管理 | antd 5.x |
| game-theme | 游戏UI | 自定义组件 |
| apple-theme | C端/营销/移动 | React + CSS/Tailwind |

## 安装/卸载 Theme

```bash
# 安装
git clone https://github.com/design-team/steam-theme.git themes/steam-theme

# 卸载
rm -rf themes/steam-theme
```

## 使用

```
/ui-design 生成一个用户管理列表页
/ui-design 创建一个产品落地页
/ui-design 用 game-theme 设计游戏HUD
```
