---
title: "trial eval"
description: "Score a run with external evaluators - export blind eval-in.json, run or import eval-out.json, and write evals.tsv without changing any verdict."
type: cli
url: https://jand-s.github.io/scion/41664431a8f5/cli/trial-eval/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/cli/trial-eval.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/evaluation/evaluators.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/files/evals.md
  - https://jand-s.github.io/scion/41664431a8f5/judge/questions.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# trial eval

`dotnet scion trial eval` sends the images of a run to the evaluators configured in `scion.json` and collects their
scores. It writes `eval-in.json` with blind items, runs `process` evaluators, or waits for an `eval-out.json` you
import (agents), and writes `evals.tsv`. Scores never change an exit code or a verdict.

## Synopsis

```text
dotnet scion trial eval <run> --with <a,b> [--questions <file>] [--dry-run]
dotnet scion trial eval <run> --import <eval-out.json>
```

## Description

1. Collects the run's ids and their `window.png`, `context.png`, stills and `metrics.json`.
2. Shuffles them under blind labels and writes `<run>/eval-in.json` with the questions (default: the questions of the
   judge heads in `judge.dir`).
3. For each `process` evaluator: runs its `command` with the two file paths and reads `eval-out.json`.
4. For `agent` evaluators: stops, with `next_step` telling the agent to answer the file and import it.
5. Maps labels back to ids and writes `evals.tsv` and the `eval` column of `summary.tsv`.

## Options

| Option | Default | Meaning |
|---|---|---|
| `<run>` | required | The run folder. |
| `--with a,b` | | Evaluators by name. |
| `--questions <file>` | judge questions | A question file. |
| `--import <file>` | | Import an answer file. |
| `--dry-run` | off | Write `eval-in.json` only. |

## Output

<!-- validate: cli/trial-eval -->
```json
{"schema_version":1,"command":"trial eval","ok":true,"run":"/Users/alex/orchard/.scion/runs/20261011-150512","evaluators":["agent"],"items":3,"exported":"eval-in.json","imported":null,"evals":"evals.tsv","next_step":"answer eval-in.json into eval-out.json, then: dotnet scion trial eval 20261011-150512 --import eval-out.json"}
```

## Exit codes

| Code | When |
|---|---|
| 0 | Exported, scored or imported. |
| 1 | A process evaluator failed. |
| 2 | Unknown evaluator, bad answer file. |

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| `no_match` | 2 | Evaluator not in `scion.json`. | `did_you_mean`. |
| `bad_value` | 2 | `eval-out.json` does not validate. | Check it against `judge-answer.schema.json`. |
| `not_supported` | 2 | An evaluator with `gate: true`. | Scores never gate; write an `ITrialCheck`. |

## Examples

### A script evaluator

```sh
dotnet scion trial eval 20261011-150512 --with contrast-script
```

### Agent round trip

```sh
dotnet scion trial eval 20261011-150512 --with agent
dotnet scion trial eval 20261011-150512 --import eval-out.json
```

## See also

- [Evaluators](/scion/41664431a8f5/evaluation/evaluators/)
- [eval-in.json, eval-out.json, evals.tsv](/scion/41664431a8f5/reference/files/evals/)
- [Questions and answers](/scion/41664431a8f5/judge/questions/)
