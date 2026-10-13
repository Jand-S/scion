---
title: "Gates"
description: "Every built-in gate - text overflow, UI grid, contrast, photosensitivity, regression, telemetry versus pixels, nothing changed, palette - and custom gates."
type: reference
url: https://jand-s.github.io/scion/41664431a8f5/evaluation/gates/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/evaluation/gates.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/evaluation/overview.md
  - https://jand-s.github.io/scion/41664431a8f5/trials/checks.md
  - https://jand-s.github.io/scion/41664431a8f5/trials/specs.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/api/specs.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Gates

A gate is an objective visual check: it measures something a person would otherwise have to notice, and fails the
recipe when the measure is out of bounds. Gates are checks with the id `gate:<name>`, enabled per recipe with
`TrialChecks.Gate` or per spec with `.Gate(Gates.X)`. Their results appear in `result.json` like any check, with the
frame and the region of the problem.

## Enabling a gate

```csharp
protected override void Checks(TrialChecks c) =>
    c.Gate(Gates.TextOverflow)
     .Gate(Gates.TextContrast, ("min_ratio", 4.5))
     .Gate(Gates.Photosensitivity);
```

`Gates` holds the ids as string constants (`Gates.TextOverflow == "text_overflow"`). Options are name and value
pairs; unknown options fail the recipe with a message listing the valid ones.

## Built-in gates

### TextOverflow

Id `text_overflow`. Fails when a TextMeshPro or UI Toolkit text in the visible UI is truncated, wraps beyond its
rect or overflows its parent, on any captured frame. Reads the UI tree in test builds. Options: `ignore` (a list of
element names).

### UiGrid

Id `ui_grid`. Fails when a UI element's edges are off the layout grid by more than `tolerance_px` (default 1).
Options: `grid_px` (default 8), `tolerance_px`, `root` (a UI element name). Use it on static HUD states.

### TextContrast

Id `text_contrast`. Fails when text contrast with the pixels behind it is under WCAG 2.2: 4.5:1 for normal text,
3:1 for large text (18 pt, or 14 pt bold). Measures the rendered frame under each text element. Options:
`min_ratio`, `min_ratio_large`.

### Photosensitivity

Id `photosensitivity`. Fails on general or red flashes above the limits of the IRIS method (more than 3 flashes in
any second over 25 % of the screen). Reads every frame of the cut; works on `window.png` sequences too. Options:
`area` (default 0.25), `max_per_second` (default 3).

### ImageRegression

Id `image_regression`. Fails when an image is `changed` against the baseline, instead of only reporting the verdict.
Use it for screens that must never change without a new approval in the same commit. No options.

### TelemetryVsPixel

Id `telemetry_vs_pixel`. Fails when telemetry says a visual effect happened (an `fx` line: `Flash`, `Shake`, `Vfx`,
`Text`) but the frames within `window_frames` (default 6) after it show no change in the region, or the frame hash
did not change. Catches effects that fire but draw nothing (a missing material, a culled layer).

### NothingChanged

Id `nothing_changed`. Fails when the cut shows less than `min_ratio` (default 0.001) of changed pixels between its
first and last frame. Catches actions that silently do nothing on screen.

### Palette

Id `palette`. Fails when an isolated 2D asset (a sprite or a UI icon captured with `TrialView.Ui` on a transparent
background) uses colors outside the palette file. Options: `palette` (a `.hex` or `.gpl` file), `max_delta` (OKLab
distance, default 0.02). Only for isolated assets: lighting and post-processing change colors in scenes.

## Custom gates

Write an `ITrialCheck` whose `Id` starts with `gate:` and register it with `[assembly: TrialCheck]`. Two examples from
Orchard:

```csharp
public sealed class FloatingPropGate : ITrialCheck   // props must touch the ground
{
    string problem;
    public string Id => "gate:floating_prop";
    public void Begin(TrialRecipe r, TrialSession s) => problem = null;
    public void OnFrame(in FrameView f) { }
    public void OnLog(string m, string st, LogType t) { }
    public CheckResult End()
    {
        foreach (var prop in Props.All)
            if (prop.GroundGap > 0.02f) return CheckResult.Fail($"{prop.name} floats {prop.GroundGap:0.00} above ground");
        return CheckResult.Pass();
    }
}
```

A blocked door gate works the same way: for each door in the room, ask the navigation mesh for a path through it
and fail with the door's name when there is none.

## Results

| Field | Value |
|---|---|
| `checks[].id` | `gate:text_contrast` |
| `checks[].status` | `pass`, `warn`, `fail`, `skipped` |
| `checks[].reason` | `"Score" 3.1:1 on #5a5a5a (needs 4.5:1)` |
| `checks[].frame`, `region` | where; region in pixels, origin at the top left |

## See also

- [The evaluation funnel](/scion/evaluation/overview/)
- [Checks](/scion/trials/checks/)
- [Specs](/scion/trials/specs/)
- [Specs API](/scion/reference/api/specs/)
