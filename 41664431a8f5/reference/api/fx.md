---
title: "Fx API"
description: "The Fx facade and the IFxSink interface the game implements - sound, shake, text, flash, hit-stop, VFX and marks."
type: api
url: https://jand-s.github.io/scion/41664431a8f5/reference/api/fx/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/reference/api/fx.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/grafts/fx.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/api/testing.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Fx API

`Fx` is the effects facade grafts call; `IFxSink` is what the game implements once. Both exist in every build.

## Fx

**Namespace:** `Scion` · **Assembly:** `Scion.Runtime` · **Define:** none

```csharp
public static class Fx
{
    public static IFxSink Sink { get; set; }
    public static string Source { get; set; }
    public static void Sound(string id, Vector3? at = null, float volume = 1f);
    public static void Shake(float amount);
    public static void Text(string text, Vector3 at, Color color, float seconds = 1.1f, int size = 12, string icon = null);
    public static void Flash(Component target, Color color, float seconds);
    public static void HitStop(float seconds);
    public static void Vfx(string id, Vector3 at, float scale = 1f);
    public static void Mark(string label);
}
```

| Member | Meaning |
|---|---|
| `Sink` | The game's sink; null makes every verb a no-op. Set once at boot. |
| `Source` | The graft currently running, written by the slot dispatcher; `by` in telemetry. |

## IFxSink

**Namespace:** `Scion` · **Assembly:** `Scion.Runtime` · **Define:** none

```csharp
public interface IFxSink
{
    void Sound(string id, Vector3? at, float volume);
    void Shake(float amount);
    void Text(string text, Vector3 at, Color color, float seconds, int size, string icon);
    void Flash(Component target, Color color, float seconds);
    void HitStop(float seconds);
    void Vfx(string id, Vector3 at, float scale);
    void Mark(string label);
}
```

In test builds, Trials wraps the sink to record each call in `telemetry.jsonl`, then forwards it.

## See also

- [Effects with Fx](/scion/41664431a8f5/grafts/fx/)
- [Testing API](/scion/41664431a8f5/reference/api/testing/)
- [telemetry.jsonl](/scion/41664431a8f5/reference/files/telemetry-jsonl/)
