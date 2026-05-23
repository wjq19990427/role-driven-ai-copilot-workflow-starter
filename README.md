# Role-driven AI Co-Pilot Workflow Starter

> **专为个人开发者设计**的 AI 协同开发模板。Claude 在五个明确角色间切换，每个角色只做一件事，通过文档交接形成可追溯的工作流。

---

## 这是什么

一套即开即用的工作流脚手架。**核心创新**：把 Claude 拆成 5 个职责互斥的角色，每次对话只扮演一个，避免认知边界混乱。

```
用户          /pm  →  /diagnose  →  /design  →  /execute  →  /review  →  用户
            澄清     诊断          设计       实现         审查
                                  任务卡      代码         + 记忆
```

适合：
- 个人开发者，希望让 Claude 更有章法
- 长期项目，需要跨会话保持上下文
- 任何 AI 协同场景（Claude 是首选实现，但角色定义 AI-agnostic）

---

## 为什么用角色驱动

实践中发现：**让 Claude 在明确角色边界内工作，比自由对话效率高 3-5 倍**。

| 自由对话 | 角色驱动 |
|---|---|
| Claude 同时思考"是什么/怎么做/对不对" | 一次只做一件事，思维边界清晰 |
| 上下文混乱，容易越界 | 角色禁止越界，强制中断点 |
| 难以追溯 | 每个角色都有固定输出物，可审计 |
| 一次性消耗 token 高 | 拆分后每个角色 token 占用低 |

这不是 Claude 的特性，而是认知科学原理——**专注边界提高产出质量**。

---

## 五个角色

| 角色 | 触发词 | 职责 |
|---|---|---|
| **PM** | `/pm` | 把用户混乱想法整理成结构化需求清单 |
| **Diagnose** | `/diagnose` | Bug 根因诊断，定位文件范围 |
| **Design** | `/design` | 产出任务卡 + 并行调度图（只写 What，不写 How） |
| **Execute** | `/execute` | 按任务卡实现代码，独立 worktree |
| **Review** | `/review` | Review diff、归档、写记忆 |

每个角色在 `roles/{role}.md` 中有完整定义：触发条件、做/不做、工作流程、权限范围、输出物格式、自查清单。

---

## 快速开始

### 1. 复制到新项目

```bash
git clone https://github.com/wjq19990427/role-driven-ai-copilot-workflow-starter.git my-project-template
cd my-project-template
rm -rf .git
cp -r * .claude /path/to/your/project/
```

或：直接 **Use this template** 创建新仓库。

### 2. 按项目实际情况修改

- `CLAUDE.md`：通用，一般不动
- `roles/execute.md`：调整 smoke test 命令（替换为项目实际启动/测试命令）
- `docs/STATUS.md`：填写当前版本和核心架构
- `docs/ARCHITECTURE.md`：填写模块表和依赖关系
- `docs/api/`：为每个代码层新建 `{layer}.md`，参考 `_HOWTO.md`

### 3. 提交基线

```bash
git add CLAUDE.md roles/ .claude/ docs/
git commit -m "docs: 角色驱动工作流基建"
```

### 4. 开始工作

```bash
claude  # 启动 Claude Code
```

Claude 会在开场询问角色。从 `/pm` 开始最自然。

---

## 仓库结构

```
.
├── README.md                       # 本文件
├── SETUP.md                        # 新项目接入引导
├── CLAUDE.md                       # 顶层规则 + 角色切换说明
├── .claude/skills/                 # 触发词薄壳（Claude Code 专用）
│   ├── pm.md
│   ├── diagnose.md
│   ├── design.md
│   ├── execute.md
│   └── review.md
├── roles/                          # 角色完整定义（任何 AI 客户端可用）
│   ├── pm.md
│   ├── diagnose.md
│   ├── design.md
│   ├── execute.md
│   └── review.md
└── docs/
    ├── STATUS.md                   # 项目快照（≤ 50 行）
    ├── ARCHITECTURE.md             # L1 模块索引
    ├── HANDOFF_PROTOCOL.md         # 角色交接协议
    ├── REVIEW_PROTOCOL.md          # Review 流程详细规范
    ├── api/
    │   └── _HOWTO.md               # L2 契约编写指南
    ├── tasks/
    │   └── _template.md            # 任务卡模板
    └── memory/                     # 跨会话记忆（核心，非可选）
        ├── MEMORY.md
        └── _HOWTO.md
```

---

## 核心设计原则

### 1. 认知分工，不是执行分工

设计师消耗 token 思考"要什么"，执行者消耗 token 思考"怎么做"。任务卡只写 What，不写 How。

### 2. 角色边界强制

每个角色都有"不做什么"清单。越界 = 退出角色。这是 token 效率和产出质量的关键。

### 3. 文档交接，不口头

角色之间通过固定格式的文档交接（PM Brief / 诊断报告 / 任务卡 / Review 报告）。每个产物可追溯、可审计。

### 4. 并行设计

设计师写完一批任务卡必须出并行执行指南。文件冲突分析只看源码（`docs/` 不算冲突），不重叠 → 可并行。

### 5. 副作用延迟

LLM Skill 只返回结果数据，不直接写 DB。持久化由用户显式保存触发。

### 6. Prompt 防御纵深

Prompt 里的约束必须在代码层同步兜底，不依赖 LLM 严格遵从。

### 7. 跨会话记忆是核心

记忆系统不是可选扩展，而是长期项目的标配。只有 `/review` 角色能写，确保单一写入源。

---

## 与 AI-Co-Pilot-Starter 的区别

| 维度 | AI-Co-Pilot-Starter | 本仓库 |
|---|---|---|
| 适用场景 | Claude + Codex 双 AI 协同 | 个人开发者 + Claude（或单 AI） |
| 角色数 | 2（架构师 + 实现工） | 5（PM/Diagnose/Design/Execute/Review） |
| 核心产物 | 任务卡 | 角色定义 + 任务卡 |
| 上下文隔离 | worktree 隔离代码 | 角色边界隔离认知 |
| 记忆系统 | 可选 | 核心 |

两个仓库可以共存：团队多 AI 协同用 `AI-Co-Pilot-Starter`，个人多角色工作流用本仓库。

---

## 适配非 Claude 客户端

虽然 `.claude/skills/` 是 Claude Code 专属，但**角色定义本身 AI-agnostic**。

- **Codex / Cursor / Gemini**：在启动 prompt 里说"按 `roles/{role}.md` 工作"
- **手动复制**：把对应 `roles/{role}.md` 内容粘进系统 prompt 或对话开头

详见 `SETUP.md`。

---

## 设计来源

本模板从 [MyPresent](https://github.com/wjq19990427/MyPresent) 项目的实际开发流程中提炼，经 60+ 任务卡实战验证。

前身：[AI-Co-Pilot-Starter](https://github.com/wjq19990427/AI-Co-Pilot-Starter)（Claude + Codex 双 AI 协同版）

---

## License

MIT
