# UI Design Skill

通用前端 UI&UX 设计规范 skill，采用 Theme 插件化架构。平台无关——任何能读取文件的 AI agent 都能使用。

## 定位

让 AI 产出**符合设计标准**的 UI。用户有 UI 需求时，手动让 AI 读取 `SKILL.md` 即可触发完整的 6 阶段工作流。

- **不依赖特定平台的触发机制**（hook / plugin / instructions file），因此无需为每个 agent 单独适配
- **任何 agent 能读文件即可使用**：把本仓库放在 agent 可访问的目录，需要时让 AI 读取 `SKILL.md`

## 架构

```
SKILL.md（骨架）+ references/（按需加载）+ {themeDir}（设计资产，用户配置的本地主题目录）
```

- **渐进式披露（progressive disclosure）**：SKILL.md 只保留 6 阶段工作流骨架，详细规则按需从 references/ 加载，避免一次性消耗大量 token
- **Theme 可以独立演进**：每个 theme 自带 theme.json + patterns/standards/assets/examples
- **设计师负责 Theme**，**AI 负责组装**

## 目录结构

```
├── SKILL.md                    # Skill 骨架（6 阶段工作流 + 交叉引用）
├── references/                 # 按需加载的详细规则
│   ├── theme-management.md     # 主题搜索/安装/卸载/校验
│   ├── theme-structure.md      # theme.json 格式与结构约定
│   └── token-dictionary.md     # Token 语义字典（6 维度）
├── explorations/
│   └── skill-workflows/        # SKILL_V{version}.md → SKILL.md 的探索版本映射（不作为运行入口）
├── theme-config.json           # 主题目录配置（themeDir 字段，绝对路径，指向本地已安装主题的唯一目录）
└── themes/                     # skill 自带示例主题（仅供参考，不参与检索/安装/校验）
    └── apple-theme/            # C端/营销/移动主题示例
```

## 使用方式

### 在任意 agent 中

把本仓库放在 agent 可访问的目录，然后对 AI 说：

```
读取 /path/to/ui-design-skill/SKILL.md，按它的规范帮我生成一个用户管理列表页
```

AI 会读取 SKILL.md，按 6 阶段工作流产出符合主题规范的 UI 代码。

> 根目录 `SKILL.md` 是唯一正式入口。历史基线与候选工作流统一收录在 `explorations/skill-workflows/`，不可直接注册为 Skill。

### 在 Claude Code 中（可选）

如果用 Claude Code 并希望以 skill 形式注册，可软链到 `~/.claude/skills/`：

```bash
ln -s /path/to/ui-design-skill ~/.claude/skills/ui-design
```

之后可用 `/ui-design` 或在对话中让 AI 调用。

## 6 阶段工作流

1. **场景识别**：语义匹配 themes-summary.json 选择主题
2. **主题加载**：读取/安装主题到 themeDir
3. **主题解析**：解析目标项目上下文、Theme requires 与设计标准
4. **模板匹配**：从 patterns/ 选择页面/区块模板
5. **物料匹配**：从 assets/ 引用素材
6. **代码生成**：结合 token + pattern + standard 生成代码（禁止硬编码颜色/间距）

## Theme 结构

```
{theme-name}/
├── theme.json          # 介绍信息 + tokens（单文件）
├── patterns/           # 页面/区块模板
├── standards/          # 场景专属规范
├── assets/             # 物料素材
└── examples/           # 市场预览截图
```

theme.json 必需字段：`name`、`version`、`scene`、`tokens`

## 配置主题目录

编辑 `theme-config.json`，将 `themeDir` 设为存放主题的绝对路径：

```json
{
  "themeDir": "/Users/yourname/ui-themes"
}
```

## 统一主题仓库

所有主题集中管理在 [ui-theme-hub](https://github.com/LeP-Ton/ui-theme-hub)，AI 从该仓库发现和安装主题。新主题通过 PR 贡献。
