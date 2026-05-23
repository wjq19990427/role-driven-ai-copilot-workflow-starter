# 角色交接协议

> 五个角色之间的输入/输出格式与流转规则。这是工作流闭环的关键。

---

## 完整流转图

```
用户混乱输入
      ↓
   ┌──/pm──┐
   │       │
   ↓       ↓
PM Brief 不是 Bug → /design
  ↓ (Bug)
/diagnose
  ↓
诊断报告
  ↓
/design
  ↓
任务卡 + 并行指南
  ↓
/execute（可能多 worktree 并行）
  ↓
diff + commit
  ↓
/review
  ↓
Review 报告 → [通过] merge + 归档 + 记忆更新 → 回到用户
            → [不通过] 回到 /execute 修复
```

---

## 文档载体一览

| 阶段 | 文件 | 生命周期 |
|---|---|---|
| PM 阶段 | `docs/_pm_brief.md` | 临时（设计完成后删除或归档到 `docs/archive/`） |
| Diagnose 阶段 | `docs/_diagnosis.md` | 临时（任务卡产出后删除） |
| Design 阶段 | `docs/tasks/task-N.md` | 持久（merge 后移到 `archive/`） |
| Execute 阶段 | git diff + commit | 永久存于 git |
| Review 阶段 | Review 报告（追加到任务卡末尾） | 跟随任务卡归档 |

带下划线前缀的（`_pm_brief.md`、`_diagnosis.md`）是临时文件，不进 git 长期历史。

---

## 各角色交接规范

### `/pm` → `/diagnose` 或 `/design`

**PM Brief 必须包含**：
- 每个需求的「下一步」字段，指向 `/diagnose` 或 `/design`
- 「待用户决策」清单，至少标记一次

**交接动作**：
1. PM 输出 Brief 后**停止**
2. 用户确认或调整需求
3. 用户显式调用下游触发词
4. 下游角色读 Brief 作为输入

**交接失败信号**：
- Brief 中没有「下一步」字段
- 待决策项未解决就被调用下游

---

### `/diagnose` → `/design`

**诊断报告必须包含**：
- 根因（不是症状的同义复述）
- 每个根因的「文件:行号」证据
- 「涉及文件」清单（完整，不遗漏关联文件）
- 「影响半径」分析
- 「下一步」字段指向 `/design`

**交接动作**：
1. 诊断师输出报告后**停止**
2. 用户调用 `/design`
3. 设计师读诊断报告 + PM Brief（如有）作为输入

**交接失败信号**：
- 证据不足以定位根因，但报告强行给了结论
- 「涉及文件」遗漏关键文件
- 偷偷给了修复方案

---

### `/design` → `/execute`

**任务卡必须包含**：
- 「变更说明」（用户视角，1-2 句）
- 「必读契约」（至少 1 项 L2 文档引用）
- 「改动范围」（修改/新建/不许碰 三类）
- 「接口约定」（函数签名 + 行为 + 副作用 + 约束）
- 「验收清单」≤ 10 条

**多张任务卡时必须包含**：
- 「⚡ 并行执行指南」：Wave N 分组

**交接动作**：
1. 设计师输出任务卡后**停止**
2. 用户确认任务卡可执行（无 BLOCKED 项）
3. 用户调用 `/execute` 或自己实现

**交接失败信号**：
- 任务卡包含伪代码或具体实现细节
- 验收清单超 10 条且未拆卡
- 多张任务卡但缺并行指南
- 「必读契约」字段为空

---

### `/execute` → `/review`

**执行者交付物**：
- branch：`codex/task-N` 或 `claude/task-N`
- commit message 符合规范：`<type>(<scope>): <描述> · 关联 #N`
- smoke test 通过的证明（控制台日志或简单说明）
- 任务卡验收清单的自查结果

**交接动作**：
1. 执行者完成实现 + 自查后**提交到自己分支**（不 push main）
2. 用户调用 `/review`
3. 复盘师读 `git diff main...codex/task-N` + 任务卡

**交接失败信号**：
- 直接 push 到 main（违反基本纪律）
- 改动超出任务卡「改动范围」
- 改了「不许碰」清单内的文件
- 公开签名变更但 L2 契约未同步

**BLOCKED 协议**：

执行中遇到以下情况立即停手，输出 `BLOCKED: <冲突点>`：

- 任务卡接口约定与 L2 契约矛盾
- 依赖的函数/类不存在于当前 HEAD
- 改动必然影响「不许碰」范围

不自行扩大改动范围——让用户介入或回到 `/design`。

---

### `/review` → 用户（闭环）

**Review 完成动作**：

通过路径：
1. 输出 Review 报告（追加到任务卡末尾）
2. 执行 merge：`git merge codex/task-N --no-ff -m "..."`
3. 归档任务卡：`git mv docs/tasks/task-N.md docs/tasks/archive/`
4. 更新 `docs/STATUS.md` 的「最近完成」段
5. 更新 `CHANGELOG.md` 的 `[Unreleased]` 段
6. 写记忆（如有沉淀价值）
7. 一次 commit 包含步骤 3-6
8. push

不通过路径：
1. 输出 `REVIEW FAILED` 报告 + 修复路径
2. **不 merge**
3. 等执行者修复后重新提交

---

## 跨角色信息传递

**所有角色共享读取**：
- `docs/STATUS.md`（项目快照）
- `docs/ARCHITECTURE.md`（L1 索引）
- `docs/api/*.md`（L2 契约）
- `docs/memory/MEMORY.md`（记忆索引）

**写入权限严格分割**：

| 文件 | 谁能写 |
|---|---|
| `docs/_pm_brief.md` | `/pm` |
| `docs/_diagnosis.md` | `/diagnose` |
| `docs/tasks/task-N.md` | `/design`（创建）+ `/review`（追加 Review 报告 + 归档） |
| `docs/api/*.md` | `/design`（设计新签名时）+ `/execute`（实现时同步语义） |
| 源代码 | `/execute` |
| `docs/STATUS.md` | `/review` |
| `CHANGELOG.md` | `/review` |
| `docs/memory/*.md` | `/review`（唯一） |

---

## 中断与恢复

工作流可在任何角色之后中断（停下来等用户）。恢复时：

1. 用户调用对应角色的触发词
2. 该角色读上一阶段的产物（Brief / 诊断报告 / 任务卡）
3. 继续工作

**好处**：用户可以随时叫停、补充信息、重新规划。每个产物都是可恢复的检查点。

---

## 跨任务的工作流并行

实际工作中常有多个任务并行：

```
任务 A：/pm → /design → /execute → /review
任务 B：    /pm → /diagnose → /design → /execute → /review
任务 C：              /design → /execute → /review
```

并行原则：
- **不同任务的同阶段可以同时进行**（一个对话做任务 A 的 /design，另一个对话做任务 B 的 /pm）
- **同一任务的不同阶段必须串行**（任务 A 没 review 完前不要开始 task A.2）
- **多个任务的 /execute 阶段最好用独立 worktree**（避免代码冲突）

---

## 协议升级原则

如果发现某个交接环节频繁出错，先检查：

1. 是否角色定义不清晰？→ 改 `roles/{role}.md`
2. 是否交接格式不明确？→ 改本文件
3. 是否项目特殊纪律？→ 改 `CLAUDE.md` 或对应 `roles/`

**不要**通过"经验"绕过协议。协议是工作流的骨架，绕过 = 累积技术债。
