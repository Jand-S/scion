---
title: "Configuration"
description: "Every key of scion.json, scion.local.json and ScionSettings.json, the scripting defines and the environment variables, with types, defaults and meaning."
type: reference
url: https://jand-s.github.io/scion/41664431a8f5/reference/configuration/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/reference/configuration.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/reference/schemas.md
  - https://jand-s.github.io/scion/41664431a8f5/cli/index.md
  - https://jand-s.github.io/scion/41664431a8f5/grafts/build-modes.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Configuration

Scion reads three files. `scion.json` at the project root (versioned) configures the CLI: paths, defaults, player
arguments, builds, evaluators, specs and the judge. `scion.local.json` beside it (per machine, in `.gitignore`) has
the same shape and overrides it. `ProjectSettings/ScionSettings.json` (versioned) configures the weaver, the build
and the analyzers inside Unity. This page lists every key, plus scripting defines and environment variables.

## scion.json

```json
{"$schema":"https://jand-s.github.io/scion/schemas/v1/scion-config.schema.json","schema_version":1,"unity_project":".","grafts":{"promoted":"Assets/Orchard/Grafts","staging":"scion/grafts"},"trials":{"recipes_staging":"scion/recipes","out":".scion/runs","baselines":"scion/baselines","keep_runs":10,"defaults":{"copies":6,"rounds":1,"headless":true,"size":"1280x720","seed":1,"timeout_s":300},"player_args":[],"host_args":["-orchard-host"],"client_args":["-orchard-join","localhost"],"port_arg":"-orchard-port"},"rule_paths":["Assets/Orchard/Core","Assets/Orchard/Data"],"log_allow":[],"evaluators":[],"build":{"test":{"method":"Scion.Editor.ScionBuild.TestFromCommandLine","output":{"osx":".scion/builds/test/Orchard.app","win":".scion/builds/test/Orchard.exe","linux":".scion/builds/test/Orchard.x86_64"}},"final":{"method":"Scion.Editor.ScionBuild.FinalFromCommandLine","output":{"osx":".scion/builds/final/Orchard.app","win":".scion/builds/final/Orchard.exe","linux":".scion/builds/final/Orchard.x86_64"}}},"specs":{"project":"Trials/Specs/Orchard.Specs/Orchard.Specs.csproj"},"judge":{"dir":"scion/judge","eyes":"siglip2-so400m","cache":".scion/judge"}}
```

| Key | Type | Default | Meaning |
|---|---|---|---|
| `$schema` | string | | Ignored by the CLI; lets editors validate. |
| `schema_version` | int | 1 | Format version. |
| `unity_project` | path | `.` | The Unity project folder, relative to `scion.json`. |
| `grafts.promoted` | path | `Assets/<Game>/Grafts` | Promoted grafts (with the `.asmref`). |
| `grafts.staging` | path | `scion/grafts` | Draft grafts. |
| `trials.recipes_staging` | path | `scion/recipes` | Recipe sources and captured or found matches. |
| `trials.out` | path | `.scion/runs` | Run folders. |
| `trials.baselines` | path | `scion/baselines` | Approved baselines. |
| `trials.keep_runs` | int | 10 | Runs kept; older ones are deleted at the start of a run. |
| `trials.defaults.copies` | int | doctor's suggestion | Default `-j`. |
| `trials.defaults.rounds` | int | 1 | Default `--rounds`. |
| `trials.defaults.headless` | bool | true | False shows windows by default. |
| `trials.defaults.size` | `WxH` | `1280x720` | Default capture size. |
| `trials.defaults.seed` | int | 1 | Default run seed. |
| `trials.defaults.timeout_s` | int | 300 | Real seconds per copy. |
| `trials.player_args` | string[] | `[]` | Arguments for every player process. |
| `trials.host_args` | string[] | `[]` | Extra arguments for hosts and solo copies. |
| `trials.client_args` | string[] | `[]` | Extra arguments for co-op clients. |
| `trials.port_arg` | string or null | null | The game's flag that takes the group's port. |
| `rule_paths` | path[] | `[]` | Files whose content forms the rule digest. |
| `log_allow` | regex[] | `[]` | Errors that do not fail the `log` check, in every recipe. |
| `evaluators` | object[] | `[]` | `{name, command, kind (process\|agent), gate (false)}`. |
| `build.test.method`, `build.final.method` | string | `Scion.Editor.ScionBuild.TestFromCommandLine`, `...FinalFromCommandLine` | Batchmode entry points. |
| `build.test.output.<os>`, `build.final.output.<os>` | path | `.scion/builds/test\|final/<Name>` | Build outputs per OS (`osx`, `win`, `linux`). |
| `specs.project` | path or null | null | The specs project. |
| `judge.dir` | path | `scion/judge` | Heads, questions, votes. |
| `judge.eyes` | string | `siglip2-so400m` | The eye model ([Models and licenses](/scion/judge/models/)). |
| `judge.cache` | path | `.scion/judge` | Embeddings cache. |

