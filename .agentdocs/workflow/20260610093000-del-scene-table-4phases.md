# 删除 SKILL.md 硬编码场景表，场景识别改为动态语义匹配

## 背景与目标
- SKILL.md Phase 1 维护了一份硬编码的「场景→关键词→默认 Theme」映射表，与 theme.json 中的 `scene`/`tags`/`description` 字段信息冗余
- 新主题发布后必须同步更新 SKILL.md 场景表，维护成本高
- AI 语义理解能力足够，不需要硬编码中文关键词做映射

## 约束与原则
- 不新增字段（如 triggers），复用现有 tags + description
- 场景识别逻辑改为算法描述，不再维护具体映射数据
- 保持原有识别优先级不变

## 阶段与 TODO
- [x] 删除 Phase 1 硬编码场景表，改为动态语义匹配算法描述
- [x] 简化 Phase 2 主题加载步骤（场景识别已在 Phase 1 完成）
- [x] 修改 Theme 安装与卸载中的「按场景表」引用为动态扫描

## 关键风险
- AI 语义匹配可能对非常规关键词匹配不到，但有兜底询问机制

## 当前进展
- 已完成所有变更，SKILL.md 不再包含任何硬编码场景数据

## 代码变更

### SKILL.md — Phase 1: 场景识别

```diff
- ### Phase 1: 场景识别
- 
- 根据用户输入中的关键词识别场景：
- 
- | 场景 | 关键词 | 默认 Theme |
- |------|--------|-----------|
- | `b-end` | 管理、后台、列表、表格、表单、仪表盘、数据看板 | boss-theme |
- | `game` | 游戏、HUD、背包、技能栏、角色面板 | game-theme |
- | `c-end` | 首页、产品、个人中心、社交、电商 | apple-theme |
- | `marketing` | 落地页、营销、推广、Landing、Banner | apple-theme |
- | `mobile` | 移动端、H5、小程序、App、手机 | apple-theme |
- 
- 识别优先级：用户指定场景 > 用户指定 theme > 关键词匹配 > 简短询问一次
+ ### Phase 1: 场景识别
+ 
+ 扫描 `themes/` 下所有 theme.json，根据 `scene` + `tags` + `description` 语义匹配用户意图：
+ 
+ 1. 读取每个 theme.json 的 `scene`、`tags`、`description`
+ 2. 将用户输入与各 theme 的语义信息做匹配
+ 3. 匹配到场景后，若该场景只有一个 theme 则直接使用，多个则询问选择
+ 4. 未匹配到任何场景时，简短询问一次
+ 
+ 识别优先级：用户指定场景 > 用户指定 theme > 语义匹配 > 简短询问一次
```

### SKILL.md — Phase 2: 主题加载

```diff
- 根据场景查找已安装 theme：
- 
- 1. 扫描 `themes/` 目录下的子目录
- 2. 读取每个子目录的 `theme.json`（市场介绍 + tokens）
- 3. 匹配场景对应的默认 theme
- 4. 若用户指定 theme 名，直接加载
- 5. 若该场景的 theme 未安装，从统一主题仓库安装到 `themes/`
+ 加载 Phase 1 识别到的 theme：
+ 
+ 1. 若用户指定 theme 名，直接加载
+ 2. 若识别到的 theme 未安装，从统一主题仓库安装到 `themes/`
```

### SKILL.md — Theme 安装与卸载

```diff
- - **查询**：先列本地已安装主题，再按场景表列出可安装主题
+ - **查询**：先列本地已安装主题，再从统一主题仓库列出可安装主题（按 `scene`/`tags` 分组展示）
```

## 测试用例
### TC-001 语义匹配 B 端场景
- 类型：功能测试
- 优先级：高
- 关联模块：Phase 1 场景识别
- 前置条件：boss-theme 已安装
- 操作步骤：
  1. 输入"创建一个工单管理页面"
  2. AI 扫描 theme.json，匹配到 boss-theme（scene: b-end, tags: management）
- 预期结果：自动选择 boss-theme，无需询问
- 是否通过：待验证

### TC-002 未匹配时兜底询问
- 类型：功能测试
- 优先级：中
- 关联模块：Phase 1 场景识别
- 前置条件：无
- 操作步骤：
  1. 输入"创建一个 XX 页面"（XX 无法匹配任何 theme）
- 预期结果：简短询问一次，不反复追问
- 是否通过：待验证
