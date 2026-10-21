# SKILL.md 工作流版本档案

本目录保存根目录 `SKILL.md` 的原始基线与探索版本。映射规则由目录路径和原文件名共同表达：

```text
explorations/skill-workflows/SKILL_V{version}.md → SKILL.md
```

根目录 `SKILL.md` 始终是唯一正式入口。本目录版本文件首部带档案注释，不应被 Agent 直接识别为 Skill。

## 版本映射

| 版本档案 | 映射目标 | 原来源 | derivedFrom | 方案特点 | 状态 |
|---|---|---|---|---|---|
| `SKILL_V1.0.md` | `SKILL.md` | `SKILL_OLD.md` / `4b513f2` | — | Phase 3 独立选择技术体系 | 原始基线 |
| `SKILL_V1.1.md` | `SKILL.md` | `SKILL_NEW1.md` / `23fa40f` | `SKILL_V1.0` | Phase 3 改为 standards 解析 | 候选 |
| `SKILL_V1.2.md` | `SKILL.md` | `81c11b5:SKILL.md` | `SKILL_V1.1` | Phase 3 合并项目上下文、requires 与 standards | 候选 |
| `SKILL_V2.0.md` | `SKILL.md` | `SKILL_NEW2.md` | `SKILL_V1.1` | 重组为项目分析到验证交付的端到端流程 | 候选 |
| `SKILL_V2.1.md` | `SKILL.md` | 会话-3 | `SKILL_V2.0` | 加入运行、截图、评估、修改与再评估的三轮视觉闭环 | 当前正式版 |

## 版本关系

```text
SKILL.md
└── SKILL_V1.0 (original@4b513f2)
    └── SKILL_V1.1 (23fa40f)
        ├── SKILL_V1.2 (81c11b5)
        └── SKILL_V2.0
            └── SKILL_V2.1 (current)
```

版本号表达方案变化幅度：

- `V1.x`：保持原六阶段总体结构，在阶段职责内部演进。
- `V2.0`：重新划分六阶段职责和执行顺序，属于主流程级变化。
- `V2.1`：沿用 V2.0 阶段职责，补齐基于真实截图的视觉验证、自修正和停止条件。

## 使用约定

- 对比方案时，先看本索引，再读取版本文件顶部的特点与取舍。
- 需要试用某个版本时，从该文件的 YAML frontmatter（首个 `---`）开始复制正文到根目录 `SKILL.md`。
- 不直接注册、软链或发布本目录中的版本文件。
- `.history/` 是编辑器自动快照，不属于正式版本档案。
- 新增探索版本时沿用 `SKILL_V{major}.{minor}.md`，补齐文件头，并在本表登记映射目标、来源、派生关系和状态。
