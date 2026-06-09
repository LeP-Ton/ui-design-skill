# requires 移除 css-variables（浏览器原生能力不属于外部依赖）

## 背景与目标
- CSS Variables 是浏览器原生能力，不需要安装或引入，不应出现在 requires 依赖项中
- requires 只放需要安装/引入的外部依赖
- 代码风格推断逻辑调整：无 UI 框架依赖 → 默认使用原生 CSS 变量写法

## 代码变更

### themes/apple-theme/theme.json
```diff
   "requires": [
-    { "name": "css-variables", "source": "https://developer.mozilla.org/en-US/docs/Web/CSS/Using_CSS_custom_properties" },
     { "name": "tailwind", "source": "https://github.com/tailwindlabs/tailwindcss" }
   ],
```

### themes/game-theme/theme.json
```diff
   "requires": [
-    { "name": "css-variables", "source": "https://developer.mozilla.org/en-US/docs/Web/CSS/Using_CSS_custom_properties" },
     { "name": "poem", "source": "https://github.com/LeP-Ton/poem.git" }
   ],
```

### SKILL.md
Phase 3 技术栈推断表：
```diff
-| requires 中的依赖 | 推断代码风格 | 典型场景 |
-|----------|-----------|----------|
-| 含 `antd` | antd 组件 + ConfigProvider 注入 | B端管理 |
-| 含 `tailwind` | Tailwind CSS class + CSS 变量 | C端/营销 |
-| 含 `css-variables` | 原生 CSS 变量（`var(--xxx)`） | 游戏 |
+| requires 中的依赖 | 推断代码风格 | 典型场景 |
+|-------------------|------------|----------|
+| 含 `antd` | antd 组件 + ConfigProvider 注入 | B端管理 |
+| 含 `tailwind` | Tailwind CSS class + CSS 变量 | C端/营销 |
+| 无 UI 框架依赖 | 原生 CSS 变量（`var(--xxx)`） | 游戏 |
```

```diff
-- `name`：依赖标识，AI 据此判断代码风格（如 `antd` → antd 组件写法，`css-variables` → CSS 变量写法）
+- `name`：依赖标识，AI 据此推断代码风格（如 `antd` → antd 组件写法，`tailwind` → Tailwind 写法）
+- `requires` 只放需要安装/引入的外部依赖，浏览器原生能力（如 CSS Variables）不列入
```

Phase 6：
```diff
-- 根据 `requires` 中的依赖推断代码风格（如含 `antd` → antd 组件 + ConfigProvider，含 `tailwind` → Tailwind class，无 UI 框架 → 原生 CSS 变量 `var(--xxx)`，不熟悉的依赖 → 从 source 获取 API）
+- 根据 `requires` 中的依赖推断代码风格（如含 `antd` → antd 组件 + ConfigProvider，含 `tailwind` → Tailwind class，无 UI 框架依赖 → 原生 CSS 变量 `var(--xxx)`，不熟悉的依赖 → 从 source 获取 API）
```

### AGENTS.md
```diff
-- requires：对象数组，每项含 name（依赖标识，决定代码风格）和 source（开源地址，AI 不熟悉时从中获取 API）
+- requires：对象数组，每项含 name（依赖标识，推断代码风格）和 source（开源地址，AI 不熟悉时从中获取 API）。只放外部依赖，浏览器原生能力（CSS Variables）不列入
```

```diff
-| boss-theme | B端管理（默认） | antd（requires.adapters: ["antd"]） |
-| game-theme | 游戏UI（默认） | CSS Variables（requires.adapters: ["css-variables"]） |
-| apple-theme | C端/营销/移动（默认） | CSS Variables + Tailwind（requires.adapters: ["css-variables", "tailwind"]） |
+| boss-theme | B端管理（默认） | antd |
+| game-theme | 游戏UI（默认） | 原生 CSS 变量 + poem |
+| apple-theme | C端/营销/移动（默认） | Tailwind CSS |
```

## 测试用例
### TC-001 game-theme 无 UI 框架依赖时默认 CSS 变量写法
- 类型：功能测试
- 优先级：高
- 操作步骤：用 game-theme 生成代码，requires 中无 antd/tailwind
- 预期结果：AI 默认使用 var(--xxx) 写法
- 是否通过：待验证
