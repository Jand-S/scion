---
title: "Reading results"
description: "The order in which an agent reads a trial run - JSON, report.md, window.png, context.png - with the image rules and token costs."
type: guide
url: https://jand-s.github.io/scion/41664431a8f5/agents/reading-results/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/agents/reading-results.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/agents/next-action.md
  - https://jand-s.github.io/scion/41664431a8f5/evaluation/motion-pack.md
  - https://jand-s.github.io/scion/41664431a8f5/trials/capture.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Reading results

A trial run writes far more than an agent should read. This guide gives the reading order (the `trial run` JSON, then
`report.md` of failed or changed recipes, then `window.png` only when pixels changed or the task is visual, then
`context.png`, never `frames/`), the image rules that keep a 1-texel shake visible to a vision model, the motion pack
for judging feel, the two-pass review and what each step costs in tokens.

## When to use

- After every `dotnet scion trial run`, `trial run --warm` or `trial loop` pass.
- When reviewing a visual change: hit effects, HUD states, rooms, animations.

## Steps

### 1. Read the JSON

The envelope is always under 1.5 KB. Decide from it alone whether anything needs a look:

| Field | Look further when |
|---|---|
| `failed[]` | not empty: read each `report` |
| `changed[]` | not empty: these recipes differ from their baseline |
| `runs.new` | above 0: read `report.md` before approving |
| `determinism.hash_stable` | `false`: see [What to do next](/scion/agents/next-action/) |
| `warnings[]` | not empty |

Paths inside lists are relative to `run`; `next_step` uses absolute paths.

### 2. Read report.md of failed and changed recipes only

`dotnet scion trial show --failed` and `trial show --changed` print the paths. Each recipe's `report.md` (about 1,200
tokens) has the verdict, the checks that did not pass, the images with their frames, the timeline of marks, the
probes, notes and logged errors. Most failures are explained here, without an image.

### 3. Open window.png only for pixel changes or visual tasks

Open an image when `changed[].hash` is `same` (only pixels moved), when the verdict is `px_changed` or `within` and the
task is visual, or when the task itself is about the look. `window.png` holds up to 12 consecutive frames around the
impact, 3 columns, numbered with their game time and marks. Prefer `grid.png` from `trial show <Id> --grid` for a
`changed` recipe: baseline, current and difference rows in one image.

### 4. Open context.png only if the window is not enough

`context.png` shows 6 key frames chosen by the marks (announcement, action, impact, peak of the number, death,
after), 3 by 2. It answers "what happened before and after"; the window answers "how did the hit feel".

### 5. Never open frames/

`frames/NNNN.png` exists only with `--frames`, and a recipe has dozens to hundreds of them. Open single frames only
when a person asks for them.

### 6. Delegate when there are many images

With more than about 5 images to judge, give the run folder and the ids to the `scion-triage` subagent (Claude Code)
or a separate agent. It returns one line per recipe ([The Scion skill](/scion/agents/skill/)).

## Reference

### Image rules

Scion writes every image so a vision model sees what matters without recompression:

| Rule | Value | Why |
|---|---|---|
| Longest side | at most 2000 px for sheets (`window.png`, `context.png`, `sheet.png`); at most 1568 px for motion pack images, which crop the action area | Above about 1568 px some readers recompress images as JPEG, which erases 1-texel shake: use `--motion` when shake matters |
| File size | under 500 KB | Same |
| Images per request | at most 20 | API limit of most vision models |
| Scaling of crops | 2x nearest neighbor | Pixel art keeps its pixels |
| Frame labels | `f07 +117ms IMPACT` on each cell | The model knows time and events without guessing |
| Fixed crosshair | drawn on `window.png` when `--motion` is on | Reveals camera shake |

Sheets for people (`sheet.png`, `preview.gif`) may be larger; agents should not open `preview.gif` (models read only
its first frame).

### The motion pack

Vision models are poor at fine motion even when they see every frame. For feel (hit-stop, shake, flash, easing),
run with `--motion` and read the numbers first:

| File | Content | Tokens |
|---|---|---|
| `metrics.json` | Measured hit-stop, shake, flash, anticipation and number easing, with targets | about 300 |
| `report.md` | Per-frame timeline plus a measured, target and status table | about 1,200 |
| `window.png` | 12 frames around the impact, 2x nearest neighbor, numbered, with a crosshair | about 1,000 |
| `motion-diff.png` | The same layout with the frame-to-frame difference amplified (a black cell is a hit-stop frame) | about 1,000 |
| `tracks.png` | Luminance, camera offset, 1 - SSIM and number height over time, with lines at the marks | about 1,250 |
| `context.png` | 6 key frames | about 1,900 |

See [The motion pack](/scion/evaluation/motion-pack/).

### Two-pass review

For a visual judgment, do two passes:

1. **Blind:** only the images. "Describe the impact frame by frame." About 4,000 tokens.
2. **With numbers:** `tracks.png`, `metrics.json` and `report.md`; check where the description and the numbers
   disagree. About 7,500 tokens.

For A/B choices, see [A/B comparisons](/scion/evaluation/ab-comparison/): variants aligned on the impact, blind labels,
asked in both orders.

## Errors

| Situation | Next action |
|---|---|
| `report.md` missing for a failed pair | The copy crashed: read `q<i>/player.log` (`player_crashed`). |
| `window.png` missing | The recipe never called `MarkImpact` and had no cut; read `report.md` and the recipe. |
| An image is blank or magenta | The `blank` or `magenta` check failed too; it is in `report.md`. |
| `grid.png` missing | The recipe keeps no images (`Tolerance = null`); compare `window.png` of both runs with `trial diff`. |

## Examples

### Deciding without an image

```json
{"changed":[{"id":"KnightFalls","vs_approved":"changed","hash":"changed","image":"q1/KnightFalls/r1/window.png"}]}
```

`hash: changed` means the rules changed. The image will not explain why: read `report.md`, the probes and the diff of
the rule code first.

### A pixel-only change

```json
{"changed":[{"id":"SlashHit","vs_approved":"changed","hash":"same","image":"q0/SlashHit/r1/window.png","diff":"diff/SlashHit/r1/window.diff.png"}]}
```

`hash: same`: the rules did not change. Open `grid.png` (`trial show SlashHit --grid`) to see the visual change and
report it to the person.

## See also

- [What to do next](/scion/agents/next-action/)
- [Capture and outputs](/scion/trials/capture/)
- [The motion pack](/scion/evaluation/motion-pack/)
- [report.md](/scion/reference/files/report-md/)
