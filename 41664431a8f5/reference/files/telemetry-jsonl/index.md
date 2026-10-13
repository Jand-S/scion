---
title: "telemetry.jsonl"
description: "The per-recipe event log - one JSON line per log message, note, mark, probe, check, fault, game event, Fx call and injection."
type: file
url: https://jand-s.github.io/scion/41664431a8f5/reference/files/telemetry-jsonl/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/reference/files/telemetry-jsonl.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/evaluation/telemetry.md
  - https://jand-s.github.io/scion/41664431a8f5/grafts/fx.md
  - https://jand-s.github.io/scion/41664431a8f5/trials/coop.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# telemetry.jsonl

`telemetry.jsonl` records everything that happened during one recipe round, one JSON object per line, in order: Unity
log messages, notes, marks, probes, check results, graft faults, the game's own events, every `Fx` call with the graft
that made it, and injections. Co-op peers compare their `turn` events from it.

## Location

`<scion-out>/<IdDir>/r<n>/telemetry.jsonl`.

## Written by and read by

Written by the test player at the end of the recipe. Read by the CLI (co-op comparison), gates (`TelemetryVsPixel`),
the motion metrics, agents and the judge.

## Schema

`https://jand-s.github.io/scion/41664431a8f5/schemas/v1/telemetry-line.schema.json` validates each line.

## Fields

Every line has `f` (frame of the cut, -1 outside it), `t` (game seconds since the recipe start) and `kind`.

| kind | Fields |
|---|---|
| `log` | `type` (`Error`, `Assert`, `Exception`, `Warning`), `msg`, `stack` (up to 2 KB), `thread` (`main`, `worker`) |
| `note` | `msg` |
| `mark` | `label` |
| `probe` | `key`, `value` |
| `check` | `id`, `status`, `reason`, `frame` |
| `fault` | `source`, `type`, `msg` |
| `event` | `name` plus the caller's fields |
| `fx` | `verb`, `args`, `by` (the graft, or null) |
| `inject` | `version`, `bound` |

## Example

<!-- validate: telemetry-line -->
```json
{"f":14,"t":1.467,"kind":"fx","verb":"Shake","args":[0.3],"by":"Orchard.Grafts.CritShake.After"}
```

More lines:

```jsonc
{"f":-1,"t":0.1,"kind":"note","msg":"battle ready"}
{"f":14,"t":1.467,"kind":"mark","label":"IMPACT"}
{"f":14,"t":1.467,"kind":"probe","key":"damage","value":12}
{"f":30,"t":2.0,"kind":"event","name":"turn","turn":1,"hash":"9F3A0C11D2E4B5A6"}
```

## Compatibility

New kinds and fields may be added in minor versions; readers must skip kinds they do not know.
