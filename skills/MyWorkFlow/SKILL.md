---
name: MyWorkFlow
description: >
  USE THIS SKILL AT THE START OF EVERY TASK — before writing any code, before
  making any plan, before any implementation. This is the master workflow
  orchestrator. It determines whether the task is frontend, backend, or
  fullstack, then walks through up to seven phases: DISCUSS → PLAN →
  DISPATCH (or IMPLEMENT) → INTEGRATE (or TEST) → TEST → REVIEW → DONE.
  Splittable tasks use the agent team path with parallel sub-agents;
  single-agent tasks use the original path. Invoke this skill when the user
  asks you to build a feature, fix a bug, refactor code, add a component,
  create an API endpoint, modify the database schema, update UI, or make any
  change to the codebase. If you are about to write code, you need this skill
  first. It loads frontend/backend supplements (MyWorkFlow-Frontend,
  MyWorkFlow-Backend) and the agent team skill (MyWorkFlow-Team) when tasks
  can be split.
---

# MyWorkFlow

Master workflow orchestrator. Every phase has three sections:

- **ENTER**: what must be true before starting this phase
- **DO**: what to accomplish in this phase
- **EXIT**: mandatory decision gate — you are NOT allowed to proceed to the
  next phase without explicitly passing this gate

## Phase Transition Protocol

After completing each phase, you MUST output:

```
## Phase Result: <Phase Name>

### Output
<what was produced>

### Decision
<explicit choice made at exit gate>

### Next Phase
<phase name>
```

**You cannot leave a phase without writing this block.** If you feel the
urge to start coding, STOP — check which phase you're in and whether the
exit gate has been passed.

---

## Phase 0: TRIAGE — Determine Task Type

**ENTER**
- User has made a request

**DO**
- Analyze the user's request and files involved

| Condition | Type | Action |
|-----------|------|--------|
| Files under `frontend/` only | Frontend | Load `MyWorkFlow-Frontend` |
| Files under `backend/` only | Backend | Load `MyWorkFlow-Backend` |
| Files in both layers | Fullstack | Load both |
| Pure docs, config, or conversation | General | No supplement |

**EXIT**
- [ ] Task type recorded: FRONTEND / BACKEND / FULLSTACK / GENERAL
- [ ] Supplements loaded (if applicable)
- → ALWAYS proceed to Phase 1

---

## Phase 1: DISCUSS — Align on the Problem

**ENTER**
- Task type known from Phase 0

**DO**
- Clarify what the problem actually is before solving it
- Misalignment here is the most expensive mistake — entire implementation
  cycles get wasted building the wrong thing
- Feature, design change, or anything ambiguous → invoke `brainstorming`
- Root cause clear AND fix scoped to a single file → skip brainstorming
- Do NOT write code in this phase

**EXIT — complete ALL before proceeding:**

- [ ] Shared understanding of the problem reached
- [ ] User has confirmed the understanding (explicitly or via `1`)
- → ALWAYS proceed to Phase 2

STOP. Do not begin planning until the user has confirmed the problem
understanding.

---

## Phase 2: PLAN — Design the Solution

**ENTER**
- Problem understanding confirmed from Phase 1

**DO**
- Read relevant modules and existing code
- Assess scope:
  - < 3 files, no new modules → verbal plan (state root cause, fix location,
    approach directly in conversation)
  - ≥ 3 files or new modules → invoke `writing-plans`, write to
    `docs/superpowers/plans/<YYYY-MM-DD-slug>.md`
- **Before presenting for user approval**, analyze split-worthiness (see EXIT)

**EXIT — you MUST complete ALL of the following before proceeding.**

STOP. Do not start implementing. Do not create a branch. First, explicitly
answer these questions:

### Split Analysis

1. List every sub-task from the plan
2. For each sub-task, identify: files touched, contracts consumed, contracts
   produced
3. Apply `MyWorkFlow-Team` split rules:
   - Hard constraint: no two AgentTasks share the same file
   - Semantic coupling: shared contract → same agent; shared hook/service →
     same agent
   - Minimum unit: at least one complete component or API endpoint
4. Count splittable AgentTasks

### Decision (pick ONE — do not proceed without picking)

- **DISPATCH**: ≥ 2 AgentTasks identified → invoke `MyWorkFlow-Team` for
  detailed split + contract generation, include `## Agent Team Assignment`
  in plan document, present to user for approval
- **SINGLE-AGENT**: < 2 AgentTasks → no AgentTeam section needed, present
  plan to user for single-agent approval

### Phase Result output:

```
## Phase Result: PLAN

### Output
- Plan type: verbal / written (path)
- Sub-tasks identified: N

### Split Analysis
- Splittable: YES / NO
- Reason: <e.g. "3 independent pickers, no shared files, no semantic coupling">

### Decision
- DISPATCH (N AgentTasks) / SINGLE-AGENT

### Next Phase
- DISPATCH → Phase 3 DISPATCH
- SINGLE-AGENT → Phase 3 IMPLEMENT (single-agent)
```

**You are NOT allowed to create a branch, write code, or invoke any
implementation skill until this decision is made and the plan is approved.**

---

## Phase 3: DISPATCH (splittable) or IMPLEMENT (single-agent)

### Path A: DISPATCH

**ENTER**
- PLAN exit decision = DISPATCH
- AgentTeam allocation table written
- Contracts defined with owner = Main Agent
- User has approved the plan (including split)

