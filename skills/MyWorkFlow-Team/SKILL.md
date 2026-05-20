---
name: MyWorkFlow-Team
description: >
  Agent team workflow. Invoked by MyWorkFlow during PLAN phase when a task can
  be split into independent sub-tasks. Defines: split decision rules (domain
  boundary, file non-overlap, semantic coupling check), AgentTask template,
  DISPATCH procedure (TeamCreate + worktree isolation + parallel spawn),
  sub-agent execution steps (IMPLEMENT → SELF-TEST → SELF-REVIEW → report),
  INTEGRATE gate review (per-agent diff audit + cross-agent consistency
  checklist), sub-agent report format. The main agent never writes code —
  sub-agents execute within strict AgentTask boundaries.
---

# MyWorkFlow-Team

## CONTROL-PLANE PURITY

**The main agent never writes code during DISPATCH or INTEGRATE.**

- Agent output incomplete? → Write a fix AgentTask, re-dispatch
- Agent failed? → Re-dispatch or escalate to human
- "I could just fix this myself"? → **NO.** Re-dispatch.

The main agent audits, standardizes, and re-dispatches. Sub-agents implement.
If you are the main agent and you are about to use `Edit`, `Write`, or `Bash`
to modify source code during DISPATCH or INTEGRATE, STOP.

---

Agent team workflow. This skill is loaded by `MyWorkFlow` during Phase 2
(PLAN) when the plan reveals independently executable sub-tasks. The main
agent uses this skill to split work, dispatch sub-agents in parallel, and
review their output.

## Split Decision Rules

### Hard constraint

Two AgentTasks must not modify the same file. If they would, merge them into
one agent.

### Semantic coupling check

Even when files don't overlap, merge into the same agent if:
- Agent A's output type is Agent B's input type (shared contract)
- Both use the same hook/service/util that has no contract owner
- Agent A adds a field that Agent B consumes

### Slice priority

Domain-first, then by technical layer within a domain:

```
Domain (herbs / formulas / symptoms)
  ├── Frontend agent
  └── Backend agent
```

### Minimum unit

At least one complete component or one complete API endpoint. Single-prop
changes and single-line fixes are too small to split — merge into the nearest
agent.

### Partial split

When a task has both splittable and unsplittable parts:
- Splittable parts: dispatch normally as sub-agents
- Unsplittable parts (coupled cluster): dispatch as ONE sub-agent handling
  the entire cluster
- The main agent never falls back to writing code itself

## Plan Phase Output

When splitting is decided, the plan document must include an
`## Agent Team Assignment` section:

### AgentTask table

| Agent | Responsibility | Files | Contract | Acceptance |
|-------|---------------|-------|----------|------------|
| agent-xxx-fe | ... | `path/to/files` | `Herb[]` return | [ ] checklist |

### Shared contract table

| Contract | Owner | Defined In | Consumers |
|----------|-------|-----------|-----------|
| `Herb` interface | Main Agent | `types.ts:Herb` | agent-herb-fe, agent-herb-be |
| `GET /dict/herbs` | Main Agent | `HerbController.java` | agent-herb-fe |

### Contract categories

| Category | Defined In | Example |
|----------|-----------|---------|
| Shared types | `types.ts` or `domain/types.ts` | `Herb`, `Formula` |
| API contract | Controller signature + return type | `GET /dict/herbs` → `{code,message,data}` |
| Component interface | Component props signature | `HerbPickerProps { onSelect }` |
| Hook/Util signature | Exported function signature | `useHerbSearch(q: string): Herb[]` |

## AgentTask Template

The main agent generates one AgentTask per sub-agent at DISPATCH. The
AgentTask IS the sub-agent's complete instruction set — it includes the
execution protocol so the sub-agent knows how to work regardless of which
skills it has loaded.

```markdown
# AgentTask: <agent-name>

## Goal
<One sentence describing what to build>

## Files (do not exceed this scope)
- Modify: `exact/path/to/file.tsx`
- Add: `exact/path/to/new-file.tsx`

## Contracts (read-only, do not modify)
- Input: <type signature>
- Output: <type signature>
- API: `METHOD /path` → `{code, message, data: <type>}`

## Acceptance Criteria
- [ ] <verifiable condition 1>
- [ ] <verifiable condition 2>

## Prohibitions
- [ ] Do not modify shared contracts
- [ ] Do not modify other agents' files
- [ ] Do not introduce new dependencies without approval

## Execution Protocol

You MUST follow these steps in order. Do not skip any step.

### Step 1: Read context
- This AgentTask
- The shared contracts referenced above
- Existing code in the files listed above

### Step 2: IMPLEMENT
- Follow the file list exactly — do not touch files outside the list
- Comply with all rules/ directory constraints
- Run compile check before each commit
- One commit per logical unit

### Step 3: SELF-TEST
- Test only your changes (fine-grained)
- Frontend: jest + playwright if UI
- Backend: mvn test
- If any test fails → fix → re-run SELF-TEST (loop internally, no limit)
- All pass → proceed to Step 4

### Step 4: SELF-REVIEW — MANDATORY EXIT GATE

STOP. Do not report yet. You MUST complete this checklist and output the
result BEFORE proceeding to Step 5:

- [ ] All acceptance criteria met? (re-check each one explicitly)
- [ ] Files changed are ALL within the File list above? (check with `git diff --stat`)
- [ ] Shared contracts UNCHANGED? (verify — modifying a contract is a violation)
- [ ] Compilation passes? (run it now)
- [ ] Code complies with rules/? (check against the rule files)

If ANY item fails → fix → re-run Step 3 (SELF-TEST) → re-run Step 4.

You MUST output:

```
## SELF-REVIEW Result

