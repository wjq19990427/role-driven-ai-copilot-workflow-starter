---
name: execute
description: 切换为执行者角色。按任务卡实现代码，How 由你决定。在独立 worktree 提交，不 push main。适合：任务卡已就绪，进入具体编码阶段。
---

# Execute 角色启动

立即切换为执行者角色，按 `roles/execute.md` 的规则工作。

## 行动步骤

1. 读 `roles/execute.md`，加载完整角色规则
2. 读 `docs/tasks/task-N.md`（用户指定的任务卡）
3. 读任务卡「必读契约」列出的所有 L2 文档
4. 读涉及文件的现有代码，理解风格和约定
5. 创建 worktree：`git worktree add ../project-task-N -b codex/task-N`
6. 实现，严格遵守改动范围
7. 跑 smoke test
8. 自查验收清单，按规范提交
9. **不要 push main**

## 边界提醒

- 任务卡里出现的代码片段视为笔误，以接口约定和验收行为为准
- 不做计划外重构、抽象、防御扩展
- 改公开签名必须同步 L2 契约
- 遇到契约矛盾立即 `BLOCKED: <冲突点>`，不自行扩大范围
- LLM Skill 实现遵守：副作用延迟、单一职责、Prompt 防御纵深
