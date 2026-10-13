---
title: "Rare-moment hunter"
description: "Search game states with coverage-guided fuzzing on the core simulator to find rare situations, and turn each find into a recipe."
type: guide
url: https://jand-s.github.io/scion/41664431a8f5/explore/rare-moments/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/explore/rare-moments.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/cli/hunt-and-search.md
  - https://jand-s.github.io/scion/41664431a8f5/trials/captured-matches.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/api/core.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Rare-moment hunter

Some situations happen once in a thousand matches: four units dying on one turn, a heal landing on a unit already
at zero, a crit on the last enemy while a banner is showing. The hunter plays matches on the core simulator with
bots and mutated commands, keeps the ones that reach new **features** (combinations of state the simulation
reports), and saves each new feature as a match file. Every find becomes a recipe you can capture, check and
approve.

## When to use

- Before a release, to find edge cases no one wrote a recipe for.
- After a new mechanic, to see what it combines with.

## Before you begin

- Adoption level 3 (`ISimulation` with `Features`).

## Steps

### 1. Report features from the simulation

```csharp
public IEnumerable<string> Features(OrchardState s)
{
    if (s.DiedThisTurn >= 3) yield return "multi_kill";
    if (s.LastHeal?.Target.Hp == 0) yield return "heal_on_zero";
    if (s.Turn > 30) yield return "long_match";
    yield return "units_alive:" + s.Alive;
}
```

### 2. Hunt

```sh
dotnet scion hunt run CoopTurn --budget 10
```

<!-- validate: cli/hunt-run -->
```json
{"schema_version":1,"command":"hunt run","ok":true,"target":"CoopTurn","budget_s":600,"states":184220,"features":41,"finds":[{"name":"multi_kill-0007","file":"scion/recipes/found/multi_kill-0007.match.json","feature":"multi_kill"},{"name":"heal_on_zero-0002","file":"scion/recipes/found/heal_on_zero-0002.match.json","feature":"heal_on_zero"}],"next_step":"dotnet scion trial run \"CapturedMatch[match=heal_on_zero-0002]\""}
```

### 3. Look at the finds

```sh
dotnet scion trial run "CapturedMatch[match=heal_on_zero-0002]"
```

Move the ones worth keeping to `scion/recipes/captured/` and approve them.

## Reference

| Option | Default | Meaning |
|---|---|---|
| `<Id\|Spec>` | | The recipe (its setup and players) or a spec whose `Balance` setup to start from. |
| `--budget <minutes>` | 5 | Time to search. |
| `--seed <n>` | random | Start seed, for repeatable hunts. |
| `--save` | on | Write finds to `scion/recipes/found/`. |

### How it searches

1. Start from bot matches with random seeds.
2. Keep a corpus of matches that produced a feature never seen before.
3. Mutate corpus matches: change one command, swap two, change a seed, cut and continue with a bot.
4. Prefer mutations of matches whose features are rare.

One find is saved per new feature, minimized to the fewest turns that still produce it.

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| `no_simulation` | 3 | No `ISimulation`. | Register one. |
| `finds: []` | 0 | Nothing new in the budget. | Report more specific features, or raise `--budget`. |

## Examples

### A long hunt overnight

```sh
dotnet scion hunt run CoopTurn --budget 480
```

### Repeat a hunt exactly

```sh
dotnet scion hunt run CoopTurn --budget 10 --seed 42
```

## See also

- [hunt, search](/scion/41664431a8f5/cli/hunt-and-search/)
- [Turn any match into a recipe](/scion/41664431a8f5/trials/captured-matches/)
- [Core API](/scion/41664431a8f5/reference/api/core/)
