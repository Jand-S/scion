---
title: "multiverse"
description: "Fork a captured match at a turn, play variants on the core simulator, report where rules diverge, and show the result or a side-by-side mosaic."
type: cli
url: https://jand-s.github.io/scion/41664431a8f5/cli/multiverse/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/cli/multiverse.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/explore/multiverse.md
  - https://jand-s.github.io/scion/41664431a8f5/trials/captured-matches.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/files/manifest.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# multiverse

`dotnet scion multiverse run` replays a captured match on the core simulator up to a turn, forks it, plays each
variant to the end and reports, per variant, whether the rules diverged, the first turn and the first field.
`dotnet scion multiverse show` prints a past result. With `--mosaic`, the test build replays the variants side by
side.

## Synopsis

```text
dotnet scion multiverse run <match|Id> --at-turn <N> --variants <list|file> [--mosaic]
dotnet scion multiverse show <run>
```

## Description

Everything except `--mosaic` runs in plain .NET on the registered `ISimulation`, in parallel, without Unity.

## multiverse run

`<match>` is a captured match name (`knight-dies-to-crit`), its file, or a `CapturedMatch[...]` id. Variants: preset
names or blends, `tunable:<Key>=<value>`, `grafts:<dll>`, `bot:<name>`, comma-separated, or a JSON file with a list of
`{name, kind, value}`. Results go to `.scion/multiverse/<yyyyMMdd-HHmmss>/multiverse.json`.

## multiverse show

Prints the envelope of a past run from its `multiverse.json`.

## Options

| Option | Commands | Meaning |
|---|---|---|
| `--at-turn <N>` | run | Fork after turn N (0 = the start). |
| `--variants <list\|file>` | run | The variants. |
| `--mosaic` | run | Also replay in the test build and write `mosaic.png`. |

## Output

<!-- validate: cli/multiverse -->
```json
{"schema_version":1,"command":"multiverse show","ok":true,"run":"/Users/alex/orchard/.scion/multiverse/20261012-101500","match":"knight-dies-to-crit","at_turn":2,"variants":[{"name":"base","diverged":false,"first_turn":null,"field":null},{"name":"crit-1200","diverged":true,"first_turn":3,"field":"units[0].hp"}],"report":"multiverse.json","mosaic":"mosaic.png","next_step":"open mosaic.png"}
```

## Exit codes

| Code | When |
|---|---|
| 0 | Done. Divergence of tunable or preset variants is information, not failure. |
| 1 | A `grafts:` variant diverged (grafts must be visual). |
| 2 | Unknown match, variant or turn. |
| 3 | No simulation, or the state cannot be saved. |

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| `no_simulation` | 3 | No `[assembly: Simulation]`. | Register an `ISimulation`. |
| `not_forkable` | 3 | No `ISaveable`. | Implement `Save` and `Load`. |
| `mosaic_unsupported` | 3 | `--mosaic` without `IMosaicAdapter`. | Implement it or drop `--mosaic`. |
| `no_match` | 2 | Unknown match or preset. | `trial list`; `scion/presets/`. |

## Examples

### Prove a draft is visual

```sh
dotnet scion multiverse run knight-dies-to-crit --at-turn 0 --variants base,grafts:.scion/grafts/ScionGrafts_51c2.dll
```

### See variants side by side

```sh
dotnet scion multiverse run knight-dies-to-crit --at-turn 2 --variants base,heavy --mosaic
```

## See also

- [The multiverse](/scion/explore/multiverse/)
- [Turn any match into a recipe](/scion/trials/captured-matches/)
- [Mosaic](/scion/trials/mosaic/)
