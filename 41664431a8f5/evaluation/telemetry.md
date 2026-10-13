---
title: "Telemetry, probes and sensors"
description: "What a recipe records besides pixels - telemetry.jsonl event kinds, probes, per-frame sensors, marks and frames.tsv columns."
type: reference
url: https://jand-s.github.io/scion/41664431a8f5/evaluation/telemetry/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/evaluation/telemetry.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/trials/session.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/files/telemetry-jsonl.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/files/tables.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Telemetry, probes and sensors

Pixels show what happened; telemetry says it. Every recipe writes `telemetry.jsonl` (one JSON line per event: log
lines, notes, marks, probes, checks, faults, game events, `Fx` calls, injections) and `frames.tsv` (one row per
captured frame, with fixed columns, the game's sensors and the marks). Probes are the numbers checks and specs test.

## Probes

```csharp
session.Probe("damage", hpBefore - hpAfter);
```

- The last value per key goes to `result.json` under `probes`; `ProbeIs` tests it.
- Every call writes a `probe` line to telemetry, with frame and game time.
- Keys: lowercase letters, digits and `_`.

Core code can declare probes without referencing Trials: `[Probe("damage")]` on a field or property of a rule type
makes the adapter's generated code record it at the end of each recipe ([Core API](/scion/41664431a8f5/reference/api/core/)).

## Sensors

```csharp
session.Sensor("enemy_hp", () => battle.State.Unit(1).Hp);
```

- A column in `frames.tsv`, read once per captured frame.
- Registered once per process, in `Prepare`; columns freeze at the first `BeginCut` of a recipe.
- `state` and `tick` are the conventional names that `--check` reads first ([Verify determinism](/scion/41664431a8f5/determinism/verify/)).

## Marks

`Mark(label)`, `MarkImpact()` and `Fx.Mark(label)` label the current frame. Labels appear in the `marks` column, as
`mark` lines in telemetry and in `report.md`'s timeline. `IMPACT` is reserved for `MarkImpact`.

## Telemetry kinds

Every line has `f` (frame of the cut, -1 outside it), `t` (game seconds since the recipe start) and `kind`:

| kind | Fields | Written by |
|---|---|---|
| `log` | `type` (`Error`, `Assert`, `Exception`, `Warning`), `msg`, `stack` (up to 2 KB), `thread` (`main`, `worker`) | Unity's log |
| `note` | `msg` | `TrialSession.Note` |
| `mark` | `label` | `Mark`, `MarkImpact`, `Fx.Mark` |
| `probe` | `key`, `value` | `Probe` |
| `check` | `id`, `status`, `reason`, `frame` | the checks at the end |
| `fault` | `source`, `type`, `msg` | a graft that threw |
| `event` | `name` and the caller's fields | `TrialSession.Event` |
| `fx` | `verb`, `args`, `by` | every `Fx` call |
| `inject` | `version`, `bound` | an injection applied |

```json
{"f":14,"t":1.467,"kind":"event","name":"turn","turn":3,"hash":"9F3A0C11D2E4B5A6"}
```

## frames.tsv columns

| Column | Meaning |
|---|---|
| `frame` | Frame index in the cut, from 0. |
| `ms` | Game milliseconds since the cut started (integer). |
| `real_ms` | Wall milliseconds (informative, never compared). |
| `px` | xxHash64 of the frame's raw bytes, 16 hex digits. |
| `rng` | xxHash64 of `UnityEngine.Random.state`. |
| `unscaled`, `time` | Virtual unscaled and scaled time since the recipe start, seconds with 6 decimals. |
| sensors | One column per sensor, in registration order. |
| `marks` | Labels of the frame, comma-separated. |

## See also

- [TrialSession](/scion/41664431a8f5/trials/session/)
- [telemetry.jsonl](/scion/41664431a8f5/reference/files/telemetry-jsonl/)
- [results.tsv, frames.tsv, determinism.tsv](/scion/41664431a8f5/reference/files/tables/)
- [Effects with Fx](/scion/41664431a8f5/grafts/fx/)
