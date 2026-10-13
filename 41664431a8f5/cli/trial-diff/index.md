---
title: "trial diff"
description: "Compare two runs or two copies id by id - hash, frame counts, the first differing frame and its cause, and image heatmaps."
type: cli
url: https://jand-s.github.io/scion/41664431a8f5/cli/trial-diff/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/cli/trial-diff.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/evaluation/ab-comparison.md
  - https://jand-s.github.io/scion/41664431a8f5/trials/image-comparison.md
  - https://jand-s.github.io/scion/41664431a8f5/determinism/verify.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# trial diff

`dotnet scion trial diff` compares two run folders (or two copies inside runs) without baselines: for each id in
both, whether the hash is the same, the frame counts, the first frame where the per-frame data differs and its cause,
and an image comparison with a heatmap per artifact. Use it for before and after a graft, or two presets.

## Synopsis

```text
dotnet scion trial diff <runA> <runB> [Id...]
```

## Description

`runA` and `runB` are run folder names, paths, or `<run>/q<i>` for one copy. Ids default to every id present in
both. Frame data is compared column by column in the cause order of `determinism.tsv` (`state`, `tick`, `rng`,
`unscaled`, `time`, `px`, `frames`). Images use each id's tolerance and masks; heatmaps go to
`<runB>/diff/<IdDir>/r<n>/<artifact>.diff.png`.

## Options

| Option | Default | Meaning |
|---|---|---|
| `<runA> <runB>` | required | The two sides. |
| `Id...` | common ids | Limit to these. |

## Output

<!-- validate: cli/trial-diff -->
```json
{"schema_version":1,"command":"trial diff","ok":true,"a":"20261011-143200","b":"20261011-150512","recipes":[{"id":"SlashHit","hash":"same","frames":[46,46],"first_frame":14,"cause":"px","images":[{"file":"window.png","status":"changed","ratio":0.021,"diff":"diff/SlashHit/r1/window.diff.png"}]},{"id":"KnightFalls","hash":"same","frames":[52,52],"first_frame":null,"cause":null,"images":[{"file":"window.png","status":"same","ratio":0,"diff":null}]}],"next_step":"look at diff/SlashHit/r1/window.diff.png"}
```

## Exit codes

| Code | When |
|---|---|
| 0 | Compared (differences do not fail). |
| 2 | A run or id not found. |

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| `no_match` | 2 | No such run or id. | Use the folder names under `trials.out`. |
| warning `pairs_missing` | 0 | Some ids exist in only one run. | Listed in `warnings`; select common ids. |

## Examples

### Before and after a graft

```sh
dotnet scion trial diff 20261011-143200 20261011-143512 SlashHit FireballArea
```

### Two copies of one run

```sh
dotnet scion trial diff 20261011-143200/q0 20261011-143200/q3 KnightFalls
```

## See also

- [A/B comparisons](/scion/evaluation/ab-comparison/)
- [Image comparison](/scion/trials/image-comparison/)
- [Verify determinism](/scion/determinism/verify/)
