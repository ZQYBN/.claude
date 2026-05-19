---
name: MyTestBasedOnGit
description: Git diff 驱动测试编排器。USE when user says `/MyTestBasedOnGit`, "run tests", "测试", "跑测试". Analyzes git diff to determine what changed, maps changes to mandatory verification anchors, executes, and reports. AI does NOT decide what to test — the topology decides.
---

# MyTestBasedOnGit

基于 `git diff` 的测试编排器：变更 → 强制检查拓扑 → 执行 → 报告。

**核心原则：AI 不判断"该测什么"，拓扑表决定。AI 只负责执行。**

## 触发

- `/MyTestBasedOnGit` — 当前分支 vs `develop`
- `/MyTestBasedOnGit feature/xxx` — 指定分支

---

## 测试锚点

四层不可绕过的验证锚点：

| 锚点        | 层  | 命令                                     | AI 能否作弊        |
| ----------- | --- | ---------------------------------------- | ------------------ |
| **A1** 编译 | F/B | `tsc --noEmit` / `mvn compile`           | 不能，机械判定     |
| **A2** 契约 | F+B | DTO 字段对比 / API response shape        | 不能，结构对比     |
| **A3** 流程 | F/E | `npx playwright test`（headless）        | 难，完整用户操作链 |
| **A4** 视觉 | F   | `npx playwright test --update-snapshots` | 难，像素级对比     |

---

## 执行流程

### Step 1: 确定范围

```bash
git diff develop...HEAD --name-only
git status --porcelain
```

有未提交变更标注 ⚠️，不阻断。

### Step 2: 文件分类

| 标签           | 规则                                                  |
| -------------- | ----------------------------------------------------- |
| `[F]` Frontend | `frontend/` 下 `*.tsx`, `*.ts`（非 `.test.ts`）       |
| `[B]` Backend  | `backend/` 下 `*.java`（非 `*Test.java`, `*IT.java`） |
| `[D]` Database | `*.sql`, `@Entity` 文件, `application.yml`            |
| `[T]` Test     | `*.test.ts`, `*Test.java`, `*IT.java`                 |

### Step 3: 强制检查拓扑

**优先规则**：

- 全部 `[T]` → 跳过，输出"仅测试文件变更"
- 仅 CSS / 文案 / 注释 → 只跑 A1

**风险等级**：

| 级别         | 文件特征                                                                    | 强制动作                |
| ------------ | --------------------------------------------------------------------------- | ----------------------- |
| **critical** | `workspaceFactory`, `types.ts`, `patientSessionReducer`, `auth/`, `router/` | A1 + A2 + A3 + 全量回归 |
| **high**     | `store/`, `shared/`, `hooks/`                                               | A1 + 全量 Jest          |
| **medium**   | `pages/`, `sections/`, `components/`, `Controller.java`                     | A1 + 对应锚点           |
| **low**      | CSS, 文案, 注释                                                             | A1 only                 |

**拓扑表**：文件匹配 → 必须经过的锚点。AI 不可跳过。

| 变更文件特征                     | risk     | A1 编译 | A2 契约        | A3 流程    | A4 视觉              |
| -------------------------------- | -------- | ------- | -------------- | ---------- | -------------------- |
| `workspaceFactory` / `types.ts`  | critical | tsc     | contract       | jest 全量  | —                    |
| `store/` / `shared/` / `hooks/`  | high     | tsc     | —              | jest 全量  | —                    |
| `pages/prescription/` (处方流程) | medium   | tsc     | —              | Playwright | form screenshot      |
| `pages/` 其他页面                | medium   | tsc     | —              | Playwright | page screenshot      |
| `sections/` 分区表单             | medium   | tsc     | —              | Playwright | —                    |
| `components/` UI 组件            | medium   | tsc     | —              | —          | component screenshot |
| `Controller.java`                | medium   | javac   | response shape | —          | —                    |
| `Service.java`                   | medium   | javac   | —              | mvn test   | —                    |
| `@Entity` / `*.sql`              | medium   | javac   | migration      | —          | —                    |
| `DTO` / `record`                 | medium   | javac   | contract       | —          | —                    |
| CSS / 文案                       | low      | —       | —              | —          | screenshot diff      |

**A4 视觉快照约束**：

- ✅ 截图：布局骨架、表单结构、表格、关键按钮、Modal 外框
- ❌ 禁止截图：动画中间态、hover/active 态、loading shimmer、时间戳、动态数据
- 快照对比用 `maxDiffPixels: 100`，避免字体渲染/Dpi 差异误报

### Step 4: 执行锚点

```
A1 编译层 (always)
  cd frontend && npx tsc --noEmit
  cd backend && ./mvnw.cmd compile

A2 契约层 (if triggered)
  读取前端 types.ts → 对比后端 DTO → 输出差异表

A3 流程层 (if triggered)
  cd frontend && npx playwright test
  前端: cd frontend && npx jest --passWithNoTests

A4 视觉层 (if triggered)
  cd frontend && npx playwright test
  screenshot diff 对比基线
```

单条命令超时 120s。一层失败不阻断其他层。

### Step 5: 输出报告

```markdown
# 测试报告

## 范围

- 分支: <name> vs develop
- 变更: N 文件
- 触发锚点: A1 A3

## 锚点执行结果

| 锚点 | 层  | 检查项       | 结果    |
| ---- | --- | ------------ | ------- |
| A1   | F   | tsc --noEmit | ✅      |
| A1   | B   | mvn compile  | ✅      |
| A3   | F   | jest         | 194/195 |
| A3   | F   | playwright   | 3/3     |

## 拓扑覆盖

| 变更文件                     | 要求锚点 | 已执行 | 状态       |
| ---------------------------- | -------- | ------ | ---------- |
| PrescriptionFormSections.tsx | A1 A3    | A1 A3  | ✅         |
| types.ts                     | A1 A2    | A1     | ⚠️ 缺少 A2 |

## 风险

| 级别 | 含义 | 问题 |
| ---- | ---- | ---- |
| P0   | 阻断 | ...  |
| P1   | 警告 | ...  |
```

## 原则

- AI 不判断"该测什么"——拓扑表决定
- AI 只负责执行和报告
- 失败不阻断其他锚点
- 优先修复已有测试，其次才是补充
- 高爆炸半径文件（store/types/shared）走全量，不走增量
- 本 skill 定位：**测试编排器**
