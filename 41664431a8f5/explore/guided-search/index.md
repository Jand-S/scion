---
title: "Guided search"
description: "Search tunables with paired Bayesian optimization and a MAP-Elites gallery of diverse good options, pre-filtered by the judge and decided by a person."
type: guide
url: https://jand-s.github.io/scion/41664431a8f5/explore/guided-search/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/explore/guided-search.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/cli/hunt-and-search.md
  - https://jand-s.github.io/scion/41664431a8f5/grafts/tunables.md
  - https://jand-s.github.io/scion/41664431a8f5/judge/presets.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Guided search

Tuning by hand stops at a few knobs. Guided search explores many: it proposes tunable values with paired Bayesian
optimization, keeps a **MAP-Elites gallery** of the best option in each niche (fast and light, slow and heavy...),
lets the judge drop the clearly worse ones, and asks a person only the duels that decide. The result is a small set
of good, different options, ready to save as presets.

## When to use

- Feel with 4 to 15 tunables (a hit effect, a camera, a UI transition).
- Balance values when a spec defines success.

## Before you begin

- The tunables, with ranges ([Tunables and presets](/scion/41664431a8f5/grafts/tunables/)).
- A spec that says what is acceptable (`Visual(...).Expect(...)`), and a judge head for the taste part.

## Steps

### 1. Search

```sh
dotnet scion search run StrongerCrits --tunables CritShake.Strength,CritShake.Stop,HitFlash.Seconds --budget 60 --duels 20
```

<!-- validate: cli/search-run -->
```json
{"schema_version":1,"command":"search run","ok":true,"spec":"StrongerCrits","tunables":["CritShake.Strength","CritShake.Stop","HitFlash.Seconds"],"evaluated":60,"duels":20,"gallery":".scion/search/20261012-120000/search.json","best":[{"name":"g07","values":{"CritShake.Strength":0.62,"CritShake.Stop":0.08,"HitFlash.Seconds":0.09},"score":0.81},{"name":"g12","values":{"CritShake.Strength":0.35,"CritShake.Stop":0.04,"HitFlash.Seconds":0.05},"score":0.77}],"next_step":"save a winner: copy its values to scion/presets/<name>.json"}
```

### 2. Vote the duels

`search run` opens the duel page when it needs a person (`--duels` caps how many). Without a person, it stops after
the judge's ranking and lists the pending duels.

### 3. Keep the winners

Copy the values of the options you like into presets (`scion/presets/punchy.json`), and use them in recipes and specs.

## Reference

| Option | Default | Meaning |
|---|---|---|
| `<Spec>` | | The spec that defines acceptable results. |
| `--tunables a,b` | every tunable the spec's recipes use | What to search. |
| `--budget <n>` | 40 | Variants to evaluate. |
| `--duels <n>` | 10 | Maximum duels to ask a person. |
| `--niches a,b` | `hit_stop_ms,shake_peak_px` | Metrics that define the gallery's niches. |

### Loop

1. Propose a batch of variants (Bayesian optimization over pairs: it models which of two variants wins).
2. Run them as recipe variants in the test build (`--motion`).
3. Drop variants that fail the spec's gates or bounds.
4. Rank the rest with the judge; place each in its niche of the gallery if it beats the current one.
5. Ask a person the duels where the judge is least sure; their votes update the model and train the head.

`search.json` holds the gallery: every niche with its best variant, values, score and image.

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| `bad_value` | 2 | A tunable has no range. | Add `[Tunable(min, max)]` (SCN005). |
| `model_missing` | 3 | The spec's judge head is not trained. | Train it, or remove `Judge` from the spec. |

## Examples

### A balance search on the simulator

```sh
dotnet scion search run CoopFloor3 --tunables Balance.CritPermille,Balance.EnemyHpPermille --budget 200 --duels 0
```

With `--duels 0` and a balance-only spec, the search runs entirely in plain .NET.

### Gallery niches by color and speed

```sh
dotnet scion search run StrongerCrits --niches flash_delta_l,hit_stop_ms
```

## See also

- [hunt, search](/scion/41664431a8f5/cli/hunt-and-search/)
- [Tunables and presets](/scion/41664431a8f5/grafts/tunables/)
- [Taste heads and feel presets](/scion/41664431a8f5/judge/presets/)
