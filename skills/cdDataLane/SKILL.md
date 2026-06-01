---
name: cdDataLane
description: >
  DATA-LANE staged file pipeline. For CSV cleaning, format conversion, reports,
  RAG preprocessing, GIS, and multi-step file workflows. Filesystem is source
  of truth — not chat memory. Invoked by cdRouter. Does not use git feature
  branches.
---

# cdDataLane

Staged pipeline v3 (compact). **No git branches** for DATA tasks.

Iron laws:

- One stage per interaction — no skipping stages in one reply
- Input dirs and prior stages are read-only
- Every stage writes artifacts + stage `README.md`
- Errors freeze in place — log to `90_temp/error_log.md`, no wipe-and-retry

---

## INIT (gate)

Pick exactly one topology:

| topology | signals | entry dir |
|----------|---------|-----------|
| general | CSV, Excel, JSON, charts | `00_input/` |
| document | papers, reports, outlines | `00_material/` |
| rag | embeddings, chunking, corpus | `00_raw_docs/` |
| gis | raster, vector, maps | `00_raw_data/` |

Actions:

1. Print absolute `$PWD` as root
2. Declare directory tree for chosen topology
3. Write `./README.md` (goal, topology, inputs)
4. Block until user sends `APPROVE:INIT`

---

## General Topology (default)

```
./
├── 00_input/         # read-only source
├── 01_cleaning/
├── 02_transform/
├── 03_validate/
├── 04_analysis/
├── 05_visualization/
├── 06_output/
├── 90_temp/
├── 99_archive/
└── README.md
```

Document / RAG / GIS variants: use extended stage trees from prior pipeline spec; keep `90_temp`, `99_archive`, `README.md`.

---

## Stage README Template

```markdown
# Stage: <name>

## Input
- ../<prev>/<file>

## Operators
1. ...

## Output
- ./<file>
```

---

## Response Header (every reply)

```
[stage]: <INIT | 01_cleaning | ...>
[topology]: <general | document | rag | gis>
[input]: <path>
[output]: <path>
[status]: in_progress | written_await_confirm
```

## Response Footer

```
Next: <next stage>
Reply APPROVE:STAGE to continue.
```

---

## Versioning

- Use `_v1`, `_v2` suffixes — never `final`, `new_final`
- Milestones → move old files to `99_archive/`

---

## Not For

Software feature work → `cdCodeLane` + git. Do not mix lanes in one task.
