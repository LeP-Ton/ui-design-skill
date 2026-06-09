# 去中心化市场设计：场景索引内置 SKILL.md + AI 安装时校验

## 背景与目标
- 经过讨论，确定不采用中心化市场方案
- 场景索引和仓库地址直接内置在 SKILL.md 的 Phase 1 场景表中
- 规范靠 SKILL.md 定义，质量靠 AI 安装时校验 theme.json 格式合规性
- 无独立市场网站或索引文件

## 代码变更

### SKILL.md
核心理念：
```diff
-- **主题市场**：theme 从远程市场安装，theme.json 兼作市场介绍页
+- **去中心化**：无中心市场，场景索引和仓库地址内置在 SKILL.md 中，规范靠 SKILL.md 定义，质量靠 AI 安装时校验
```

Phase 1 场景表新增仓库地址列：
```diff
-| 场景 | 关键词 | 默认 Theme |
+| 场景 | 关键词 | 默认 Theme | 仓库地址 |
+|------|--------|-----------|----------|
+| `b-end` | ... | boss-theme | `https://github.com/example/boss-theme.git` |
+| `game` | ... | game-theme | `https://github.com/example/game-theme.git` |
+| `c-end` | ... | apple-theme | `https://github.com/example/apple-theme.git` |
```

Phase 2 主题加载：
```diff
-5. 若该场景的 theme 未安装，提示用户从市场安装
+5. 若该场景的 theme 未安装，从场景表中的仓库地址 `git clone` 安装到 `themes/`
```

新增安装校验：
```diff
+**安装校验**：安装 theme 后，校验 theme.json 格式是否合规：
+- 必需字段完整：`name`、`version`、`scene`、`tokens`
+- tokens 6 维度完整：`color`、`spacing`、`typography`、`border`、`shadow`、`motion`
+- color 基础字段完整：`primary`、`background.base`、`text.primary`、`border.default`
+- 若校验不通过，提示用户修复或更换 theme
```

Phase 2.5 从"主题市场"改为"主题搜索与安装"：
```diff
-### Phase 2.5: 主题市场
+### Phase 2.5: 主题搜索与安装
-- **搜索**：从远程市场仓库列出可用主题
+- **搜索**：根据场景表中的仓库地址，列出可用主题
-- **查询**：先列本地已安装，再提示可从市场搜索更多
+- **查询**：先列本地已安装主题，再按场景表列出可安装主题
```

Theme 安装与卸载：
```diff
-- **安装**：从市场 `git clone` 主题仓库到 `themes/` 目录
+- **安装**：根据场景表中的仓库地址 `git clone` 到 `themes/`，安装后自动校验 theme.json 格式合规性
-- **查询**：先列本地已安装，再提示可从市场搜索更多
+- **查询**：先列本地已安装主题，再按场景表列出可安装主题
```

### AGENTS.md
```diff
-通用前端 UI&UX 设计规范 Claude Code Skill，Theme 插件化架构 + 主题市场。
+通用前端 UI&UX 设计规范 Claude Code Skill，Theme 插件化架构。无中心化市场，场景索引和仓库地址内置在 SKILL.md 中，规范靠 SKILL.md 定义，质量靠 AI 安装时校验。
```

```diff
-- 设计师：维护 Theme（独立 Git 仓库，发布到主题市场）
+- 设计师：维护 Theme（独立 Git 仓库）
-- AI：从市场安装 Theme + 加载 + 组装代码
+- AI：从 SKILL.md 场景表中的仓库地址安装 Theme + 校验 + 加载 + 组装代码
```

```diff
-- 引入主题市场：theme 从远程市场安装，theme.json 兼作市场介绍页，examples/ 改为预览截图
+- 无中心化市场：场景索引和仓库地址内置在 SKILL.md 场景表中，规范靠 SKILL.md 定义，质量靠 AI 安装时校验 theme.json 格式合规性
```

## 测试用例
### TC-001 场景表包含仓库地址
- 类型：检查测试
- 优先级：高
- 操作步骤：读取 SKILL.md Phase 1 场景表，确认每行都有仓库地址列
- 预期结果：5 个场景行都有仓库地址
- 是否通过：待验证

### TC-002 AI 安装时校验 theme.json
- 类型：功能测试
- 优先级：高
- 操作步骤：模拟安装一个格式不合规的 theme（缺少 tokens.spacing），验证 AI 提示修复
- 预期结果：AI 检测到缺失字段并提示
- 是否通过：待验证