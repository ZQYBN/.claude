---
name: cdCodeLane
description: >
  CODE-LANE workflow: ALIGN → CHANGE → VERIFY → SHIP. Default single-agent
  implementation. Invoked by cdRouter for all code tasks. Use cdTeamLane only
  when user explicitly requests /team or plan marks multi-agent.
---

# cdCodeLane

Four steps. No phase ledger. No mandatory Phase Result blocks.

```
ALIGN → CHANGE → VERIFY → SHIP
```

Read `rules/cdConstitution.md` and `rules/cdEngineering.md` before CHANGE.

---

## Step 1: ALIGN

**Goal:** Same understanding of scope before edits.

| size (from cdRouter) | action |
|----------------------|--------|
| small | State goal, files, approach in ≤5 lines. No plan file. |
| medium | Write `docs/plans/YYYY-MM-DD-slug.md` (≤1 page) |
| large | Full plan + user token `APPROVE` before CHANGE |

UI/flow tasks: apply `rules/cdDelivery.md` — prefer schema → registry → host.

Ambiguous feature or design → ask clarifying questions (one at a time). Skip long brainstorming unless user asks.

**Do not write code in ALIGN.**

---

## Step 2: CHANGE

1. Create branch from `develop`: `feature/*` or `bugfix/*`
2. Implement scoped changes only (Law 1)
3. One logical change per commit
4. Before each commit: run compile check if project defines it in stack rules
5. New UI pages/components → invoke `cdUiBuild` when polish matters

### Optional: cdTeamLane

Only if user said `/team` OR plan includes `multi-agent: true`:

- Invoke `cdTeamLane` for split, dispatch, merge
- Main agent may resolve **one** merge conflict file if human is blocked; otherwise re-dispatch

Default: **single agent** implements everything.

---

## Step 3: VERIFY

Always invoke `cdTestByDiff`.

- All required anchors must pass before SHIP
- Failures → fix → re-run VERIFY

---

## Step 4: SHIP

- Push branch; user opens PR
- AI never merges to `develop` or `main`
- Optionally invoke `cdReviewLite` before user PR (recommended for medium/large)

Archive plan after merge if user wants: `docs/archive/YYYY-MM/<plan-file>`.

---

## Prohibited

- Skipping VERIFY
- Incidental refactors (Law 1)
- Merging branches without user
- Default multi-agent without `/team`
