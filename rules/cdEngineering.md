# cdEngineering — Engineering Standards

Applies to all CODE-LANE work. Stack-specific rules live in the **project** repo:
`.claude/rules/cdStackFrontend.md`, `cdStackBackend.md` (not global).

---

## Decision Priority

When rules conflict, decide in this order:

1. Data safety
2. Correctness
3. Architecture stability (prefer not restructuring)
4. Maintainability
5. Minimal blast radius
6. Performance (only with evidence)
7. Style
8. Literal rule text (most specific wins)

If still unclear → stop and ask the user.

---

## Change Priority

1. Remove redundant code
2. Merge duplicate logic
3. Reduce file count
4. Add new code last

Constraints:

- Do not change existing behavior unless requested
- Do not add unrequested features
- No premature abstraction: extract shared logic only after **3+** repetitions
- No speculative `Base*`, `Abstract*`, `Generic*Factory` shells

---

## Git — Commits

Format: `<type>(<scope>): <short description>`

| type | meaning |
|------|---------|
| feat | new feature |
| fix | bug fix |
| refactor | refactor, same behavior |
| perf | performance |
| docs | documentation |
| style | formatting only |
| test | tests |
| chore | tooling, deps |
| revert | revert |

- Description ≤ 50 chars, no trailing period
- scope: lowercase English
- One logical change per commit

---

## Git — Branches

| branch | role | AI permission |
|--------|------|---------------|
| main | production | never touch |
| develop | integration | never merge into |
| feature/* | new work | may create |
| bugfix/* | fixes | may create |

AI creates feature/bugfix from `develop`. User merges PRs.

---

## CLI

- One-off low-risk ops (rename, search, single replace): CLI OK
- Reusable batch work: prefer existing `script/`; archive one-offs to `script/archive/`
- Do not create throwaway scripts with no reuse value

---

## Plans and Specs

| artifact | path | when |
|----------|------|------|
| Plan | `docs/plans/YYYY-MM-DD-slug.md` | ≥3 files or new module |
| Spec | `docs/superpowers/specs/YYYY-MM-DD-slug.md` | design-first features |

≤2 files, no new API: verbal ALIGN in chat only — no plan file.
