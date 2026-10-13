---
title: "SCN120-SCN131"
description: "Build guard and injection diagnostics - slots on IL2CPP, test defines in a final build, and graft DLLs built for another build or ABI."
type: diagnostics
url: https://jand-s.github.io/scion/41664431a8f5/reference/diagnostics/builds-and-injection/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/reference/diagnostics/builds-and-injection.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/grafts/build-modes.md
  - https://jand-s.github.io/scion/41664431a8f5/grafts/injection.md
  - https://jand-s.github.io/scion/41664431a8f5/contribute/compatibility.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# SCN120-SCN131

Build guards stop a player build that would not work or would ship lab code; injection checks refuse a graft DLL
that does not match the running build. They appear in `build test`, `build final`, `graft inject`,
`InjectReport.Errors` and `trial run --grafts`.

## Diagnostics

### SCN120

**Severity:** error · **Source:** build

**Message:** `SCION_SLOTS needs a JIT backend (Mono); this build uses IL2CPP.` **Hint:** `Build tests with
ScionBuild.BuildTestPlayer (forces Mono), or remove SCION_SLOTS.`

**Cause.** A player build with `SCION_SLOTS` on IL2CPP. Slots are replaced at run time, which needs a JIT.

**Fix.** Use `dotnet scion build test` or `ScionBuild.BuildTestPlayer`.

**Suppress.** Not allowed.

### SCN121

**Severity:** error · **Source:** build

**Message:** `Final build with test defines: {0}.` **Hint:** `Remove SCION_SLOTS and SCION_TRIALS from the build, or
use ScionBuild.BuildTestPlayer.`

**Cause.** `BuildFinalPlayer` (or `build final`) found `SCION_SLOTS` or `SCION_TRIALS` in the build's defines.

**Fix.** Remove them from Player Settings or the build profile; test builds add them for the build only.

**Suppress.** Not allowed.

### SCN130

**Severity:** error · **Source:** runtime, CLI

**Message:** `Graft DLL targets build {0}, but this player is build {1}.` **Hint:** `Run dotnet scion graft build
against this build, then inject again.`

**Cause.** The DLL was compiled against another test build (another `build_id`), so its references may not match.

**Fix.** `dotnet scion graft build` against the running build, then inject.

**Suppress.** Not allowed.

### SCN131

**Severity:** error · **Source:** runtime, CLI

**Message:** `Graft DLL has ABI {0}; this player expects ABI {1}.` **Hint:** `Use the same Scion version for the CLI
and the build.`

**Cause.** The CLI that built the DLL and the package in the build have different weaver ABIs (`Anchors.AbiVersion`).

**Fix.** Install the same Scion version for the package and the CLI; rebuild.

**Suppress.** Not allowed.

## See also

- [Build modes](/scion/grafts/build-modes/)
- [Live injection](/scion/grafts/injection/)
- [Versioning and compatibility](/scion/contribute/compatibility/)
- [Diagnostics](/scion/reference/diagnostics/)
