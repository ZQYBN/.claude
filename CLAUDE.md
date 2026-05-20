# Claude Code 全局规则

本文档是 AI 代理在所有项目中的行为约束。当项目级 `.claude/CLAUDE.md` 冲突时以本文档为准。

---

## 工作流

```
DISCUSS → PLAN → IMPLEMENT → TEST → REVIEW → DONE
```

入口 skill：`MyWorkFlow`（任务开始时自动触发，判定任务类型，加载前端/后端补充）。
禁止跳过阶段。方案未确认前禁止编写代码。

---

## 规则索引

规则定义"做什么 / 不做什么"，始终生效。见 `rules/` 目录：

| 文件 | 约束范围 |
|------|----------|
| `coding-principles.md` | 裁决顺序、修改优先级、实现约束、禁止过早抽象 |
| `scope.md` | 范围控制、架构保护 |
| `frontend.md` | React（无 class/any/抖动）、AntD 6（4 条）、树结构（4 必须 + 3 禁止） |
| `backend.md` | API 格式 `{code,message,data}`、错误处理（禁空 catch、日志三要素）、数据库（确认 schema、禁破坏性 SQL） |
| `comments.md` | 文件头注释格式、何时写/不写注释、禁止事项 |
| `commit.md` | Commit 格式、分支命名、文档模板 |
| `cli.md` | 命令行规范（脚本复用、临时脚本归档） |

---

## Skill 索引

Skill 定义"怎么做"，封装流程与领域知识。见 `skills/` 目录：

### 流程编排（MyWorkFlow 系列）

| Skill | 触发 | 用途 |
|-------|------|------|
| `MyWorkFlow` | 自动：任务开始 | 流程编排器，判定任务类型，7 phase（可拆分时）或 6 phase（单 agent） |
| `MyWorkFlow-Team` | 自动：可拆分任务 | Agent team 编排（拆分规则、AgentTask 模板、DISPATCH、INTEGRATE gate review） |
| `MyWorkFlow-Frontend` | 由 MyWorkFlow 加载 | 前端补充（编译检查、skill 调用、review 项） |
| `MyWorkFlow-Backend` | 由 MyWorkFlow 加载 | 后端补充（编译检查、schema 确认、review 项） |
| `MyWorkFlow-Rules` | 手动：规则缺口/冲突/过时时 | 规则管理流程（发现→讨论→草拟→确认→写入） |
| `MyWorkFlow-Skills` | 手动：skill 变更时 | Skill 管理流程（提升/降级/删除/新增） |

### 领域 Skills

由 `MyWorkFlow` 在各阶段自动调用，不在此列出完整目录。新增 skill 通过 `MyWorkFlow-Skills` 管理。

---

## 项目适配

项目级 `.claude/CLAUDE.md` 可覆盖：技术栈版本、目录结构、项目特定约束。
禁止覆盖全局规则文件和 skill 触发配置。
