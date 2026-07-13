# 主题安装稳定性提升 + themeDir 配置化 + URL 修复

## 背景与目标
- 原安装方式为"从统一主题仓库复制主题目录"，没有具体步骤，AI 手动复制容易遗漏文件
- 引入 `theme-config.json` 配置主题存放目录（`themeDir`），仅支持绝对路径，skill 不再硬编码主题存储位置
- 安装方式改为 `curl + unzip` 下载 zip 包，加入完整性校验
- 修复 GitHub Pages URL 错误（`le-p-ton` → `lep-ton`，去掉多余的 `docs/` 前缀）
- 新增主题市场 URL 参数说明（scene/tags/q/installed）

## 约束与原则
- themeDir 必须为绝对路径，不支持相对路径
- 安装命令通过 `curl -f` + `unzip -t` + `mkdir -p` 保证稳定性
- 不硬编码 skill 安装路径，适配所有 agent 平台

## 阶段与 TODO
- [x] 安装方式改为 zip 下载（curl + unzip），加入校验步骤
- [x] 新增 theme-config.json，themeDir 仅支持绝对路径
- [x] 修复 GitHub Pages zip 包 URL（用户名和路径前缀）
- [x] 新增主题市场 URL 参数表格（scene/tags/q/installed）
- [x] 主题搜索与安装改为按触发词分类（查询/安装/卸载/访问）
- [x] Phase 2 读取 theme-config.json 获取 themeDir
- [x] AGENTS.md 更新 theme-config.json 描述

## 代码变更

### 新增 theme-config.json

```diff
+ {
+   "themeDir": "/Users/didi/Desktop/ui-themes"
+ }
```

### SKILL.md — 主题搜索与安装

```diff
- 主题安装目标为本 SKILL.md 所在目录下的 `themes/`（即 AI 读取此文件时，此文件的父目录就是 skill 根目录，记为 `{SKILL_DIR}`）。
+ 主题安装目标由 `theme-config.json`（与 SKILL.md 同目录）中的 `themeDir` 字段决定。`themeDir` 必须为绝对路径（如 `/Users/xxx/project/themes/`），不支持相对路径。首次使用时需先设置 `themeDir`，否则提示用户配置。
+ 
+ `{themeDir}` 指 theme-config.json 中配置的主题存放绝对路径。

- - **查询**：...若用户想浏览预览图和交互式筛选，提供带参数的主题市场链接：`https://lep-ton.github.io/ui-theme-hub/?scene={scene}&tags={tags}`
- - **安装**：从 GitHub Pages 下载主题 zip 包并解压到 `{SKILL_DIR}/themes/`，然后校验完整性：
-   ```
-   curl -L https://le-p-ton.github.io/ui-theme-hub/packages/{theme-id}.zip -o /tmp/{theme-id}.zip \
-     && unzip -o /tmp/{theme-id}.zip -d {SKILL_DIR}/themes/ \
-     && rm /tmp/{theme-id}.zip
-   ```
- - **卸载**：删除 `themes/{theme-name}/` 目录
+ - 当用户提到"搜索主题"、"查询主题"、"有哪些主题"等要求时触发：
+   **查询**：从 themes-summary.json 按条件筛选主题，展示 name/description/scene/tags
+ - 当用户提到"安装主题"、"下载主题"、"使用主题"等要求时触发：
+   **安装**：从 GitHub Pages 下载主题 zip 包并解压到 `{themeDir}`，然后校验完整性。安装前先确保 `{themeDir}` 目录存在：
+     ```
+     mkdir -p {themeDir} \
+       && curl -fL https://lep-ton.github.io/ui-theme-hub/packages/{theme-id}.zip -o /tmp/{theme-id}.zip \
+       && unzip -t /tmp/{theme-id}.zip \
+       && unzip -o /tmp/{theme-id}.zip -d {themeDir} \
+       && rm /tmp/{theme-id}.zip
+     ```
+ - 当用户提到"卸载主题"、"删除主题"、"去除主题"等要求时触发：
+   **卸载**：删除 `{themeDir}/{theme-name}/` 目录
+ - 当用户提到"访问主题仓库"，"查看主题仓库"，"打开主题仓库"等要求时触发：
+   **访问**：从`{themeDir}` 目录获取本地已安装主题 ID，主题市场链接支持的 URL 参数如下：
+     | 参数 | 用途 | 示例 |
+     |------|------|------|
+     | `scene` | 按场景筛选 | `?scene=enterprise` |
+     | `tags` | 按标签筛选（逗号分隔） | `?tags=blue,management` |
+     | `q` | 搜索关键词回填 | `?q=管理` |
+     | `installed` | 已安装主题 ID 排前并标记"已下载" | `?installed=boss-theme-blue,apple-theme` |
+     组合示例：`https://lep-ton.github.io/ui-theme-hub/?scene=enterprise&installed=boss-theme-blue`
+     按照示例打开浏览器访问主题市场
```

### SKILL.md — Phase 2 主题加载

```diff
- 1. 先读取 `theme-config.json` 获取 `themeDir`（默认 `./themes/`）
+ 1. 先读取 `theme-config.json` 获取 `themeDir`（绝对路径，若未配置则提示用户设置）
```

### AGENTS.md — 新增 theme-config.json 描述

```diff
  ## 架构
  - **SKILL.md**：完整引擎逻辑
  - **统一主题仓库（ui-theme-hub）**：集中管理所有主题
+ - **theme-config.json**：主题目录配置文件，`themeDir` 字段指定主题存放目录（仅支持绝对路径）
  - **themes/**：本地已安装 theme 目录（默认）
```

## 测试用例
### TC-001 zip 安装完整性
- 类型：功能测试
- 优先级：高
- 操作步骤：
  1. 执行安装命令安装 cyberpunk-theme
  2. 检查目录是否完整（含 patterns/standards/assets）
- 预期结果：目录完整，assets/ 空目录存在
- 是否通过：已验证通过

### TC-002 themeDir 未配置时的提示
- 类型：功能测试
- 优先级：中
- 操作步骤：
  1. 清空 theme-config.json 中的 themeDir
  2. 触发安装流程
- 预期结果：提示用户先配置 themeDir
- 是否通过：待验证

### TC-003 GitHub Pages URL 可达性
- 类型：功能测试
- 优先级：高
- 操作步骤：
  1. curl -sI https://lep-ton.github.io/ui-theme-hub/packages/boss-theme-blue.zip
- 预期结果：HTTP 200
- 是否通过：已验证通过
