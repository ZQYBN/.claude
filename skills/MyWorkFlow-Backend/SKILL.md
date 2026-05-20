---
name: MyWorkFlow-Backend
description: >
  Backend workflow supplement. Provides backend-specific compile check
  (./mvnw.cmd compile), database schema verification, and backend-specific
  review checklist (API response format, error handling, SQL safety). Only
  invoked by MyWorkFlow — do not invoke this skill directly.
---

# MyWorkFlow-Backend

Backend workflow supplement. This skill is loaded by `MyWorkFlow` when
Phase 0 detects a backend task. It does not define its own workflow — it
adds backend-specific checks to MyWorkFlow's phases.

## IMPLEMENT phase: additional steps

### Compile check

Run before every commit:

```bash
./mvnw.cmd compile
```

Java compilation catches signature mismatches, missing dependencies, and
annotation errors. Running it locally before pushing avoids wasting time on
CI failures that take minutes to surface.

### Database schema verification

Before writing any code that touches the database:

- Confirm the actual database schema — column nullability, index definitions,
  valid values for status fields. ORM entities can drift from the real schema;
  the database is the source of truth.
- Never infer schema from ORM entity classes alone. A `@Column(nullable =
  false)` annotation means nothing if the actual column allows nulls.
- Never execute destructive SQL: no `DROP TABLE`, no `TRUNCATE`, no
  `DELETE` without a `WHERE` clause. These operations are irreversible and
  have no safety net in a demo/staging environment.

## REVIEW phase: additional checklist

Run these on top of MyWorkFlow's generic review checklist:

- **API response format**: Every endpoint returns `{ code, message, data }`.
  No bare entities, no raw primitives. The frontend depends on this envelope
  for unified error handling — breaking the contract breaks the UI.
- **Error handling**: No empty catch blocks. Every caught exception is either
  logged (with timestamp, operation context, and stack trace), re-thrown, or
  translated into an error response. Silently swallowed exceptions are the
  hardest bugs to diagnose — they leave no trace.
- **Log quality**: Error logs include when it happened, what operation was
  running, and the full stack trace. Log level matches actual severity —
  don't log a recoverable validation error at ERROR level.
- **SQL safety**: No injection vectors (use parameterized queries / JPA
  bind parameters). No destructive statements without explicit user
  confirmation.
- **Schema consistency**: Code matches the actual database. If you added a
  field to an entity, does the corresponding column exist? If you made a
  column non-null, do all existing rows have values?

## TEST phase

`MyTestBasedOnGit` automatically determines the backend test scope (mvn
compile, mvn test, API contract verification) from the git diff. Do not
duplicate that logic here.
