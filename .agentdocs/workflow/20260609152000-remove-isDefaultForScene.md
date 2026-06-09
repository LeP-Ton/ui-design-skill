# theme.json 删除冗余字段 isDefaultForScene

## 背景与目标
- theme.json 中 `isDefaultForScene` 字段在当前场景下永远为 true，属于冗余字段
- 当某场景只有一个 theme 时，该 theme 自动为该场景的默认主题，无需显式声明
- 删除该字段，简化 theme.json 结构

## 约束与原则
- 仅删除 `isDefaultForScene`，其余 theme.json 字段保留
- 不改变 AI 工作流逻辑，仅将"显式声明默认"改为"隐式约定默认"

## 阶段与 TODO
- [x] 删除三个 theme.json 中的 isDefaultForScene 字段
- [x] 更新 SKILL.md 中 theme.json 格式说明
- [x] 更新 SKILL.md 必需字段说明，补充默认主题约定
- [x] 更新 AGENTS.md 项目认知

## 代码变更

### themes/apple-theme/theme.json
```diff
   "tags": ["c-end", "marketing", "mobile", "apple", "elegant"],
-  "isDefaultForScene": true,
   "requires": {
```

### themes/boss-theme/theme.json
```diff
   "tags": ["b-end", "management", "dashboard", "enterprise"],
-  "isDefaultForScene": true,
   "requires": {
```

### themes/game-theme/theme.json
```diff
   "tags": ["game", "hud", "dark", "monitoring"],
-  "isDefaultForScene": true,
   "requires": {
```

### SKILL.md
```diff
-  "isDefaultForScene": true,
   "requires": {
```

```diff
-`theme.json` 必需字段：`name`、`version`、`scene`、`tokens`、`registry`
+`theme.json` 必需字段：`name`、`version`、`scene`、`tokens`、`registry`。当某场景只有一个 theme 时，该 theme 自动为该场景的默认主题，无需显式声明。
```

### AGENTS.md
```diff
-- name/version/scene/tags/isDefaultForScene
+- name/version/scene/tags
```

## 测试用例
### TC-001 场景识别仍能正确匹配默认主题
- 类型：功能测试
- 优先级：高
- 关联模块：SKILL.md Phase 2
- 前置条件：themes/ 目录下三个 theme 完整
- 操作步骤：
  1. AI 识别场景为 b-end
  2. 扫描 themes/ 目录
  3. 匹配 boss-theme（scene=b-end，且为唯一匹配）
- 预期结果：
  - 自动选择 boss-theme，无需用户确认
  - 行为与删除前一致
- 是否通过：待验证
