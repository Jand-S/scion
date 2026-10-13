---
title: "trial show"
description: "Show what a run found for one id, the failed or the changed ones - verdicts, hashes, reasons and the paths of reports, images and grids."
type: cli
url: https://jand-s.github.io/scion/41664431a8f5/cli/trial-show/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/cli/trial-show.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/agents/reading-results.md
  - https://jand-s.github.io/scion/41664431a8f5/trials/baselines.md
  - https://jand-s.github.io/scion/41664431a8f5/trials/image-comparison.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# trial show

`dotnet scion trial show` reads a run folder and prints the essentials for some ids: rounds ok and failed, hashes,
frame counts, the verdict against the baseline with its reasons, and the **paths** of `report.md` and the images.
It never prints image content: open the paths. With `--grid` it writes baseline, current and difference side by side.

## Synopsis

```text
dotnet scion trial show <Id...> | --failed | --changed [--run <run>] [--image window|context|still|sheet|motion-diff] [--grid]
```

## Description

Defaults to the last run. For each id it merges every round and copy. `--image` limits `images[]` to one kind. `--grid`
writes `diff/<IdDir>/r<n>/<artifact>.grid.png` (baseline, current, heatmap in three columns) and returns its path.

## Options

| Option | Default | Meaning |
|---|---|---|
| `<Id...>` | | Ids or patterns. |
| `--failed` | | Ids with a failed round. |
| `--changed` | | Ids with verdict `changed`, `stale`, `within` or `px_changed`. |
| `--run <run>` | last run | A run folder name or path. |
| `--image <kind>` | all | Only that image kind. |
| `--grid` | off | Write and return comparison grids. |

## Output

<!-- validate: cli/trial-show -->
```json
{"schema_version":1,"command":"trial show","ok":true,"run":"/Users/alex/orchard/.scion/runs/20261011-143200","count":1,"recipes":[{"id":"KnightFalls","rounds":{"ok":1,"failed":1},"hashes":["1B44E07AC9D20F13"],"frames":[52],"vs_approved":"same","round":1,"copy":3,"report":"q3/KnightFalls/r1/report.md","images":["q3/KnightFalls/r1/window.png","q3/KnightFalls/r1/context.png"],"failed":[{"round":1,"copy":3,"error":"timeout (30s): unit down animation"}]}],"next_step":"read q3/KnightFalls/r1/report.md"}
```

| Field | Meaning |
|---|---|
| `recipes[].rounds` | `ok` and `failed` counts. |
| `hashes`, `frames` | Distinct values seen across rounds. |
| `vs_approved`, `reasons`, `stale` | Verdict and why. |
| `px_diff` | Fraction of frames whose pixel hash differs from the baseline. |
| `report`, `images` | Paths relative to the run. |
| `grid` | With `--grid`. |

## Exit codes

| Code | When |
|---|---|
| 0 | Shown. |
| 2 | No such id or run. |

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| `no_match` | 2 | The id is not in the run. | `did_you_mean`. |
| warning `result_unreadable` | 0 | A `result.json` is missing or broken. | The copy crashed; read its `player.log`. |
| warning `summary_not_written` | 0 | The run was interrupted before `summary.json`. | Run again. |

## Examples

### Everything that failed

```sh
dotnet scion trial show --failed
```

### A grid for review

```sh
dotnet scion trial show SlashHit --grid --image window
```

## See also

- [Reading results](/scion/agents/reading-results/)
- [Baselines and approval](/scion/trials/baselines/)
- [Image comparison](/scion/trials/image-comparison/)
