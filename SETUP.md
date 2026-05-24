# 新项目接入指南

## Step 1：复制基建文件

```bash
# 进入新项目根目录
cd /path/to/your/project

# 从本模板复制
cp -r /path/to/role-driven-ai-copilot-workflow-starter/{CLAUDE.md,roles,.claude,docs} .
```

或 GitHub 「Use this template」按钮创建新仓库后 clone。

---

## Step 2：按项目实际情况调整

### 必改

| 文件 | 改什么 |
|---|---|
| `roles/execute.md` | 把 `{import_test_command}` 和 `{start_command}` 替换为项目实际命令 |
| `docs/STATUS.md` | 填写当前版本、核心架构（核心模块的一句话描述） |
| `docs/ARCHITECTURE.md` | 填写模块表和依赖关系 |
| `docs/api/` | 为每个代码层新建 `{layer}.md`，参考 `_HOWTO.md` |

### 按需改

| 文件 | 何时改 |
|---|---|
| `CLAUDE.md` | 项目有特殊纪律时追加规则（如必须用某框架的某 API） |
| `roles/{role}.md` | 角色行为需要项目特定约束时（很少需要） |
| `docs/REVIEW_PROTOCOL.md` | 团队有特定 Review 流程时 |

### 不要改

| 文件 | 为什么 |
|---|---|
| `.claude/skills/*.md` | 通用薄壳，跨项目复用 |
| `docs/HANDOFF_PROTOCOL.md` | 角色交接协议，跨项目稳定 |
| `docs/api/_HOWTO.md` | 通用指南 |
| `docs/memory/_HOWTO.md` | 通用指南 |

---

## Step 3：提交基线

```bash
git add CLAUDE.md roles/ .claude/ docs/
git commit -m "docs: 角色驱动工作流基建"
```

---

## Step 4：开始第一次工作流

```bash
claude  # 启动 Claude Code
```

新对话开始，Claude 会询问角色。从 `/pm` 开始：

```
你: /pm
   我想加个登录页面，用户名密码登录，要有"记住我"
```

Claude（PM 模式）会：
1. 复述意图
2. 追问关键缺失（用户场景、密码加密方式、记住我有效期等）
3. 输出 `docs/_pm_brief.md`
4. 停下等你确认

确认后切换：

```
你: /design
   按 Brief 写任务卡
```

如此循环直到 `/review` 合并完成。

---

## 触发词用法速查

每个角色支持**一步法**和**两步法**两种调用：

### 一步法（推荐，传参直达）

```bash
/execute task-60         # 进入执行模式，立即读 docs/tasks/task-60.md
/review task-60          # 进入复盘模式，对照 task-60 审 codex/task-60 分支
/diagnose 登录页报错 500  # 进入诊断模式，立即开始定位
```

### 两步法（先切角色，再说要做什么）

```
你: /execute
Claude: [进入执行者模式] 请告诉我做哪个任务卡？
你: task-60
Claude: [读 task-60，开始实现]
```

### 各角色参数约定

| 角色 | 是否需参数 | 参数语义 |
|---|---|---|
| `/pm` | 否（推荐两步法） | 直接进入后自然描述需求 |
| `/diagnose` | 可选 | 一步法附症状/报错；或两步法描述 |
| `/design` | 否 | 自动读 `_pm_brief.md` 或 `_diagnosis.md` |
| `/execute` | **必需** | `task-N` 形式指定任务卡 |
| `/review` | **必需** | `task-N` 形式指定要 review 的任务 |

### 切换角色

同一对话中可显式切换：

```
你: /design
Claude: [设计师模式] 输出 task-60
你: /review task-60
Claude: [复盘师模式] 开始审查...
```

**但不建议高频切换**——每次切换会重新加载角色规则，浪费 token。最稳妥的做法是：一个角色完成产物 + 停下 → 开新对话调下一角色。

---

## 适配非 Claude 客户端

### Codex

在项目根放 `AGENTS.md`：

```markdown
# Codex 工作指引

本项目使用角色驱动工作流。请按 `roles/{role}.md` 中的规则工作。

用户在对话中会指明当前角色（如"扮演 design 角色"），
你必须读对应 `roles/{role}.md` 后按规则行事。

通用纪律见 `CLAUDE.md`。
```

### Cursor

在 `.cursorrules` 中加：

```
本项目使用角色驱动工作流。用户对话开始时会指明角色。
完整规则见 CLAUDE.md 和 roles/{role}.md。
```

### Gemini CLI / 其他

启动时把当前角色的 `roles/{role}.md` 内容粘到系统 prompt，或对话开头说：

```
你的角色定义在以下文档中：
{粘贴 roles/design.md 内容}
请严格按此角色规则工作。
```

---

## 文档层级速查

| 层级 | 文件 | 用途 | 谁更新 | 更新时机 |
|---|---|---|---|---|
| L0 | `CLAUDE.md` + `roles/*.md` + `.claude/skills/*.md` | 角色规则 | 框架升级时 | 工作流变化 |
| L1 | `docs/STATUS.md` + `docs/ARCHITECTURE.md` | 项目快照 | `/review` 角色 | 每次任务完成 |
| L2 | `docs/api/*.md` | API 契约 | `/design` 或 `/execute` 角色 | 改公开签名时 |
| L3 | 源代码 | 实现细节 | `/execute` 角色 | 随代码变化 |
| 记忆 | `docs/memory/*.md` | 跨会话上下文 | `/review` 角色 | 有沉淀价值时 |

---

## 常见问题

### Q: 必须严格走完五个角色吗？

不必。简单任务可跳过某些步骤：

- typo 修复：跳过 `/pm` `/diagnose` `/design`，直接 `/execute` + `/review`
- 已经清楚的 Bug：跳过 `/pm`，从 `/diagnose` 开始
- 探索性想法：从 `/pm` 开始，可能在 PM Brief 后就停下

但**复杂任务建议走完整流程**，每个角色都有不可替代的价值。

### Q: 一次对话可以切换角色吗？

可以但**不建议**。每次切换角色等于重置认知边界，更稳妥的做法是：

1. 当前角色完成产物 + 停下
2. 用户开新对话，触发下一角色

如果在同一对话切换，明确说 `/design` 等触发词，Claude 会读新角色定义并切换上下文。

### Q: 记忆文件多大算合理？

- `MEMORY.md` 索引：保持每行 ≤ 150 字，全文 ≤ 100 行
- 每个具体记忆文件（`user.md` 等）：≤ 200 行，超出说明粒度过粗，考虑拆分

### Q: 我可以删掉某个角色吗？

可以。最少配置：`/design` + `/execute` + `/review`（三个角色覆盖核心流程）。

但**记忆系统建议保留**——长期项目跨会话上下文价值大。

---

## 升级模板

模板更新时，参考 [本仓库 CHANGELOG](https://github.com/wjq19990427/role-driven-ai-copilot-workflow-starter/blob/main/CHANGELOG.md)。

升级通常只需替换：

```bash
# 只替换框架文件，不动项目自定义内容
cp -r /path/to/updated-template/{roles,.claude,docs/REVIEW_PROTOCOL.md,docs/HANDOFF_PROTOCOL.md,docs/api/_HOWTO.md,docs/memory/_HOWTO.md} .
```

---

## 贡献

发现工作流问题、有改进建议？欢迎提 Issue 或 PR。

特别欢迎：
- 新角色定义（如 `/research` `/refactor` `/spike`）
- 适配其他 AI 客户端的指南
- 实战案例分享
