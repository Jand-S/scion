---
title: "judge"
description: "Manage judge heads - list, show, create, vote, train and run them on a run's captures - with data in scion/judge."
type: cli
url: https://jand-s.github.io/scion/41664431a8f5/cli/judge/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/cli/judge.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/judge/overview.md
  - https://jand-s.github.io/scion/41664431a8f5/judge/training.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/files/judge-data.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# judge

The `judge` commands manage the Scion Judge: `judge list` and `judge show` describe heads, `judge new` creates one
from a question file, `judge vote` opens a local page for blind duels and labels, `judge train` trains and promotes a
new head version, and `judge run` scores a run's captures. Heads, questions and votes live in `judge.dir`
(`scion/judge/`); embeddings in `judge.cache`.

## Synopsis

```text
dotnet scion judge list
dotnet scion judge show <head>
dotnet scion judge new <head> --question <file> [--view full|hud|corners|actor] [--sequence]
dotnet scion judge vote [--run <run>] [--head <head>] [--synthetic N] [--gold <set> --count N] [--regold] [--top N]
dotnet scion judge train <head>
dotnet scion judge run <run> [--heads a,b] [--references N] [--pairs <run>]
```

## Description

The judge never changes a verdict or an exit code of trials. It runs locally with the eye model in `judge.eyes`.

## judge list

Every head with its version, type, vote count and gold agreement.

## judge show

One head: question, view, eyes, training history, calibration, and where it disagrees most with people.

## judge new

Creates `scion/judge/heads/<head>.json` from a question file. `--sequence` makes a motion head that also reads
`metrics.json`.

## judge vote

Starts a local page at `http://localhost:<port>/` and opens it. Pairs come from `--run` (or the latest runs), blind
and in both orders; the judge picks the most informative pairs first. `--synthetic N` adds N agent-made pairs
(degradations, fabricated bugs). `--gold <set> --count N` collects gold votes (people only). `--regold` re-asks a
sample of gold votes to measure the ceiling. `--top N` shows only the N best variants per id.

## judge train

Trains a new version with cross-validation and promotes it only if it agrees more with the gold set.

## judge run

Scores every capture of a run with the given heads (default: all), writes `<run>/judge.json` and `escalate`.
`--references N` adds the N most similar reference screens per capture; `--pairs <run>` compares each id with the
same id in another run.

## Options

| Option | Commands | Meaning |
|---|---|---|
| `--question <file>` | new | The question. |
| `--view <view>` | new | `full`, `hud`, `corners`, `actor`. |
| `--sequence` | new | A motion head. |
| `--run`, `--head` | vote | Where pairs come from; which head. |
| `--synthetic N`, `--gold <set>`, `--count N`, `--regold`, `--top N` | vote | See above. |
| `--heads a,b`, `--references N`, `--pairs <run>` | run | See above. |

## Output

<!-- validate: cli/judge -->
```json
{"schema_version":1,"command":"judge run","ok":true,"results":".scion/runs/20261011-150512/judge.json","escalate":["HudStates/turn-banner.720p.png"],"next_step":"look at the escalated captures, or ask an evaluator: dotnet scion trial eval 20261011-150512 --with agent"}
```

`judge list` returns `heads[]`; `judge show` and `judge train` return `head` (and `training`); `judge vote` returns
`url` and `votes`.

## Exit codes

| Code | When |
|---|---|
| 0 | Done. |
| 2 | Unknown head, bad question file. |
| 3 | The eye model is missing. |

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| `model_missing` | 3 | The eye model is not downloaded. | Run the command in `hint` (also in `doctor --level 3`). |
| `no_match` | 2 | Unknown head. | `judge list`. |
| `bad_value` | 2 | The question file does not validate. | Check it against `judge-question.schema.json`. |
| `not_supported` | 2 | `judge vote` without a browser (an agent). | Votes are for people; agents use `trial eval`. |

## Examples

### Score a run with two heads

```sh
dotnet scion judge run 20261011-150512 --heads readability,has_glitch
```

### Ten minutes of votes

```sh
dotnet scion judge vote --head hit_weight
```

## See also

- [The Judge](/scion/41664431a8f5/judge/overview/)
- [Train a head](/scion/41664431a8f5/judge/training/)
- [Judge data files](/scion/41664431a8f5/reference/files/judge-data/)
