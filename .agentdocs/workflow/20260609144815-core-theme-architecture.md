# Core + Theme 插件化架构重构

## 背景与目标
- 原架构为单体结构，所有规范/模板/主题/示例打包在一起，随场景增多会持续膨胀
- 重构为 Core + Theme 插件化架构：Core 永远很小，Theme 可独立演进、动态下载
- 设计师负责 Theme，前端负责 Core，AI 负责组装

## 约束与原则
- Core 不含任何预设内容（场景/主题/规范），纯运行时引擎
- Theme 是格式约定、来源不限（git clone / npm / 手动复制）
- 每个场景一个默认 theme
- 协作模型：设计师维护 Theme，前端维护 Core，AI 负责组装

## 阶段与 TODO
- [x] 创建 Core 引擎（engine/index/rules/adapters）
- [x] 创建 boss-theme（B端默认主题）
- [x] 创建 game-theme（游戏默认主题）
- [x] 创建 apple-theme（C端/营销/移动主题）
- [x] 重写 SKILL.md
- [x] 合并引擎逻辑到 SKILL.md（解决 core/ 文件不被自动加载的问题）
- [x] 合并 tokens 为单个 theme.json（解决 6 文件读取成本高的问题）
- [x] 新增通用标准 core/standards/（解决跨主题通用规范无处放的问题）
- [x] 补全 Token 语义字典（解决 AI 无法推断 token 用途的问题）
- [x] 更新 AGENTS.md 和 README.md

## 关键设计决策

### 1. 引擎逻辑内联到 SKILL.md
- **问题**：core/engine.md、index.md、rules.md 不会被 Claude 自动读取，写了等于白写
- **决策**：把 3 个文件的核心逻辑全部合并进 SKILL.md，core/ 只保留适配器和通用标准
- **原因**：Claude 触发 skill 时只看 SKILL.md

### 2. tokens 合并为单个 theme.json
- **问题**：原来每个 theme 有 manifest.json + 6 个 token JSON + registry.json = 8 个文件，AI 需读 8 次
- **决策**：合并为 theme.json 一个文件，包含 manifest + tokens + registry
- **原因**：token 量不大，没必要拆碎；单文件一次读取，效率最高

### 3. 通用标准独立存放
- **问题**：无障碍、响应式等规范是跨主题通用的，但 core 说不含预设内容，每个 theme 又不该重复维护
- **决策**：core/standards/ 放通用标准，theme.json 中通过 `requires.sharedStandards` 声明依赖
- **原因**：通用标准写一次，theme 声明引用即可

### 4. Token 格式与语义字典
- **格式标准**：参考 W3C Design Tokens Community Group（https://design-tokens.github.io/community-group/format/）的分组思路（color/spacing/typography/border/shadow/motion），但简化了嵌套——W3C 标准 $value/$type 对 Claude 不友好，用扁平 JSON
- **语义字典**：仅靠字段名（如 `color.background.elevated`）AI 无法推断用途，所以在 SKILL.md 中为每个 token 字段提供语义+使用场景说明
- **extended 扩展**：每个 token 维度可添加 extended 字段存放场景专属值，自定义字段需在 theme 的 standards/ 中说明用途

## 代码变更

### 新增文件
```diff
+ SKILL.md                          # 重写：完整引擎逻辑 + Token 语义字典
+ core/engine.md                    # 引擎定义（后合并到 SKILL.md，已删除）
+ core/index.md                     # 索引机制（后合并到 SKILL.md，已删除）
+ core/rules.md                     # 规则引擎（后合并到 SKILL.md，已删除）
+ core/adapters/antd.md             # antd 适配器
+ core/adapters/tailwind.md         # Tailwind 适配器
+ core/adapters/css-variables.md    # CSS Variables 适配器
+ core/standards/accessibility.md   # 通用无障碍标准
+ core/standards/responsive.md      # 通用响应式标准
+ core/standards/interaction.md     # 通用交互标准
+ themes/boss-theme/theme.json      # B端主题（合并 manifest+tokens+registry）
+ themes/boss-theme/patterns/       # 页面+区块模板（4+6）
+ themes/boss-theme/standards/      # B端专属规范
+ themes/boss-theme/examples/       # 示例代码
+ themes/game-theme/theme.json      # 游戏主题
+ themes/game-theme/patterns/       # 游戏模板
+ themes/game-theme/standards/      # 游戏专属规范
+ themes/apple-theme/theme.json     # C端/营销/移动主题
+ themes/apple-theme/patterns/      # C端+营销+移动模板
+ themes/apple-theme/standards/     # C端/营销/移动专属规范
```

### 删除文件
```diff
- themes/presets/                    # 旧预设主题目录
- themes/schemas/                    # 旧 schema 目录
- themes/scenes/                     # 旧场景 token 目录
- themes/adapters/                   # 旧适配器（迁移到 core/adapters/）
- themes/index.json                  # 旧主题索引
- standards/                         # 旧顶层标准目录
- references/                        # 旧参考文档目录
- assets/                            # 旧顶层物料目录
- templates/                         # 旧顶层模板目录
- examples/                          # 旧顶层示例目录
- core/engine.md                     # 合并到 SKILL.md 后删除
- core/index.md                      # 合并到 SKILL.md 后删除
- core/rules.md                      # 合并到 SKILL.md 后删除
- themes/*/tokens/                   # 合并到 theme.json 后删除
- themes/*/manifest.json             # 合并到 theme.json 后删除
- themes/*/components-registry/      # 合并到 theme.json 后删除
```

## 测试用例
### TC-001 Core 不含预设内容
  - 类型：结构验证
  - 优先级：高
  - 操作步骤：
  1. 检查 core/ 目录只有 adapters/ 和 standards/
  - 预期结果：无场景/主题/规范具体内容
  - 是否通过：待验证

### TC-002 theme.json 完整性
  - 类型：结构验证
  - 优先级：高
  - 操作步骤：
  1. 读取每个 theme 的 theme.json
  - 预期结果：包含 name/version/scene/tokens/registry 五个必需字段

### TC-003 通用标准引用
  - 类型：功能测试
  - 优先级：中
  - 操作步骤：
  1. 检查 theme.json 中 requires.sharedStandards
  2. 检查 core/standards/ 对应文件存在
  - 预期结果：boss-theme 声明依赖 accessibility/responsive/interaction，对应文件均存在

### TC-004 Token 语义可理解
  - 类型：功能测试
  - 优先级：高
  - 操作步骤：
  1. 触发 /ui-design 生成 B 端列表页
  2. 检查生成代码中的背景色使用
  - 预期结果：卡片背景用 tokens.color.background.elevated，不是 base 或 muted
