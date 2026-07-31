# 本地主题检索：补"零维护"探测命令，替代索引文件

## 背景与目标
- design-skill 检索本地主题时缺乏索引层。`references/theme-management.md` 原文"从 `{themeDir}` 目录获取本地已安装主题 ID"措辞模糊，AI 只能 `ls` + 逐个 `cat theme.json`，违背"渐进式披露 / 省 token"哲学。
- 评估两种方案：① 在 `{themeDir}` 维护 `installed-summary.json` 派生索引；② 固化一条本地探测命令直接读源文件。
- **采用方案②**：派生索引有一致性维护成本（安装/卸载需同步更新，用户手动删目录会脏数据），且本地主题数量少，索引收益不显著；探测命令零维护、即时正确。

## 约束与原则
- 保持平台无关：只用 `grep`/`for`/`ls`/`xargs`/`paste`，任何能跑 bash 的 agent 可用。
- 不引入需维护的派生数据文件，保持项目"简洁、不引入多余机制"哲学。
- 行为不变：仅把模糊描述写实成可执行命令，不改变检索语义。

## 阶段与 TODO
- [x] 在"查询主题"补本地主题检索命令（id/scene/tags）
- [x] 在"访问主题仓库"把"获取本地已安装主题 ID"写实成可执行命令
- [x] 校验两条命令在当前 themeDir 下实际可用
- [x] 更新 .agentdocs/index.md 与 AGENTS.md

## 关键风险
- zsh 默认 `nomatch`：glob 无匹配时报 `no matches found` 中断命令。已修复——TC-001 加 `setopt nullglob`，TC-002 改用 `find`。bash 不受影响。
- `find` 输出顺序不保证字典序，`?installed=` 参数顺序会变化。不影响功能（市场页只按 id 匹配，与顺序无关）。

## 当前进展
- `references/theme-management.md` 两处补命令，均为零维护探测式，不新增索引文件。
- 边界修复：实测发现 zsh 默认 `nomatch` 会在 glob 无匹配时报 `no matches found`。TC-001 加 `setopt nullglob 2>/dev/null`（bash 静默忽略）；TC-002 改用 `find -mindepth 1 -maxdepth 1` 替代 `ls -d */`，空目录静默无报错。真实目录与空目录均实测通过。

## 代码变更
- references/theme-management.md（"查询主题"小节，新增本地主题检索命令块）

```diff
 从 [themes-summary.json](https://raw.githubusercontent.com/LeP-Ton/ui-theme-hub/main/docs/themes-summary.json) 按条件筛选主题，展示 `name` / `description` / `scene` / `tags`。

+**本地主题检索**：当需要列出本地已安装主题（id/scene/tags）时，直接读源文件，无需维护索引文件（避免派生数据不一致）。一条命令提取所有本地主题的关键字段：
+
+```bash
+# 列出 {themeDir} 下所有本地已安装主题的 id/scene/tags
+# setopt nullglob：兼容 zsh，目录为空时 glob 不报错（bash 默认即此行为）
+setopt nullglob 2>/dev/null
+for f in {themeDir}/*/theme.json; do
+  echo "=== $(dirname "$f" | xargs basename) ==="
+  grep -E '"(name|scene|tags)"' "$f" | head -3
+done
+```
+
+> 该命令直接读源头 `theme.json`，零维护成本；用户手动增删主题后结果即时正确，不存在索引脏数据问题。
+
 ### 安装主题
```

- references/theme-management.md（"访问主题仓库"小节，把模糊描述写实成命令）

```diff
-从 `{themeDir}` 目录获取本地已安装主题 ID，主题市场链接支持的 URL 参数如下：
+从 `{themeDir}` 目录获取本地已安装主题 ID。提取所有本地主题 id（取目录名，无需逐个读 theme.json）：
+
+```bash
+# 输出 {themeDir} 下所有已安装主题的 id（目录名），逗号分隔，用于拼装市场 URL 的 installed 参数
+# 用 find -mindepth 1 -maxdepth 1 兼容空目录，避免 zsh 的 nomatch 报错
+find {themeDir} -mindepth 1 -maxdepth 1 -type d -exec basename {} \; | paste -sd ',' -
+```
+
+主题市场链接支持的 URL 参数如下：
```

## 测试用例
### TC-001 本地主题检索命令输出正确
- 类型：功能测试
- 优先级：高
- 关联模块：references/theme-management.md「查询主题」
- 前置条件：themeDir（/Users/didi/Desktop/ui-themes）已存在且含 ai-ppt-theme、boss-theme-orange
- 操作步骤：
  1. 执行 for 循环命令（{themeDir} 替换为实际路径）
- 预期结果：
  - 逐个打印各主题目录名 + name/scene/tags 三行
  - 空目录时不报错（zsh 加 setopt nullglob）
- 是否通过：通过（实测 ai-ppt-theme、boss-theme-orange 正确输出；空目录静默无报错）

### TC-002 installed 参数拼装命令输出正确
- 类型：功能测试
- 优先级：中
- 关联模块：references/theme-management.md「访问主题仓库」
- 前置条件：同 TC-001
- 操作步骤：
  1. 执行 `find {themeDir} -mindepth 1 -maxdepth 1 -type d -exec basename {} \; | paste -sd ',' -`
- 预期结果：
  - 输出逗号分隔的目录名（如 `boss-theme-orange,ai-ppt-theme`，顺序不限）
  - 空目录时输出空，不报错
- 是否通过：通过（实测输出 `boss-theme-orange,ai-ppt-theme`；空目录静默无报错）
