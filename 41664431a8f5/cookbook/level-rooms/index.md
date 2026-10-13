---
title: "Audit level rooms"
description: "Capture every room template in two views with a static trial, probe props and doors, fail on custom gates, and recapture only changed rooms."
type: guide
url: https://jand-s.github.io/scion/41664431a8f5/cookbook/level-rooms/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/cookbook/level-rooms.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/trials/static-trials.md
  - https://jand-s.github.io/scion/41664431a8f5/evaluation/gates.md
  - https://jand-s.github.io/scion/41664431a8f5/trials/checks.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Audit level rooms

Level rooms break quietly: a prop floats, a door is blocked, a decoration from another biome slips in. This recipe
audits every room template with a `StaticTrial<RoomTemplate>`: two views per room (empty from above, and with units
from the game camera), probes for props and doors, custom gates that fail on the common breakages, and reruns of only
the rooms a change touched.

## When to use

- Games with authored or generated rooms, tiles or arenas.
- After changes to room data, prop prefabs or the placement code.

## Before you begin

- The game can build one room in an empty scene from its template (Orchard's `RoomBuilder.Build(template)`), the same
  code path the game uses.

## Steps

### 1. The static trial

```csharp
public sealed class RoomAudit : StaticTrial<RoomTemplate>
{
    public RoomAudit() { Tags = new[] { "level" }; SettleFrames = 5; }

    protected override IEnumerable<RoomTemplate> Enumerate() => RoomTemplates.All.OrderBy(t => t.Id);
    protected override string ItemName(RoomTemplate t) => t.Id;

    protected override IEnumerator Show(RoomTemplate t, TrialSession s)
    {
        yield return RoomBuilder.Build(t);
    }

    protected override IEnumerator Clear(RoomTemplate t, TrialSession s)
    {
        RoomBuilder.Clear();
        yield break;
    }

    protected override IEnumerable<TrialView> Views(RoomTemplate t)
    {
        yield return TrialView.Top(RoomBuilder.Bounds, pxPerUnit: 16, grid: true);
        yield return TrialView.Screen;
    }

    protected override void Measure(RoomTemplate t, TrialSession s)
    {
        s.Probe(t.Id + "_props", RoomBuilder.Props.Count);
        s.Probe(t.Id + "_doors_open", RoomBuilder.Doors.Count(d => d.Reachable));
    }
}
```

### 2. Custom gates

Register `FloatingPropGate` and a `BlockedDoorGate` as `ITrialCheck` ([Gates](/scion/41664431a8f5/evaluation/gates/) shows the
first). Others worth writing: a prop clipping another prop, a prop from a foreign biome, an invisible blocker on the
navigation mesh, a room that does not match its template's size.

### 3. Run and review

```sh
dotnet scion trial run RoomAudit -j 6
dotnet scion trial show RoomAudit --image sheet
```

### 4. Recapture only what changed

```sh
dotnet scion trial run RoomAudit --changed
```

`--changed` takes the ids that were `changed` or `stale` in the last run; for static trials, the stills that changed
are listed in `compare.json`.

## Reference

| Output | Content |
|---|---|
| `<room>.top.png` | From above with a 1-unit grid. |
| `<room>.png` | The game camera with units placed. |
| `sheet.png` | Every room, labeled. |
| probes | Props and reachable doors per room. |

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| check `gate:floating_prop` fails | 1 | A prop is above the ground. | Fix the prefab pivot or the placement. |
| check `gate:blocked_door` fails | 1 | No path through a door. | Move the blocking prop. |

## Examples

### A judge head trained on past room reviews

```sh
dotnet scion judge new room_quality --question scion/judge/questions/room_quality.json --view full
dotnet scion judge run <run> --heads room_quality
```

### Only one biome

```csharp
protected override IEnumerable<RoomTemplate> Enumerate() => RoomTemplates.All.Where(t => t.Biome == "orchard");
```

## See also

- [Static trials](/scion/41664431a8f5/trials/static-trials/)
- [Gates](/scion/41664431a8f5/evaluation/gates/)
- [Checks](/scion/41664431a8f5/trials/checks/)
