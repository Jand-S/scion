---
title: "Capture every HUD state"
description: "A static trial over 20 HUD states at two resolutions, with gates for overflow, contrast and grid, a contact sheet, and judge ranking."
type: guide
url: https://jand-s.github.io/scion/41664431a8f5/cookbook/hud-states/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/cookbook/hud-states.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/trials/static-trials.md
  - https://jand-s.github.io/scion/41664431a8f5/evaluation/gates.md
  - https://jand-s.github.io/scion/41664431a8f5/judge/ui-review.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Capture every HUD state

The HUD has many states nobody looks at together: the turn banner, low health, four players, a status effect, a long
unit name. This recipe captures all of them with one `StaticTrial<HudState>`, at 360p and 720p, checks them with
gates (text overflow, contrast, grid), writes a labeled sheet, and lets the judge rank them for review.

## When to use

- Any change to HUD layout, fonts, colors or localization.
- As the input of a [UI review with the judge](/scion/41664431a8f5/judge/ui-review/).

## Before you begin

- The HUD can be put in any state from data, without a battle (Orchard's `Hud.Apply(HudState)`).
- Modes declared: `[assembly: TrialModes("360p=640x360", "720p=1280x720")]`.

## Steps

### 1. Describe the states

```csharp
public readonly struct HudState
{
    public readonly string Name; public readonly int Players; public readonly int Hp; public readonly string Status;
    public HudState(string name, int players, int hp, string status) { Name = name; Players = players; Hp = hp; Status = status; }
}
```

### 2. Write the static trial

```csharp
public sealed class HudStates : StaticTrial<HudState>
{
    public HudStates()
    {
        Description = "Every HUD state at the current mode.";
        Tags = new[] { "ui" };
        Tolerance = ImageTolerance.Exact;
    }

    protected override IEnumerable<HudState> Enumerate()
    {
        yield return new HudState("idle", 1, 20, null);
        yield return new HudState("low-hp", 1, 3, null);
        yield return new HudState("four-players", 4, 20, null);
        yield return new HudState("burning", 1, 12, "burn");
        yield return new HudState("long-name", 1, 20, null);
        // ... 15 more
    }

    protected override string ItemName(HudState s) => s.Name;

    protected override IEnumerator Show(HudState s, TrialSession session)
    {
        Hud.Current.Apply(s);
        yield return null;
    }

    protected override IEnumerable<TrialView> Views(HudState s) { yield return TrialView.Ui; }

    protected override void Checks(TrialChecks c) =>
        c.Gate(Gates.TextOverflow).Gate(Gates.TextContrast).Gate(Gates.UiGrid, ("grid_px", 4));
}
```

### 3. Run both modes

```sh
dotnet scion trial run HudStates --mode 360p
dotnet scion trial run HudStates --mode 720p
dotnet scion trial show "HudStates@*" --image sheet
```

### 4. Rank for review

```sh
dotnet scion judge run <run> --heads readability,has_glitch
```

## Reference

| Output | Content |
|---|---|
| `<state>.ui.png` | One still per state, UI layer only. |
| `sheet.png` | All states, labeled, longer side at most 2000 px. |
| `checks[]` | `gate:text_overflow`, `gate:text_contrast`, `gate:ui_grid` with frame and region. |

Ids are `HudStates@360p` and `HudStates@720p`; each has its own baseline.

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| check `gate:text_overflow` fails | 1 | A label is cut. | Fix the layout or the text; the region shows where. |
| check `gate:text_contrast` fails | 1 | Text below 4.5:1. | Change the color or add a backing panel. |

## Examples

### Only one state while fixing it

```sh
dotnet scion trial run HudStates --mode 360p
dotnet scion trial show HudStates@360p --image still
```

### Add a color-blind mode

```csharp
[assembly: TrialModes("360p=640x360", "720p=1280x720", "deuter")]
```

## See also

- [Static trials](/scion/41664431a8f5/trials/static-trials/)
- [Gates](/scion/41664431a8f5/evaluation/gates/)
- [Review a UI with the judge](/scion/41664431a8f5/judge/ui-review/)
