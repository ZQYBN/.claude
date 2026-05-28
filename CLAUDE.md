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

### 五、依赖校验协议

**所有硬依赖 skill 必须在 Phase 0 完成三段校验，缺一不可：**

1. **索引存在** — CLAUDE.md 的"工作流硬依赖"表中已注册
2. **文件存在** — 自定义 skill 在 `skills/<name>/SKILL.md` 可读；built-in 命令在当前 runtime 可用
3. **运行时可达** — 当前 session 可以实际调用该 skill/命令

**任一校验失败 → STOP，输出缺失报告，禁止进入 Phase 1。**

---

## 工作流

```
DISCUSS → PLAN → DISPATCH(或IMPLEMENT) → INTEGRATE(或TEST) → TEST → REVIEW → DONE
```

入口 skill：`MyWorkFlow`（**所有任务**的唯一入口，由 Phase 0 TRIAGE 判定类型并分流）。

MyWorkFlow Phase 0 TRIAGE 路由表：

| 任务类型 | 判定条件 | 路由目标 |
|---------|---------|---------|
| CODE | 涉及代码文件（`.java`/`.tsx`/`.ts`/`.py` 等）、功能开发、Bug 修复、重构 | 继续 MyWorkFlow → Phase 1 DISCUSS |
| FILE | 数据清洗、格式转换、文档生成、报表、可视化、RAG/GIS 处理 | 调用 `MyWorkFlow-Docs`，退出 MyWorkFlow |
| SKILL | 新增/修改/删除 skill、调整触发配置 | 调用 `MyWorkFlow-Skills`，退出 MyWorkFlow |
| RULE | 新增/修改/废弃 rule、规则缺口或冲突 | 调用 `MyWorkFlow-Rules`，退出 MyWorkFlow |

子 skill（MyWorkFlow-Docs、MyWorkFlow-Skills、MyWorkFlow-Rules）禁止直接调用，
必须通过 MyWorkFlow Phase 0 TRIAGE 进入。

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
| `MyWorkFlow` | 自动：所有任务唯一入口 | Phase 0 TRIAGE 判定类型并分流：CODE 内部执行，FILE/SKILL/RULE 调用子 skill |
| `MyWorkFlow-Docs` | 由 MyWorkFlow Phase 0 TRIAGE 调用 | 阶段化文件流水线，目录状态机，README 落盘，只读隔离 |
| `MyWorkFlow-Skills` | 由 MyWorkFlow Phase 0 TRIAGE / Phase 6 REVIEW 调用 | Skill 生命周期管理（新增/修改/删除/调整触发） |
| `MyWorkFlow-Rules` | 由 MyWorkFlow Phase 0 TRIAGE / Phase 6 REVIEW 调用 | Rule 生命周期管理（新增/修改/废弃/删除） |
| `MyWorkFlow-Team` | 由 MyWorkFlow Phase 2 PLAN 调用 | Agent team 编排（拆分、派发、集成审查） |
| `MyWorkFlow-Frontend` | 由 MyWorkFlow Phase 0 TRIAGE 调用 | 前端编译检查、skill 调用、review 项 |
| `MyWorkFlow-Backend` | 由 MyWorkFlow Phase 0 TRIAGE 调用 | 后端编译检查、schema 确认、review 项 |


## 工作流硬依赖

以下 skill 是 MyWorkFlow Phase N 的强制调用项，非可选。缺失任何一个 → Phase 0 STOP。

| Skill | 类型 | 调用 Phase | 缺失行为 |
|-------|------|-----------|---------|
| `code-review-expert` | 自定义 skill | Phase 6 REVIEW | STOP，报告"code-review-expert 未安装" |
| `simplify` | built-in 命令 | Phase 6 REVIEW | STOP，报告"当前环境不支持 simplify 命令" |
| `MyTestBasedOnGit` | 自定义 skill | Phase 4/5 TEST | STOP，报告"MyTestBasedOnGit 未安装" |

上述 skill 不因"领域 Skills"条款而豁免索引注册。

---

## 项目适配

项目级 `.claude/CLAUDE.md` 可覆盖：技术栈版本、目录结构、项目特定约束。
禁止覆盖本文件的铁律和 skill 触发配置。
