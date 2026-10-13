---
title: "The run folder"
description: "The folder tree a trial run writes - per-copy outputs, per-recipe and per-round files, CLI summaries and diffs - with one line per file."
type: file
url: https://jand-s.github.io/scion/41664431a8f5/reference/files/run-folder/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/reference/files/run-folder.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/trials/capture.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/files/summary.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/files/result-json.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# The run folder

Every `dotnet scion trial run` creates one run folder under `trials.out`, named by its start time. Inside, each copy
of the test build has its own folder `q<i>/` (the player's `-scion-out`), with one folder per id and round; the CLI
adds its summaries and diffs at the top. The CLI never writes inside a copy's folder.

## Location

```text
<trials.out>/<yyyyMMdd-HHmmss>/        default .scion/runs/20261011-143200/
```

A player started by hand writes the `q<i>/` part directly into its `-scion-out`.

## Written by and read by

| Part | Written by | Read by |
|---|---|---|
| `q<i>/` and below | The test player | The CLI, agents, people |
| `summary.*`, `compare.json`, `diff/`, `evals.tsv`, `judge.json` | The CLI | Agents, people, CI |

## Schema

Each JSON file has its own schema ([JSON Schemas](/scion/41664431a8f5/reference/schemas/)); TSV files follow the rules in
[results.tsv, frames.tsv, determinism.tsv](/scion/41664431a8f5/reference/files/tables/).

## Fields

```text
20261011-143200/
  summary.json  summary.tsv  compare.json      CLI: verdicts and the full envelope
  evals.tsv  eval-in.json  eval-out.json       trial eval
  judge.json                                   judge run
  diff/<IdDir>/r<n>/<artifact>.diff.png        heatmaps against the baseline
  diff/<IdDir>/r<n>/<artifact>.grid.png        trial show --grid
  q0/                                          one copy (the player's -scion-out)
    run.json  results.tsv  report.md  player.log
    recipes.json                               only with -scion-list
    control.json                               only with -scion-control
    determinism.tsv                            only with -scion-check
    SlashHit/r1/
      result.json  report.md  frames.tsv  telemetry.jsonl  log.txt
      window.png  context.png  before.png      stills by name
      sheet.png                                static trials
      motion-diff.png  tracks.png  metrics.json  preview.gif  idmask/   with --motion
      frames/0000.png ...                      with --frames
      frames/0000.raw ...  frames/raw.json     with --raw
```

| File | Page |
|---|---|
| `run.json` | [run.json](/scion/41664431a8f5/reference/files/run-json/) |
| `result.json` | [result.json](/scion/41664431a8f5/reference/files/result-json/) |
| `results.tsv`, `frames.tsv`, `determinism.tsv` | [results.tsv, frames.tsv, determinism.tsv](/scion/41664431a8f5/reference/files/tables/) |
| `telemetry.jsonl` | [telemetry.jsonl](/scion/41664431a8f5/reference/files/telemetry-jsonl/) |
| `report.md` | [report.md](/scion/41664431a8f5/reference/files/report-md/) |
| `recipes.json` | [recipes.json](/scion/41664431a8f5/reference/files/recipes-json/) |
| `frames/*.raw`, `raw.json` | [Raw frames](/scion/41664431a8f5/reference/files/raw-frames/) |
| `summary.*`, `compare.json` | [summary.json, summary.tsv, compare.json](/scion/41664431a8f5/reference/files/summary/) |
| `control.json` | [control.json and live files](/scion/41664431a8f5/reference/files/control-json/) |
| `eval-*.json`, `evals.tsv` | [eval-in.json, eval-out.json, evals.tsv](/scion/41664431a8f5/reference/files/evals/) |
| images and `metrics.json` | [Capture and outputs](/scion/41664431a8f5/trials/capture/), [The motion pack](/scion/41664431a8f5/evaluation/motion-pack/) |

`<IdDir>` is the id with every character outside `[A-Za-z0-9_.,=@#[]-]` replaced by `_`.

## Example

```sh
ls .scion/runs/20261011-143200/q0/SlashHit/r1
```

```text
before.png  context.png  frames.tsv  log.txt  report.md  result.json  telemetry.jsonl  window.png
```

## Compatibility

New files may be added in minor versions; existing names and places never change within 1.x. Tools should ignore
files they do not know.