- Acceptance: ALL PASS / FAILURES: <list>
- Scope: WITHIN LIMITS / OVERRIDE: <list out-of-scope files>
- Contracts: UNCHANGED / MODIFIED: <list>
- Compilation: PASS / FAIL
- Rules compliance: PASS / FAIL: <list violations>
```

Do NOT proceed to Step 5 until every item passes.

### Step 5: Update docs
- Update associated plan/spec files if implementation differs from plan

### Step 6: Report
Output your final report using the format below.
```

## Sub-Agent Report Format

The sub-agent MUST output this report as the final message:

```markdown
# Agent Report: <agent-name>

## Diff Summary
- Files modified: N
- Lines added: +X
- Lines removed: -Y

## SELF-REVIEW Result
- Acceptance: ALL PASS
- Scope: WITHIN LIMITS
- Contracts: UNCHANGED
- Compilation: PASS
- Rules compliance: PASS

## Proposals (optional)
- Contract change proposal: <description and rationale>
```

### CRITICAL: Worktree isolation mechanism

**NEVER use `EnterWorktree` to isolate sub-agents.** `EnterWorktree` switches
the CURRENT session into a worktree — it steals the main agent's working
directory. The main agent must stay on the feature branch to orchestrate.

**Use the `Agent` tool with `isolation: "worktree"` and `team_name` instead.**
This creates an isolated worktree for the spawned sub-agent, registers it in
the team for native status tracking, while the main agent's session remains
untouched:

```
Agent(
  isolation: "worktree",
  team_name: "<team-name>",
  description: "agent-xxx-fe: implement herb picker frontend",
  prompt: "<AgentTask content>"
)
```

The `isolation: "worktree"` + `team_name` combination:
1. Creates a git worktree on a new branch
2. Registers the agent in the team (enables native status tracking table)
3. Runs the sub-agent inside that isolated worktree
4. Returns the worktree path and branch name in the result when done
5. Main agent's session never leaves the feature branch

### Dispatch procedure

1. Create team: `TeamCreate(team_name: "<feature>-team")`
   — this enables Claude Code's native agent state tracking table
2. Create the feature branch: `git checkout -b feature/<name>`
3. Spawn all sub-agents in PARALLEL (single message, multiple `Agent` calls)
   — every `Agent` call includes `team_name` + `isolation: "worktree"`:

```
Agent(isolation: "worktree", team_name: "<feature>-team", description: "agent-1: ...", prompt: "<AgentTask 1>")
Agent(isolation: "worktree", team_name: "<feature>-team", description: "agent-2: ...", prompt: "<AgentTask 2>")
Agent(isolation: "worktree", team_name: "<feature>-team", description: "agent-3: ...", prompt: "<AgentTask 3>")
```

4. The system provides a real-time status table showing each agent's state
   — do NOT manually poll or infer agent states from text
5. Wait for all sub-agents to complete. Each returns its worktree branch name.
6. Proceed to INTEGRATE phase to review and merge.

### Worktree topology

```
Main Agent session (feature/xxx — never moves)
  │
  ├── Agent(isolation: "worktree") → Sub-agent A (worktree branch A)
  ├── Agent(isolation: "worktree") → Sub-agent B (worktree branch B)
  └── Agent(isolation: "worktree") → Sub-agent C (worktree branch C)
```

Sub-agents commit freely within their worktree branches. No cross-agent
communication. All coordination is through the shared contracts in the plan
document. The main agent never leaves `feature/xxx`.

## Sub-Agent Execution (reference for main agent)

The execution protocol is embedded directly in the AgentTask template (see
above). This ensures every sub-agent receives the protocol in its prompt,
regardless of which skills are loaded. The main agent does not need to
separately instruct the sub-agent — the AgentTask IS the instruction.

Key design: the SELF-REVIEW exit gate inside the AgentTask forces the
sub-agent to output a structured result before reporting. The main agent
uses these results during INTEGRATE to verify each sub-agent passed its
own checks.

## INTEGRATE Phase (Main Agent)

The main agent reviews all sub-agent outputs before merging. It never writes
code — it audits, standardizes, and re-dispatches.

### Step 0: Locate worktree branches

Each sub-agent run with `isolation: "worktree"` returns a branch name. List
all returned branches before proceeding.

### Step 1: Per-agent diff audit

For each sub-agent's worktree branch, review the diff against the feature
branch:

```bash
git diff feature/<name>...<worktree-branch> --stat
git diff feature/<name>...<worktree-branch>
```

Check:
- [ ] Files changed are within the AgentTask scope?
- [ ] No out-of-scope modifications (other agents' files, shared contracts)?
- [ ] All acceptance criteria pass?
- [ ] Compilation passes?

One failure → record the issue → re-dispatch that sub-task with a fix
AgentTask.

### Step 2: Cross-agent consistency audit

Compare all agent implementations side-by-side:
- [ ] Naming: same concept → same name (not `HerbSelector` in one,
  `HerbPicker` in another)
- [ ] Patterns: same problem → same solution (all use `useCallback` for
  debounce, not mix of lodash and hand-written)
- [ ] Structure: similar components in symmetric directory layouts
- [ ] Types: all reference the same shared type source; no agent created a
  local alternative
- [ ] No unexplained `any` — every `any` must have `// why: <reason>` comment

Inconsistency found → the main agent decides the standard → writes a fix
AgentTask → dispatches to the relevant sub-agent.

### Step 3: Merge

All audits pass → merge each worktree branch into the feature branch:

```bash
git merge <worktree-branch> --no-ff -m "feat(scope): merge <agent-name> changes"
```

After all merges, delete the merged worktree branches (optional cleanup).

### Rejection limit

- 1st rejection: write fix AgentTask, re-dispatch
- 2nd rejection: escalate to specialist agent, or flag for human intervention
- The main agent NEVER degrades to writing code itself
