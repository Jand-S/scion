---
title: "Build modes"
description: "Build the test player in slot mode on Mono and the final player in call mode, with the scripting defines, ScionBuild entry points and build guards."
type: guide
url: https://jand-s.github.io/scion/41664431a8f5/grafts/build-modes/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/grafts/build-modes.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/grafts/overview.md
  - https://jand-s.github.io/scion/41664431a8f5/cli/build.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/api/editor.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/files/scion-build-json.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Build modes

Scion has two kinds of player build. The **test build** (Mono, `SCION_SLOTS` and `SCION_TRIALS`) routes grafts
through slots, accepts injection and runs trials. The **final build** (IL2CPP or Mono, no test defines) calls grafts
directly and contains no lab code. This guide covers the defines, `dotnet scion build test|final`, the editor API
`ScionBuild`, Build Profiles, the guards SCN120 and SCN121, and the `scion-build.json` written next to every build.

## When to use

- Use the test build for everything Scion does at run time: trials, live injection, warm copies, the multiverse.
- Use the final build for players, stores and performance measurements. It is what you ship.
- Rebuild the test build after any change to game code, promoted grafts or `AnchorScope`; the CLI warns with
  `build_stale` when the build is older than the source.

## Steps

### 1. Build the test player

```sh
dotnet scion build test
```

<!-- validate: cli/build -->
```json
{"schema_version":1,"command":"build test","ok":true,"channel":"batchmode","seconds":35.2,"build_id":"4c49a1e0b2f3","output":".scion/builds/test/Orchard.app","errors":[],"next_step":"dotnet scion trial run"}
```

The CLI finds the Unity editor of the project and builds in batchmode. With the editor open and the
`com.unity.pipeline` package installed, it builds through the open editor instead; with the editor open and no
pipeline, it stops with [`editor_open`](/scion/reference/errors/#editor_open).

### 2. Build the final player

```sh
dotnet scion build final
```

The final build uses the project's backend (IL2CPP by default) and refuses any test define.

### 3. Build from your own editor script

```csharp
using Scion.Editor;
using UnityEditor;

public static class OrchardBuilds
{
    [MenuItem("Orchard/Build test player")]
    public static void Test()
    {
        ScionBuild.BuildTestPlayer(new BuildPlayerOptions
        {
            scenes = new[] { "Assets/Orchard/Scenes/Battle.unity" },
            locationPathName = ".scion/builds/test/Orchard.app",
            target = BuildTarget.StandaloneOSX,
        });
    }
}
```

`BuildTestPlayer` forces Mono, adds `ScionBuild.TestDefines` and restores the project's backend and defines in a
`finally`, so a failed build never leaves the project in test mode.

## Reference

### Defines

| Define | Set by | Effect |
|---|---|---|
| none | final build | Call mode: anchors call `__ScionGenerated.D_<id>` directly. No slots, no lab. |
| `SCION_SLOTS` | test build | Slot mode: one delegate field per anchor, `Injection`, the control server. Needs Mono. |
| `SCION_TRIALS` | test build | `Scion.Trials`, the virtual clock and screen, `SeededRandom`, captures. |
| `SCION_NETCLOCK` | you, optional | Moves the Netcode tick onto the virtual clock ([Netcode clock](/scion/determinism/netclock/)). |
| `SCION_VERBOSE` | you, optional | Extra `[Scion]` log lines from the weaver and the session. Off by default. |

Code that uses test-only types sits inside the matching define:

```csharp
#if SCION_TRIALS
TrialSession.Current?.Probe("damage", amount);
#endif
```

`Before`, `After`, `Call`, `Call<T>`, `Anchor`, `AnchorScope`, `Anchors`, `Faults`, `Fx` and `WallClock` exist in
every build. `Scion.Trials`, `Clock`, `VirtualScreen`, `SeededRandom` and `VirtualWaitForSecondsRealtime` need
`SCION_TRIALS`; `Injection` and `InjectReport` need `SCION_SLOTS`.

### Entry points

| Member | Use |
|---|---|
| `ScionBuild.BuildTestPlayer(BuildPlayerOptions)` | Test player from your script. |
| `ScionBuild.BuildFinalPlayer(BuildPlayerOptions)` | Final player; fails with SCN121 on test defines. |
| `ScionBuild.TestFromCommandLine()` | Batchmode: `-executeMethod Scion.Editor.ScionBuild.TestFromCommandLine -scion-build-out <dir> [-scion-build-target osx\|win\|linux]`. |
| `ScionBuild.FinalFromCommandLine()` | The same for the final build. |
| `ScionBuild.FromEditor(mode, outDir)` | Used by the CLI through the open editor; returns the JSON of `build`. |

Batchmode builds print one line the CLI reads: `BUILD_RESULT mono Succeeded errors=0 seconds=35.2`.

### Build Profiles

With Unity Build Profiles, keep one profile for the final build and let Scion build the test player: the test
defines are added only for the duration of `BuildTestPlayer`, never saved into a profile. The scenes come from the
active profile unless `BuildPlayerOptions.scenes` is set.

### What the build writes

| File | Content |
|---|---|
| `<out>/<player_name>.app` (or `.exe`) | The player. `player_name` comes from `build.player_name` in `ScionSettings.json`. |
| `<out>/scion-build.json` | Build id, git commit, Unity version, backend, defines, slots and recipe names ([scion-build.json](/scion/reference/files/scion-build-json/)). |

`build_id` is the first 12 hex digits of the SHA-256 of the main game assembly as built. Injection and baselines use
it to tell builds apart.

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| `SCN120` | 1 | `SCION_SLOTS` on an IL2CPP build. | Build tests with `ScionBuild.BuildTestPlayer` or `dotnet scion build test`. |
| `SCN121` | 1 | Final build with `SCION_SLOTS` or `SCION_TRIALS`. | Remove the test defines or use `BuildFinalPlayer`. |
| `editor_open` | 3 | The editor is open and `com.unity.pipeline` is not installed. | Close the editor or install the pipeline package. |
| `unity_not_found` | 3 | No Unity of the project's version in the Hub path. | Set `unity.editor` in `scion.local.json`. |
| `build_wrong_mode` | 3 | A trials command found a build without `SCION_TRIALS`. | `dotnet scion build test`. |
| `build_stale` | warning | Source changed since the build. | Rebuild before trusting the run. |

## Examples

### Build into another folder

```sh
dotnet scion build test --out /tmp/orchard-test
dotnet scion trial run --build /tmp/orchard-test/Orchard.app
```

### Build in CI without the CLI

```sh
"$UNITY" -batchmode -quit -projectPath . -executeMethod Scion.Editor.ScionBuild.TestFromCommandLine -scion-build-out .scion/builds/test
```

## See also

- [build test, build final](/scion/cli/build/)
- [Editor API](/scion/reference/api/editor/)
- [scion-build.json](/scion/reference/files/scion-build-json/)
- [SCN120-SCN131](/scion/reference/diagnostics/builds-and-injection/)
