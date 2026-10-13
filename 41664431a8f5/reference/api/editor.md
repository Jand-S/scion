---
title: "Editor API"
description: "ScionBuild entry points for test and final players from scripts, batchmode and the open editor, and the ScionBuildEntry stub the CLI uses."
type: api
url: https://jand-s.github.io/scion/41664431a8f5/reference/api/editor/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/reference/api/editor.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/grafts/build-modes.md
  - https://jand-s.github.io/scion/41664431a8f5/cli/build.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Editor API

`ScionBuild` builds players the way the CLI does: test players on Mono with the test defines, final players without
them. It works from your editor scripts, from batchmode and through the open editor.

## ScionBuild

**Namespace:** `Scion.Editor` · **Assembly:** `Scion.Editor` · **Define:** editor only

```csharp
public static class ScionBuild
{
    public static readonly string[] TestDefines = { "SCION_SLOTS", "SCION_TRIALS" };
    public static BuildReport BuildTestPlayer(BuildPlayerOptions o);
    public static BuildReport BuildFinalPlayer(BuildPlayerOptions o);
    public static void TestFromCommandLine();
    public static void FinalFromCommandLine();
    public static string FromEditor(string mode, string outDir);
}
```

| Member | Meaning |
|---|---|
| `BuildTestPlayer` | Forces Mono, adds `TestDefines` for the build, restores backend and defines in a `finally`, writes `scion-build.json`. |
| `BuildFinalPlayer` | Builds with the project's backend; fails with SCN121 when a test define is set. |
| `TestFromCommandLine`, `FinalFromCommandLine` | Batchmode: `-scion-build-out <dir> [-scion-build-target osx\|win\|linux]`; exits 0 or 1 and prints `BUILD_RESULT`. |
| `FromEditor` | `mode` is `test` or `final`; returns the JSON of `build`. Used through the open editor. |

## ScionBuildEntry

**Namespace:** global · **Assembly:** written by the CLI into `Temp/Scion/` · **Define:** editor only

```csharp
public static class ScionBuildEntry
{
    public static string Run(string mode, string outDir) => Scion.Editor.ScionBuild.FromEditor(mode, outDir);
}
```

The CLI writes this stub to `Temp/Scion/ScionBuildEntry.cs` and runs it in the open editor through the
`com.unity.pipeline` command line. You never write it.

## See also

- [Build modes](/scion/41664431a8f5/grafts/build-modes/)
- [build test, build final](/scion/41664431a8f5/cli/build/)
- [scion-build.json](/scion/41664431a8f5/reference/files/scion-build-json/)
