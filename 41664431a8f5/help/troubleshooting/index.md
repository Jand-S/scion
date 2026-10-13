---
title: "Troubleshooting"
description: "Symptoms, causes and fixes for common problems - copies that do not start, stale verdicts, editor_open, missing slots, IL2CPP and empty captures."
type: troubleshooting
url: https://jand-s.github.io/scion/41664431a8f5/help/troubleshooting/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/help/troubleshooting.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/reference/errors.md
  - https://jand-s.github.io/scion/41664431a8f5/agents/next-action.md
  - https://jand-s.github.io/scion/41664431a8f5/cli/doctor.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Troubleshooting

Each entry starts from what you see, names the usual cause and gives the fix. Run `dotnet scion doctor --level 3`
first: it catches most setup problems with a hint. Error codes link to [Error and warning codes](/scion/41664431a8f5/reference/errors/).

## No copies start

**Symptom.** `trial run` ends at once with `build_not_found` or `player_crashed`.

**Cause.** No test build at the discovered path, a build from another machine (paths in `scion-build.json`), or a
macOS quarantine flag on a downloaded build.

**Fix.** `dotnet scion build test`; or `--build <path>`; on macOS, `xattr -dr com.apple.quarantine <app>`.

## Every verdict is stale

**Symptom.** All ids are `stale` after switching machines or Unity versions.

**Cause.** Images are compared per platform (OS, graphics API, size, capture), and the baseline has no folder for
this platform, or Unity, fps or player args changed.

**Fix.** The hash is still compared, so `stale` with the same hash is safe. A person approves once on the new
platform: `dotnet scion trial approve --changed --reason "baselines for linux-vulkan"`.

## editor_open when building

**Symptom.** `build test` exits 3 with `editor_open`.

**Cause.** The project is open in Unity and `com.unity.pipeline` is not installed, so the CLI cannot build through the
editor and cannot start a second one on the same project.

**Fix.** Close the editor, or install the pipeline package and build again.

## SCN113 when injecting

**Symptom.** `graft inject` reports `SCN113: HitPresenter.PlayCritical has no slot in this build.`

**Cause.** The test build has no slot for that method: no promoted graft on it and no `AnchorScope` covering it.

**Fix.** Add `[assembly: AnchorScope(typeof(HitPresenter))]`, `dotnet scion build test`, inject again.

## SCN120 on a test build

**Symptom.** The build fails with `SCION_SLOTS needs a JIT backend (Mono)`.

**Cause.** The test defines were set in Player Settings and the project builds with IL2CPP.

**Fix.** Remove the defines from Player Settings; build tests with `dotnet scion build test`, which sets Mono and the
defines only for the build.

## Empty captures in batchmode

**Symptom.** Exit 3 `capture_broken`, or black `window.png`.

**Cause.** A player started by hand with `-batchmode` and without offscreen capture, or a camera that renders only
to the screen.

**Fix.** Let the CLI start players, or add `-scion-offscreen`. Cameras with a target texture of their own need to be
captured with a `TrialView`.

## Ports already in use

**Symptom.** `no_free_port`, or co-op clients that never connect.

**Cause.** Another server holds ports, or `port_arg` is missing, so every group uses the game's default port.

**Fix.** Set `trials.port_arg` to the game's port flag; close stray servers (`dotnet scion trial serve --stop`).

## px_unstable warnings

**Symptom.** `px_unstable` and `px_changed` verdicts that come and go.

**Cause.** Something visual is not deterministic: particles started before the recipe, an animator on unscaled real
time, a shader reading `_Time` from a non-redirected source.

**Fix.** Keep `Control = DeterminismControl.All`; run `--rounds 3 --check` and read `determinism.tsv`; mask only what is
random on purpose.

## Frame count off by one

**Symptom.** `frame_jitter: ["CoopTurn 212/213"]`.

**Cause.** The Netcode tick runs on the real clock.

**Fix.** Turn on `SCION_NETCLOCK` with `netclock_assemblies` ([Netcode clock](/scion/41664431a8f5/determinism/netclock/)).

## Low memory

**Symptom.** `low_memory` warnings, copies killed by the OS, `player_crashed`.

**Cause.** `-j` above what free memory allows (about 550 MB per copy for Orchard).

**Fix.** Lower `-j` to `doctor`'s `suggested_copies`, or use warm copies or the mosaic.

## A recipe passes alone and fails in the queue

**Symptom.** `trial run SlashHit` passes; `trial run` fails it.

**Cause.** State leaks from an earlier recipe: a static, `Time.timeScale`, a pool, a scene left loaded.

**Fix.** Reset it in the adapter's `Teardown` or on `TrialSession.Boundary`.

## See also

- [Error and warning codes](/scion/41664431a8f5/reference/errors/)
- [What to do next](/scion/41664431a8f5/agents/next-action/)
- [doctor](/scion/41664431a8f5/cli/doctor/)
