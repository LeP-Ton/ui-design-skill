# Token 语义字典

> 本文件由 SKILL.md 的「Token 语义字典」部分抽离，按需加载。
> 生成代码时，必须按以下语义使用 token 值，不可凭猜测。

## color（色彩）

| Token | 语义 | 用在哪里 |
|-------|------|----------|
| `color.primary` | 品牌主色 | 主按钮、链接、选中态、进度条 |
| `color.secondary` | 次要色 | 辅助按钮、次要图标 |
| `color.accent` | 强调色 | 少量点缀高亮 |
| `color.success` | 成功色 | 成功状态标签、成功提示 |
| `color.warning` | 警告色 | 警告状态标签、警告提示 |
| `color.error` | 错误色 | 错误状态标签、错误提示、删除按钮 |
| `color.info` | 信息色 | 信息提示 |
| `color.background.base` | 页面最底层背景 | body、layout 背景 |
| `color.background.elevated` | 浮层背景 | 卡片、弹窗、下拉面板 |
| `color.background.muted` | 次要背景 | 表头、分割区域、代码块背景 |
| `color.text.primary` | 主要文字 | 标题、正文 |
| `color.text.secondary` | 次要文字 | 描述、标签、辅助信息 |
| `color.text.muted` | 最弱文字 | 占位符、禁用文字、注释 |
| `color.text.inverse` | 反色文字 | 主色背景上的文字（按钮内文字） |
| `color.border.default` | 默认边框 | 输入框、卡片边框 |
| `color.border.muted` | 淡边框 | 分割线、表格内边框 |
| `color.extended.*` | 场景专属扩展 | 由 theme 自定义，如 tableHeaderBg、sidebarBg 等 |

## spacing（间距）

| Token | 语义 | 用在哪里 |
|-------|------|----------|
| `spacing.unit` | 基础单位（4px） | 所有间距值的计算基础 |
| `spacing.scale` | 间距阶梯数组 | 索引对应：[0]=0, [1]=4, [2]=8, [3]=12, [4]=16, [5]=20, [6]=24, [7]=32, [8]=40, [9]=48, [10]=64, [11]=80, [12]=96 |
| `spacing.extended.*` | 场景专属扩展 | 如 headerHeight、sidebarWidth 等 |

**间距层级规则**：
- 页面/场景间距 → scale[10-12]（64-96px）
- 区域/区块间距 → scale[7-8]（32-40px）
- 卡片/组件间距 → scale[4]（16px）
- 元素组间距 → scale[2]（8px）
- 同组元素间距 → scale[1]（4px）

## typography（字体）

| Token | 语义 | 用在哪里 |
|-------|------|----------|
| `typography.fontFamily.base` | 正文字体 | body、段落、输入框 |
| `typography.fontFamily.heading` | 标题字体 | h1-h6、页面标题 |
| `typography.fontFamily.code` | 代码字体 | 代码块、行内代码 |
| `typography.fontSize.xs` | 12px | 辅助文字、标签 |
| `typography.fontSize.sm` | 13px | 小文字、表格辅助信息 |
| `typography.fontSize.base` | 14px | 正文（默认） |
| `typography.fontSize.lg` | 16px | 大正文、小标题 |
| `typography.fontSize.xl` | 20px | 区块标题 |
| `typography.fontSize.2xl` | 24px | 页面标题 |
| `typography.fontSize.3xl` | 30px | 大标题 |
| `typography.fontSize.4xl` | 36px | Hero 标题 |
| `typography.fontWeight.normal` | 400 | 正文 |
| `typography.fontWeight.medium` | 500 | 强调文字 |
| `typography.fontWeight.semibold` | 600 | 小标题 |
| `typography.fontWeight.bold` | 700 | 大标题 |
| `typography.lineHeight.tight` | 1.25 | 标题行高 |
| `typography.lineHeight.base` | 1.5 | 正文行高 |
| `typography.lineHeight.relaxed` | 1.75 | 长文行高 |

## border（边框/圆角）

| Token | 语义 | 用在哪里 |
|-------|------|----------|
| `border.radius.none` | 0 | 无圆角 |
| `border.radius.sm` | 4px | 小圆角：标签、小按钮 |
| `border.radius.base` | 6px | 基础圆角：输入框、按钮、卡片 |
| `border.radius.lg` | 8px | 大圆角：弹窗、大卡片 |
| `border.radius.xl` | 12px | 超大圆角：C端卡片 |
| `border.radius.full` | 9999px | 完全圆角：头像、圆形按钮 |
| `border.width.thin` | 1px | 细边框：分割线 |
| `border.width.base` | 1px | 默认边框：输入框、卡片 |
| `border.width.thick` | 2px | 粗边框：焦点态 |

## shadow（阴影）

| Token | 语义 | 用在哪里 |
|-------|------|----------|
| `shadow.sm` | 微阴影 | 卡片默认态 |
| `shadow.base` | 基础阴影 | 按钮、下拉 |
| `shadow.lg` | 大阴影 | 悬浮态卡片、弹窗 |
| `shadow.xl` | 超大阴影 | 全屏弹窗 |

## motion（动效）

| Token | 语义 | 用在哪里 |
|-------|------|----------|
| `motion.enabled` | 是否启用动效 | false 时跳过所有动画 |
| `motion.duration.fast` | 0.1s | hover、焦点变化 |
| `motion.duration.base` | 0.2s | 展开/收起、显隐切换 |
| `motion.duration.slow` | 0.3s | 页面切换、弹窗出现 |
| `motion.easing.standard` | 标准缓动 | 大部分场景 |
| `motion.easing.decelerate` | 减速缓动 | 进入动画 |
| `motion.easing.accelerate` | 加速缓动 | 退出动画 |

## extended 扩展说明

每个 token 维度都可添加 `extended` 字段存放场景专属扩展值，`extended` 中的字段由 theme 自定义，需在 theme 的 `standards/` 中说明其用途。
