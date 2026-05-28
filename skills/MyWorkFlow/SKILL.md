---
name: MyWorkFlow
description: >
  USE THIS SKILL AT THE START OF EVERY TASK — before writing any code, before
  making any plan, before any implementation. This is the SINGLE entry point
  for ALL tasks. Phase 0 TRIAGE determines the task type and routes to the
  correct execution path: CODE tasks continue inside MyWorkFlow (DISCUSS →
  PLAN → DISPATCH/IMPLEMENT → INTEGRATE/TEST → TEST → REVIEW → DONE), while
  FILE tasks route to MyWorkFlow-Docs, SKILL tasks route to MyWorkFlow-Skills,
  and RULE tasks route to MyWorkFlow-Rules. Splittable code tasks use the
  agent team path with parallel sub-agents; single-agent tasks use the
  original path. It loads frontend/backend supplements (MyWorkFlow-Frontend,
  MyWorkFlow-Backend) and the agent team skill (MyWorkFlow-Team) when tasks
  can be split.
---

# MyWorkFlow

## CONTROL-PLANE PURITY — READ THIS FIRST

**When this skill dispatches sub-agents, the main agent NEVER writes code.**

- Sub-agent output is incomplete? → Re-dispatch with a fix AgentTask
- Sub-agent failed? → Re-dispatch or escalate to human
- Only 1 sub-agent finished and 4 are stuck? → Wait, re-dispatch, or escalate
- "It would be faster if I just did it myself"? → **NO. Never.**

The main agent orchestrates. Sub-agents implement. This separation is
absolute. If you find yourself reaching for `Edit` or `Write` during
DISPATCH or INTEGRATE, STOP — you are violating the architecture.

---

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

## Runtime Ledger (phase_ledger.json)

In addition to the conversational Phase Result block, every phase MUST write a
machine-readable checkpoint to disk. Path: `./.claude/runtime/phase_ledger.json`

This is the **code-path analogue of MyWorkFlow-Docs's per-stage README.md**.
Without it, context collapse = total amnesia.

### Ledger schema

```json
{
  "task_id": "<kebab-case-summary>",
  "repo_root": "/absolute/path/to/repo",
  "phase": "<0|1|2|3|4|5|6|7>",
  "phase_name": "<TRIAGE|DISCUSS|PLAN|DISPATCH|IMPLEMENT|INTEGRATE|TEST|REVIEW|DONE>",
  "decision": "<exit gate decision summary>",
  "approved": "<APPROVE:* token or null>",
  "branch": "<feature/xxx or bugfix/xxx or null>",
  "plan_path": "<path to plan doc or null>",
  "updated_at": "<ISO 8601>"
}
```

### ENTER rule

At the start of each phase (except Phase 0), read the ledger and verify:
- The file exists
- `phase` equals the preceding phase number
- `decision` is non-empty (the exit gate was actually passed)

**Violation → STOP. Do not proceed. Recover from the last valid checkpoint.**

### EXIT rule

After writing the `## Phase Result` conversational block, update the ledger
immediately. The conversational block and the ledger must agree.

### Path constraint

`./.claude/runtime/` is **project-relative** — it lives inside the repository
being worked on (the `repo_root` field). Never write to `~/.claude/runtime/`.
Two parallel repos must never share the same ledger.

---

## Phase 0: TRIAGE — Determine Task Type

**ENTER**
- User has made a request
- **Ledger INIT:** Determine `task_id` (kebab-case from request summary), resolve
  `repo_root` to absolute `$PWD`, create `./.claude/runtime/` directory, write
  initial `phase_ledger.json` with `phase: "0"`, `phase_name: "TRIAGE"`,
  `decision: null`, `approved: null`
