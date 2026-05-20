# Claude Code 全局规则

本文档是 AI 代理在所有项目中的行为约束。当项目级 `.claude/CLAUDE.md` 冲突时以本文档为准。

---

## 铁律（始终生效，不可绕过）

### 一、最小修改原则

**只修改用户明确要求修改的内容。禁止一切顺手操作。**

- 禁止顺手修复：发现无关 bug/typo/warning → 不修复，记下来事后报告
- 禁止顺手删除：发现未使用变量/函数/文件 → 不删除
- 禁止顺手优化：发现可改进的写法 → 不改
- 禁止顺手重构：发现可抽取的公共逻辑 → 不抽取

**每次修改前自问：用户让我改这个了吗？** 答案否定 → 不改。

### 二、PLAN 阶段不可跳过

**AI 必须先说明要修改什么、如何修改，获得确认后才能开始写代码。**
即使修改范围小（单文件、单行），也必须口头说明方案。禁止直接跳到实现。

### 三、DISPATCH/INTEGRATE 时主 Agent 不写代码

**仅当走 agent team 路径（Phase 3 DISPATCH / Phase 4 INTEGRATE）时生效。**
单 agent 回退路径下，主 agent 在 IMPLEMENT 阶段正常写代码不受此条限制。

DISPATCH/INTEGRATE 期间：
- 主 agent 只做编排、审查、合并。不写代码。
- Sub-agent 产出不完整 → 写修复 AgentTask 重新派发。
- Agent 失败 → 重新派发或升级人工。
- "我自己来更快" → **不行。** 使用 `Edit`/`Write` 修改源码是违规。

### 四、Phase Exit Gate

**每个 phase 结束时必须显式输出决策，才能进入下一 phase。**
禁止 phase 间无声滑入。PLAN 结束必须输出 `Decision: DISPATCH / SINGLE-AGENT`。

---

## 工作流

```
DISCUSS → PLAN → DISPATCH(或IMPLEMENT) → INTEGRATE(或TEST) → TEST → REVIEW → DONE
```

入口 skill：`MyWorkFlow`（任务开始时自动触发，判定类型，加载补充）。
详细流程见 skill 文件。铁律在此，skill 文件不得弱化。

---

## 规则索引

完整约束在 `rules/` 目录。关键规则已在上方铁律中直接生效，其余按需读取：

| 文件 | 何时读 |
|------|--------|
| `scope.md` | 上方铁律一已覆盖核心；当涉及跨模块改动、新增分层、引入设计模式时读取 |
| `coding-principles.md` | 裁决顺序、修改优先级、实现约束、禁止过早抽象 |
| `frontend.md` | 修改前端文件时读取 |
| `backend.md` | 修改后端文件时读取 |
| `comments.md` | 新建文件时读取 |
| `commit.md` | 提交前读取 |
| `cli.md` | 使用命令行时读取 |

---

## Skill 索引

| Skill | 触发 | 用途 |
|-------|------|------|
| `MyWorkFlow` | 自动：任务开始 | 流程编排，7 phase（可拆分）或 6 phase（单 agent） |
| `MyWorkFlow-Team` | 由 MyWorkFlow 在 Phase 2 加载 | Agent team 编排（拆分、派发、集成审查） |
| `MyWorkFlow-Frontend` | 由 MyWorkFlow 加载 | 前端编译检查、skill 调用、review 项 |
| `MyWorkFlow-Backend` | 由 MyWorkFlow 加载 | 后端编译检查、schema 确认、review 项 |
| `MyWorkFlow-Rules` | 手动 | 规则变更流程 |
| `MyWorkFlow-Skills` | 手动 | Skill 变更流程 |

领域 Skills 由 MyWorkFlow 在各阶段自动调用，不在此列出。

---

## 项目适配

项目级 `.claude/CLAUDE.md` 可覆盖：技术栈版本、目录结构、项目特定约束。
禁止覆盖本文件的铁律和 skill 触发配置。
