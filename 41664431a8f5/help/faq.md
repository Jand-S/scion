---
title: "FAQ"
description: "Short answers about shipping, Mono and IL2CPP, dependencies, platforms, Harmony and MonoMod, agent costs, the license and offline use."
type: reference
url: https://jand-s.github.io/scion/41664431a8f5/help/faq/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/help/faq.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/start/overview.md
  - https://jand-s.github.io/scion/41664431a8f5/start/requirements.md
  - https://jand-s.github.io/scion/41664431a8f5/help/license.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# FAQ

Short answers to the questions people ask first. Each links to the page with the details.

## Questions

### Does Scion ship to players

Only as direct calls. The final build calls promoted grafts directly (call mode): no slots, no injection, no trials,
no lab code, no reflection. A final build with test defines fails (SCN121). See [Build modes](/scion/41664431a8f5/grafts/build-modes/).

### Why does the test build need Mono

Slots are replaced while the game runs, which needs a JIT. IL2CPP compiles ahead of time. The test build is Mono; the
final build can be IL2CPP.

### Do I need Netcode or URP

No. Neither is a dependency. Scion knows how to redirect their time reads when you list them in `ScionSettings.json`,
and co-op trials work with any networking that takes command-line flags.

### Which platforms

The editor and the CLI run on macOS, Windows and Linux. Test builds run on the desktop platforms. See
[Requirements](/scion/41664431a8f5/start/requirements/).

### Why not Harmony or MonoMod

They patch at run time, which IL2CPP does not support, and their errors appear only when the code runs. Scion weaves
at compile time: errors come with file and line, and final builds have plain direct calls.

### Can grafts change rules

No. Grafts are visual; rules change in the core through `RuleModifier<T>`. See
[Grafts are visual only](/scion/41664431a8f5/grafts/visual-only/).

### How much does an agent spend

The CLI returns compact JSON (about 1.5 KB) with a `next_step`, and images are paths, not content, so an agent reads
only what it needs. The project index is under 2 KB. The judge runs locally and costs nothing per image.

### Does Scion call AI models

No. The CLI, the MCP server and the judge never call a hosted model. Agent evaluators answer through files.

### Does it work offline

Yes. After installing the package, the CLI and (optionally) the judge's eye model, nothing needs the network.

### What is the license

Apache-2.0. See [License](/scion/41664431a8f5/help/license/).

### Where do I report a bug

On the GitHub repository's issues, with the output of `dotnet scion doctor --level 3` and the run's `report.md`.

## See also

- [What is Scion](/scion/41664431a8f5/start/overview/)
- [Requirements](/scion/41664431a8f5/start/requirements/)
- [License](/scion/41664431a8f5/help/license/)
