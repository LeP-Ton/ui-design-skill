# 改为手动启动并精简主题解析命名

## 背景与目标

- 项目理念调整为不主动干扰用户，只有用户明确要求读取 `SKILL.md` 时才启动 UI 工作流。
- Phase 3 原名称“实现上下文与标准解析”过长，需要使用四字名称概括。

## 约束与原则

- 删除根 `SKILL.md` 的整个 YAML frontmatter，包括 `name`、`description`、`license` 和边界标记。
- 保留文件内的手动调用说明，不增加其他平台触发配置。
- Phase 3 只修改显示名称和衔接文案，不改变其项目上下文、requires 和 standards 解析职责。
- 探索目录中的版本档案保留历史原貌，不随正式入口追改。

## 阶段与 TODO

- [x] 删除自动发现所依赖的 YAML frontmatter。
- [x] 将 Phase 3 命名改为“主题解析”。
- [x] 同步阶段跳转文案。
- [x] 更新 README、AGENTS 和 AgentDocs 索引认知。
- [x] 验证文件入口、阶段名称和旧认知残留。

## 关键风险

- 删除 frontmatter 后，不再兼容依赖 `name` 和 `description` 自动发现 Skill 的平台；这是“仅手动调用”的预期结果。
- “主题解析”是对复合职责的简写，具体边界仍由该阶段正文定义，不能仅凭标题推断为只读 theme.json。

## 当前进展

根 `SKILL.md` 已改为普通 Markdown 手动入口，Phase 3 已统一命名为“主题解析”，核心项目文档已同步。

## 代码变更

- `SKILL.md`
```diff
----
-name: ui-design
-description: Use when creating any web page, UI component, form, landing page, dashboard, mobile interface, or game UI - before writing frontend code. Triggers on 生成页面/创建列表页/设计界面/落地页/营销页/移动端界面/游戏UI/C端页面/管理页面/数据看板/按主题生成. Ensures a theme-based design system (tokens + patterns + standards) is applied instead of hardcoded values.
-license: Complete terms in LICENSE.txt
----
-
-**NEXT:** 主题加载完成后，进入 Phase 3 确定设计标准。
-### Phase 3: 实现上下文与标准解析
-**NEXT:** 实现边界与设计标准确定后，进入 Phase 4 匹配模板。
+**NEXT:** 主题加载完成后，进入 Phase 3 解析主题。
+### Phase 3: 主题解析
+**NEXT:** 主题解析完成后，进入 Phase 4 匹配模板。
```

- `README.md`
```diff
-3. **技术体系选择**：根据 requires 推断技术栈
+3. **主题解析**：解析目标项目上下文、Theme requires 与设计标准
```

- `AGENTS.md`
```diff
-- **SKILL.md**：6 阶段工作流骨架（场景识别 + 主题加载 + 实现上下文与标准解析 + 模板匹配 + 物料匹配 + 代码生成），以 `dir` 贯穿主题选择/安装，先适配目标项目技术体系，再按需组合 Theme 资产；含交叉引用指向 references/
+- **SKILL.md**：无触发 frontmatter 的手动调用入口；6 阶段工作流骨架（场景识别 + 主题加载 + 主题解析 + 模板匹配 + 物料匹配 + 代码生成），以 `dir` 贯穿主题选择/安装，先适配目标项目技术体系，再按需组合 Theme 资产；含交叉引用指向 references/
-- **保留的收益**：SKILL.md 骨架化（14065→5018 字节，省 token）、description 按 SDO 规则优化（只写 when 不写 what）
+- **保留的收益**：SKILL.md 骨架化，详细规则按需加载，降低调用时的上下文消耗
-## description 编写规则（SDO，保留）
- 只写"何时用"（when），绝不写"做什么"（what）——否则模型会自以为懂了而跳过读正文
- 以 `Use when...` 开头，第三人称
- 含"任务阶段词"和"时机锚点"（如 `before writing frontend code`）
- frontmatter 总计 ≤1024 字符
-
+- **无触发 frontmatter**：根 `SKILL.md` 不声明 `name`、`description`、`license`，避免平台按描述自动触发；用户主动要求 AI 读取文件后才启动工作流
```

- `.agentdocs/index.md`
```diff
+`workflow/20260814151727-manual-skill-entry-theme-parse.md` - 删除 SKILL 自动触发 frontmatter，并将 Phase 3 精简命名为“主题解析”
-- description 规则（SDO）：只写 when 不写 what，以 "Use when..." 开头，含时机锚点
+- 启动方式：根 `SKILL.md` 无 name/description/license frontmatter，只在用户主动要求读取文件时启动
```

- 新增文件：`.agentdocs/workflow/20260814151727-manual-skill-entry-theme-parse.md`，记录本次理念调整、具体 diff 和验证方式。

## 测试用例

### TC-001 手动入口无触发元数据
- 类型：静态检查
- 优先级：高
- 操作步骤：检查 `SKILL.md` 文件前十行，并检索 `name:`、`description:`、`license:`。
- 预期结果：文件以手动调用说明注释开头，不包含三个 frontmatter 字段。
- 是否通过：通过

### TC-002 Phase 3 命名一致
- 类型：静态检查
- 优先级：高
- 操作步骤：检索 `Phase 3` 及相邻 `NEXT` 文案。
- 预期结果：标题为“主题解析”，前后跳转均使用对应短名称。
- 是否通过：通过

### TC-003 项目认知同步
- 类型：文档检查
- 优先级：中
- 操作步骤：检索 README、AGENTS 和 AgentDocs 索引中的旧阶段名及 description 规则。
- 预期结果：正式入口相关认知与手动调用理念一致；探索版本中的历史内容不纳入失败判定。
- 是否通过：通过

### TC-004 Skill 通用校验器兼容性
- 类型：兼容性检查
- 优先级：中
- 操作步骤：使用 `skill-creator/scripts/quick_validate.py` 检查项目根目录。
- 预期结果：由于本项目主动取消标准 Skill frontmatter，不以通用自动发现校验通过为目标。
- 是否通过：未执行；本地 Python 缺少 `yaml` 模块。已用等价静态检查确认 frontmatter 不存在，该缺失不影响本次手动入口目标。
