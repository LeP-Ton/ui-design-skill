# AgentDocs 索引

## 当前变更文档
`workflow/20260731140000-revert-to-platform-agnostic.md` - 回退 plugin/hook，改为平台无关手动调用 skill（保留骨架化 + SDO）
`workflow/20260731133000-superpowers-auto-trigger-plugin.md` - 借鉴 superpowers 自动触发机制：升级为 Plugin + SessionStart hook 注入 bootstrap（已回退）
`workflow/20260713093000-install-stability-themedir.md` - 主题安装稳定性提升 + themeDir 配置化 + URL 修复
`workflow/20260710160000-zip-install.md` - 主题安装方式改为 zip 下载，提升稳定性
`workflow/20260706154000-split-index-two-layer.md` - themes-index.json 拆为两层：轻量索引 + 按需详情
`workflow/20260610093000-del-scene-table-4phases.md` - 删除 SKILL.md 硬编码场景表，场景识别改为动态语义匹配
`workflow/20260609181500-init-ui-theme-hub.md` - 初始化 ui-theme-hub 统一主题仓库
`workflow/20260609180000-unified-theme-repo.md` - 统一主题仓库：从去中心化改为集中管理
`workflow/20260609174500-decentralized-market.md` - 去中心化市场：场景索引内置 SKILL.md + AI 安装时校验
`workflow/20260609173000-theme-market.md` - 主题市场设计：theme.json 新增市场字段 + examples 改为预览截图
`workflow/20260609170000-remove-css-variables-require.md` - requires 移除 css-variables（浏览器原生能力不属于外部依赖）
`workflow/20260609164500-requires-object-array.md` - requires 从字符串数组改为对象数组（name + source）
`workflow/20260609163000-game-poem-dep.md` - game-theme 新增 poem 文字动画库依赖及使用规范
`workflow/20260609161500-simplify-requires.md` - requires 从嵌套对象简化为数组，支持关键字@版本号和 URL 格式
`workflow/20260609160000-remove-core.md` - 删除 core/ 目录，架构从 Core+Theme 简化为纯 Theme 插件化
`workflow/20260609154500-remove-adapters.md` - 删除 core/adapters/ 适配器文件
`workflow/20260609153000-remove-registry.md` - 删除 theme.json 中 registry 字段，技术栈推断改为 requires.adapters
`workflow/20260609152000-remove-isDefaultForScene.md` - 删除 theme.json 冗余字段 isDefaultForScene
`workflow/20260609144815-core-theme-architecture.md` - Core + Theme 插件化架构重构（含 Token 语义字典设计思路）

## 关键记忆
- 项目定位：通用前端 UI&UX 设计规范 skill，**开源、平台无关**，Theme 插件化架构
- 架构：SKILL.md（骨架）+ references/（按需加载）+ themes/{theme}
- 设计哲学：**手动调用而非自动触发**——用户有 UI 需求时主动让 AI 读取 SKILL.md；不采用 superpowers 的 hook 自动触发（平台专属，会引入多 agent 兼容问题）
- 平台无关：skill 内容不写死平台专属变量（${CLAUDE_PLUGIN_ROOT}）或工具名（Skill 工具），任何 agent 能读文件即可用
- 渐进式披露：SKILL.md 骨架 5018 字节（原 14065），详细规则按需从 references/ 加载
- description 规则（SDO）：只写 when 不写 what，以 "Use when..." 开头，含时机锚点
- Token 格式：参考 W3C Design Tokens 分组思路，简化为扁平 JSON + references/token-dictionary.md 语义字典
- 协作模型：设计师维护 Theme，AI 负责组装
- 已安装 Theme：boss-theme(B端) / game-theme(游戏) / apple-theme(C端/营销/移动)
- 使用方式：任意 agent 让 AI 读取 SKILL.md；Claude Code 可软链到 ~/.claude/skills/ui-design
