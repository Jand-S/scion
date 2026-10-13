---
title: "Live injection"
description: "Compile draft grafts against a test build and swap them into running copies at the next recipe boundary, then restore or disable slots."
type: guide
url: https://jand-s.github.io/scion/41664431a8f5/grafts/injection/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/grafts/injection.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/grafts/lifecycle.md
  - https://jand-s.github.io/scion/41664431a8f5/trials/warm-copies.md
  - https://jand-s.github.io/scion/41664431a8f5/cli/graft.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/api/injection.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Live injection

Injection loads a graft DLL compiled outside Unity into a running Mono test build and swaps the slots of the anchors
it targets. This guide covers `dotnet scion graft build` (compiles drafts against the build's `Managed/*.dll`),
`graft inject --to all|<pid> [--isolate]`, `graft restore`, `graft disable <slot>`, the pending state until the next
recipe boundary, `Injection.Version`, `InjectReport`, and `trial run --grafts <dll>` for injection at startup.
Errors: SCN113, SCN130, SCN131.

## When to use

- Use live injection to iterate on a draft graft in seconds, without a Unity rebuild.
- Use `trial run --grafts` to compare recipes with and without a draft in a cold run.
- Promote and rebuild when the draft is done: runs with injected grafts cannot be approved as baselines.

## Before you begin

- A test build (`dotnet scion build test`).
- Running copies with a control channel: warm copies (`dotnet scion trial serve -j 2`) or a game you started with
  `-scion-control 0`.
- Drafts in `scion/grafts/` (`grafts.staging` in `scion.json`).

## Steps

### 1. Compile the drafts against the build

```sh
dotnet scion graft build
```

<!-- validate: cli/graft-build -->
```json
{"schema_version":1,"command":"graft build","ok":true,"dll":".scion/grafts/ScionGrafts_51c2.dll","build_id":"4c49a1e0b2f3","abi":1,"grafts":["Orchard.Grafts.CritShake.After"],"seconds":1.9,"diagnostics":[],"next_step":"dotnet scion graft inject --to all"}
```

The build compiles every file in `scion/grafts/` with Roslyn and the test defines, references the build's
`Managed/*.dll`, and generates the dispatcher code the weaver would. The DLL records the target `build_id` and the
weaver `AbiVersion`.

### 2. Inject it into the running copies

```sh
dotnet scion graft inject --to all
```

<!-- validate: cli/graft-inject -->
```json
{"schema_version":1,"command":"graft inject","ok":true,"dll":".scion/grafts/ScionGrafts_51c2.dll","copies":[{"pid":4242,"version":3,"ok":true,"pending":false,"bound":["A__Orchard_Presentation_HitPresenter__Play__HitEvent"],"errors":[],"recycle":false},{"pid":4243,"version":3,"ok":true,"pending":true,"bound":["A__Orchard_Presentation_HitPresenter__Play__HitEvent"],"errors":[],"recycle":false}],"next_step":"dotnet scion trial run --warm SlashHit"}
```

A copy in the middle of a recipe answers `pending: true` and applies the swap at the next recipe boundary, so one
recipe never mixes two versions of a graft.

### 3. Run and look

```sh
dotnet scion trial run --warm SlashHit
```

### 4. Undo

```sh
dotnet scion graft restore               # every slot back to the promoted grafts
dotnet scion graft disable A__Orchard_Presentation_HitPresenter__Play__HitEvent   # empty one slot
```

## Reference

### Merge or isolate

| Mode | What one anchor runs after the injection |
|---|---|
| Default (merge) | The promoted grafts plus the injected ones. An injected graft with the same identity (`Orchard.Grafts.CritShake.After`) replaces the promoted one. Order: observers, `Priority`, full name. |
| `--isolate` | Only the injected grafts of the DLL, for the anchors the DLL touches. |

### Commands

| Command | Use |
|---|---|
| `graft build [files]` | Compile drafts (all of `scion/grafts/` by default) into `.scion/grafts/ScionGrafts_<hash>.dll`. |
| `graft inject [--to all\|<pid>] [--isolate] [--dll <file>]` | Send the DLL to copies (default `all` live copies of this build). Builds first when the drafts changed. |
| `graft restore [--to ...]` | Every slot back to the promoted grafts. |
| `graft disable <slot> [--to ...]` | Empty one slot, promoted grafts included. |
| `trial run --grafts <dll>` | Inject at startup into cold copies (`-scion-inject`). |

### From code

Inside `#if SCION_SLOTS`:

```csharp
InjectReport r = Injection.Inject("/Users/alex/orchard/.scion/grafts/ScionGrafts_51c2.dll");
if (!r.Ok) foreach (var e in r.Errors) Debug.LogWarning(e);   // "SCN113: ..."
Injection.Applied += report => Debug.Log(report.ToJson());
Injection.Restore();
Injection.Disable("A__Orchard_Presentation_HitPresenter__Play__HitEvent");
```

`Injection.Version` counts applied loads; it is the `grafts` column of `results.tsv`, so a run shows which recipes ran
with injected code.

### Rules

- An injected graft must be `public`, in a `public` type: it lives in another assembly.
- Its target must have a slot in this build: a promoted graft on it, or an `AnchorScope` that covers it (SCN113).
- A DLL built for another `build_id` is refused (SCN130); a DLL of another ABI is refused (SCN131).
- Mono never unloads assemblies. After 50 injections `InjectReport.Recycle` is true and `trial serve` restarts that
  copy at the next boundary.
- A run with injected grafts cannot be approved as a baseline: promote the graft and rebuild.

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| `SCN113` | 1 | The target has no slot in this build. | Add its type to `AnchorScope`, rebuild, inject again. |
| `SCN130` | 1 | The DLL targets another build. | `dotnet scion graft build` against this build. |
| `SCN131` | 1 | The DLL has another weaver ABI. | Use the same Scion version for the CLI and the build. |
| `no_copies` | 3 | No live copy of this build answers. | `dotnet scion trial serve -j 2`. |
| `not_supported` | 3 | The copy was built without `SCION_SLOTS`. | Use a test build. |

## Examples

### Isolate one draft in one copy

```sh
dotnet scion graft inject --to 4242 --isolate
```

### Compare with and without a draft in cold runs

```sh
dotnet scion trial run SlashHit --rounds 2
dotnet scion trial run SlashHit --rounds 2 --grafts .scion/grafts/ScionGrafts_51c2.dll
dotnet scion trial diff 20261011-143200 20261011-143512 SlashHit
```

## See also

- [graft (CLI)](/scion/41664431a8f5/cli/graft/)
- [Warm copies](/scion/41664431a8f5/trials/warm-copies/)
- [Injection API](/scion/41664431a8f5/reference/api/injection/)
- [InjectReport JSON](/scion/41664431a8f5/reference/files/inject-report/)
- [Control protocol](/scion/41664431a8f5/reference/control-protocol/)
