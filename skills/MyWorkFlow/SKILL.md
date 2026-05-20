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

Master workflow orchestrator. You are about to work on a task — before you
write a single line of code, walk through these phases. Each phase exists
for a reason; skipping one creates rework.

## Phase 0: Determine task type

Analyze the user's request and the files involved. This decides which
supplement to load, so you run the right compile checks and call the right
domain skills.

| Condition | Type | Action |
|-----------|------|--------|
| Files under `frontend/` only | Frontend | Load `MyWorkFlow-Frontend` |
| Files under `backend/` only | Backend | Load `MyWorkFlow-Backend` |
| Files in both, or the request touches both layers | Fullstack | Load both supplements |
| Pure docs, config, or conversation | General | No supplement needed |

## Phase 1: DISCUSS

**Goal**: Align on what the problem actually is before solving it.

Misalignment at this stage is the most expensive mistake — you can waste
entire implementation cycles building the wrong thing. Spend time here.

- If the request is a feature, design change, or anything ambiguous: invoke
  `brainstorming` to explore intent, constraints, and approaches
- If the root cause is already clear and the fix is scoped to a single file:
  skip brainstorming and proceed directly to Phase 2 (PLAN)
- Output: a shared understanding of the problem
- Do not write code in this phase — you don't yet know what to build

## Phase 2: PLAN

**Goal**: Read the code, form a solution, and get approval before touching anything.

Guessing at a solution without reading the surrounding code produces plans
that fall apart on contact with reality. This phase prevents that.

- Read the relevant modules and existing code
- Assess scope:
  - **Verbal plan**: fewer than 3 files changed, no new modules. State the
    root cause, fix location, and approach directly in conversation.
  - **Written plan**: 3+ files changed or new modules created. Invoke
    `writing-plans` and write to `docs/superpowers/plans/<YYYY-MM-DD-slug>.md`.
    Written plans serve as external memory — long conversations lose context.
- Never skip this phase. State what you will change and how before doing it.

### Split decision

Before presenting the plan for user approval, assess whether the task can be
split into parallel sub-tasks:

1. List the sub-tasks from the plan
2. Apply split rules from `MyWorkFlow-Team` (hard constraint: no file overlap;
   semantic coupling check; minimum unit ≥ one component/endpoint)
3. If ≥ 2 AgentTasks emerge:
   - Invoke `MyWorkFlow-Team` for detailed split + contract generation
   - Include `## Agent Team Assignment` section in the plan document
   - Mark task as **splittable** → Phase 3 will be DISPATCH
   - The user reviews the split alongside the rest of the plan
4. If < 2 AgentTasks → mark as **not splittable** → Phase 3 will be IMPLEMENT

## Phase 3: DISPATCH (splittable) or IMPLEMENT (single-agent)

### If task is splittable

Invoke `MyWorkFlow-Team` for the full DISPATCH flow:

1. Generate one AgentTask per sub-agent using the AgentTask template from
   `MyWorkFlow-Team`
2. Create the team with `TeamCreate`
3. For each AgentTask, in parallel:
   - Create an isolated worktree from the feature branch
   - Spawn a sub-agent with the AgentTask + plan + contracts
   - The sub-agent follows `MyWorkFlow-Team` sub-agent execution steps:
     IMPLEMENT → SELF-TEST → SELF-REVIEW → report
4. Wait for all sub-agents to complete and submit their reports

### If task is not splittable

Follow the single-agent IMPLEMENT path:

1. Wait for the user to approve the plan
2. Create a feature branch from `develop`
3. Implement in commits — one change per commit
4. Run compile checks from the loaded supplement before each commit
5. Invoke domain skills from the loaded supplement when its dispatch table
   says to (e.g. the frontend supplement says: new UI → `frontend-design`,
   design decisions → `ui-ux-pro-max`)
6. Update associated docs (plan, spec) to match the final implementation —
   stale docs are as harmful as missing docs
7. Push: `git push origin feature/<name>`

## Phase 4: INTEGRATE (splittable) or TEST (single-agent)

### If task is splittable

Follow `MyWorkFlow-Team` INTEGRATE gate review:

1. **Per-agent diff audit**: files in scope? no out-of-scope changes?
   acceptance criteria all pass? compiles?
2. **Cross-agent consistency audit**: naming, patterns, structure, types
   consistent across all agents? no unexplained `any`?
3. Failures → write fix AgentTask → re-dispatch (max 2 rejections, then
   escalate to specialist or human)
4. All pass → merge worktree diffs to feature branch → unified commit →
   `ExitWorktree` (remove)
5. **The main agent never writes code during INTEGRATE** — it audits,
   standardizes, and re-dispatches

### If task is not splittable

Invoke `MyTestBasedOnGit` for full integration testing:

- It analyzes git diff to determine what changed and maps changes to
  mandatory verification anchors
- Tests pass → Phase 6
- Tests fail → fix and re-run; do not proceed with failures

## Phase 5: TEST (splittable only)

After INTEGRATE merges all worktree changes, run full integration tests on
the unified feature branch:

- Invoke `MyTestBasedOnGit` — analyzes git diff, maps to verification anchors
- Tests pass → Phase 6
- Tests fail → dispatch fix agent and re-run; do not proceed with failures

## Phase 6: REVIEW

Self-audit before handing off to a human reviewer:

- Invoke `code-review-expert` for a senior-engineer perspective
- For refactoring tasks, also invoke `simplify` to check reuse and efficiency
- Run the checklist:
  - [ ] Implementation matches the plan's goals and boundaries
  - [ ] Every change in the diff has a clear purpose
  - [ ] No unauthorized architecture changes (new layers, new patterns,
    directory restructuring)
  - [ ] Rule gaps: did this bug reveal a missing or incomplete rule? If a
    rule should have prevented this but didn't exist, flag it
  - [ ] No extractable patterns (same pattern appears 3+ times — flag it;
    fewer than 3 — leave it)
  - [ ] Associated docs are updated and consistent with the code
- If the task was splittable, also check:
  - [ ] All sub-agent reports are accounted for
  - [ ] Cross-agent consistency audit passed in INTEGRATE
  - [ ] No sub-agent exceeded its AgentTask file scope
- If you find a rule gap or extractable pattern, raise it with the user and
  follow the `MyWorkFlow-Rules` process

## Phase 7: DONE

Hand off cleanly:

- The user creates the PR, merges to `develop`, and deletes the feature
  branch — AI does not merge or touch `main`
- Ask the user whether to archive the plan file:
  - One-shot task → archive after PR merge
  - Multi-phase feature → archive after all phases complete
  - Code-change record → user decides
- Archive command: `git mv docs/superpowers/plans/<file> docs/archive/YYYY-MM/<file>`
- Never archive without user confirmation

## Branch lifecycle

```
develop
  └── feature/<name>   (AI implements → user merges PR → branch deleted)
```

Feature branches are created from `develop`. AI never merges to `develop`,
never operates on `main`.
