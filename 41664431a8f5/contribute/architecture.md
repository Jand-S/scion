---
title: "Architecture"
description: "How Scion is built - runtime, editor, weaver and tools, the data flow from build to verdict, the generated ABI, and how the package is tested."
type: concept
url: https://jand-s.github.io/scion/41664431a8f5/contribute/architecture/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/contribute/architecture.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/contribute/compatibility.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/api.md
  - https://jand-s.github.io/scion/41664431a8f5/grafts/overview.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Architecture

This page is for contributors and for anyone who needs to know how Scion works inside: which assemblies exist and
what each does, how data moves from a build to a verdict, what the weaver generates (the ABI that injection depends
on), and how the package is tested. For using Scion, start at [What is Scion](/scion/41664431a8f5/start/overview/).

## How it works

### Components

Scion has code Unity compiles into the game (runtime), code that runs in the editor at build time, and .NET tools
outside Unity. Folders ending in `~` (`Tools~`, `Tests~`, `Documentation~`) travel with the package and are ignored by
Unity's importer.

| Component | Assembly | Role |
|---|---|---|
| Runtime | `Scion.Runtime` | Graft attributes, `Call`, `Fx`, `Faults`, `Anchors`; under `SCION_TRIALS` the virtual clock, screen and random. |
| Injection | `Scion.Injection` | Slots, `Injection`, `InjectReport` (`SCION_SLOTS`). |
| Trials | `Scion.Trials` | The session, recipes, adapters, checks, capture, static trials, co-op, mosaic (`SCION_TRIALS`). Its pure logic (flags, shards, writers, checks) has no `UnityEngine` and is tested with `dotnet test`. |
| Control | `Scion.Control` | The control server for warm copies and live injection. |
| Core | `Scion.Core` | The engine-free rules API: `RuleModifier<T>`, `ISimulation`, tunables, probes. |
| Editor | `Scion.Editor` | `ScionBuild`, build guards, `scion-build.json`. |
| Weaver | `Unity.Scion.CodeGen` | The `ILPostProcessor`. Only `ScionWeaver.cs` touches Unity's ILPP API; the model, resolver and emitters are plain Mono.Cecil, shared with the CLI. |
| CLI | `Scion.Cli` (dotnet tool) | Every command; reads builds with Mono.Cecil and never loads them. |
| Analyzers | `Scion.Analyzers` | Roslyn analyzers with the weaver's ids and texts; loaded by Unity through the `RoslynAnalyzer` label. |
| Compare | `Scion.Compare` | Image comparison, masks, heatmaps, its own PNG reader and writer. |
| Testing | `Scion.Testing` | `FxRecorder` and `Scion.Runtime` for .NET. |
| Specs | `Scion.Specs` | Specs, gates ids, metamorphic properties. |

### Data flow

1. Unity compiles the game; the weaver rewrites anchors (call mode or slot mode) and writes the determinism scan.
2. `build test` produces the player and `scion-build.json` (build id, slots, recipes).
3. `trial run` starts copies with `-scion-*` flags; each `TrialSession` plays its shard and writes `q<i>/`.
4. The CLI merges copies, compares with `scion/baselines/` using `Scion.Compare`, and writes `summary.json`.
5. `graft build` compiles drafts with Roslyn against the build's assemblies and the same emitters as the weaver;
   `graft inject` sends the DLL over the control protocol; `Injection` swaps slots at the next boundary.

### The generated ABI

Injection depends on names the weaver generates. They are fixed by `Anchors`:

| Name | Meaning |
|---|---|
| `__ScionGenerated` | The generated type in each woven assembly. |
| `D_<id>` | The dispatcher of an anchor side (call mode). |
| `F_<id>` | The slot field of an anchor side (slot mode). |
| `__Restore` | Restores every slot to the promoted grafts. |
| `AbiVersion` | 1; a constant in `Anchors` and in every `__ScionGenerated`. A DLL of another ABI is refused (SCN131). |
| `Anchors.Id` | The id format shared by the weaver, the CLI and the injection. |

### Diagnostics in one place

`DiagnosticIds.cs` is the single catalog of SCN ids, severities, messages and hints. The weaver and the analyzers
compile it; the runtime and the CLI repeat only the id, and a parity test checks every id used exists.

### Tests

```sh
cd Packages/me.jandson.scion/Tests~
dotnet test Scion.Tests.slnx
```

| Project | Covers |
|---|---|
| Weaver tests | Weaving real assemblies with Cecil, call and slot mode, diagnostics, redirects. |
| Analyzer tests | Each SCN analyzer with positive and negative cases. |
| Trials tests | Flags, shards, filters, writers, checks, adapter choice, without Unity. |
| CLI tests | Commands against fake builds and run folders; envelopes validated against the JSON Schemas. |
| Compare tests | Metric, masks, learned masks, heatmaps. |

Play-mode tests in Unity cover capture, offscreen rendering and the control server.

## Key terms

| Term | Meaning |
|---|---|
| ILPP | `ILPostProcessor`: Unity's hook to rewrite compiled assemblies. |
| Call mode, slot mode | How anchors reach grafts in final and test builds. |
| ABI | The generated names and `AbiVersion` that injection relies on. |
| Pure logic | Code without `UnityEngine`, compiled twice: by Unity and by `dotnet test`. |

## Guarantees and limits

- The weaver never weaves assemblies named `Scion.*` or `Unity.Scion.*`.
- Redirect targets (`Clock`, `VirtualScreen`, `SeededRandom`, `VirtualWaitForSecondsRealtime`) stay in `Scion.Runtime`
  forever, because assemblies such as Unity UI do not reference other Scion assemblies.
- Renaming a redirect target is never silent: SCN114 and a test fail.
- The CLI never loads game DLLs; it reads them with Cecil.

## See also

- [Versioning and compatibility](/scion/41664431a8f5/contribute/compatibility/)
- [C# API](/scion/41664431a8f5/reference/api/)
- [How grafts work](/scion/41664431a8f5/grafts/overview/)
