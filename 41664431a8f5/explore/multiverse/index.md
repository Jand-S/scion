---
title: "The multiverse"
description: "Fork a deterministic match at a turn, run variants of code or numbers from that point, and see the first turn and field where the rules diverge."
type: guide
url: https://jand-s.github.io/scion/41664431a8f5/explore/multiverse/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/explore/multiverse.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/cli/multiverse.md
  - https://jand-s.github.io/scion/41664431a8f5/trials/captured-matches.md
  - https://jand-s.github.io/scion/41664431a8f5/trials/mosaic.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/api/core.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# The multiverse

The multiverse replays a captured match up to turn N, forks it, and plays K variants from there: other grafts,
other tunables, other presets, other bots. Because the rules are deterministic, any difference is caused by the
variant. Scion reports, per variant, whether the rules diverged, the first turn and the first field that changed,
and shows the visuals side by side in a mosaic. It is how you prove a graft is only visual, and how you see what a
balance change does to a real match.

## When to use

- Before promoting a graft: run the match with and without it; nothing may diverge.
- When tuning rules: see the first turn where a new damage value changes the outcome.
- When a bug appears late in a match: fork just before it and try fixes.

## Before you begin

- Adoption level 3: the rules registered as an `ISimulation`, deterministic in plain .NET
  ([Find non-determinism](/scion/determinism/diagnose/)).
- A captured match ([Turn any match into a recipe](/scion/trials/captured-matches/)).

## Steps

### 1. Fork and run

```sh
dotnet scion multiverse run knight-dies-to-crit --at-turn 2 --variants base,heavy,crit-1200
```

`crit-1200` is a preset that sets the rule tunable `Balance.CritPermille` to 1200.

<!-- validate: cli/multiverse -->
```json
{"schema_version":1,"command":"multiverse run","ok":true,"run":"/Users/alex/orchard/.scion/multiverse/20261012-101500","match":"knight-dies-to-crit","at_turn":2,"variants":[{"name":"base","diverged":false,"first_turn":null,"field":null},{"name":"heavy","diverged":false,"first_turn":null,"field":null},{"name":"crit-1200","diverged":true,"first_turn":3,"field":"units[0].hp"}],"report":"multiverse.json","mosaic":null,"next_step":"heavy is visual only; crit-1200 changes units[0].hp at turn 3: dotnet scion multiverse show 20261012-101500"}
```

### 2. Read the report

```sh
dotnet scion multiverse show 20261012-101500
```

`multiverse.json` has the hash of every turn for the base and each variant, and for diverged variants the field's
value before and after.

### 3. See it

```sh
dotnet scion multiverse run knight-dies-to-crit --at-turn 2 --variants base,heavy --mosaic
```

With `--mosaic`, the test build replays the variants side by side from the fork turn and writes `mosaic.png`.

## Reference

### Variants

| Kind | Given as | Changes |
|---|---|---|
| `preset` | a preset name or blend | tunables |
| `tunables` | `tunable:Balance.CritPermille=1200` | one or more tunables |
| `grafts` | `grafts:<dll>` | presentation code (the rules must not diverge) |
| `bot` | `bot:<name>` | who plays the remaining turns |

`--variants` takes a comma-separated list, or a JSON file with one object per variant.

### How it runs

1. The `ISimulation` replays the match's commands up to `--at-turn` and saves the state (`ISaveable`).
2. For each variant, it loads the state, applies the variant and plays the remaining commands (or the bot).
3. After every turn it records `Hash()`; the first turn whose hash differs from the base is `first_turn`.
4. At that turn, `IInspectable.Diff` names the first field that differs (`units[0].hp`).

Steps 1 to 4 run in plain .NET, in parallel, in milliseconds per turn. Only `--mosaic` starts the test build.

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| `no_simulation` | 3 | No `[assembly: Simulation]`. | Register an `ISimulation`. |
| `not_forkable` | 3 | The simulation does not implement `ISaveable`. | Implement `Save` and `Load` on the state. |
| `no_match` | 2 | The match file or a variant does not exist. | `trial list` shows captured matches. |
| variant `diverged` with `grafts` | 1 | A graft changed rules. | Move the change to the core ([Grafts are visual only](/scion/grafts/visual-only/)). |

## Examples

### Prove a draft graft is visual

```sh
dotnet scion multiverse run knight-dies-to-crit --at-turn 0 --variants base,grafts:.scion/grafts/ScionGrafts_51c2.dll
```

### Let bots finish the match

```sh
dotnet scion multiverse run knight-dies-to-crit --at-turn 2 --variants bot:aggressive,bot:cautious
```

## See also

- [multiverse (CLI)](/scion/cli/multiverse/)
- [Turn any match into a recipe](/scion/trials/captured-matches/)
- [Mosaic](/scion/trials/mosaic/)
- [Core API](/scion/reference/api/core/)
