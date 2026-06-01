---
name: cdUiBuild
description: >
  OPT-IN UI implementation helper for cdCodeLane. USE when creating new pages,
  components, or visual polish. Combines design direction with implementation.
  For enterprise Ant Design projects follow project cdStackFrontend.md over
  aesthetic defaults here.
---

# cdUiBuild

Invoke during cdCodeLane CHANGE when building or heavily restyling UI.

## When to Use

- New page, dashboard, landing section, modal flow
- User asks to beautify, redesign, or improve UX
- Generative UI: new registry component that needs visual baseline

Skip for: one-line CSS fix, label change, schema-only delivery per `cdDelivery`.

---

## Priority Order

1. **Project stack rules** (`cdStackFrontend.md`) — Ant Design, layout stability, no `any`
2. **Existing app patterns** — match neighboring components
3. **Design direction below** — only where stack is silent

---

## Design Direction

Before coding, pick one clear direction (state it in one line):

- Purpose and primary user action
- Tone: minimal | dense admin | marketing | clinical/form-heavy
- Constraints: a11y, performance, mobile

Implement working code — not mockups.

### Quality Bar

- Explicit typography and spacing; avoid generic "AI slop" palettes
- Fixed widths for Select/Input where layout stability matters (see stack rules)
- Motion: purposeful, not decorative overload
- Prefer CSS variables / design tokens consistent with host

### Enterprise Override

If project uses Ant Design admin UI:

- Do not fight the design system for novelty
- Prefer composition of existing primitives
- `cdDelivery` schema-driven widgets should look native to the host

---

## Generative UI Hook

When task is registry + schema:

1. Implement component with stable props interface (contract)
2. Document component id for schema whitelist
3. Provide example schema snippet in plan or PR description

---

## Output

- Component files within cdCodeLane scope
- Brief note: design direction chosen + any schema id registered
