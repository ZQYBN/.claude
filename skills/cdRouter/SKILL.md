---
name: cdRouter
description: >
  USE AT THE START OF EVERY TASK. Single entry router for the cd workflow
  system. Classifies the request into CODE, DATA, or META lane and loads the
  matching skill. Replaces legacy MyWorkFlow Phase 0 TRIAGE.
---

# cdRouter

Classify the user request in one pass, then hand off. Do not implement in this skill.

## Three Questions

1. **What is being changed?** code / data files / rules or skills
2. **How large?** ≤2 files | 3–9 files | ≥10 files or new module
3. **Fast UI path?** read `cdDelivery` — schema-first vs registry vs host

## Lane Table

| Condition | Lane | Next skill |
|-----------|------|------------|
| `.java`, `.tsx`, `.ts`, `.py`, features, bugs, refactors | CODE | `cdCodeLane` |
| CSV, reports, RAG, GIS, doc pipelines, format conversion | DATA | `cdDataLane` |
| Add/change/remove rules or skills or triggers | META | `cdMetaGovernance` then return to lane if code also needed |
| User says `/team` | CODE + team | `cdCodeLane` with `cdTeamLane` |

## Size Hint (pass to cdCodeLane)

| size | ALIGN requirement |
|------|-------------------|
| small | ≤2 files, no new public API → verbal only |
| medium | 3–9 files → short plan in `docs/plans/` |
| large | ≥10 files or new module → plan + user `APPROVE` |

## Branch Prefix (CODE only)

- bug / defect / crash → `bugfix/`
- else → `feature/`

## Stack Rules

Load from **project** repo if present:

- `.claude/rules/cdStackFrontend.md`
- `.claude/rules/cdStackBackend.md`
- `.claude/rules/cdTestTopology.md` (overrides for cdTestByDiff)

Global stack defaults are intentionally absent — use project `cdStack*` rules.

## Output

Before leaving router, state:

```
Lane: CODE | DATA | META
Size: small | medium | large
Next: cdCodeLane | cdDataLane | cdMetaGovernance
Delivery mode: schema | registry | host | n/a
```
