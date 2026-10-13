---
title: "Evaluators"
description: "Score runs with external evaluators - scripts or agents - through eval-in.json and eval-out.json, blind, with scores in evals.tsv that never fail a run."
type: guide
url: https://jand-s.github.io/scion/41664431a8f5/evaluation/evaluators/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/evaluation/evaluators.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/cli/trial-eval.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/files/evals.md
  - https://jand-s.github.io/scion/41664431a8f5/judge/questions.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Evaluators

An evaluator scores the images of a run from outside Scion: a script, a model you run, or an agent. `dotnet scion
trial eval <run>` writes `eval-in.json` (questions and blind items), runs the evaluators configured in `scion.json`
or waits for an answer file, reads `eval-out.json` and writes `evals.tsv`. The CLI never calls a model itself, and
scores only order variants: an evaluator can never fail a run.

## When to use

- Use an evaluator for a question the trained judge does not cover yet, or to get a second opinion.
- Use an agent evaluator when you are an agent: export, answer the file yourself, import.

## Steps

### 1. Configure evaluators

```json
{"schema_version":1,"evaluators":[{"name":"contrast-script","command":["python3","tools/eval_contrast.py"],"kind":"process","gate":false},{"name":"agent","command":[],"kind":"agent","gate":false}]}
```

A `process` evaluator is called with the paths of `eval-in.json` and `eval-out.json` as its two arguments. An
`agent` evaluator is answered by hand or by an agent through `--import`.

### 2. Run them

```sh
dotnet scion trial eval 20261011-150512 --with contrast-script
```

### 3. Or answer as an agent

```sh
dotnet scion trial eval 20261011-150512 --with agent
# read .scion/runs/20261011-150512/eval-in.json, look at the images, write eval-out.json
dotnet scion trial eval 20261011-150512 --import eval-out.json
```

<!-- validate: cli/trial-eval -->
```json
{"schema_version":1,"command":"trial eval","ok":true,"run":"/Users/alex/orchard/.scion/runs/20261011-150512","evaluators":["agent"],"items":3,"exported":null,"imported":"eval-out.json","evals":"evals.tsv","scores":[{"id":"SlashHit[preset=heavy]","evaluator":"agent","question":"hit_weight","score":0.8}],"next_step":"dotnet scion trial show --grid"}
```

## Reference

### Files

| File | Written by | Content |
|---|---|---|
| `eval-in.json` | the CLI | Questions and items with blind labels (X, Y, Z), image paths, frames and metrics. |
| `eval-out.json` | the evaluator | Answers per item and question, with probabilities, confidence and evidence. |
| `evals.tsv` | the CLI | `id evaluator question score label`, one line per answer. |

The shapes are the judge's question and answer formats ([eval-in.json, eval-out.json, evals.tsv](/scion/reference/files/evals/)).

### Rules

- Items are blind: the evaluator sees labels, never variant names. The CLI maps labels back on import.
- `gate` must be `false`: scores never change exit codes. An evaluator that should block is a gate, written as an
  `ITrialCheck`.
- Scores also go to the `eval` column of `summary.tsv`.

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| `no_match` | 2 | `--with` names an evaluator not in `scion.json`. | Use `did_you_mean`. |
| `bad_value` | 2 | `eval-out.json` does not match the schema. | Validate it against `judge-answer.schema.json`. |
| none | 1 | A process evaluator exited non-zero; its scores are missing and `ok` is false. | Read its stderr in the run's `evals.log`. |

## Examples

### Two evaluators at once

```sh
dotnet scion trial eval 20261011-150512 --with contrast-script,agent
```

### Export only

```sh
dotnet scion trial eval 20261011-150512 --with agent --dry-run
```

## See also

- [trial eval](/scion/cli/trial-eval/)
- [eval-in.json, eval-out.json, evals.tsv](/scion/reference/files/evals/)
- [Questions and answers](/scion/judge/questions/)
