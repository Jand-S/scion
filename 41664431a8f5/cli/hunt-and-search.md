---
title: "hunt, search"
description: "Hunt rare game states with coverage-guided fuzzing on the simulator, and search tunables with guided optimization and a gallery of diverse options."
type: cli
url: https://jand-s.github.io/scion/41664431a8f5/cli/hunt-and-search/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/cli/hunt-and-search.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/explore/rare-moments.md
  - https://jand-s.github.io/scion/41664431a8f5/explore/guided-search.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/files/manifest.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# hunt, search

Two exploration commands. `dotnet scion hunt run` plays matches on the core simulator with mutated commands, keeps
those that reach new features and saves each find as a match in `scion/recipes/found/`. `dotnet scion search run`
explores tunables with paired Bayesian optimization, keeps a MAP-Elites gallery, filters with the spec and the judge
and asks a person the deciding duels.

## Synopsis

```text
dotnet scion hunt run <Id|Spec> [--budget <minutes>] [--seed <n>] [--save]
dotnet scion search run <Spec> [--tunables a,b] [--budget <n>] [--duels <n>] [--niches a,b]
```

## Description

Both are long-running: they report progress, stop cleanly on Ctrl-C and keep what they found so far.

## hunt run

Runs for `--budget` minutes (default 5) on the `ISimulation`, from the recipe's setup or the spec's balance setup.
Each new feature reported by `ISimulation.Features` produces one find, minimized to the fewest turns. Finds go to
`scion/recipes/found/<feature>-<n>.match.json` with `source: hunt`.

## search run

Evaluates `--budget` variants (default 40) of the selected tunables as recipe variants in the test build (or on the
simulator for balance-only specs), drops those failing the spec, ranks with the judge, keeps the best per niche in
`search.json`, and asks up to `--duels` blind duels (default 10).

## Options

| Option | Commands | Meaning |
|---|---|---|
| `--budget` | both | Minutes (hunt) or variants (search). |
| `--seed <n>` | hunt | Repeatable hunts. |
| `--save` | hunt | Save finds (default on; `--save false` to only report). |
| `--tunables a,b` | search | What to tune. |
| `--duels <n>` | search | Maximum duels for a person (0 = none). |
| `--niches a,b` | search | Metrics that define the gallery's niches. |

## Output

<!-- validate: cli/hunt-run -->
```json
{"schema_version":1,"command":"hunt run","ok":true,"target":"CoopTurn","budget_s":300,"states":91540,"features":33,"finds":[{"name":"multi_kill-0003","file":"scion/recipes/found/multi_kill-0003.match.json","feature":"multi_kill"}],"next_step":"dotnet scion trial run \"CapturedMatch[match=multi_kill-0003]\""}
```

<!-- validate: cli/search-run -->
```json
{"schema_version":1,"command":"search run","ok":true,"spec":"StrongerCrits","tunables":["CritShake.Strength","CritShake.Stop"],"evaluated":40,"duels":8,"gallery":".scion/search/20261012-120000/search.json","best":[{"name":"g03","values":{"CritShake.Strength":0.55,"CritShake.Stop":0.07},"score":0.79}],"next_step":"save g03 as a preset in scion/presets/"}
```

## Exit codes

| Code | When |
|---|---|
| 0 | Done (no finds is still 0). |
| 2 | Unknown target, spec or tunable. |
| 3 | No simulation; judge model missing. |

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| `no_simulation` | 3 | No `ISimulation`. | Register one. |
| `model_missing` | 3 | The spec uses a judge head and the eyes are missing. | `doctor --level 3`. |
| `bad_value` | 2 | A tunable without a range. | `[Tunable(min, max)]`. |
| `no_match` | 2 | Unknown spec or recipe. | `did_you_mean`. |

## Examples

### A ten-minute hunt

```sh
dotnet scion hunt run CoopTurn --budget 10
```

### A search without people

```sh
dotnet scion search run StrongerCrits --tunables CritShake.Strength,CritShake.Stop --duels 0
```

## See also

- [Rare-moment hunter](/scion/41664431a8f5/explore/rare-moments/)
- [Guided search](/scion/41664431a8f5/explore/guided-search/)
- [Turn any match into a recipe](/scion/41664431a8f5/trials/captured-matches/)
