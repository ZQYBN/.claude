---
name: cdTeamLane
description: >
  OPT-IN multi-agent path. USE only when user says /team or plan includes
  multi-agent true. Invoked from cdCodeLane CHANGE step. Split, parallel
  worktree agents, sequential merge. Not the default workflow.
---

# cdTeamLane

**Opt-in only.** User must say `/team` or plan must contain `multi-agent: true`.

Main agent orchestrates. Sub-agents implement. Main agent avoids code edits except **one** conflict file when human is blocked.

---

## Enable Checklist

- [ ] User explicitly opted in
- [ ] ≥2 sub-tasks with **zero file overlap**
- [ ] Shared contracts listed in plan
- [ ] VERIFY anchors available after merge

If any fail → fall back to single-agent `cdCodeLane`.

---

## Split Rules

**Hard:** two agents must not edit the same file.

**Merge into one agent if:**

- A's output type is B's input (shared contract)
- Both touch same hook/service without contract owner

**Minimum unit:** one complete component or one API endpoint.

---

## Plan Section Required

```markdown
## Agent Team Assignment

| Agent | Files | Contract | Acceptance |
|-------|-------|----------|------------|

### Shared Contracts
| Contract | Owner | Consumers |
|----------|-------|-----------|
```

---

## AgentTask Template (compact)

```markdown
# AgentTask: <name>

## Goal
<one sentence>

## Files (exclusive)
- Modify: ...
- Add: ...

## Contracts (read-only)
...

## Acceptance
- [ ] ...

## Steps
1. Read context
2. Implement (Edit/Write only listed files)
3. Self-test compile + scoped tests
4. Report diff summary + acceptance checklist
```

Embed file contents in prompt if worktree base may not see feature-branch-only files.

---

## Dispatch

1. Create feature branch on main session
2. Spawn parallel agents with worktree isolation (runtime-specific API)
3. Wait for all reports
4. Per-agent diff audit vs AgentTask scope
5. **Sequential merge** — stop on first conflict
6. On conflict: write `{repo}/.claude/runtime/cdMergeRecovery.md` with pre-merge SHA

Re-dispatch fix tasks on failure (max 2 per agent, then escalate human).

---

## After Merge

Return to `cdCodeLane` → VERIFY (`cdTestByDiff`) → SHIP.
