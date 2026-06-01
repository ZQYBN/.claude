# cdStackFrontend — Project Template

Copy to: `{repo}/.claude/rules/cdStackFrontend.md`

> Stack: React + TypeScript + Ant Design 6 (adjust for your project)

## React

- Function components and hooks only
- No `any` — explicit types
- Fixed widths for Select/Input; explicit Table column widths — no layout jitter

## Ant Design 6

- No deprecated APIs
- Notification: use `title`, not `message`
- Drawer: use `size`, `destroyOnHidden` — not `width`, `destroyOnClose`
- Avoid new `antd.List` — prefer semantic containers

## Large Trees

- Virtual scroll, flat projection, stable keys, O(1) expand ops
- No deep recursive render; memoize tree builds

## Compile (VERIFY)

```bash
cd frontend && npx tsc --noEmit
```

## Comments (new files)

File header block describing responsibility and key constraints.
