# cdDelivery — Config-Driven and Generative UI

Cross-project delivery protocol. Aligns fast iteration with a stable host app.
Not micro-frontends by default — schema-driven UI first.

---

## Fixed Layer (slow change)

- Host shell: routing, auth, layout, telemetry
- Component Registry: local components shipped via normal CI
- Renderer + schema validator: maps schema → registry components

---

## Variable Layer (fast change)

- UI schema (JSON): layout, component ids, props, actions
- Optional flow schema: steps, branches, validations
- Backend API or config service returns schema version

LLM should output **constrained schema**, not arbitrary remote JS.

---

## Delivery Priority (vibe coding)

1. **Schema-only change** — no app release, or config publish only
2. **New registry component** — small PR, then schema references it
3. **Host shell change** — large PR, requires explicit user APPROVE

---

## ALIGN Questions (when user asks to change UI or flow)

1. Can this be done by schema alone?
2. If not, is a new registry component enough?
3. Only then: change host, routes, or shared state

---

## Schema Safety

- Whitelist component ids from Registry
- Validate schema against JSON Schema or project contract
- Version schemas; support rollback via config flag
- Never `eval` or load unreviewed JS from CDN for core product paths

---

## Module Federation (optional, not default)

Use page-level remotes only when:

- Separate team owns a whole domain
- Long-lived independent release cadence
- Acceptable isolation vs integration tradeoff

Fine-grained component remotes are discouraged — prefer schema + registry.
