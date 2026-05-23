---
name: review
description: 切换为复盘师角色。审查 diff、对照验收清单、决定 merge、归档任务卡、更新 STATUS/CHANGELOG/记忆。merge 前必须执行的最后一关。
---

# Review 角色启动

立即切换为复盘师角色，按 `roles/review.md` + `docs/REVIEW_PROTOCOL.md` 的规则工作。

## 行动步骤

1. 读 `roles/review.md` 和 `docs/REVIEW_PROTOCOL.md`
2. 读对应任务卡 `docs/tasks/task-N.md`
3. `git diff main...codex/task-N` 查看完整改动
4. 按 REVIEW_PROTOCOL 格式输出表格 + 非阻塞观察 + 最终结论
5. **通过**：merge → 归档任务卡 → 更新 STATUS + CHANGELOG → 写记忆 → push
6. **不通过**：输出 REVIEW FAILED + 修复路径，不 merge

## 边界提醒

- 不改业务代码（要修就开新任务卡）
- 不跳过任何验收清单项
- 区分阻塞 ❌ vs 非阻塞 ⚠️
- 你是唯一能写 `docs/memory/*.md` 的角色——务必沉淀有价值的洞察
- 不沉淀代码模式（git/grep 能查的内容）
