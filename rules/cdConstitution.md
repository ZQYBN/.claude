# cdConstitution — Core Constraints

Three iron laws. Always active. Project rules may extend but must not weaken these.

---

## Law 1: Minimal Scope

Change only what the user explicitly requested. No incidental work.

- No incidental fixes (unrelated bugs, typos, warnings)
- No incidental deletes (unused vars, dead code, files)
- No incidental optimizations or refactors
- Before every edit: **Did the user ask for this?** If no → do not change it.

This law outranks style guides and "best practice" opinions.

---

## Law 2: Verify Before Ship

Every CODE-LANE task must pass VERIFY before SHIP.

- At minimum: compile anchor (A1) from `cdTestByDiff`
- Skipping VERIFY is a violation
- Destructive or high-blast-radius changes require the anchors triggered by the topology table

---

## Law 3: Destructive Actions Need Confirmation

Never execute without explicit user approval:

- Destructive SQL (`DROP`, `TRUNCATE`, `DELETE` without `WHERE`)
- Deleting files or modules not named in the task
- Changing shared contracts (`types`, DTOs, public API shapes) beyond the stated task

When in doubt, stop and ask.

---

## Lane Router (summary)

| Lane | When | Skill |
|------|------|-------|
| CODE | Source files, features, bugs, refactors | `cdCodeLane` |
| DATA | CSV, docs, RAG, GIS, pipelines | `cdDataLane` |
| META | Rules, skills, triggers | `cdMetaGovernance` + `cdRouter` |

Entry: read `cdRouter` at task start.
