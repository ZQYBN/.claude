# cdTestTopology — Project Overrides

Copy to: `{repo}/.claude/rules/cdTestTopology.md`

Merge with global defaults in `cdTestByDiff`. **Project rows win** on conflict.

## Project Topology Overrides

| change pattern | risk | A1 | A2 | A3 | A4 |
|----------------|------|----|----|----|-----|
| example: `pages/checkout/` | medium | tsc | — | playwright | — |
| example: `workspaceFactory` | critical | tsc | contract | jest full | — |

## Notes

- Add paths specific to your repo's high-blast-radius modules
- Do not duplicate global rows unless overriding them