**DO**
- Invoke `MyWorkFlow-Team` for DISPATCH procedure
- Create feature branch: `git checkout -b feature/<name>`
- Generate one AgentTask per sub-agent
- Spawn all sub-agents in PARALLEL using `Agent` tool with
  `isolation: "worktree"` — this creates an isolated worktree for each
  sub-agent while the main agent's session stays on the feature branch
- **NEVER use `EnterWorktree` for sub-agents** — it steals the main agent's
  session directory
- Wait for all sub-agents to report (each returns its worktree branch name)

**EXIT**
- [ ] All sub-agents have submitted reports
- [ ] All reports include diff summary + self-check results
- → ALWAYS proceed to Phase 4 INTEGRATE

---

### Path B: IMPLEMENT (single-agent fallback)

**ENTER**
- PLAN exit decision = SINGLE-AGENT
- User has approved the plan

**DO**
- Create feature branch from `develop`
- Implement in commits — one change per commit
- Run compile checks from loaded supplement before each commit
- Invoke domain skills from supplement dispatch table (e.g. new UI →
  `frontend-design`, design decisions → `ui-ux-pro-max`)
- Update associated docs
- Push: `git push origin feature/<name>`

**EXIT**
- [ ] All planned changes implemented and pushed
- [ ] Docs updated to match implementation
- → ALWAYS proceed to Phase 4 TEST (single-agent path)

---

## Phase 4: Path-dependent

### Path A: INTEGRATE (splittable)

**ENTER**
- All sub-agent reports received from Phase 3 DISPATCH

**DO**
- Follow `MyWorkFlow-Team` INTEGRATE gate review:

**Step 1 — Per-agent diff audit:**
- [ ] Files changed within AgentTask scope?
- [ ] No out-of-scope modifications?
- [ ] All acceptance criteria pass?
- [ ] Compilation passes?

**Step 2 — Cross-agent consistency audit:**
- [ ] Same concept → same name across all agents
- [ ] Same problem → same solution across all agents
- [ ] Similar components → symmetric directory layout
- [ ] All reference shared types; no local type alternatives
- [ ] No unexplained `any` (every `any` has `// why:` comment)

**Step 3 — Handle failures:**
- Failure → write fix AgentTask → re-dispatch (max 2 rejections per agent)
- 2nd rejection → escalate to specialist agent or flag for human
- Main agent NEVER writes code

**Step 4 — Merge:**
- All pass → merge worktree diffs to feature branch
- Unified commit (main agent writes commit message)
- `ExitWorktree` (remove)

**EXIT**
- [ ] All worktree diffs merged and committed
- [ ] Consistency audit passed or escalated
- → ALWAYS proceed to Phase 5 TEST

---

### Path B: TEST (single-agent)

**ENTER**
- Implementation complete from Phase 3 IMPLEMENT (single-agent)

**DO**
- Invoke `MyTestBasedOnGit` — analyzes git diff, maps to verification anchors
- Tests pass → Phase 6 REVIEW
- Tests fail → fix and re-run

**EXIT**
- [ ] All triggered test anchors pass
- → ALWAYS proceed to Phase 6 REVIEW

---

## Phase 5: TEST (splittable only)

**ENTER**
- INTEGRATE complete, all worktree diffs merged to feature branch

**DO**
- Invoke `MyTestBasedOnGit` — full integration test on unified branch
- Tests pass → Phase 6
- Tests fail → dispatch fix agent and re-run

**EXIT**
- [ ] All triggered test anchors pass on the merged branch
- → ALWAYS proceed to Phase 6 REVIEW

---

## Phase 6: REVIEW

**ENTER**
- All tests passing (from Phase 4 single-agent path or Phase 5 splittable
  path)

**DO**
- Invoke `code-review-expert` for senior-engineer perspective
- Refactoring tasks → also invoke `simplify`
- Run checklist:
  - [ ] Implementation matches plan's goals and boundaries
  - [ ] Every change in diff has a clear purpose
  - [ ] No unauthorized architecture changes
  - [ ] Rule gaps: did this bug reveal a missing or incomplete rule? Flag it
  - [ ] No extractable patterns (3+ occurrences → flag)
  - [ ] Associated docs updated and consistent
- If splittable, also check:
  - [ ] All sub-agent reports accounted for
  - [ ] Cross-agent consistency audit passed
  - [ ] No sub-agent exceeded AgentTask file scope
- Rule gap or extractable pattern found → follow `MyWorkFlow-Rules`

**EXIT**
- [ ] Code-review-expert report reviewed
- [ ] All checklist items addressed
- → ALWAYS proceed to Phase 7 DONE

---

## Phase 7: DONE

**ENTER**
- REVIEW complete, all issues resolved

**DO**
- User creates PR, merges to `develop`, deletes feature branch
- AI never merges, never operates on `main`
- Ask user whether to archive plan file:
  - One-shot task → archive after PR merge
  - Multi-phase feature → archive after all phases complete
  - Code-change record → user decides
- Archive: `git mv docs/superpowers/plans/<file> docs/archive/YYYY-MM/<file>`

**EXIT**
- [ ] PR created (by user)
- [ ] Archive decision made
- → Workflow complete

---

## Branch Lifecycle

```
develop
  └── feature/<name>   (AI implements → user merges PR → branch deleted)
```

Feature branches are created from `develop`. AI never merges to `develop`,
never operates on `main`.
