---
title: "Effects with Fx"
description: "Call sound, shake, text, flash, hit-stop, VFX and marks from grafts through the Fx facade, register the game's sink, and read the calls in telemetry."
type: guide
url: https://jand-s.github.io/scion/41664431a8f5/grafts/fx/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/grafts/fx.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/reference/api/fx.md
  - https://jand-s.github.io/scion/41664431a8f5/grafts/testing.md
  - https://jand-s.github.io/scion/41664431a8f5/evaluation/telemetry.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Effects with Fx

`Fx` is the facade grafts use for presentation: `Fx.Sound`, `Fx.Shake`, `Fx.Text`, `Fx.Flash`, `Fx.HitStop`,
`Fx.Vfx` and `Fx.Mark`. The game implements them once in an `IFxSink` registered at boot; Trials records every call
in `telemetry.jsonl` with the graft that made it (`by`); tests replace the sink with `FxRecorder`. Grafts never
reach the game's audio, camera or HUD systems directly, so they stay visual, testable and portable.

## When to use

- Use `Fx` for any effect a graft triggers.
- Call the game's own presentation code from a graft only for things `Fx` does not cover, and keep it visual.

## Steps

### 1. Implement the sink

```csharp
using Scion;
using UnityEngine;

namespace Orchard.Presentation
{
    public sealed class OrchardFxSink : IFxSink
    {
        public void Sound(string id, Vector3? at, float volume) => AudioBank.Play(id, at, volume);
        public void Shake(float amount) => CameraRig.Current.Shake(amount);
        public void Text(string text, Vector3 at, Color color, float seconds, int size, string icon) =>
            FloatingText.Spawn(text, at, color, seconds, size, icon);
        public void Flash(Component target, Color color, float seconds) => SpriteFlash.On(target, color, seconds);
        public void HitStop(float seconds) => TimeFreeze.For(seconds);
        public void Vfx(string id, Vector3 at, float scale) => VfxPool.Play(id, at, scale);
        public void Mark(string label) { }
    }
}
```

### 2. Register it at boot

```csharp
[RuntimeInitializeOnLoadMethod(RuntimeInitializeLoadType.BeforeSceneLoad)]
static void InstallFx() => Fx.Sink = new OrchardFxSink();
```

### 3. Call Fx from grafts

```csharp
[After(typeof(HitPresenter), nameof(HitPresenter.Play))]
public static void After(HitPresenter presenter, HitEvent e)
{
    Fx.Sound(e.Crit ? "hit_crit" : "hit", presenter.transform.position);
    Fx.Text(e.Amount.ToString(), presenter.TargetPosition, e.Crit ? Color.yellow : Color.white);
}
```

## Reference

### Verbs

| Verb | Signature | Meaning |
|---|---|---|
| `Sound` | `(string id, Vector3? at = null, float volume = 1f)` | Play a sound by id, optionally positioned. |
| `Shake` | `(float amount)` | Shake the camera, 0 to 1. |
| `Text` | `(string text, Vector3 at, Color color, float seconds = 1.1f, int size = 12, string icon = null)` | Floating text. |
| `Flash` | `(Component target, Color color, float seconds)` | Tint a target briefly. |
| `HitStop` | `(float seconds)` | Freeze presentation briefly. |
| `Vfx` | `(string id, Vector3 at, float scale = 1f)` | Spawn an effect by id. |
| `Mark` | `(string label)` | A mark in the capture timeline (`frames.tsv`, `IMPACT`). |

`IFxSink` has one method per verb with the same parameters and no defaults. With no sink, every verb does nothing.

### What Trials records

In test builds Trials decorates the sink. Every call writes one line to `telemetry.jsonl` and then calls your sink:

```json
{"f":14,"t":1.467,"kind":"fx","verb":"Shake","args":[0.3],"by":"Orchard.Grafts.CritShake.After"}
```

`by` is `Fx.Source`, written by the slot dispatcher around each graft; calls from game code have `"by": null`.
`Fx.Mark` also adds the label to the frame's `marks` column, like `TrialSession.Mark`.

### Rules for the sink

- Keep it thin: map the verb to the game's system and return.
- Do not change rule state in it: it runs from grafts, which are visual.
- Use ids, not asset references, so recipes, telemetry and tests can name the effect.

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| none from `Fx` | | A missing sink makes every verb a no-op. | Register `Fx.Sink` at boot; check with `Fx.Sink != null`. |

## Examples

### A heal number

```csharp
[After(typeof(DamageNumbers), nameof(DamageNumbers.SpawnHeal))]
public static void After(DamageNumbers numbers, int amount, Vector3 at) =>
    Fx.Text("+" + amount, at, new Color(0.4f, 1f, 0.5f), size: 14, icon: "leaf");
```

### Assert an effect in a recipe

```csharp
protected override void Checks(TrialChecks c) => c.Marked("IMPACT").ProbeIs("shake_calls", v => v >= 1, "crit shakes");
```

## See also

- [Fx API](/scion/reference/api/fx/)
- [Test grafts without Unity](/scion/grafts/testing/)
- [Telemetry, probes and sensors](/scion/evaluation/telemetry/)
