---
title: "Taste heads and feel presets"
description: "Swap judge heads per audience, keep feel presets and blend them, and know where art-style variations come from."
type: guide
url: https://jand-s.github.io/scion/41664431a8f5/judge/presets/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/judge/presets.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/grafts/tunables.md
  - https://jand-s.github.io/scion/41664431a8f5/judge/training.md
  - https://jand-s.github.io/scion/41664431a8f5/explore/guided-search.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Taste heads and feel presets

Taste is not one thing. A head trained on the studio's votes predicts the studio; a head trained on playtesters
predicts casual players; an accessibility head favors contrast and calm motion. Heads are swappable per audience.
Feel works the same way: named **presets** of tunable values (`heavy`, `snappy`) that blend with weights, so an agent
turns two or three knobs instead of fifteen numbers.

## When to use

- Use audience heads when the team and the players disagree, or for accessibility reviews.
- Use presets to name a feel once and reuse it across effects and recipes.

## Steps

### 1. Train a head per audience

```sh
dotnet scion judge new hit_weight.casual --question scion/judge/questions/hit_weight.json --view actor --sequence
dotnet scion judge vote --head hit_weight.casual --run 20261012-091500
```

Head names may have a suffix after a dot; `judge run --heads hit_weight.casual` picks that one.

### 2. Save presets

`scion/presets/snappy.json`:

<!-- validate: preset -->
```json
{"schema_version":1,"name":"snappy","description":"Short, crisp hits.","tunables":{"CritShake.Strength":0.2,"CritShake.Stop":0.03,"HitFlash.Seconds":0.05}}
```

### 3. Blend and compare

```sh
dotnet scion trial run SlashHit --variants base,heavy,snappy,"base + 0.7 heavy + 0.3 snappy" --motion
dotnet scion judge run <run> --heads hit_weight,hit_weight.casual
```

## Reference

### The cycle

1. Generate many variants (presets, blends, guided search).
2. The judge orders them per audience.
3. A person picks between the top two.
4. The vote trains the head; a winning blend can become a preset.

### Where art-style variations come from

Scion varies what code and numbers control: grafts, tunables, presets. New art in the game's style (sprites, icons)
comes from your art pipeline or image tools, outside Scion; once in the project, trials capture it and the judge
ranks it like anything else.

### Blend syntax

`base + 0.7 heavy + 0.3 snappy`: weights are decimals, presets are names in `scion/presets/`, `base` is the initial
values. See [Tunables and presets](/scion/41664431a8f5/grafts/tunables/).

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| `no_match` | 2 | A preset or head name does not exist. | List `scion/presets/` or `judge list`. |
| `bad_value` | 2 | A blend does not parse. | Use `name`, `+` and decimal weights. |

## Examples

### An accessibility head

```sh
dotnet scion judge new calm_motion.access --question scion/judge/questions/calm_motion.json --sequence
```

### Use a blend in a spec run

```sh
dotnet scion trial run --spec StrongerCrits --variants base,"base + 0.7 heavy + 0.3 snappy"
```

## See also

- [Tunables and presets](/scion/41664431a8f5/grafts/tunables/)
- [Train a head](/scion/41664431a8f5/judge/training/)
- [Guided search](/scion/41664431a8f5/explore/guided-search/)
