---
title: "build test, build final"
description: "Build the Mono test player with slots and trials, or the final player without test code, in batchmode or through the open editor."
type: cli
url: https://jand-s.github.io/scion/41664431a8f5/cli/build/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/cli/build.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/grafts/build-modes.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/files/scion-build-json.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/api/editor.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# build test, build final

`dotnet scion build test` builds the test player (Mono, `SCION_SLOTS` and `SCION_TRIALS`) and writes
`scion-build.json` next to it; `dotnet scion build final` builds the shipping player without test defines. The CLI
runs Unity in batchmode with `build.test.method` or `build.final.method` from `scion.json`, or, when the editor is
open and `com.unity.pipeline` is installed, builds through the open editor. Compile errors come back with file,
line and code.

## Synopsis

```text
dotnet scion build test  [--out <dir>] [--dry-run]
dotnet scion build final [--out <dir>] [--dry-run]
```

## Description

1. Finds Unity and checks the editor lock.
2. Closed editor: runs `Unity -batchmode -quit -projectPath <unity_project> -executeMethod <method> -scion-build-out <dir>`.
   Open editor with the pipeline package: writes `Temp/Scion/ScionBuildEntry.cs` and runs it in the editor.
3. Reads the `BUILD_RESULT` line and the log, and collects compile errors and SCN diagnostics.
4. On success, reads `scion-build.json` and prints the build id.

The output folder defaults to `build.test.output.<os>` (or `build.final.output.<os>`) in `scion.json`.

## build test

Builds with `ScionBuild.TestFromCommandLine` (or the method in `build.test.method`): Mono backend, `SCION_SLOTS` and
`SCION_TRIALS` added for the build only, `scion-build.json` written with the slots and recipe names. Trials, warm
copies, injection and the multiverse mosaic all need this build.

## build final

Builds with `ScionBuild.FinalFromCommandLine` (or `build.final.method`): the project's backend (IL2CPP by default),
no test defines, grafts called directly. It fails with SCN121 if a test define is set anywhere in the build.

## Options

| Option | Default | Meaning |
|---|---|---|
| `--out <dir>` | from `scion.json` | Output folder. |
| `--dry-run` | off | Print the program and arguments; build nothing. |

## Output

<!-- validate: cli/build -->
```json
{"schema_version":1,"command":"build test","ok":false,"channel":"batchmode","seconds":21.4,"log":".scion/logs/build-20261012-130000.log","build_id":null,"output":".scion/builds/test/Orchard.app","errors":[{"file":"Assets/Orchard/Grafts/CritShake.cs","line":18,"code":"SCN104","message":"The signature of CritShake.After does not match HitPresenter.Play.","hint":"Expected: public static void After(HitPresenter presenter, HitEvent e, ref Call<float> c)","did_you_mean":[]}],"errors_total":1,"next_step":"fix Assets/Orchard/Grafts/CritShake.cs:18, then: dotnet scion build test"}
```

| Field | Meaning |
|---|---|
| `channel` | `batchmode` or `editor`. |
| `seconds` | Build time. |
| `build_id` | 12 hex, or null on failure. |
| `output` | The player path. |
| `errors[]` | `file`, `line`, `code` (`CS0103`, `SCN104`...), `message`, `hint`. |
| `errors_total` | All errors, when the list is cut. |
| `program`, `args` | With `--dry-run`. |

## Exit codes

| Code | When |
|---|---|
| 0 | Built. |
| 1 | Compile errors or a build guard (SCN120, SCN121). |
| 2 | Bad option. |
| 3 | No Unity, editor open without the pipeline, Unity crashed. |

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| `editor_open` | 3 | The editor is open and `com.unity.pipeline` is missing. | Close the editor, or install the pipeline package. |
| `unity_not_found` | 3 | No Unity of the project's version. | Set `unity.editor` in `scion.local.json`. |
| `unity_failed` | 3 | Unity exited without a `BUILD_RESULT` line. | Read the log in `log`. |
| warning `unity_exit` | 0 | Unity exited non-zero after a successful build. | Usually harmless; read the log. |
| `SCN120`, `SCN121` | 1 | Build guards. | See [Build modes](/scion/grafts/build-modes/). |

## Examples

### Show the Unity command

```sh
dotnet scion build test --dry-run
```

### A final build for a store

```sh
dotnet scion build final --out /tmp/orchard-release
```

## See also

- [Build modes](/scion/grafts/build-modes/)
- [scion-build.json](/scion/reference/files/scion-build-json/)
- [Editor API](/scion/reference/api/editor/)
