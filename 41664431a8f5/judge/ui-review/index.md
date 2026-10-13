---
title: "Review a UI with the judge"
description: "A nine-step UI review - capture every state at two sizes, run gates and the judge, fix the worst, re-judge variations in pairs and let a person pick."
type: tutorial
url: https://jand-s.github.io/scion/41664431a8f5/judge/ui-review/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/judge/ui-review.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/cookbook/hud-states.md
  - https://jand-s.github.io/scion/41664431a8f5/evaluation/gates.md
  - https://jand-s.github.io/scion/41664431a8f5/judge/training.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Review a UI with the judge

This tutorial reviews Orchard's battle HUD in about 30 to 45 minutes instead of hours. Scion captures 20 HUD states
at two resolutions, the gates catch objective problems, the judge scores and ranks every capture, an agent reviews
only the worst and the uncertain ones next to reference screens, makes variations, the judge compares them in
pairs, and a person picks between the final two. The vote trains the judge for next time.

## Before you begin

- The `HudStates` static trial ([Capture every HUD state](/scion/41664431a8f5/cookbook/hud-states/)).
- Trained heads `readability`, `has_glitch` and `looks_like_orchard` ([Train a head](/scion/41664431a8f5/judge/training/)).
- Optional: a reference library of screens from other games, indexed by the judge (`scion/judge/references/`).

## Steps

### 1. Capture (about 1 minute)

```sh
dotnet scion trial run HudStates --mode 360p
dotnet scion trial run HudStates --mode 720p
```

40 stills, two sheets.

### 2. Gates (seconds)

`HudStates` enables `TextOverflow`, `TextContrast` and `UiGrid`. Failures come first in `trial show --failed`.

### 3. Judge (seconds)

```sh
dotnet scion judge run <run> --heads readability,has_glitch,looks_like_orchard
```

Every capture gets a score per head; low-confidence answers are listed under `escalate`.

### 4. References (milliseconds)

```sh
dotnet scion judge run <run> --references 5
```

For each capture, the five most similar reference screens and their notes.

### 5. Agent review of the worst (5 to 10 minutes)

The agent reads only the 10 to 15 lowest-scored and escalated captures, each next to its references, and writes a
short list of problems with evidence (state, region).

### 6. Variations (5 to 15 minutes)

The agent writes graft or theme changes for the problems (spacing, contrast, icon size) as drafts and runs the
states again with `--grafts`.

### 7. Re-judge in pairs (3 to 5 minutes)

```sh
dotnet scion trial diff <before> <after> HudStates
dotnet scion judge run <after> --heads readability --pairs <before>
```

### 8. A person picks (about 5 minutes)

```sh
dotnet scion judge vote --run <after> --head readability --top 2
```

The person sees the top two variants per state, blind, and picks.

### 9. The vote trains

```sh
dotnet scion judge train readability
```

## Check the result

- `trial show HudStates --failed` lists no gate failures.
- `judge run` scores the winning variant above the original on `readability` for most states.
- `judge show readability` reports a new version, promoted, with agreement on the gold set equal or higher.

## Next steps

- Promote the winning grafts and approve the new stills ([From draft to final build](/scion/41664431a8f5/grafts/lifecycle/)).
- Add the HUD review to CI with only steps 1 and 2 ([Run Scion in CI](/scion/41664431a8f5/cookbook/ci/)).
- Train an audience head for accessibility ([Taste heads and feel presets](/scion/41664431a8f5/judge/presets/)).

## See also

- [Capture every HUD state](/scion/41664431a8f5/cookbook/hud-states/)
- [Gates](/scion/41664431a8f5/evaluation/gates/)
- [Train a head](/scion/41664431a8f5/judge/training/)
