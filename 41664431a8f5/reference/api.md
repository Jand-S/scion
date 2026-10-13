---
title: "C# API"
description: "Scion's public C# API by assembly, namespace and scripting define, with one line per type and a link to its reference."
type: api
url: https://jand-s.github.io/scion/41664431a8f5/reference/api/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/reference/api.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/grafts/build-modes.md
  - https://jand-s.github.io/scion/41664431a8f5/contribute/architecture.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# C# API

Scion's public API is small and split by where it runs. `Scion.Runtime` has what every build needs (grafts, `Fx`,
faults); test-only types sit behind `SCION_TRIALS` or `SCION_SLOTS`; the editor and the out-of-Unity packages
(`Scion.Testing`, `Scion.Compare`, `Scion.Specs`, `Scion.Core`) have their own assemblies. Every public member has XML
documentation in the package; these pages give the signatures and the rules.

## Assemblies

| Assembly | Namespace | Define | Contains |
|---|---|---|---|
| `Scion.Runtime` | `Scion` | none | Grafts attributes, `Call`, `Anchors`, `Faults`, `Fx`, `IFxSink`, `WallClock`, `ImageTolerance` |
| `Scion.Runtime` | `Scion` | `SCION_TRIALS` | `Clock`, `VirtualScreen`, `VirtualWaitForSecondsRealtime`, `SeededRandom`, `Determinism`, `DeterminismControl` |
| `Scion.Injection` | `Scion` | `SCION_SLOTS` | `Injection`, `InjectReport`, `ScionPromotedAttribute` |
| `Scion.Trials` | `Scion.Trials` | `SCION_TRIALS` | Recipes, adapters, session, checks, static trials, captured matches, mosaic |
| `Scion.Control` | `Scion.Control` | `SCION_SLOTS` or `SCION_TRIALS` | The control server (no public API beyond the protocol) |
| `Scion.Editor` | `Scion.Editor` | editor | `ScionBuild`, `ScionBuildEntry` |
| `Unity.Scion.CodeGen` | | editor | The weaver (no public API) |
| `Scion.Core` (NuGet and package) | `Scion.Core` | none, no engine references | `RuleModifier<T>`, `Stage`, `TunableAttribute`, `ProbeAttribute`, `ISimulation` and capabilities |
| `Scion.Testing` (NuGet) | `Scion.Testing` | | `FxRecorder`, `FxCall`; `Scion.Runtime` for .NET |
| `Scion.Compare` (NuGet) | `Scion.Compare` | | `Image`, `Mask`, `ImageDiff`, `DiffStatus`, `DiffResult` |
| `Scion.Specs` (NuGet) | `Scion.Specs` | | `TrialSpec`, `SpecBuilder`, `Gates`, `Metamorphic` |

## Types

| Type | Page |
|---|---|
| `BeforeAttribute`, `AfterAttribute`, `AnchorAttribute`, `AnchorScopeAttribute`, `Call`, `Call<T>`, `Faults`, `Anchors`, `WallClockAttribute`, `ScionPromotedAttribute` | [Grafts API](/scion/41664431a8f5/reference/api/grafts/) |
| `Injection`, `InjectReport` | [Injection API](/scion/41664431a8f5/reference/api/injection/) |
| `Clock`, `VirtualScreen`, `VirtualWaitForSecondsRealtime`, `SeededRandom`, `Determinism`, `DeterminismControl`, `NetClock` | [Determinism API](/scion/41664431a8f5/reference/api/determinism/) |
| `TrialRecipe`, `VaryAttribute`, `TrialArgs`, `TrialModesAttribute`, `ITrialModeAware`, `CapturedMatch` | [Recipes API](/scion/41664431a8f5/reference/api/recipes/) |
| `ITrialAdapter`, `TrialAdapter<TRecipe>`, `TrialAdapterAttribute`, `StaticTrial`, `StaticTrial<TItem>`, `TrialView`, `IMatchCapture`, `MatchCaptureAttribute`, `IMosaicAdapter` | [Adapters API](/scion/41664431a8f5/reference/api/adapters/) |
| `TrialSession`, `TrialRole` | [Session API](/scion/41664431a8f5/reference/api/session/) |
| `TrialChecks`, `ITrialCheck`, `TrialCheckAttribute`, `CheckResult`, `CheckStatus`, `FrameView` | [Checks API](/scion/41664431a8f5/reference/api/checks/) |
| `Fx`, `IFxSink` | [Fx API](/scion/41664431a8f5/reference/api/fx/) |
| `FxRecorder`, `FxCall` | [Testing API](/scion/41664431a8f5/reference/api/testing/) |
| `ImageTolerance`, `Image`, `Mask`, `ImageDiff`, `DiffStatus`, `DiffResult` | [Compare API](/scion/41664431a8f5/reference/api/compare/) |
| `ScionBuild`, `ScionBuildEntry` | [Editor API](/scion/41664431a8f5/reference/api/editor/) |
| `TrialSpec`, `SpecBuilder`, `Gates`, `Metamorphic` | [Specs API](/scion/41664431a8f5/reference/api/specs/) |
| `RuleModifier<T>`, `Stage`, `TunableAttribute`, `ProbeAttribute`, `ISimulation`, `SimulationAttribute`, `IBot`, `BotAttribute`, `ISaveable`, `IInspectable`, `IMirrorable`, `IPermutable`, `Tunables` | [Core API](/scion/41664431a8f5/reference/api/core/) |

## Conventions

- In call mode nothing allocates and nothing uses reflection.
- Coordinates in C# have the origin at the bottom left, like `Screen`; JSON and PNG use the top left.
- Session and check methods run on the main thread.

## See also

- [Build modes](/scion/41664431a8f5/grafts/build-modes/)
- [Architecture](/scion/41664431a8f5/contribute/architecture/)
- [Versioning and compatibility](/scion/41664431a8f5/contribute/compatibility/)
