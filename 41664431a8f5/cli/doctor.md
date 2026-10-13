---
title: "doctor"
description: "Check the Scion setup - configuration, CLI and package versions, Unity, the test build, the machine and the judge - and report the adoption level."
type: cli
url: https://jand-s.github.io/scion/41664431a8f5/cli/doctor/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/cli/doctor.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/start/install.md
  - https://jand-s.github.io/scion/41664431a8f5/trials/parallel.md
  - https://jand-s.github.io/scion/41664431a8f5/determinism/diagnose.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# doctor

`dotnet scion doctor` checks that everything Scion needs is in place and says what to do about anything that is not.
It reports one check per line with a `hint`, the suggested number of parallel copies for this machine, the
project's adoption level (`adoption_level`, 1 to 3) and the next step to the next level (`adoption_next`).
`--level` sets how deep the checks go.

## Synopsis

```text
dotnet scion doctor [--level 1|2|3] [--format json|text]
```

## Description

| Level | Checks |
|---|---|
| 1 | `scion.json` and `ScionSettings.json` parse; CLI and package versions match; .NET SDK; git; the output folder is writable; free memory and cores. |
| 2 (default) | Level 1, plus: Unity of the project's version; the editor lock and `com.unity.pipeline`; the test build exists, has `SCION_TRIALS`, is not stale; `scion-build.json` reads; free disk under `trials.out`. |
| 3 | Level 2, plus: starts the test build with `-scion-list` (recipes, adapters, SCN201 to SCN203; up to 60 s); the judge's eye model; Git LFS for baselines; suppressions with justifications. |

On a fresh project the `build` check warns until the first `dotnet scion build test`; a warning never fails `doctor`.

Each check has an `id`, a `status` (`ok`, `warn`, `fail`), a `detail` and, unless ok, a `hint`.

`suggested_copies` is the smaller of (cores - 1) and (free memory / memory per copy), using the last run's
measurement or 600 MB.

## Options

| Option | Default | Meaning |
|---|---|---|
| `--level 1\|2\|3` | 2 | Check depth (not the adoption level). |
| `--format json\|text` | auto | Output format. |

## Output

<!-- validate: cli/doctor -->
```json
{"schema_version":1,"command":"doctor","ok":true,"level":2,"version":"1.0.0","adoption_level":2,"adoption_next":"level 3: register an ISimulation over Orchard.Core (see Core API)","checks":[{"id":"config","status":"ok","detail":"/Users/alex/orchard/scion.json"},{"id":"versions","status":"ok","detail":"CLI 1.0.0, package 1.0.0"},{"id":"unity","status":"ok","detail":"6000.3.25f1"},{"id":"build","status":"ok","detail":".scion/builds/test/Orchard.app (4c49a1e0b2f3)"},{"id":"machine","status":"ok","detail":"10 cores, 9.1 GB free of 16.0 GB"}],"suggested_copies":6,"next_step":"dotnet scion trial run -j 6"}
```

| Field | Meaning |
|---|---|
| `level` | The check depth that ran. |
| `version` | The CLI version. |
| `adoption_level` | 1 grafts, 2 trials, 3 multiverse. |
| `adoption_next` | What is missing for the next level, or null at 3. |
| `checks[]` | `id`, `status`, `detail`, `hint`. |
| `suggested_copies` | For `trial run -j`. |

Schema: `https://jand-s.github.io/scion/41664431a8f5/schemas/v1/cli/doctor.schema.json`.

## Exit codes

| Code | When |
|---|---|
| 0 | No check failed (warnings allowed). |
| 2 | Bad option. |
| 3 | A check failed. |

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| `config_not_found` | 3 | No `scion.json`. | `dotnet scion init`. |
| warning `version_mismatch` | 0 | The CLI and the package differ. | Install the same version of both. |
| warning `build_stale` | 0 | The test build is older than the sources. | `dotnet scion build test`. |
| warning `low_memory` | 0 | Not enough free memory for 2 copies. | Close apps or run `-j 1`. |
| `model_missing` | 3 | Level 3: the judge's eye model is not in the model cache. | `dotnet scion judge model pull` ([Models and licenses](/scion/41664431a8f5/judge/models/)). |

## Examples

### The deepest check

```sh
dotnet scion doctor --level 3
```

### Only the failing checks, in a script

```sh
dotnet scion doctor --format json | jq '.checks[] | select(.status != "ok")'
```

## See also

- [Install](/scion/41664431a8f5/start/install/)
- [Parallel copies](/scion/41664431a8f5/trials/parallel/)
- [Find non-determinism](/scion/41664431a8f5/determinism/diagnose/)