- **Dependency verification (IRON LAW #5):** Verify all hard dependencies from CLAUDE.md "工作流硬依赖" table:
  1. `code-review-expert` — skill file exists at `skills/code-review-expert/SKILL.md`
  2. `simplify` — built-in command available in current runtime
  3. `MyTestBasedOnGit` — skill file exists at `skills/MyTestBasedOnGit/SKILL.md`
  - Any missing → STOP, output missing dependency report, do NOT proceed

**DO**
- Analyze the user's request and files involved

| Condition | Type | Action |
|-----------|------|--------|
| Skill/rule management request | Meta | Hand off to `MyWorkFlow-Skills` or `MyWorkFlow-Rules`; exit MyWorkFlow |
| Non-code file processing request | File | Hand off to `MyWorkFlow-Docs`; exit MyWorkFlow |
| Files under `frontend/` only | Frontend | Load `MyWorkFlow-Frontend` |
| Files under `backend/` only | Backend | Load `MyWorkFlow-Backend` |
| Files in both layers | Fullstack | Load both |
| Pure docs, config, or conversation | General | No supplement |

- **Branch prefix determination:** Classify the task intent:
  - Bug fix / defect repair / crash fix → prefix: `bugfix/`
  - Everything else (new feature, enhancement, refactor) → prefix: `feature/`
  - Record in ledger: write tentative branch name (e.g., `feature/herb-picker`)

**EXIT**
- [ ] Dependency verification passed (all 3 hard dependencies available)
- [ ] Task type recorded: FRONTEND / BACKEND / FULLSTACK / GENERAL / FILE / SKILL / RULE
- [ ] Branch prefix determined: `feature/` or `bugfix/`
- [ ] Ledger updated: `decision` = task type, `updated_at` = now
- [ ] If CODE (FRONTEND/BACKEND/FULLSTACK/GENERAL): supplements loaded (if applicable) → proceed to Phase 1 DISCUSS
- [ ] If FILE: invoked `MyWorkFlow-Docs` → exit MyWorkFlow
- [ ] If SKILL: invoked `MyWorkFlow-Skills` → exit MyWorkFlow
- [ ] If RULE: invoked `MyWorkFlow-Rules` → exit MyWorkFlow

---

## Phase 1: DISCUSS — Align on the Problem

**ENTER**
- Ledger check: `phase` must be `"0"`, `decision` non-empty → OK
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
- [ ] User has confirmed the understanding with approval token: `APPROVE` / `GO` / `OK`
- [ ] Ledger updated: `phase: "1"`, `decision` = confirmed understanding, `approved` = user's token
- → ALWAYS proceed to Phase 2

STOP. Do not begin planning until the user has confirmed the problem
understanding.

---

## Phase 2: PLAN — Design the Solution

**ENTER**
- Ledger check: `phase` must be `"1"`, `approved` non-null → OK
- Problem understanding confirmed from Phase 1

**DO**
- Read relevant modules and existing code
- Assess scope:
  - < 3 files, no new modules → verbal plan (state root cause, fix location,
    approach directly in conversation; still write a 2-line summary to ledger's
    `decision` field — verbal does not mean no trace)
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

- [ ] User has approved the plan with token `APPROVE:PLAN`
- [ ] Ledger updated: `phase: "2"`, `decision` = DISPATCH/SINGLE-AGENT,
  `approved` = `APPROVE:PLAN`, `plan_path` = path to plan doc (or "verbal"),
  `branch` = proposed branch name

**You are NOT allowed to create a branch, write code, or invoke any
implementation skill until this decision is made and the plan is approved.**

---

## Phase 3: DISPATCH (splittable) or IMPLEMENT (single-agent)

### Path A: DISPATCH

**ENTER**
- Ledger check: `phase` must be `"2"`, `decision` = `DISPATCH`, `approved` = `APPROVE:PLAN` → OK
- PLAN exit decision = DISPATCH
- AgentTeam allocation table written
- Contracts defined with owner = Main Agent

**DO — execute in this exact order. Do not reorder.**

### Step 1: Create team (ALWAYS FIRST)

```
TeamCreate(team_name: "<feature>-team")
```

Without `TeamCreate`, there is NO native status tracking table. The main
agent will be forced to manually poll agent states from text, which is
unreliable and wastes context. This step is NOT optional.

### Step 2: Create branch

Use the prefix determined in Phase 0 (`feature/` or `bugfix/`):

```
git checkout -b <prefix>/<name>
```
**Ledger update:** Write `branch` field immediately after branch creation.

### Step 3: Load supplement and prepare AgentTasks

- Invoke `MyWorkFlow-Team` for DISPATCH procedure
- Generate one AgentTask per sub-agent using the template

### Step 4: Spawn sub-agents in PARALLEL

**Use `Agent` tool with `isolation: "worktree"` + `team_name`.**
NEVER use `EnterWorktree` — it hijacks the main agent's session directory.
The `Agent` tool creates the worktree inside the sub-agent, leaving the
main agent untouched.

Spawn all agents in a SINGLE message (multiple `Agent` calls):

```
Agent(isolation: "worktree", team_name: "<feature>-team",
      description: "agent-1: <role>", prompt: "<AgentTask 1>")
Agent(isolation: "worktree", team_name: "<feature>-team",
      description: "agent-2: <role>", prompt: "<AgentTask 2>")
```

### Step 5: Wait and observe

- The system provides a real-time MD status table — use it, do not manually
  poll agent states from text
- Each agent runs its worktree, completes, and returns a branch name
- If an agent fails or produces incomplete output: write a fix AgentTask and
  re-dispatch. NEVER write code yourself
- Wait for ALL agents to report

**EXIT**
- [ ] All sub-agents have submitted reports
- [ ] All reports include diff summary + self-check results
- [ ] Ledger updated: `phase: "3"`, `phase_name: "DISPATCH"`, `decision` = N sub-agents completed
- → ALWAYS proceed to Phase 4 INTEGRATE

---

### Path B: IMPLEMENT (single-agent fallback)

**ENTER**
- Ledger check: `phase` must be `"2"`, `decision` = `SINGLE-AGENT`, `approved` = `APPROVE:PLAN` → OK
- PLAN exit decision = SINGLE-AGENT

**DO**
- Create branch from `develop` using Phase 0 prefix (`feature/` or `bugfix/`)
- **Ledger update:** Write `branch` field immediately after branch creation
- Implement in commits — one change per commit
- Run compile checks from loaded supplement before each commit
- Invoke domain skills from supplement dispatch table (e.g. new UI →
  `frontend-design`, design decisions → `ui-ux-pro-max`)
- Update associated docs
- Push: `git push origin feature/<name>`

**EXIT**
- [ ] All planned changes implemented and pushed
- [ ] Docs updated to match implementation
- [ ] Ledger updated: `phase: "3"`, `phase_name: "IMPLEMENT"`, `decision` = changes implemented
- → ALWAYS proceed to Phase 4 TEST (single-agent path)

---

## Phase 4: Path-dependent

### Path A: INTEGRATE (splittable)

**ENTER**
- Ledger check: `phase` must be `"3"`, `phase_name` = `"DISPATCH"`, `branch` non-null → OK
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

**Step 4 — Merge transaction:**
1. Snapshot HEAD: `git rev-parse HEAD` → record in ledger
2. Merge worktree branches **sequentially** (not parallel); after each merge:
   - Conflict? → STOP immediately → write `./.claude/runtime/merge_recovery.md`
     (merged branches / failed branch / conflict files / recovery commands) →
     wait for human
   - Success? → continue to next branch
3. All merged → unified commit (main agent writes commit message)
4. `ExitWorktree` (remove merged worktrees)
5. Delete merged worktree branches (optional cleanup)

Merge recovery file format (`./.claude/runtime/merge_recovery.md`):
```markdown
# Merge Recovery — <feature-branch>

## Snapshot
- Pre-merge HEAD: <sha>
- Timestamp: <ISO 8601>

## Merge Status
| Branch | Status | Notes |
|--------|--------|-------|
| worktree-a | merged | — |
| worktree-b | **CONFLICT** | file.ts:45, util.ts:12 |

## Recovery Commands
git reset --hard <pre-merge-sha>   # rollback all merges
```

**EXIT**
- [ ] All worktree diffs merged and committed (or recovery file written)
- [ ] Consistency audit passed or escalated
- [ ] Ledger updated: `phase: "4"`, `phase_name: "INTEGRATE"`, `decision` = merged N branches
- → ALWAYS proceed to Phase 5 TEST

---

### Path B: TEST (single-agent)

**ENTER**
- Ledger check: `phase` must be `"3"`, `phase_name` = `"IMPLEMENT"`, `branch` non-null → OK
- Implementation complete from Phase 3 IMPLEMENT (single-agent)

**DO**
- Invoke `MyTestBasedOnGit` — analyzes git diff, maps to verification anchors
- Tests pass → Phase 6 REVIEW
- Tests fail → fix and re-run

**EXIT**
- [ ] All triggered test anchors pass
- [ ] Ledger updated: `phase: "4"`, `phase_name: "TEST"`, `decision` = test results summary
- → ALWAYS proceed to Phase 6 REVIEW

---

## Phase 5: TEST (splittable only)

**ENTER**
- Ledger check: `phase` must be `"4"`, `phase_name` = `"INTEGRATE"`, `branch` non-null → OK
- INTEGRATE complete, all worktree diffs merged to feature branch

**DO**
- Invoke `MyTestBasedOnGit` — full integration test on unified branch
- Tests pass → Phase 6
- Tests fail → dispatch fix agent and re-run

**EXIT**
- [ ] All triggered test anchors pass on the merged branch
- [ ] Ledger updated: `phase: "5"`, `phase_name: "TEST"`, `decision` = test results summary
- → ALWAYS proceed to Phase 6 REVIEW

---

## Phase 6: REVIEW

**ENTER**
- Ledger check: `phase` must be `"4"` or `"5"`, `phase_name` = `"TEST"` → OK
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
- [ ] Ledger updated: `phase: "6"`, `phase_name: "REVIEW"`, `decision` = review summary
- → ALWAYS proceed to Phase 7 DONE

---

## Phase 7: DONE

**ENTER**
- Ledger check: `phase` must be `"6"`, `phase_name` = `"REVIEW"` → OK
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
- [ ] Ledger updated: `phase: "7"`, `phase_name: "DONE"`, `decision` = archive decision
- → Workflow complete

---

## Branch Lifecycle

```
develop
  ├── feature/<name>   (new feature, enhancement, refactor)
  └── bugfix/<name>    (bug fix, defect repair)
```

Branches are created from `develop` with the prefix determined in Phase 0 TRIAGE.
AI never merges to `develop`, never operates on `main`.
