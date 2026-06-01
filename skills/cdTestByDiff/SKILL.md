---
name: cdTestByDiff
description: >
  Git-diff test orchestrator. USE in cdCodeLane VERIFY, or when user says
  run tests, test, cdTestByDiff. AI does not choose what to test — topology
  table decides. Executes anchors and reports.
---

# cdTestByDiff

**Principle:** The topology decides anchors. AI executes and reports only.

## Scope

```bash
git diff develop...HEAD --name-only
git status --porcelain
```

Uncommitted changes: mark with warning, do not block.

## File Tags

| tag | pattern |
|-----|---------|
| F | `frontend/**` `*.tsx` `*.ts` (not `*.test.ts`) |
| B | `backend/**` `*.java` (not `*Test.java`) |
| D | `*.sql`, `@Entity`, migration configs |
| T | test files only |

All T → skip with message "test-only diff".

## Anchors

| anchor | layer | command (typical) |
|--------|-------|-------------------|
| A1 | compile | `npx tsc --noEmit` / `mvn compile` / project stack rule |
| A2 | contract | DTO ↔ types shape compare |
| A3 | flow | `jest` / `playwright` / `mvn test` |
| A4 | visual | playwright snapshots — **off by default** |

Enable A4 only when user says `/visual-test`.

## Risk Levels (3 tiers)

| level | triggers | required anchors |
|-------|----------|------------------|
| low | CSS, copy, comments | A1 if F/B touched |
| medium | pages, components, controllers, services | A1 + scoped A2/A3 |
| critical | types, auth, router, store/shared/hooks | A1 + A2 + full jest/playwright |

## Default Topology

| change pattern | risk | A1 | A2 | A3 | A4 |
|----------------|------|----|----|----|-----|
| global types / DTO | critical | yes | yes | full | no |
| store / shared / hooks | critical | yes | yes | full | no |
| pages / forms | medium | yes | no | playwright | no |
| components | medium | yes | no | optional | no |
| Controller / Service | medium | yes | shape | mvn test | no |
| Entity / sql | medium | yes | migration | no | no |
| css / copy | low | optional | no | no | no |

## Project Overrides

If `{repo}/.claude/rules/cdTestTopology.md` exists, **merge** rows; project wins on conflict.

## Execution

- Run triggered anchors; 120s timeout per command
- One anchor failure does not skip others
- Report markdown table: file → required anchors → executed → status

## Report Template

```markdown
# cdTestByDiff Report

Branch: <name> vs develop
Files changed: N
Anchors run: A1 A2 ...

| Anchor | Result |
|--------|--------|
| A1 F tsc | pass |

| File | Required | Executed | Status |
|------|----------|----------|--------|
```

Fix existing tests before adding new ones.
