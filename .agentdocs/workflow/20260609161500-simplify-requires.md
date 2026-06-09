# 简化 requires 字段：从嵌套对象改为数组

## 背景与目标
- `requires.adapters` 嵌套层级冗余，简化为 `requires` 直接作为数组
- 支持三种格式：关键字（antd）、关键字@版本号（antd@5.x）、URL（https://github.com/xxx/yyy.git）
- 语义从"依赖的适配器文件"进一步明确为"主题的依赖项列表"

## 代码变更

### themes/apple-theme/theme.json
```diff
-  "requires": {
-    "adapters": ["css-variables", "tailwind"]
-  },
+  "requires": ["css-variables", "tailwind"],
```

### themes/boss-theme/theme.json
```diff
-  "requires": {
-    "adapters": ["antd"]
-  },
+  "requires": ["antd"],
```

### themes/game-theme/theme.json
```diff
-  "requires": {
-    "adapters": ["css-variables"]
-  },
+  "requires": ["css-variables"],
```

### SKILL.md
Phase 3 整体替换：
```diff
-根据 theme.json 中 `requires.adapters` 确定技术栈：
-
-| adapters | 推断技术栈 | 典型场景 |
-|----------|-----------|----------|
-| `["antd"]` | antd 组件库 | B端管理 |
-| `["css-variables"]` | 原生 CSS 变量 | 游戏 |
-| `["css-variables", "tailwind"]` | Tailwind CSS + CSS 变量 | C端/营销 |
-| `["tailwind"]` | Tailwind CSS | C端/营销 |
+根据 theme.json 中 `requires` 确定技术栈：
+
+| requires 值 | 推断技术栈 | 典型场景 |
+|------------|-----------|----------|
+| `["antd"]` | antd 组件库 | B端管理 |
+| `["antd@5.x"]` | antd 5.x | B端管理（指定版本） |
+| `["css-variables"]` | 原生 CSS 变量 | 游戏 |
+| `["css-variables", "tailwind"]` | Tailwind CSS + CSS 变量 | C端/营销 |
+| `["https://github.com/xxx/yyy.git"]` | 自定义框架（需从 URL 获取） | 特定场景 |
+
+`requires` 格式规则：
+- 业界主流框架：直接写关键字，如 `"antd"`、`"tailwind"`、`"css-variables"`
+- 需指定版本：关键字@版本号，如 `"antd@5.x"`、`"tailwind@4.0"`
+- 不知名或自定义框架：写仓库地址，如 `"https://github.com/LeP-Ton/poem.git"`
```

```diff
-- 根据 `requires.adapters` 声明的代码风格使用 token 值（如 `["css-variables"]` → 用 `var(--xxx)` 引用，`["antd"]` → 用 antd 组件 + ConfigProvider 注入）
+- 根据 `requires` 声明的依赖确定代码风格（如 `["css-variables"]` → 用 `var(--xxx)` 引用，`["antd"]` → 用 antd 组件 + ConfigProvider 注入，URL 类型 → 从仓库获取组件 API 后使用）
```

theme.json 格式示例：
```diff
-  "requires": {
-    "adapters": ["antd"]
-  },
+  "requires": ["antd"],
```

### AGENTS.md
```diff
-- requires.adapters：声明代码风格（同时决定技术栈，如 antd → antd 组件库，css-variables → CSS 变量写法，tailwind → Tailwind CSS）
+- requires：依赖列表，决定代码风格和技术栈。主流框架写关键字（antd/tailwind/css-variables），指定版本写关键字@版本号（antd@5.x），不知名框架写 URL
```

## 测试用例
### TC-001 requires 数组格式正确解析
- 类型：功能测试
- 优先级：高
- 操作步骤：
  1. 读取 boss-theme 的 requires: ["antd"]
  2. AI 识别为 antd 组件库
- 预期结果：正确推断技术栈
- 是否通过：待验证

### TC-002 requires 支持版本号和 URL
- 类型：功能测试
- 优先级：中
- 操作步骤：
  1. 测试 requires: ["antd@5.x"] → 识别为 antd 5.x
  2. 测试 requires: ["https://github.com/LeP-Ton/poem.git"] → 识别为需从 URL 获取的自定义框架
- 预期结果：格式规则正确应用
- 是否通过：待验证