Paths are relative to `scion.json`; `~` expands to the home folder.

## scion.local.json

The same shape, only the keys that differ on this machine, plus:

| Key | Type | Meaning |
|---|---|---|
| `unity.editor` | path | The Unity editor executable, when not in the Hub's default path. |
| `unity.pipeline_cli` | path | The `com.unity.pipeline` command used for builds through the open editor. |

Objects merge deeply; arrays and values in `scion.local.json` replace those of `scion.json`.

## ScionSettings.json

`ProjectSettings/ScionSettings.json`:

```json
{"schema_version":1,"build":{"main_assembly":"Orchard.Runtime","player_name":"Orchard"},"trials":{"redirect_assemblies":["UnityEngine.UI","Unity.RenderPipelines.Universal.Runtime"],"netclock_assemblies":["Unity.Netcode.Runtime"]},"analyzers":{"rule_assemblies":["Orchard.Core"],"rule_types":["Unity.Netcode.*"],"graft_paths":["Assets/Orchard/Grafts/","../scion/grafts/"]}}
```

| Key | Default | Meaning |
|---|---|---|
| `build.main_assembly` | `Assembly-CSharp` | The assembly whose hash is the `build_id`. |
| `build.player_name` | product name | The player file name. |
| `trials.redirect_assemblies` | `[]` | Extra assemblies whose time and screen reads are redirected in test builds. |
| `trials.netclock_assemblies` | `[]` | Assemblies whose Netcode time moves to the virtual clock (with `SCION_NETCLOCK`). |
| `analyzers.rule_assemblies` | `[]` | Rule code: SCN001, SCN002, SCN004, SCN005; their types are rule types for SCN003. |
| `analyzers.rule_types` | `[]` | Extra rule types (`Ns.Type`, `Ns.Outer.Inner`, `Ns.*`). |
| `analyzers.graft_paths` | `[]` | Folders of graft code (relative to the Unity project). |

Unity does not track this file: recompile the affected assemblies after editing it.

## Scripting defines

| Define | Meaning |
|---|---|
| `SCION_SLOTS` | Slot mode, injection, the control server. Test builds. |
| `SCION_TRIALS` | Trials, virtual clock and screen, captures. Test builds. |
| `SCION_NETCLOCK` | Netcode tick on the virtual clock. Optional. |
| `SCION_VERBOSE` | Extra `[Scion]` log lines. Optional. |

## Environment variables

| Variable | Meaning |
|---|---|
| `SCION_BUILD` | The test build, when `--build` is not given. |
| `NO_COLOR` | Plain text output in a TTY. |

## See also

- [JSON Schemas](/scion/reference/schemas/)
- [CLI overview](/scion/cli/)
- [Build modes](/scion/grafts/build-modes/)
- [Player command line](/scion/reference/player-flags/)
