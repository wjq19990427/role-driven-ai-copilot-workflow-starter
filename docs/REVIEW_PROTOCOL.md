# Review 协议

> `/review` 角色的执行规范。每次 merge 前由复盘师按此格式执行。

---

## 触发时机

- 执行者提交到 `codex/task-N` 分支，等待 Review
- 用户调用 `/review`

---

## Review 步骤

### 1. 读 diff

```bash
git diff main...codex/task-N
```

对照任务卡的「接口约定」和「验收清单」逐项检查。

### 2. 读对应任务卡

`docs/tasks/task-N.md`，加载验收标准到上下文。

### 3. 逐点审查表

输出表格，每行对应一个审查点。常见审查项（不限于此）：

| 审查项 | 结论 | 备注 |
|---|---|---|
| 接口签名与任务卡一致 | ✅ | |
| 副作用范围符合约定（写 DB / 写 session state） | ✅ | |
| 未触碰「不许碰」范围 | ✅ | |
| L2 契约同步更新 | ✅ / ⚠️ | 若签名有变更 |
| 没有计划外重构/抽象 | ✅ | |
| 没有防御性扩展 | ✅ | |
| LLM Skill 副作用延迟原则遵守 | ✅ | （若涉及 AI 调用） |
| LLM Skill Prompt 兜底实现 | ✅ | （若涉及 AI 调用） |
| 验收清单第 N 项 | ✅ / ⚠️ / ❌ | 逐条核对 |
| smoke test 通过 | ✅ | |
| commit message 符合规范 | ✅ | |
| 未 push main | ✅ | |

**结论符号含义**：
- **✅** 通过
- **⚠️** 非阻塞观察（记录，不阻塞 merge；后续跟进或开新任务卡）
- **❌** 阻塞项（必须修复后重新提交）

### 4. 非阻塞观察清单

```markdown
## 非阻塞观察

- ⚠️ {描述}（处理方式：开新任务卡 #N+1 / 记入技术债 / 忽略）
```

### 5. 最终结论

```markdown
## 最终结论

整体：通过 / 不通过

[通过时]
PASS — {一句话总结}

[不通过时]
FAIL — {阻塞原因}
```

---

## 通过路径

1. **执行 merge**：
   ```bash
   git merge codex/task-N --no-ff -m "$(cat <<'EOF'
   <type>(<scope>): <描述> [task-N]

   - {要点 1}
   - {要点 2}

   Co-Authored-By: ...
   EOF
   )"
   ```

2. **归档任务卡**：
   ```bash
   git mv docs/tasks/task-N.md docs/tasks/archive/task-N.md
   ```

3. **追加 Review 报告**到任务卡末尾（归档前）：
   ```markdown
   ---

   ## Review 报告（{日期}）

   {完整 Review 表格}

   ## 最终结论

   PASS — {总结}
   ```

4. **更新 STATUS.md**「最近完成」段（保持 ≤ 50 行）

5. **更新 CHANGELOG.md** `[Unreleased]` 段（1-2 句话）

6. **写记忆**（如有沉淀价值，见下文）

7. **一次 commit**：
   ```bash
   git add docs/STATUS.md CHANGELOG.md docs/tasks/archive/task-N.md docs/memory/
   git commit -m "chore: 归档 task-N，更新 STATUS 与 CHANGELOG"
   ```

8. **Push**：
   ```bash
   git push origin main
   ```

---

## 不通过路径

输出：

```markdown
## REVIEW FAILED

**阻塞项：**
- ❌ {具体问题 1，引用文件:行号}
- ❌ {具体问题 2}

**修复路径：**
{建议修复方向，不写具体代码}

**下一步：**
等待执行者修复后重新提交，不执行 merge。
```

**追加到任务卡末尾，但不归档**（任务卡留在 `docs/tasks/` 等待修复）。

git 不动。

---

## 跳过 Review 的场景

仅限以下情况可跳过完整 Review：

