---
title: "judge"
description: "Download the judge's eye model and manage judge heads - list, show, create, vote, train and run them on a run's captures."
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

The `judge` commands manage the Scion Judge: `judge model pull` downloads the eye model into the model cache,
`judge list` and `judge show` describe heads, `judge new` creates one
from a question file, `judge vote` opens a local page for blind duels and labels, `judge train` trains and promotes a
new head version, and `judge run` scores a run's captures. Heads, questions and votes live in `judge.dir`
(`scion/judge/`); embeddings in `judge.cache`.

## Synopsis

```text
dotnet scion judge model pull [<eyes>]
dotnet scion judge list
dotnet scion judge show <head>
dotnet scion judge new <head> --question <file> [--view full|hud|corners|actor] [--sequence]
dotnet scion judge vote [--run <run>] [--head <head>] [--synthetic N] [--gold <set> --count N] [--regold] [--top N]
dotnet scion judge train <head>
dotnet scion judge run <run> [--heads a,b] [--references N] [--pairs <run>]
```

## Description

The judge never changes a verdict or an exit code of trials. It runs locally with the eye model in `judge.eyes`.

## judge model pull

Downloads the eye model (`<eyes>`, default `judge.eyes`) into the model cache, `~/.cache/scion/models/<eyes>/`, and
checks its hash. This is the only judge command that uses the network; every other one reads the cache and fails with
`model_missing` when the model is not there. It is idempotent: a model already in the cache is not downloaded again
(`downloaded: false`). On a machine without network, copy the `<eyes>/` folder from another machine's cache.

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
| `<eyes>` | model pull | `siglip2-so400m`, `siglip2-base`, `dinov2-base` or `uiclip` ([Models and licenses](/scion/41664431a8f5/judge/models/)); default `judge.eyes`. |
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

`judge model pull` returns `model` (`eyes`, `path`, `bytes`, `downloaded`); `judge list` returns `heads[]`;
`judge show` and `judge train` return `head` (and `training`); `judge vote` returns `url` and `votes`.

<!-- validate: cli/judge -->
```json
{"schema_version":1,"command":"judge model pull","ok":true,"model":{"eyes":"siglip2-so400m","path":"/Users/alex/.cache/scion/models/siglip2-so400m","bytes":1612000000,"downloaded":true},"next_step":"dotnet scion judge list"}
```

## Exit codes

| Code | When |
|---|---|
| 0 | Done. |
| 2 | Unknown head or eyes, bad question file. |
| 3 | The eye model is missing, or `judge model pull` could not download it. |

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| `model_missing` | 3 | The eye model is not in the model cache. | `dotnet scion judge model pull` (the `hint` names the eyes). |
| `no_match` | 2 | Unknown head. | `judge list`. |
| `bad_value` | 2 | The question file does not validate. | Check it against `judge-question.schema.json`. |
| `not_supported` | 2 | `judge vote` without a browser (an agent). | Votes are for people; agents use `trial eval`. |

## Examples

### Get the eye model once per machine

```sh
dotnet scion judge model pull
```

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
