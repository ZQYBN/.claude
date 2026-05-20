---
name: MyWorkFlow-Skills
description: >
  Skill lifecycle manager. Invoke this skill when the user asks to promote a
  skill to auto-trigger, demote a skill to manual-only, delete a skill, or
  add a new skill. Also invoke when you notice a skill consistently
  under-triggers or over-triggers and needs its trigger configuration
  adjusted. Skills are AI capability modules — each one encapsulates domain
  knowledge and execution workflows for a specific task family.
---

# MyWorkFlow-Skills

Skill lifecycle manager. Skills are AI capability modules — each one
encapsulates domain knowledge and an execution flow. The management goal:
high-frequency scenarios trigger automatically, low-frequency ones are
invoked manually.

## Change rules

| Action | Trigger | Process |
|--------|---------|---------|
| Promote to auto-trigger | Same skill invoked manually 3+ times | You propose, user confirms |
| Demote to manual | Auto-triggered skill is frequently skipped or produces poor output | You propose removing auto-trigger, user confirms |
| Delete | Skill doesn't meet quality bar | User directly instructs deletion; you scan all rule files and remove every reference |
| Add | New skill installed | Manually invoke, validate it works, then follow the promote flow |

- Never add, remove, or change skill trigger configuration without user
  confirmation
- When deleting a skill, scan every rule file for dangling references —
  a deleted skill with leftover trigger rules causes confusion

## How skills and rules relate

- **Rules** define boundaries — what you can and cannot do (constraints)
- **Skills** provide capabilities — domain knowledge and processes (tools)
- **Rules** decide when to invoke which **Skill** (trigger conditions)

They are complementary: rules prevent mistakes, skills enable quality.
Neither replaces the other.
