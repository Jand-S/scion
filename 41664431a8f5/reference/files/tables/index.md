---
title: "results.tsv, frames.tsv, determinism.tsv"
description: "The tab-separated tables a test player writes - one line per recipe round, one line per captured frame, and one line per round comparison."
type: file
url: https://jand-s.github.io/scion/41664431a8f5/reference/files/tables/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/reference/files/tables.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/evaluation/telemetry.md
  - https://jand-s.github.io/scion/41664431a8f5/determinism/verify.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/files/run-folder.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# results.tsv, frames.tsv, determinism.tsv

Three tables complement the JSON files: `results.tsv` has one line per (id, round) a process played, `frames.tsv` one
line per captured frame of a recipe, `determinism.tsv` one line per round comparison with `--check`. All three are
tab-separated with a header line, so they load in any spreadsheet or `awk`.

## Location

| File | Path |
|---|---|
| `results.tsv` | `<scion-out>/results.tsv` |
| `frames.tsv` | `<scion-out>/<IdDir>/r<n>/frames.tsv`, and `scion/baselines/<IdDir>/<platform>/frames.tsv` |
| `determinism.tsv` | `<scion-out>/determinism.tsv` (with `-scion-check`) |

## Written by and read by

Written by the test player (`frames.tsv` in baselines by `trial approve`). Read by the CLI (comparisons), agents and
people.

## Schema

TSV rules: the first line is the header; lines starting with `#` are comments; the format version is the
`schema_version` of the sibling JSON (`run.json`); free text comes last, with tabs and line breaks replaced by spaces.
Hashes are 16 lowercase hex digits.

## Fields

### results.tsv columns

`round id ok seconds switch_seconds frames hash px_hash objects mono_mb native_mb setup_s setup_fps cut_s read_s wait_s outputs_s checks grafts error`

| Column | Meaning |
|---|---|
| `ok` | 1 or 0, the same as `result.json`. |
| `objects`, `mono_mb`, `native_mb` | Live objects and memory after the recipe. |
| `setup_fps` | Frame rate during setup (real time). |
| `checks` | Checks that did not pass, `id=status` comma-separated; empty when all passed. |
| `grafts` | `Injection.Version`. |
| `error` | Free text, last. |

Footer comment: `# prepare <s>s, total <s>s`.

### frames.tsv columns

`frame ms real_ms px rng unscaled time <sensors...> marks`

| Column | Meaning |
|---|---|
| `frame` | Index in the cut. |
| `ms` | Game milliseconds since the cut began (integer). |
| `real_ms` | Wall milliseconds; informative, never compared. |
| `px` | xxHash64 (seed 0) of the frame's raw bytes; empty when hashing is off. |
| `rng` | xxHash64 of the 16 bytes of `UnityEngine.Random.state`. |
| `unscaled`, `time` | Virtual times minus the recipe start, seconds with 6 decimals. |
| sensors | One per `Sensor`, decimal, in registration order; frozen at the first `BeginCut`. |
| `marks` | Labels, comma-separated, last. |

### determinism.tsv columns

`id round vs_round status frame cause detail`

| Column | Meaning |
|---|---|
| `status` | `same` or `diff`. |
| `frame` | First differing frame; -1 when only the count or final hash differs. |
| `cause` | First differing of `state`, `tick`, `rng`, `unscaled`, `time`, `px`, `frames`; `none` when equal. |
| `detail` | Free text. |

## Example

```tsv
frame	ms	real_ms	px	rng	unscaled	time	state	tick	marks
0	0	12	3f1c0a9b7e22d410	9a0b11c2d3e4f506	0.000000	0.000000	7781	0	
14	466	201	a1b2c3d4e5f60718	9a0b11c2d3e4f506	0.466667	0.466667	7790	14	IMPACT
```

## Compatibility

Columns may be added before the last column (`error` or `marks`) in minor versions; read columns by header name.
