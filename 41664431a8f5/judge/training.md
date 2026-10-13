---
title: "Train a head"
description: "Collect blind duels and labels, add synthetic degradations, keep a gold set out of training, train a head in seconds and check its calibration."
type: guide
url: https://jand-s.github.io/scion/41664431a8f5/judge/training/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/judge/training.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/judge/questions.md
  - https://jand-s.github.io/scion/41664431a8f5/cli/judge.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/files/judge-data.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Train a head

A head learns one question from votes. This guide creates a head, collects votes with `judge vote` (blind duels in
the browser), adds synthetic examples (degraded versions and fabricated bugs), keeps a gold set that never enters
training, trains with `judge train` (cross-validated, seconds) and promotes the new version only when it agrees
more with the people's gold votes than the previous one.

## When to use

- Train a head when the same taste question comes up on every change (HUD readability, hit weight, glitches).
- Start with three heads: UI readability, visual glitch, and "looks like our game".

## Before you begin

- The judge configured in `scion.json` (`judge.dir`, `judge.eyes`, `judge.cache`); `dotnet scion doctor` checks that
  the eye model is present.
- A few runs with captures of the thing to judge.

## Steps

### 1. Create the head

```sh
dotnet scion judge new readability --question scion/judge/questions/readability.json --view hud
```

### 2. Vote

```sh
dotnet scion judge vote --run 20261011-150512 --head readability
```

A local page opens with blind pairs (X and Y), each asked in both orders. Ten minutes gives about 60 duels. Votes go
to `scion/judge/votes/readability.jsonl`.

### 3. Add synthetic examples

```sh
dotnet scion judge vote --head readability --synthetic 200
```

Scion degrades approved captures (lower contrast, 1 px misalignment, clutter, off-palette color, overflowing text)
and fabricates bugs (missing-shader magenta, floating props, z-fighting). Each pair "original beats degraded" is a
vote with `source: agent` and the degradation as its tag.

### 4. Set the gold set apart

```sh
dotnet scion judge vote --head readability --gold ui_v1 --count 150
```

Gold votes come only from people, never from agents, and never enter training. A week later, re-vote 30 to 50 of
them (`--regold`) to measure how much people agree with themselves: that is the ceiling a head can reach.

### 5. Train

```sh
dotnet scion judge train readability
```

<!-- validate: cli/judge -->
```json
{"schema_version":1,"command":"judge train","ok":true,"head":{"schema_version":1,"name":"readability","version":4,"eyes":"siglip2-so400m","view":"hud","question":{"type":"score","instructions":"How easy is the HUD to read at 1x on a laptop screen?"},"trained_at":"2026-10-12T08:10:00Z","votes":412,"calibration":{"gold_set":"ui_v1","agreement":0.81,"ceiling":0.86}},"training":{"folds":5,"accuracy":0.83,"gold_agreement":0.81,"previous":0.78,"promoted":true},"next_step":"dotnet scion judge run <run> --heads readability"}
```

### 6. Use it

```sh
dotnet scion judge run 20261011-150512 --heads readability,has_glitch
```

## Reference

### Promotion rule

A new version replaces the current head only when its agreement with the gold set is higher than the previous
version's. Otherwise it is kept as `readability@4-rejected` for inspection, and `promoted` is false.

### What makes votes useful

| Do | Because |
|---|---|
| Include near-misses: pairs almost equal where one is worse | They teach the boundary. |
| Mix sources: old versions of the game (git), degradations, fabricated bugs, real variants | One source teaches its own artifacts. |
| Vote in both orders | Position bias cancels out. |
| Let the judge pick the next duels (`judge vote` does by default) | It asks where it is least sure. |

### Agents in the loop

Agents may generate synthetic pairs, label pairs in batch as `source: agent`, pick the most informative screens for a
person, and explain disagreements ("it learned that dark means bad"). They never vote in the gold set, and a head is
promoted only by agreement with people.

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| `model_missing` | 3 | The eye model is not downloaded. | `dotnet scion doctor` prints the download command. |
| `no_match` | 2 | No head with that name. | `dotnet scion judge list`. |
| training `promoted: false` | 0 | The new version does not beat the old one on gold. | Add near-miss votes; check `judge show` for where it errs. |

## Examples

### Inspect a head

```sh
dotnet scion judge show readability
```

### A duel head for feel

```sh
dotnet scion judge new hit_weight --question scion/judge/questions/hit_weight.json --view actor --sequence
```

## See also

- [Questions and answers](/scion/judge/questions/)
- [judge (CLI)](/scion/cli/judge/)
- [Judge data files](/scion/reference/files/judge-data/)
- [Judge motion](/scion/judge/motion/)
