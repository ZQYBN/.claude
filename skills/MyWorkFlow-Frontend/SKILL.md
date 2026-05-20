---
name: MyWorkFlow-Frontend
description: >
  Frontend workflow supplement. Provides frontend-specific compile check
  (npx tsc --noEmit), domain skill dispatch (frontend-design for new UI,
  ui-ux-pro-max for design decisions), and frontend-specific review
  checklist. Only invoked by MyWorkFlow — do not invoke this skill directly.
---

# MyWorkFlow-Frontend

Frontend workflow supplement. This skill is loaded by `MyWorkFlow` when
Phase 0 detects a frontend task. It does not define its own workflow — it
adds frontend-specific checks and skill calls to MyWorkFlow's phases.

## IMPLEMENT phase: additional steps

### Compile check

Run before every commit:

```bash
npx tsc --noEmit
```

TypeScript compilation catches contract mismatches — props that don't match,
imports that broke, types that drifted. Catching these before commit keeps
the branch green and avoids wasting CI cycles.

### Domain skill dispatch

| When | Invoke | Why |
|------|--------|-----|
| Creating a new UI component or page | `frontend-design` | Produces polished, production-grade interfaces instead of generic AI aesthetics |
| Making design decisions (style, color, layout) | `ui-ux-pro-max` | Provides 50+ styles, 161 palettes, 57 font pairings — avoids design drift across the app |

These skills are the frontend counterpart to the brainstorming and planning
that happens in DISCUSS/PLAN. They keep the UI consistent and intentional.

## REVIEW phase: additional checklist

Run these on top of MyWorkFlow's generic review checklist:

- **Component cohesion**: Does each component have a single, clear
  responsibility? Are props well-defined? A component that does too much is
  hard to test and hard to reuse.
- **State mutation radius**: Does this change touch `store/`, `shared/`, or
  `hooks/`? Changes to high-level state affect the entire app — verify the
  blast radius is understood and intentional.
- **Layout stability**: Select and Input components must use fixed widths;
  Table columns must have explicit widths. Dynamic-width elements cause the
  page to reflow as content changes, which feels broken to the user.
- **Ant Design 6 compliance**: No deprecated APIs. Notification uses `title`
  not `message`. Drawer uses `size` and `destroyOnHidden` not `width` and
  `destroyOnClose`. No new `antd.List` usage — prefer native semantic
  containers.
- **TypeScript safety**: No `any` types. Every type is explicit so the
  compiler can catch mistakes before they reach runtime.

## TEST phase

`MyTestBasedOnGit` automatically determines the frontend test scope (tsc,
jest, Playwright) from the git diff. Do not duplicate that logic here.
