# cdStackBackend — Project Template

Copy to: `{repo}/.claude/rules/cdStackBackend.md`

> Stack: Spring Boot 3 + JPA + MySQL (adjust for your project)

## API

- All responses wrapped: `{ code, message, data }`
- Never return raw entities or primitives

## Errors

- No empty catch blocks
- Error logs: timestamp, operation context, stack trace

## Database

- Confirm actual schema before coding — not ORM alone
- No destructive SQL without explicit user approval

## Compile (VERIFY)

```bash
cd backend && ./mvnw.cmd compile
```

## Comments (new files)

Class-level Javadoc: responsibility and module context.
