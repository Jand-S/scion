---
title: "Judge motion"
description: "Train sequence heads that judge feel from motion metrics and per-frame embeddings, such as hit weight, anticipation and too much or too little juice."
type: guide
url: https://jand-s.github.io/scion/41664431a8f5/judge/motion/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/judge/motion.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/evaluation/motion-pack.md
  - https://jand-s.github.io/scion/41664431a8f5/judge/training.md
  - https://jand-s.github.io/scion/41664431a8f5/cookbook/tune-hit-feel.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Judge motion

Feel lives in motion: how long the hit-stop lasts, how the shake decays, when the flash peaks. A **sequence head**
combines two inputs: the numbers of the motion pack (`metrics.json`) and the embeddings of each frame of the cut. A
small tree model learns the taste over the numbers from few examples and explains its answer; the frame embeddings
add style. This guide creates and uses a sequence head.

## When to use

- Use a sequence head for feel questions: "which hit is heavier?", "is the anticipation visible?", "too much juice?".
- Use an image head for anything that one frame shows.

## Before you begin

- The eye model in the model cache (`dotnet scion judge model pull`, once per machine).
- Runs made with `--motion`, so `metrics.json` exists for each recipe.
- The gates still run first: a sequence that fails photosensitivity is never ranked.

## Steps

### 1. Create a sequence head

```sh
dotnet scion judge new hit_weight --question scion/judge/questions/hit_weight.json --view actor --sequence
```

### 2. Collect duels on sequences

```sh
dotnet scion trial run SlashHit --variants base,heavy,snappy --motion
dotnet scion judge vote --run 20261012-091500 --head hit_weight
```

Duels show the two `preview.gif` side by side, aligned on the impact.

### 3. Train and run

```sh
dotnet scion judge train hit_weight
dotnet scion judge run 20261012-091500 --heads hit_weight
```

## Reference

### Inputs

| Input | From | Used for |
|---|---|---|
| Motion metrics | `metrics.json`: hit-stop, shake, flash, anticipation, numbers | The tree model: explainable, works with 50 votes |
| Frame embeddings | The eyes on each frame of `window.png`, `actor` view | Style: does the motion look like our approved hits |

### Output

A sequence head answers like any head, and its `legend` names the metrics that weighed most:

```text
hit_weight 0.74 (heavier): hit_stop.ms 100 (+), shake.peak_px 9 (+), flash.ms 80 (-)
```

### Speed

A 30-frame sequence takes under a second: metrics are already computed, and frame embeddings are cached like any
image.

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| `bad_value` | 2 | The run has no `metrics.json`. | Run with `--motion`. |
| `model_missing` | 3 | The eye model is not in the model cache. | `dotnet scion judge model pull`. |

## Examples

### Too much or too little juice

```json
{"schema_version":1,"run":null,"blind":true,"orders":2,"questions":{"juice":{"type":"choice","instructions":"How much juice does the hit have?","options":[{"id":"too_little","description":"Flat"},{"id":"right","description":"Right"},{"id":"too_much","description":"Noisy"}]}},"items":[]}
```

### Rank presets

```sh
dotnet scion judge run 20261012-091500 --heads hit_weight,juice
```

## See also

- [The motion pack](/scion/41664431a8f5/evaluation/motion-pack/)
- [Train a head](/scion/41664431a8f5/judge/training/)
- [Tune a hit effect](/scion/41664431a8f5/cookbook/tune-hit-feel/)
