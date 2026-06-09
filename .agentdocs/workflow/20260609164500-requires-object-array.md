# requires 从字符串数组改为对象数组

## 背景与目标
- requires 之前是字符串数组（如 `["antd", "tailwind"]`），无法同时表达依赖名称和开源地址
- 改为对象数组，每项包含 `name`（依赖标识）和 `source`（开源地址）
- AI 可通过 name 快速判断代码风格，不熟悉时通过 source 获取 API

## 代码变更

### themes/apple-theme/theme.json
```diff
-  "requires": ["css-variables", "tailwind"],
+  "requires": [
+    { "name": "css-variables", "source": "https://developer.mozilla.org/en-US/docs/Web/CSS/Using_CSS_custom_properties" },
+    { "name": "tailwind", "source": "https://github.com/tailwindlabs/tailwindcss" }
+  ],
```

### themes/boss-theme/theme.json
```diff
-  "requires": ["antd"],
+  "requires": [
+    { "name": "antd", "source": "https://github.com/ant-design/ant-design" }
+  ],
```

### themes/game-theme/theme.json
```diff
-  "requires": ["css-variables", "https://github.com/LeP-Ton/poem.git"],
+  "requires": [
+    { "name": "css-variables", "source": "https://developer.mozilla.org/en-US/docs/Web/CSS/Using_CSS_custom_properties" },
+    { "name": "poem", "source": "https://github.com/LeP-Ton/poem.git" }
+  ],
```

### SKILL.md
Phase 3 表格和格式规则整体替换，示例：
```diff
-`requires` 格式规则：
-- 业界主流框架：直接写关键字，如 `"antd"`、`"tailwind"`、`"css-variables"`
-- 需指定版本：关键字@版本号，如 `"antd@5.x"`、`"tailwind@4.0"`
-- 不知名或自定义框架：写仓库地址，如 `"https://github.com/LeP-Ton/poem.git"`
+`requires` 格式规则：对象数组，每项包含 `name`（依赖名称）和 `source`（开源地址）
+- `name`：依赖标识，AI 据此判断代码风格（如 `antd` → antd 组件写法，`css-variables` → CSS 变量写法）
+- `source`：仓库地址或文档地址，AI 不熟悉该依赖时从中获取 API 信息
```

theme.json 格式示例：
```diff
-  "requires": ["antd"],
+  "requires": [
+    { "name": "antd", "source": "https://github.com/ant-design/ant-design" }
+  ],
```

### AGENTS.md
```diff
-- requires：依赖列表，决定代码风格和技术栈。主流框架写关键字（antd/tailwind/css-variables），指定版本写关键字@版本号（antd@5.x），不知名框架写 URL
+- requires：对象数组，每项含 name（依赖标识，决定代码风格）和 source（开源地址，AI 不熟悉时从中获取 API）
```

## 测试用例
### TC-001 对象数组格式正确解析
- 类型：功能测试
- 优先级：高
- 操作步骤：读取 game-theme 的 requires，识别 name 为 poem、source 为 GitHub 仓库
- 预期结果：AI 从 source 获取 poem API 并按 standards/game.md 描述使用
- 是否通过：待验证
