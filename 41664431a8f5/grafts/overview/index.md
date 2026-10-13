---
title: "How grafts work"
description: "Compile-time IL weaving with an ILPostProcessor, anchors, call mode and slot mode, and why grafts cost nothing in the final build."
type: concept
url: https://jand-s.github.io/scion/41664431a8f5/grafts/overview/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/grafts/overview.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/grafts/write.md
  - https://jand-s.github.io/scion/41664431a8f5/grafts/build-modes.md
  - https://jand-s.github.io/scion/41664431a8f5/grafts/injection.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# How grafts work

A graft is a `static void` C# method marked `[Before]` or `[After]` that Scion's weaver hangs onto an existing game
method, its **anchor**, without editing the anchor's source. This page explains the mechanism: the weaver is a Unity
`ILPostProcessor` built on Mono.Cecil, it rewrites the IL of anchors when Unity compiles, final builds call grafts
directly (call mode) and test builds route them through swappable slots (slot mode, `SCION_SLOTS`) so new grafts can
be injected into a running game.

## How it works

![Diagram: graft source files compile into the game assembly; the weaver rewrites each anchor; the final build calls a generated dispatcher directly; the test build calls a swappable slot that injection can replace.](../../../assets/diagrams/weaver-flow.svg)

### The weaver

`Unity.Scion.CodeGen` is an `ILPostProcessor`. After Unity compiles an assembly that references `Scion.Runtime`, the
weaver:

1. finds every method marked `[Before]` or `[After]` in that assembly;
2. resolves each target method (exact signature first, then a unique prefix match);
3. checks the rules (shape, eligibility, `Call` usage) and reports violations as compile errors with file and line;
4. rewrites each anchor's IL so it calls a generated dispatcher at its start (befores) and at every return (afters).

The weaver only rewrites the assembly it is compiling. A graft must therefore compile **into the anchor's own
assembly**: you keep grafts in their own folder and point it at the game assembly with an `.asmref`
([SCN102](/scion/reference/diagnostics/grafts/#scn102)).

### Two modes

| | Call mode (final build) | Slot mode (test build) |
|---|---|---|
| Define | none | `SCION_SLOTS` |
| Backend | IL2CPP or Mono | Mono (needs a JIT) |
| How an anchor reaches its grafts | A direct static call to `__ScionGenerated.D_<id>`, which calls the grafts in order | A static delegate field per anchor (`__ScionGenerated.F_<id>`), filled with the promoted grafts at startup |
| Run-time cost | none measurable; the dispatcher is inlined | about 1 to 3 ns per call |
| Registration, reflection, allocation | none | none per call |
| Injection | not available | `Injection.Inject` swaps slots at the next recipe boundary |
| A graft that throws | propagates like any other code | is caught, disabled alone and reported to `Faults` |

In slot mode, `[assembly: AnchorScope(typeof(T))]` gives methods a slot even when no graft targets them yet, so a
new graft can be injected there without a rebuild ([Anchors](/scion/grafts/anchors/)).

### Why compile time

Run-time patching (Harmony, MonoMod, detours) does not work under IL2CPP and hides errors until the code runs.
Scion's weaver gives:

- direct calls in the shipped build, which IL2CPP inlines;
- compile errors with file, line and a fix (`Did you mean 'Play'?`), also live in the IDE through the analyzers;
- one dispatcher per anchor with a fixed order, so the order never depends on load order.

The per-call context `ref Call<T>` follows the model of Mixin's `CallbackInfoReturnable`: grafts can skip the
anchor, return a value or change its result ([Call and Call\<T\>](/scion/grafts/call-context/)).

### Grafts are visual

The weaver does not run in plain `dotnet test`, in the rules simulator or anywhere outside Unity. A graft that
changed a rule would work in the game and vanish everywhere else, and co-op peers would disagree. So every graft is
presentation: effects, sound, camera, HUD, telemetry. Rules change in the core through `RuleModifier<T>`
([Grafts are visual only](/scion/grafts/visual-only/)).

## Key terms

| Term | Meaning |
|---|---|
| Graft | A `static void` method with `[Before]` or `[After]`, public or internal, non-generic. |
| Anchor | A method the weaver rewrote. Every target of a graft is one; `AnchorScope` adds more in test builds. |
| Anchor id | `B__` or `A__` plus the type, method and parameter type names, for example `A__Orchard_Presentation_HitPresenter__Play__HitEvent`. |
| Dispatcher | The generated method that calls an anchor's grafts in order (`__ScionGenerated.D_<id>`). |
| Slot | In test builds, the delegate field that holds an anchor's dispatcher; injection replaces it. |
| Promoted graft | A graft compiled into the game (`grafts.promoted` folder). |
| Draft graft | A graft in `grafts.staging` (`scion/grafts/`), compiled to a DLL and injected; it enters the game when promoted. |
| Observer | A `[Before(..., Observe = true)]` graft: always runs, cannot skip. |

## Guarantees and limits

- Removing the grafts folder leaves a game that compiles and plays as before: nothing may depend on a graft
  ([SCN110](/scion/reference/diagnostics/grafts/#scn110)).
- Order within one call is fixed: observers, then other befores by `Priority`, then the body, then afters by
  `Priority`; ties break on the graft's full name.
- The final build contains no slots, no injection and no trials code; a final build with test defines fails
  ([SCN121](/scion/reference/diagnostics/builds-and-injection/#scn121)).
- Some methods cannot be anchors: iterators, `async` methods, Netcode RPCs, struct methods, generic methods, methods
  with `out` parameters and a few more ([SCN105](/scion/reference/diagnostics/grafts/#scn105)). Hang the graft on the
  plain method they call.
- Injected grafts must be `public` in a `public` type: they live in another assembly.
- Mono never unloads assemblies; a warm copy asks to be recycled after 50 injections (`InjectReport.Recycle`).

## See also

- [Write a graft](/scion/grafts/write/)
- [Build modes](/scion/grafts/build-modes/)
- [Live injection](/scion/grafts/injection/)
- [Architecture](/scion/contribute/architecture/)