- typo 修复
- 文档 only 变更（且无新约束）
- 无逻辑变更的格式调整（如 prettier / black 自动格式化）

其他情况**一律走完整 Review**。

---

## 记忆写入决策树

每次 Review 完成后问自己：**这次任务有什么是未来该记住的？**

```
有值得记的洞察？
  ↓ 是
属于哪类？
  ├─ 用户偏好的协作方式 → docs/memory/feedback.md
  ├─ 用户身份/背景信息   → docs/memory/user.md
  ├─ 项目特殊语境       → docs/memory/project.md
  └─ 外部资源指针       → docs/memory/reference.md

不值得记？
  ↓
跳过。不强行写入。
```

**值得记的标准**：

- 用户明确表达偏好（"这种粒度刚好" / "下次别这样"）
- 用户验证过的非显然决策（采纳了某个 trade-off 方案）
- 项目踩过的具体坑（某个框架的怪行为）
- 跨模块的隐性约束

**不值得记的标准**：

- 代码模式（git log / grep 能查）
- 文件路径（实时读取更准）
- 单次任务的临时上下文
- 已写进 L2 契约的内容

### 写入格式

```markdown
## {规则标题}
{规则本身，简洁一句}

**Why：** {用户给出的理由或发生过的事故}
**How to apply：** {何时何地适用，让未来 Review 能判断是否仍适用}
```

`Why` + `How to apply` 让记忆能"自解释"，不只是教条。

---

## 常见阻塞项清单

以下问题在 Review 时一旦发现必须阻塞：

| 阻塞项 | 处理 |
|---|---|
| 改动了「不许碰」范围 | FAIL，要求回退 |
| L2 契约未同步（公开签名变了但文档没更新） | FAIL，要求补 L2 |
| 副作用延迟原则违反（Skill 内部直接写 DB） | FAIL，要求重构 |
| Prompt 兜底缺失（依赖 LLM 严格遵从约束） | FAIL，要求加兜底 |
| 分支未隔离（直接提交到 main） | FAIL，要求 reset + 在 branch 上重做 |
| 计划外的重构/抽象 | FAIL，要求剥离到新任务卡 |
| smoke test 失败 | FAIL，要求修复 |
| commit message 不规范 | ⚠️ 非阻塞，但要求下次注意 |

---

## Review 报告示例

```markdown
## Review: task-58（AI 标签直接填入 + 视觉区分）

**整体：通过**

| 审查项 | 结论 | 备注 |
|---|---|---|
| toast 时机 | ✅ | toast_key 在 rerun 后才消费，精确触发一次 |
| 入库时机延迟 | ✅ | 移除了即时 add_tag 调用，符合副作用延迟原则 |
| apply_key 映射 | ✅ | `f"{safe_sid}_topics"` 与 multiselect key 一致 |
| 新标签入 options | ✅ | _structured_options() 已追加 ai_new_tags |
| 橙色信息块 | ✅ | 过滤"仍在选中值"才显示，取消勾选后自动消失 |
| 保存/归档两路均清理 | ✅ | _clear_ai_new_tags_state() 在两处都调用 |
| 无 suggestions 时 | ✅ | st.info(...) 后直接 return |
| L2 契约同步 | ✅ | docs/api/components.md 已更新 |

## 非阻塞观察

- ⚠️ AI 标签建议填入 topics 而非 flat tags，是合理选择但与历史路径不同（处理方式：忽略，已在备注中说明）

## 最终结论

PASS — 实现质量高，副作用延迟原则严格执行，UI 交互流畅。
```

---

## 自查清单（给 `/review` 角色）

Review 完成后问自己：
- [ ] 每个验收清单项都核对了？没漏？
- [ ] 区分了阻塞 ❌ vs 非阻塞 ⚠️？
- [ ] 通过/不通过的结论明确？
- [ ] 通过时归档 + STATUS + CHANGELOG 都做了？
- [ ] 有没有需要记入记忆的洞察？
- [ ] 不通过时给了清晰的修复路径？

**所有项满足才算完成 Review。**
