---
title: "A/B comparisons"
description: "Compare variants fairly - aligned on the impact frame, one row each plus a difference row, with blind shuffled labels asked in both orders."
type: guide
url: https://jand-s.github.io/scion/41664431a8f5/evaluation/ab-comparison/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/evaluation/ab-comparison.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/trials/image-comparison.md
  - https://jand-s.github.io/scion/41664431a8f5/judge/training.md
  - https://jand-s.github.io/scion/41664431a8f5/cli/trial-diff.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# A/B comparisons

Comparing two versions of an effect by eye is easy to get wrong: the frames are not aligned, the labels give the
answer away, and the first one shown wins more often. Scion's comparisons align variants on the impact event, show
one row per variant plus a difference row, hide which is which behind shuffled labels (X, Y, Z) and ask each pair in
both orders. The same rules apply when an agent, the judge or a person compares.

## When to use

- Use `trial show --grid` to compare a run with its baseline.
- Use `trial diff` to compare two runs (before and after a graft, two presets).
- Use `judge vote` to collect blind duels from people ([Train a head](/scion/judge/training/)).

## Steps

### 1. Produce the variants

```sh
dotnet scion trial run SlashHit --variants base,heavy,snappy
```

### 2. Compare two runs

```sh
dotnet scion trial diff 20261011-143200 20261011-150512 SlashHit
```

<!-- validate: cli/trial-diff -->
```json
{"schema_version":1,"command":"trial diff","ok":true,"a":"20261011-143200","b":"20261011-150512","recipes":[{"id":"SlashHit","hash":"same","frames":[46,48],"first_frame":14,"cause":"px","images":[{"file":"window.png","status":"changed","ratio":0.021,"diff":"diff/SlashHit/r1/window.diff.png"}]}],"next_step":"look at diff/SlashHit/r1/window.diff.png"}
```

### 3. Ask blind

```sh
dotnet scion judge vote --run 20261011-150512 --head hit_weight
```

The page shows two sequences labeled X and Y, aligned on the impact, and asks which is better (or a tie, and how
strongly). Each pair comes back later in the other order.

## Reference

### Alignment

Variants are aligned on the first `IMPACT` mark. When a variant has none, on the first mark with the same label in
both; otherwise on the cut's first frame, and the grid says so. Frames before and after are padded so every row has
the same length.

### Grid layout

| Row | Content |
|---|---|
| One per variant | The `window.png` frames, aligned. |
| Difference | Per column, the heatmap between the first two rows. |

Labels in the grid are the variant names for `--grid` and `trial diff` (not blind), and X, Y, Z for duels.

### Blind rules

- Labels are shuffled per pair and stored in the vote, never shown with names.
- Each pair is asked in both orders; votes that disagree with themselves count as a tie.
- The control variant (`base`) is included in every comparison set.

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| `no_match` | 2 | A run or id is not found. | Use the run folder name from `trial run`. |
| warning in output `not aligned` | 0 | No common mark. | Mark the impact in both variants. |

## Examples

### Compare two copies of one run

```sh
dotnet scion trial diff 20261011-143200/q0 20261011-143200/q3 KnightFalls
```

### Grid against the baseline

```sh
dotnet scion trial show SlashHit --grid
```

## See also

- [Image comparison](/scion/trials/image-comparison/)
- [trial diff](/scion/cli/trial-diff/)
- [Train a head](/scion/judge/training/)
