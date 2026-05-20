---
name: MyWorkFlow-Rules
description: >
  Rule lifecycle manager. Invoke this skill when you detect a rule gap (a bug
  that should have been prevented by an existing rule but wasn't), a rule
  conflict (two rules pulling in opposite directions), an obsolete rule
  (technology upgrade made it irrelevant), or an over-constraining rule (the
  rule forces a worse outcome). Also invoke when the user explicitly asks to
  create, modify, deprecate, or delete a rule. This skill defines the
  end-to-end process: discover → discuss → draft → confirm → write → record.
---

# MyWorkFlow-Rules

Rule lifecycle manager. Rules exist to prevent past mistakes from repeating.
The goal is not to record everything — it's to record what will matter again.

## When a rule is justified

A valid rule must satisfy all three:

- **Clear trigger**: you can name the specific situation where it applies
- **Traceable source**: you can point to the incident or discussion that
  motivated it
- **Verifiable effect**: obeying vs. violating produces a clear difference in
  outcome

If you can't answer all three, it's not a rule — it's a preference. Don't
write it.

## Where to put a new rule

Prefer the smallest, closest scope — in priority order:

1. **Adjust an existing rule file** — tighten wording, add a bullet, expand
   a condition. An edit is better than a new file.
2. **Add a checklist item to templates** — if the lesson fits in a plan or
   spec template, put it there.
3. **Strengthen automation** — if a linter, typechecker, or test can catch
   it, automate it. Don't write a rule a machine can enforce.
4. **Write to a global rule file** — only if the problem is cross-project,
   recurring, and long-lived.

**Litmus test**: will this same problem reappear in the same form? If no,
don't write a rule.

## When to create or change a rule

| Trigger | Example |
|---------|---------|
| Repeated mistake | Same bug type appears 2+ times |
| Major incident | Data loss, production outage, security breach |
| Rule conflict | Two rules can't both be followed |
| Obsolete rule | Tech stack upgrade makes it irrelevant |
| Over-constraining rule | Rule forces a clearly worse choice |

Don't create a rule for: one-off mistakes, context-specific experience,
things automation can catch, or pure aesthetic preference.

## Rule lifecycle

| State | Meaning | Marking |
|-------|---------|---------|
| Active | In effect, must be followed | None (default) |
| Deprecated | Phasing out, reference only | `[DEPRECATED]` prefix in rule text |
| Archived | Gone | Delete the text |

Review rules for staleness after major tech stack upgrades, phase changes
(demo → production), or when you notice a rule hasn't fired in a long time.

## Change process

1. **Discover** — user flags an issue, or you identify a gap during REVIEW
2. **Discuss** — is this a rule-level gap or a one-time execution mistake?
3. **Draft** — you propose rule text with: behavior, motivation, trigger
4. **Confirm** — user approves before you write
5. **Write** — place the rule in the right file
6. **Record** — commit with the rule change template:

```
chore(rules): add <rule-name> rule

Source:
- Date: YYYY-MM-DD
- Type: repeated-issue / major-incident / rule-conflict / stale-cleanup
- Event: <brief description>
```

## Writing a good rule

Every rule needs three elements:
- **Behavior**: what to do or not do
- **Motivation**: one sentence on why this constraint exists
- **Trigger**: when the rule applies

Example of a well-formed rule:

> "Don't abstract until the same pattern appears 3+ times. Early abstraction
> creates empty shells (BaseXXX, AbstractXXX, GenericXXXFactory) that add
> indirection without value."
>
> Source:
> - Date: 2026-05
> - Type: repeated-issue
> - Event: AI repeatedly created BaseXXX, AbstractXXX shell classes that
>   increased maintenance cost without reuse

## Anti-patterns

- **Over-specification**: writing an extremely narrow rule after a single
  incident. Abstract to the principle level instead.
- **Rule inflation**: adding a new rule every conversation. Edit existing
  rules first.
- **Zombie rules**: rules that exist but never trigger. Mark Deprecated,
  then delete.
- **Conflicting rules**: two rules pointing opposite directions. Resolve
  using the priority order (data safety > correctness > architecture
  stability > ...) or merge into one.

## Permission

Never modify a rule file without user confirmation. Rules are the user's
guardrails, not yours to adjust unilaterally.
