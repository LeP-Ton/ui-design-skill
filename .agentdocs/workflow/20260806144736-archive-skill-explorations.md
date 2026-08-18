# 归档 SKILL 工作流探索方案

## 背景与目标

根目录积累了多个 `SKILL_NEW*`、`SKILL_OLD` 探索稿，命名无法说明差异，也缺少与原始正式版本的映射。此次将有价值的方案集中归档，同时确保根目录 `SKILL.md` 仍是唯一正式入口。

## 约束与原则

- 只归档根目录人工产生的候选稿，不处理 `.history/` 的编辑器自动快照。
- 使用原文件名 `SKILL` 加版本号命名，形成 `SKILL_V{version}.md → SKILL.md` 的路径映射，并保留原候选名或 Git commit 来源。
- 所有方案以 `original@4b513f2` 为共同基线，同时记录真实 `derivedFrom`。
- 归档文件首行使用 HTML 注释，使其成为不可直接发现的设计档案。
- 不修改根目录 `SKILL.md` 的内容，不处理 `.DS_Store` 等无关改动。

## 阶段与 TODO

- [x] 定位原始基线和三个候选方案的 Git 关系。
- [x] 归档现有候选文件并建立 V1.0、V1.1、V1.2、V2.0 版本序列。
- [x] 从 `81c11b5` 提取折中版本。
- [x] 为每个方案补充统一文件头。
- [x] 创建方案映射索引并更新项目认知。
- [x] 验证正文完整性、派生关系和正式入口唯一性。

## 关键风险

- 归档文件仍保留 YAML 正文，但文件首部的档案注释使其不再是合法的直接发现入口；试用时必须从 YAML frontmatter 开始复制。
- 当前工作区还有本任务范围外的未提交改动，提交时必须使用明确路径，避免混入。

## 当前进展

已建立 `explorations/skill-workflows/`，收录 `SKILL.md` 的一个原始基线与三个候选版本；`README.md` 同时提供路径映射、版本来源和派生树。

## 代码变更（见 4.3，使用 git diff）

以下记录依据本次路径范围的 `git diff` 和新增文件清单生成：

- 新增 `explorations/skill-workflows/README.md`，维护来源、基线、派生关系、差异、收益、风险和状态。
- 将根目录 `SKILL_OLD.md`、`SKILL_NEW1.md`、`SKILL_NEW2.md` 迁移为 `SKILL_V1.0.md`、`SKILL_V1.1.md`、`SKILL_V2.0.md`，并补充统一文件头。
- 从 Git commit `81c11b5` 新增 `SKILL_V1.2.md` 折中档案。
- 更新 `README.md` 和 `AGENTS.md`，明确探索目录边界和检索方式。
- 新增文件：`.agentdocs/workflow/20260806144736-archive-skill-explorations.md`，记录本次归档决策和验证方式。

## 测试用例

1. 根目录 `find`：预期只有 `SKILL.md`，不再出现 `SKILL_NEW*` 或 `SKILL_OLD.md`。
2. 文件头检查：预期四个版本文件第一行均为 `<!--`，并包含方案、状态、来源、映射目标、基线、`derivedFrom`、特点、收益、取舍和用途。
3. 正文哈希检查：去除新增档案注释后，三个迁移方案分别与原始文件哈希一致；折中方案与 `81c11b5:SKILL.md` 一致。
4. Git 来源检查：`SKILL_V1.0` 对应 `4b513f2`，`SKILL_V1.1` 对应 `23fa40f`，`SKILL_V1.2` 对应 `81c11b5`。
5. `git diff --check`：预期无空白错误。
