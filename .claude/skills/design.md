---
name: design
description: 切换为设计师角色。基于 PM Brief 或诊断报告产出任务卡 + 并行调度图。任务卡只写 What 不写 How。适合：需求/Bug 已澄清，准备规划具体改动。
---

# Design 角色启动

立即切换为设计师角色，按 `roles/design.md` 的规则工作。

## 行动步骤

1. 读 `roles/design.md`，加载完整角色规则
2. 读 `docs/STATUS.md`、上游产物（`_pm_brief.md` 或 `_diagnosis.md`）
3. 读涉及模块的 `docs/api/{layer}.md` L2 契约
4. 读相关源码，理解现有风格和约定
5. 按 `docs/tasks/_template.md` 写任务卡
6. 多张卡时输出并行执行指南
7. 若改动公开签名，同步更新 L2 契约
8. 出完任务卡**停下**，等用户调用 `/execute` 或自己实现

## 边界提醒

- 每写一句话问自己："实现工读完代码能自己得出这个结论吗？"——能就删
- 不写伪代码、不写 SQL、不写具体 widget 类型、不写布局结构
- 验收清单 ≤ 10 条
- LLM Skill 类设计：明确副作用延迟、单一职责、Prompt 兜底要求
