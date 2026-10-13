---
title: "Static trials"
description: "Capture every state of a screen, a unit or a room with StaticTrial, in one or more views, with measurements and a labeled contact sheet."
type: guide
url: https://jand-s.github.io/scion/41664431a8f5/trials/static-trials/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/trials/static-trials.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/cookbook/hud-states.md
  - https://jand-s.github.io/scion/41664431a8f5/cookbook/level-rooms.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/api/adapters.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Static trials

Some things have no action to play: a HUD in 20 states, every unit in its idle pose, every room template. A
`StaticTrial<TItem>` enumerates items, shows each one, lets it settle, captures it in one or more **views** and
writes a labeled `sheet.png` with all of them. The package registers the adapter that plays static trials, so a
static trial needs no adapter of its own.

## When to use

- Use a static trial to review many states at once, or to catch a regression in any of them.
- Use a normal recipe when something must happen over time (a hit, a turn).

## Steps

### 1. Enumerate, show, clear

```csharp
using System.Collections;
using System.Collections.Generic;
using Scion.Trials;

namespace Orchard.Trials
{
    public sealed class UnitGallery : StaticTrial<UnitKind>
    {
        public UnitGallery() { Description = "Every unit in its idle pose."; Tags = new[] { "art" }; }

        protected override IEnumerable<UnitKind> Enumerate() => UnitKinds.All;
        protected override string ItemName(UnitKind k) => k.Id;

        protected override IEnumerator Show(UnitKind k, TrialSession s)
        {
            Gallery.Place(k);
            yield return s.WaitUntil(() => Gallery.Ready, "unit placed");
        }

        protected override IEnumerator Clear(UnitKind k, TrialSession s)
        {
            Gallery.Clear();
            yield break;
        }

        protected override IEnumerable<TrialView> Views(UnitKind k)
        {
            yield return TrialView.Screen;
            yield return TrialView.Top(new Bounds(Vector3.zero, new Vector3(4, 2, 4)), pxPerUnit: 32, grid: true);
        }

        protected override void Measure(UnitKind k, TrialSession s) => s.Probe(k.Id + "_height_px", Gallery.HeightPx);
    }
}
```

### 2. Run it

```sh
dotnet scion trial run UnitGallery
dotnet scion trial show UnitGallery --image sheet
```

## Reference

### Members

| Member | Default | Meaning |
|---|---|---|
| `Enumerate()` | abstract | The items, in a stable order. |
| `ItemName(TItem)` | `ToString()` | The still's name and the sheet label. Names follow the still rules. |
| `Show(TItem, TrialSession)` | abstract | Put the item on screen and wait until it is ready. |
| `Clear(TItem, TrialSession)` | nothing | Remove it. |
| `Views(TItem)` | `TrialView.Screen` | The cameras to capture with. |
| `Measure(TItem, TrialSession)` | nothing | Probes for the item, after it settled. |
| `SettleFrames` | 3 | Frames to render after `Show` before capturing. |

### Views

| View | Captures |
|---|---|
| `TrialView.Screen` | The game camera, as the player sees it. |
| `TrialView.Ui` | Only the UI layer, on a transparent background. |
| `TrialView.Top(bounds, pxPerUnit, grid)` | An orthographic camera from above, framing `bounds` at `pxPerUnit`; `grid` draws a 1-unit grid. |

### Outputs

For each item and view: `<item>.<view>.png` (the view name is omitted for `Screen`). Then `sheet.png`: every
capture in a grid, labeled with the item name, with the longer side at most 2000 px. A static trial has no cut, so
`window.png` and `context.png` are not written; baselines compare the stills.

### Adapter choice

The package registers `StaticTrialAdapter : TrialAdapter<StaticTrial>`, which only calls `Play`. To load a scene
first, register an adapter for your type; the more specific type wins:

```csharp
public sealed class HudAdapter : TrialAdapter<HudStates>
{
    protected override IEnumerator Run(HudStates r, TrialSession s)
    {
        yield return SceneLoader.Load("HudPreview");
        yield return r.Play(s);
    }
}
```

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| recipe error `duplicate item name <name>` | 1 | Two items give the same `ItemName`. | Make names unique. |
| recipe error `timeout (30s): <what>` | 1 | `Show` never got ready. | Raise `TimeoutSeconds` for big galleries, or split the trial. |

## Examples

### Only the items that changed

```sh
dotnet scion trial run UnitGallery
dotnet scion trial show UnitGallery --changed --grid
```

### HUD states at two resolutions

See [Capture every HUD state](/scion/41664431a8f5/cookbook/hud-states/).

## See also

- [Capture every HUD state](/scion/41664431a8f5/cookbook/hud-states/)
- [Audit level rooms](/scion/41664431a8f5/cookbook/level-rooms/)
- [Adapters API](/scion/41664431a8f5/reference/api/adapters/)
