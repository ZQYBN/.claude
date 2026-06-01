# cdMetaGovernance — Rules and Skills Lifecycle

META-LANE only. Never modify rules or skill triggers without user confirmation.

---

## When a Rule Is Justified

All three must be true:

- **Clear trigger** — named situation where it applies
- **Traceable source** — incident or discussion that motivated it
- **Verifiable effect** — obeying vs violating changes outcome clearly

Otherwise it is a preference — do not write it.

REVIEW must **not** auto-propose new rules. User must say "add a rule" to enter META-LANE.

---

## Where to Put Rules

1. Tighten an existing file (preferred)
2. Add a checklist item to plan/spec templates
3. Automate (lint, types, tests) instead of prose
4. New global rule file — only if cross-project and recurring

---

## Rule States

| state | meaning |
|-------|---------|
| Active | default |
| Deprecated | prefix `[DEPRECATED]` in text |
| Archived | delete text |

---

## Skill Changes

| action | trigger | process |
|--------|---------|---------|
| Promote to auto | manual invoke 3+ times | propose → user confirms |
| Demote to manual | poor auto triggers | propose → user confirms |
| Delete | user instructs | remove skill + scan all rules for dangling refs |
| Add | new skill installed | validate manually → promote if needed |

Rules = boundaries. Skills = capabilities. Rules route to skills; neither replaces the other.

---

## Change Commit Template

```
chore(rules): add <name>

Source:
- Date: YYYY-MM-DD
- Type: repeated-issue | major-incident | rule-conflict | stale-cleanup
- Event: <brief>
```
