---
title: "Image comparison"
description: "Compare captures with baselines using a perceptual OKLab metric, tolerances, masks and learned unstable masks, and read the heatmaps."
type: guide
url: https://jand-s.github.io/scion/41664431a8f5/trials/image-comparison/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/trials/image-comparison.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/trials/baselines.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/api/compare.md
  - https://jand-s.github.io/scion/41664431a8f5/cli/trial-diff.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Image comparison

Images are compared with `Scion.Compare`: a per-pixel perceptual distance (OKLab with the HyAB metric), a tolerance
per recipe (`ImageTolerance`), masks for regions that must be ignored, and an optional mask learned from runs that
shows which pixels are unstable. The result is a status per artifact (`Same`, `Within`, `Changed`), a changed ratio,
the bounds of the change and a heatmap `diff.png`.

## When to use

- Keep the default tolerance for most recipes: it ignores dithering and GPU rounding and catches visible changes.
- Use `ImageTolerance.Exact` for pixel art and UI that must not move a pixel.
- Set `Tolerance = null` for recipes where only the rules matter: images are captured but never compared.

## Steps

### 1. Choose the tolerance

```csharp
public HudStates()
{
    Tolerance = ImageTolerance.Exact;
}

public FireballArea()
{
    Tolerance = new ImageTolerance(pixel: 0.08f, maxRatio: 0.005f, maxPixels: -1);
}
```

### 2. Mask what changes on purpose

```csharp
public SlashHit()
{
    Mask("*", new Rect(0f, 0.94f, 1f, 0.06f), "turn timer");   // every artifact, top strip
    Mask("window.png", new Rect(0.8f, 0f, 0.2f, 0.1f), "fps counter");
}
```

Coordinates are normalized (0 to 1) with the origin at the bottom left, like `Screen`. At run time,
`session.MaskRect(screenPixels, why)` adds a mask in pixels.

### 3. Learn unstable pixels

```sh
dotnet scion trial run FireballArea --learn-unstable 3
```

The recipe runs 3 times; pixels that differ between those runs form `unstable.png`, saved with the baseline when
approved. A learned mask may cover at most 2 % of the image: above that, the recipe is not deterministic enough to
mask, and the run warns `mask_cap` instead of hiding the problem.

### 4. Read a change

```sh
dotnet scion trial show FireballArea --grid
dotnet scion trial diff 20261011-143200 20261011-150512 FireballArea
```

## Reference

### The metric

Each pixel pair is converted to OKLab and compared with HyAB (absolute lightness difference plus Euclidean distance
in a and b), scaled to 0 to 1. A pixel counts as changed when its distance is above `Pixel`. The image is:

| Status | When |
|---|---|
| `Same` | No changed pixel. |
| `Within` | Changed pixels are at most `MaxRatio` of the unmasked area and at most `MaxPixels` (when not -1). |
| `Changed` | Otherwise. |

The same metric gives the same result on every machine, so baselines from one Mac compare with captures from
another Mac of the same platform id.

### Tolerances

| Value | Pixel | MaxRatio | MaxPixels |
|---|---|---|---|
| `ImageTolerance.Default` | 0.1 | 0.001 | -1 |
| `ImageTolerance.Exact` | 0 | 0 | 0 |
| `null` (recipe) | images not compared; frame hash changes give `px_changed` | | |

### Outputs

| File | Content |
|---|---|
| `<run>/diff/<IdDir>/r<n>/<artifact>.diff.png` | Heatmap: changed pixels in red over a grey copy of the current image; masks hatched. |
| `compare.json` | Per id: verdict, reasons, image statuses, ratios, bounds (pixels, top-left origin) and diff paths. |
| `trial show --grid` output | Baseline, current and diff side by side. |

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| warning `mask_cap` | 0 | The learned mask would cover more than 2 %. | Make the recipe deterministic ([Particles, animators, UI and physics](/scion/41664431a8f5/determinism/control/)). |
| warning `px_unstable` | 0 | Frame hashes vary between rounds of this run. | Same as above; `px_changed` is noise until fixed. |
| warning `baseline_unreadable` | 0 | A baseline image or JSON cannot be read. | Check Git LFS (`git lfs pull`). |

## Examples

### Compare two images from code

```csharp
var a = Image.Load("scion/baselines/SlashHit/osx-metal-1280x720-offscreen/window.png");
var b = Image.Load(".scion/runs/20261011-143200/q0/SlashHit/r1/window.png");
var r = ImageDiff.Compare(a, b, Mask.Empty(a.Width, a.Height), ImageTolerance.Default);
Console.WriteLine($"{r.Status} {r.Ratio:P2} at {r.Bounds.X},{r.Bounds.Y}");
```

### Exact UI, tolerant world

```csharp
Tolerance = ImageTolerance.Default;
Mask("*", new Rect(0f, 0f, 1f, 0.15f), "HUD compared by HudStates with Exact");
```

## See also

- [Baselines and approval](/scion/41664431a8f5/trials/baselines/)
- [Compare API](/scion/41664431a8f5/reference/api/compare/)
- [trial diff](/scion/41664431a8f5/cli/trial-diff/)
- [A/B comparisons](/scion/41664431a8f5/evaluation/ab-comparison/)
