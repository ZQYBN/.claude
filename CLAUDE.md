# Claude Code Global Rules (cd namespace)

Single entry for the **cd** workflow system. Project `.claude/CLAUDE.md` may override stack and paths — not the three laws in `cdConstitution`.

---

## Start Every Task

1. Read `rules/cdConstitution.md`
2. Invoke skill **`cdRouter`** — classify lane and size
3. Follow the lane skill to completion

```
cdRouter → cdCodeLane | cdDataLane | cdMetaGovernance
```

---

## Three Laws (summary)

Full text: `rules/cdConstitution.md`

1. **Minimal scope** — only requested changes
2. **Verify before ship** — `cdTestByDiff` in CODE-LANE
3. **Destructive actions need confirmation**

---

## Rules Index

| file | when to read |
|------|----------------|
| `rules/cdConstitution.md` | always |
| `rules/cdEngineering.md` | CODE-LANE, git, plans |
| `rules/cdDelivery.md` | UI flow, schema, generative UI |
| `rules/cdMetaGovernance.md` | META-LANE only |

### Project-local (in repo)

| file | purpose |
|------|---------|
| `.claude/rules/cdStackFrontend.md` | React, TS, UI lib |
| `.claude/rules/cdStackBackend.md` | API, DB, server stack |
| `.claude/rules/cdTestTopology.md` | cdTestByDiff overrides |

Templates: `.claude/templates/cdStackFrontend.template.md`, `cdStackBackend.template.md`, `cdTestTopology.template.md`

---

## Skills Index

| skill | trigger | role |
|-------|---------|------|
| `cdRouter` | every task | lane + size classification |
| `cdCodeLane` | CODE-LANE | ALIGN → CHANGE → VERIFY → SHIP |
| `cdDataLane` | DATA-LANE | staged file pipeline |
| `cdTestByDiff` | VERIFY / "run tests" | diff-driven test anchors |
| `cdReviewLite` | pre-PR / "review" | 10-item checklist |
| `cdTeamLane` | `/team` opt-in | multi-agent split + merge |
| `cdUiBuild` | new UI opt-in | design + implementation |

---

## CODE-LANE Quick Reference

```
ALIGN → CHANGE → VERIFY → SHIP
```

| size | ALIGN |
|------|-------|
| ≤2 files | verbal |
| 3–9 files | short plan |
| ≥10 files | plan + APPROVE |

---

## Language

Rules and cd skills are **English**. User communication may follow user preference (e.g. 中文).
