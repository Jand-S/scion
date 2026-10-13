---
title: "Requirements"
description: "Unity, .NET, operating systems, scripting backends, memory, disk and git LFS needed to run Scion 1.0.0."
type: reference
url: https://jand-s.github.io/scion/41664431a8f5/start/requirements/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/start/requirements.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/start/install.md
  - https://jand-s.github.io/scion/41664431a8f5/cli/doctor.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Requirements

This page lists what Scion 1.0.0 needs: the Unity version, the .NET SDK for the `dotnet scion` CLI, the supported
operating systems, the Mono and IL2CPP backends for the test and final builds, and the memory, disk and git LFS that
parallel trials use. `dotnet scion doctor` checks every item here and says which one fails.

## Software

| Component | Requirement | Checked by `doctor` |
|---|---|---|
| Unity | 6000.3 or any later Unity 6 release (`"unity": "6000.3"` in the package manifest) | `unity` |
| Unity package dependency | `com.unity.nuget.mono-cecil` 1.11.4 (installed automatically) | `unity` |
| Unity modules | Mono for the test build of your target platform; IL2CPP for the final build | `build`, `defines` |
| .NET SDK | 10.0 or later, for the CLI (`Scion.Cli` targets `net10.0`) and for specs and graft DLLs | `dotnet` |
| git | 2.30 or later on the `PATH`, for approvals (`git add`), `build_stale`, `bisect` and `rule_digest` | `git` |
| git LFS | Needed when baselines keep images (`*.png` under `scion/baselines/`) | `git` |
| Optional: `com.unity.pipeline` | Lets the CLI build while the editor has the project open | `pipeline` |

Scion has no dependency on Netcode for GameObjects, URP or any render pipeline. It works with the built-in
renderer, URP and HDRP. When your game uses Netcode for GameObjects, the optional `SCION_NETCLOCK` define keeps its
tick on the recipe clock ([Netcode clock](/scion/41664431a8f5/determinism/netclock/)).

## Operating systems

| OS | CLI | Test and final builds | Notes |
|---|---|---|---|
| macOS 14 or later (Apple silicon or Intel) | yes | `osx` | Headless capture uses Metal offscreen rendering. |
| Windows 10 or 11, x64 | yes | `win` | Paths with spaces are supported; `~` means `%USERPROFILE%`. |
| Linux x64 (Ubuntu 22.04 or later) | yes | `linux` | Set `unity.editor` in `scion.local.json` when the editor is not installed by the Unity Hub. |

## Scripting backends

| Build | Backend | Why |
|---|---|---|
| Test build (`dotnet scion build test`) | Mono | Slots are swappable delegates and injection loads assemblies at run time; both need a JIT. `ScionBuild.BuildTestPlayer` forces Mono and restores your setting afterwards. |
| Final build (`dotnet scion build final`) | IL2CPP (or the project setting) | Grafts become direct static calls; nothing is registered at run time. |

A build with `SCION_SLOTS` on IL2CPP fails with [SCN120](/scion/41664431a8f5/reference/diagnostics/builds-and-injection/#scn120).

## Project layout

- Your game code is in an assembly definition (`.asmdef`). `Scion.Runtime` is not auto-referenced, so the default
  `Assembly-CSharp` cannot see it. `dotnet scion init` adds the reference to the assemblies you name.
- At least one scene is enabled in **File > Build Profiles > Scene List**. Scion's builds use exactly those scenes.
- For level 3 features, the rules live in an assembly without engine references (`noEngineReferences: true`), so
  the CLI can run them in plain .NET.

## Machine sizing

Numbers measured on a 10-core Apple M5 with 16 GB, a tactics game with network host in each copy, headless
capture at 1280x720 and 30 fps.

| Measure | Value |
|---|---|
| RAM per player copy | about 0.5 to 0.7 GB (about 0.96 GB for a single copy) |
| Recipe cost inside a running copy | 0.33 to 0.50 s for a 2 s scene; a copy starts in under 1 s after the game loads |
| 120 captures (5 recipes x 24 rounds) | 27 s with 6 headless copies; 67 s with 1 copy |
| Best copy count | 4 to 6 headless copies on that machine; `doctor` suggests a number from cores and free RAM |
| Disk per round of 120 captures | about 600 MB in the run folder (raw frames are deleted unless `--frames` or `-scion-raw`) |
| Graft DLL build (`graft build`) | 0.3 to 0.6 s |
| Weaver time inside a Unity build | under 1 s |
| Graft call cost, final build | not measurable (the graft is inlined) |
| Graft call cost, test build slot | about 1 to 3 ns |

`trial run` deletes run folders beyond `trials.keep_runs` (10 by default), so disk use stays bounded. Keep
`trials.out` outside git (the default `.scion/runs` is in `.gitignore` after `dotnet scion init`).

## Judge hardware

The Judge runs on the CPU, or on the GPU through ONNX Runtime when one is available (CoreML on macOS, DirectML on
Windows, CUDA on Linux). Computing embeddings for 10,000 images takes about 5 to 10 minutes on an M5; training a head
takes seconds; judging one image takes 10 to 50 ms. Eye models take 0.4 to 1.6 GB of disk each in
`~/.scion/models/`. See [Models and licenses](/scion/41664431a8f5/judge/models/).

## See also

- [Install](/scion/41664431a8f5/start/install/)
- [doctor](/scion/41664431a8f5/cli/doctor/)
- [Parallel copies](/scion/41664431a8f5/trials/parallel/)
- [FAQ](/scion/41664431a8f5/help/faq/)
