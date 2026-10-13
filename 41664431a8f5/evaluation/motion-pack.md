---
title: "The motion pack"
description: "The images and metrics Scion writes to describe motion - motion diff, tracks, id masks, metrics.json and a preview GIF - and how agents read them."
type: guide
url: https://jand-s.github.io/scion/41664431a8f5/evaluation/motion-pack/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/evaluation/motion-pack.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/trials/capture.md
  - https://jand-s.github.io/scion/41664431a8f5/judge/motion.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/files/run-folder.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# The motion pack

A sheet of frames shows what a hit looks like, not how it moves. With `--motion`, each recipe also writes a
**motion pack**: `motion-diff.png` (what changed between frames), `tracks.png` (where things moved), `idmask/`
(which object is which), `metrics.json` (hit-stop, shake, flash, number easing as numbers) and `preview.gif` for
people. Agents read the metrics and the two PNGs; they never need to watch a video.

## When to use

- Turn on `--motion` when the task is about feel: hit-stop, shake, flash, timing.
- Use the metrics as spec targets (`Expect("hit_stop_ms", 50, 120)`).

## Steps

### 1. Run with the pack

```sh
dotnet scion trial run SlashHit --motion
```

### 2. Read the metrics

`q0/SlashHit/r1/metrics.json`:

<!-- validate: metrics -->
```json
{"schema_version":1,"id":"SlashHit","round":1,"impact_frame":14,"hit_stop":{"frames":2,"ms":66.7},"shake":{"peak_px":7.5,"rms_px":3.1,"decay_ms":200},"flash":{"delta_l":0.42,"ms":100,"area":0.03,"per_second":1},"anticipation_ms":133.3,"numbers":[{"text":"12","rise_px":38,"ms":900,"easing":"ease-out"}],"targets":{"hit_stop_ms":{"min":50,"max":120,"status":"pass"}}}
```

### 3. Look at the images when needed

```sh
dotnet scion trial show SlashHit --image motion-diff
```

## Reference

### Files

| File | Content | For |
|---|---|---|
| `window.png` | 12 frames around the impact | agents, people |
| `context.png` | 6 frames across the cut | agents, people |
| `motion-diff.png` | The same frames as `window.png`, each showing only what changed since the previous frame (brighter = more) | agents |
| `tracks.png` | The first frame with the path of each tracked object drawn over it, one color per object, dots every frame | agents |
| `idmask/NNNN.png` | Per frame, every renderer drawn in a flat color by object id; `idmask/ids.json` maps colors to object paths | tools |
| `metrics.json` | Motion metrics (below) | agents, specs, the judge |
| `preview.gif` | The cut at real speed, at most 640 px wide | people |
| `report.md` | Links and a metrics table | both |

### Metrics

| Metric | How it is measured |
|---|---|
| `hit_stop.frames`, `ms` | Consecutive frames after the impact where tracked objects do not move. |
| `shake.peak_px`, `rms_px`, `decay_ms` | Camera offset per frame (from the camera transform), in screen pixels. |
| `flash.delta_l`, `ms`, `area`, `per_second` | Mean OKLab lightness change in the flashed region, its duration and screen fraction. |
| `anticipation_ms` | Time between the command and the impact. |
| `numbers[]` | Floating text: rise in pixels, lifetime, the easing that fits best (`linear`, `ease-out`, `ease-in-out`, `back`). |
| `targets` | The bounds from the active spec and whether each passes. |

### Image rules

- Every PNG keeps the longer side at or under 2000 px.
- Frames carry their number and marks in a label strip, so an agent can cite "frame 14".
- `motion-diff.png` and `tracks.png` share the frame order and size of `window.png`.

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| warning in `report.md` `no impact mark` | 0 | No `MarkImpact` in the cut; metrics measured from the cut's middle. | Mark the impact from an observer graft. |
| `bad_value` | 2 | `--motion` with `--warm` (warm runs capture no images). | Run cold. |

## Examples

### Ask for motion only on combat recipes

```sh
dotnet scion trial run --tag combat --motion
```

### Use a metric in a spec

```csharp
s.Visual("SlashHit").Expect("shake_peak_px", min: 4, max: 12);
```

## See also

- [Capture and outputs](/scion/trials/capture/)
- [Judge motion](/scion/judge/motion/)
- [The run folder](/scion/reference/files/run-folder/)
