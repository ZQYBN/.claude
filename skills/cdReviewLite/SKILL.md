---
name: cdReviewLite
description: >
  Lightweight pre-PR review. USE after cdTestByDiff passes, or when user asks
  for review. Ten-item checklist with P0-P2 severity. Review-only unless user
  asks to fix.
---

# cdReviewLite

Structured review of current git diff. Default: report only.

## Preflight

```bash
git status -sb
git diff --stat
git diff
```

Empty diff → ask if staged or commit range review needed.

## Severity

| level | meaning | action |
|-------|---------|--------|
| P0 | security, data loss, correctness | block merge |
| P1 | logic error, contract break | fix before merge |
| P2 | maintainability, smell | fix or follow-up |

## Checklist (10)

1. Diff matches stated task scope — no drive-by changes
2. No empty catch / swallowed errors
3. API responses match project envelope if applicable
4. Types/DTOs consistent across F/B if both changed
5. No secrets in diff
6. SQL parameterized; no destructive SQL without approval
7. No new `any` without `// why:` comment
8. No unauthorized architecture layers or new abstractions
9. Docs/plans updated if behavior changed
10. Schema/registry changes align with `cdDelivery` if UI task

## Output

```markdown
## cdReviewLite Summary

**Verdict:** ship | fix-first | block

### P0
- ...

### P1
- ...

### P2
- ...

### Scope OK
yes | no — <files>
```

Do not implement fixes unless user requests.
