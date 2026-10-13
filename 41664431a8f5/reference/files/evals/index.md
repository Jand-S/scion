---
title: "eval-in.json, eval-out.json, evals.tsv"
description: "The files evaluators exchange with the CLI - blind questions and items in, answers out, and the scores table - in the judge's question and answer shapes."
type: file
url: https://jand-s.github.io/scion/41664431a8f5/reference/files/evals/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/reference/files/evals.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/evaluation/evaluators.md
  - https://jand-s.github.io/scion/41664431a8f5/cli/trial-eval.md
  - https://jand-s.github.io/scion/41664431a8f5/judge/questions.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# eval-in.json, eval-out.json, evals.tsv

Evaluators talk to Scion through files. `trial eval` writes `eval-in.json` (questions and blind items with image
paths), the evaluator writes `eval-out.json` (answers per item and question), and the CLI writes `evals.tsv` with one
score per line after mapping labels back to ids. The shapes are the judge's question and answer formats.

## Location

`<run>/eval-in.json`, `<run>/eval-out.json` (or any path given to `--import`), `<run>/evals.tsv`.

## Written by and read by

| File | Written by | Read by |
|---|---|---|
| `eval-in.json` | `trial eval` | Evaluators (scripts, agents) |
| `eval-out.json` | Evaluators | `trial eval --import` |
| `evals.tsv` | `trial eval` | People, CI, `summary.tsv` (`eval`) |

## Schema

- `eval-in.json`: `https://jand-s.github.io/scion/41664431a8f5/schemas/v1/judge-question.schema.json`
- `eval-out.json`: `https://jand-s.github.io/scion/41664431a8f5/schemas/v1/judge-answer.schema.json`
- `evals.tsv`: TSV rules; columns `id evaluator question score label`.

## Fields

`eval-in.json` adds to a question file: `run`, `blind`, `orders` (1 or 2), `items[]` (`label`, `images`, `frames`,
`telemetry`, `metrics`) and `pairs[]` for duels. `eval-out.json` has `model`, `items.<label>.answers.<question>`,
`escalate`, `preferences[]` and `usage`. Field meanings: [Questions and answers](/scion/41664431a8f5/judge/questions/).

## Example

<!-- validate: judge-question -->
```json
{"schema_version":1,"run":"/Users/alex/orchard/.scion/runs/20261011-150512","blind":true,"orders":2,"questions":{"hit_weight":{"type":"score","instructions":"How heavy does the hit feel?","levels":[{"what":"Light"},{"what":"Medium"},{"what":"Heavy"}]}},"items":[{"label":"X","images":["q0/SlashHit_preset=heavy_/r1/window.png"],"frames":[14],"telemetry":null,"metrics":"q0/SlashHit_preset=heavy_/r1/metrics.json"},{"label":"Y","images":["q1/SlashHit_preset=snappy_/r1/window.png"],"frames":[14],"telemetry":null,"metrics":null}],"pairs":[["X","Y"]]}
```

<!-- validate: judge-answer -->
```json
{"schema_version":1,"model":"agent","head_version":null,"items":{"X":{"answers":{"hit_weight":{"type":"score","score":1.8,"confidence":0.7}}},"Y":{"answers":{"hit_weight":{"type":"score","score":1.1,"confidence":0.6}}}},"preferences":[{"pair":["X","Y"],"winner":"X","strength":2}]}
```

## Compatibility

Fields may be added in minor versions. Unknown fields in `eval-out.json` are ignored.
