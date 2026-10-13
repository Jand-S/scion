---
title: "Changelog"
description: "Notable changes to Scion by version, in Keep a Changelog format, starting with the 1.0.0 release grouped by module."
type: reference
url: https://jand-s.github.io/scion/41664431a8f5/help/changelog/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/help/changelog.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/contribute/compatibility.md
  - https://jand-s.github.io/scion/41664431a8f5/start/install.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Changelog

All notable changes to Scion are listed here, newest first, in the [Keep a Changelog](https://keepachangelog.com/en/1.1.0/)
format. Scion follows [Semantic Versioning](https://semver.org/spec/v2.0.0.html); what counts as a breaking change is
on [Versioning and compatibility](/scion/41664431a8f5/contribute/compatibility/). The same entries are in the package's
`CHANGELOG.md`.

## [1.0.0] - 2026-10-13

First stable release.

### Grafts

- `[Before]` and `[After]` with `Priority` and `Observe`; compile-time weaver (ILPostProcessor on Mono.Cecil) with call
  mode for final builds and slot mode for test builds.
- `ref Call` and `ref Call<T>`: `Skip`, `Return`, `Result`, `Skipped`, `Tag`.
- Anchors: `[Anchor(Id, Doc)]`, `[assembly: AnchorScope]`, `Anchors.Id`, `anchor list` and `anchor show`.
- Live injection: `graft build`, `graft inject` (merge or isolate), `graft restore`, `graft disable`, `InjectReport`.
- Draft lifecycle: `graft new`, `graft check`, `graft promote`, `graft watch`.
- `Faults`, the `Fx` facade with `IFxSink`, `[Tunable]` with presets and blends.
- Diagnostics SCN101 to SCN114, SCN120, SCN121, SCN130, SCN131.

### Trials

- Recipes, adapters, `TrialSession`, checks and `ITrialCheck`; variants with `[Vary]` and `Variants()`, modes and
  seed ranges.
- Static trials with views and sheets; co-op groups of 1 to 4 players; captured matches (F9); the mosaic.
- Parallel copies with shards, warm copies with `trial serve`, `trial run`, `show`, `approve`, `diff`, `eval`, `loop`.
- Baselines per platform with perceptual image comparison, masks and learned unstable masks.
- Specs (`TrialSpec`) with balance, visual, judge, metamorphic and curve parts.
- Diagnostics SCN201 to SCN203.

### Determinism

- Virtual clock, screen, random and real-time waits in test builds; `[WallClock]`; `DeterminismControl` for
  particles, animators, UI Toolkit and physics; `SCION_NETCLOCK`.
- `--check`, `--jitter`, `determinism.tsv`; `scion diagnose` with SCN301 to SCN307 and adoption levels.

### Evaluation and the Judge

- Gates: text overflow, UI grid, contrast, photosensitivity, regression, telemetry versus pixels, nothing changed,
  palette.
- The motion pack and metrics; blind A/B comparisons; file-based evaluators.
- The Judge: eyes and heads, `judge list`, `show`, `new`, `vote`, `train`, `run`.

### Exploration

- The multiverse, the live loop, bisect, metamorphic tests, the rare-moment hunter, guided search, balance with the
  simulator.

### Standard and core

- `Scion.Core`: `RuleModifier<T>`, `Stage`, `ISimulation`, bots and capabilities; analyzers SCN001 to SCN005.

### Tools and agents

- `dotnet scion` CLI with JSON envelopes and schemas; `init`, `new`, `uninstall`, `doctor`, `build`, `index`,
  `guard`, `gen`, `mcp`.
- Claude Code plugin with the skill and hooks; MCP server with docs tools; `llms.txt` and Markdown pages.
- `Scion.Testing`, `Scion.Compare`, `Scion.Specs` and `Scion.Core` on NuGet.

## See also

- [Versioning and compatibility](/scion/41664431a8f5/contribute/compatibility/)
- [Install](/scion/41664431a8f5/start/install/)
