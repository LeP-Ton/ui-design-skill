# 主题管理详细规则

> 本文件由 SKILL.md 的「主题搜索与安装」和 Phase 2/校验 部分抽离，按需加载。
> 调用 ui-design skill 后，涉及主题增删改查时读取本文件。

## 主题目录配置

主题安装目标由 `theme-config.json`（与 SKILL.md 同目录）中的 `themeDir` 字段决定。

- `themeDir` 必须为**绝对路径**（如 `/Users/xxx/project/themes/`），不支持相对路径
- 首次使用时需先设置 `themeDir`，否则提示用户配置
- `{themeDir}` 在下文中指 theme-config.json 中配置的主题存放绝对路径

## 主题搜索与安装

### 查询主题

当用户提到"搜索主题"、"查询主题"、"有哪些主题"等要求时触发：

从 [themes-summary.json](https://raw.githubusercontent.com/LeP-Ton/ui-theme-hub/main/docs/themes-summary.json) 按条件筛选主题，展示 `name` / `description` / `scene` / `tags`。

### 安装主题

当用户提到"安装主题"、"下载主题"、"使用主题"等要求时触发：

从 GitHub Pages 下载主题 zip 包并解压到 `{themeDir}`，然后校验完整性。安装前先确保 `{themeDir}` 目录存在：

```bash
mkdir -p {themeDir} \
  && curl -fL https://lep-ton.github.io/ui-theme-hub/packages/{theme-id}.zip -o /tmp/{theme-id}.zip \
  && unzip -t /tmp/{theme-id}.zip \
  && unzip -o /tmp/{theme-id}.zip -d {themeDir} \
  && rm /tmp/{theme-id}.zip
```

**参数说明**：
- `-f`：让 curl 在 HTTP 错误时失败（不生成损坏文件）
- `unzip -t`：先验证 zip 完整性再解压
- `mkdir -p`：确保目标目录存在
- `{theme-id}`：主题唯一标识，从 themes-summary.json 获取

### 卸载主题

当用户提到"卸载主题"、"删除主题"、"去除主题"等要求时触发：

删除 `{themeDir}/{theme-name}/` 目录。

### 访问主题仓库

当用户提到"访问主题仓库"、"查看主题仓库"、"打开主题仓库"等要求时触发：

从 `{themeDir}` 目录获取本地已安装主题 ID，主题市场链接支持的 URL 参数如下：

| 参数 | 用途 | 示例 |
|------|------|------|
| `scene` | 按场景筛选 | `?scene=enterprise` |
| `tags` | 按标签筛选（逗号分隔） | `?tags=blue,management` |
| `q` | 搜索关键词回填 | `?q=管理` |
| `installed` | 已安装主题 ID 排前并标记"已下载" | `?installed=boss-theme-blue,apple-theme` |

组合示例：`https://lep-ton.github.io/ui-theme-hub/?scene=enterprise&installed=boss-theme-blue`

按照示例打开浏览器访问主题市场。

## 主题询问规则

| 条件 | 行为 |
|------|------|
| 用户指定 theme | 直接使用，不询问 |
| 场景有唯一 theme | 直接使用，不询问 |
| 场景有多个 theme | 简短询问选择哪个 |
| 场景无 theme | 提示安装默认 theme |

**仅询问一次**，不得反复追问。

## 安装校验

安装 theme 后，校验完整性：

- **必需目录存在**：`patterns/`（若缺失提示重新安装）
- **必需字段完整**：`name`、`version`、`scene`、`tokens`
- **tokens 6 维度完整**：`color`、`spacing`、`typography`、`border`、`shadow`、`motion`
- **color 基础字段完整**：`primary`、`background.base`、`text.primary`、`border.default`
- 若校验不通过，提示用户修复或重新安装
