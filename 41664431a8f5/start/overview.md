---
title: "What is Scion"
description: "The problem Scion solves, its four modules, its three pieces and the three adoption levels."
type: concept
url: https://jand-s.github.io/scion/41664431a8f5/start/overview/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/start/overview.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/start/install.md
  - https://jand-s.github.io/scion/41664431a8f5/start/quickstart.md
  - https://jand-s.github.io/scion/41664431a8f5/agents.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# What is Scion

Scion changes and tests a Unity game without editing the game's existing code, and lets an AI agent do it safely.
This page explains the problem, the four modules (Grafts, Trials, Judge, Exploration), the three pieces you install
(the `me.jandson.scion` package, the `dotnet scion` CLI and the agent plugin) and the adoption levels 1 to 3 that
`dotnet scion doctor` reports.

## How it works

### The problem

Reviewing a visual change the usual way is slow and serial: recompile in the editor, press Play, set up the
situation by hand, record a clip and compare by eye. One editor per project means one check at a time. An AI agent
makes it worse: effects live in large files, so every change means reading and editing big methods, and a missed
case goes unnoticed.

Scion's answer is that the editor is the slow part, not the game. A **test build** of the game is compiled once,
opened directly in the state a recipe describes, recorded, measured and closed. Many copies run at the same time,
each with a queue of recipes.

### The four modules

| Module | What it does | Start reading |
|---|---|---|
| **Grafts** | New code in small files (`[Before]` and `[After]` methods) hung on existing methods by a compile-time weaver. The agent creates files next to the game instead of editing large ones. | [How grafts work](/scion/41664431a8f5/grafts/overview/) |
| **Trials** | Recipes (C# classes) played by the real test player in parallel, deterministic copies. Each result has a state hash, frames, images and checks, compared with approved baselines. | [How trials work](/scion/41664431a8f5/trials/overview/) |
| **Judge** | A trainable taste model: frozen vision models plus small heads trained on your votes. It ranks candidates so a person only looks at the finalists. | [The Judge](/scion/41664431a8f5/judge/overview/) |
| **Exploration** | Tools built on determinism: fork a match at turn N (multiverse), loop a moment of play, bisect regressions, metamorphic tests, a rare-moment hunter and guided search. | [The multiverse](/scion/41664431a8f5/explore/multiverse/) |

Around them sit the **evaluation funnel** (state checks, objective gates, then taste) and the **Scion Standard**
(rules and analyzers that keep a game easy to graft and test).

### The three pieces

| Piece | What it is | How you get it |
|---|---|---|
| Unity package `me.jandson.scion` | Attributes, base classes, the weaver (an ILPostProcessor), the trials runner, capture and telemetry, the analyzers | Git URL with a tag, or OpenUPM |
| CLI `dotnet scion` | Builds the test player, runs trials in parallel, compares, approves, trains the Judge, serves MCP | `dotnet tool install --local Scion.Cli` |
| Agent plugin `scion` | The skill, the hooks, the MCP server entry and the `scion-triage` subagent for Claude Code; an `AGENTS.md` block for other agents | `/plugin install scion@scion`, or [agent setup](/scion/41664431a8f5/agents/setup/) |

### How one change flows

```text
agent edits scion/grafts/CritShake.cs
  -> dotnet scion graft check        Roslyn + weaver rules, no Unity, < 1 s
  -> dotnet scion graft build        DLL compiled against the test build
  -> dotnet scion graft inject       swapped into running copies at the next recipe boundary
  -> dotnet scion trial run --warm   recipes replayed, hash and frames compared
  -> dotnet scion trial show         verdict, report.md, window.png
  -> a person approves               dotnet scion trial approve ... (changed needs a terminal)
  -> dotnet scion graft promote      the graft moves into the game; the final build calls it directly
```

### Adoption levels

| Level | Needs | Unlocks |
|---|---|---|
| **1. Grafts** | The package | Grafts with automatic anchors, `graft check`, the project index |
| **2. Trials** | An adapter that turns a recipe into game state | Parallel recipes, captures, checks, baselines, the Judge |
| **3. Multiverse** | A deterministic rules core with a command log, registered as an `ISimulation` | Forking a match by turn, proof that visuals do not change rules, matches turned into recipes, balance specs, the rare-moment hunter |

`dotnet scion doctor` prints the current level (`adoption_level`) and what is missing for the next one
(`adoption_next`). `dotnet scion diagnose` turns that into a plan with file and line.

## Key terms

| Term | Meaning |
|---|---|
| Graft | A `static void` method marked `[Before]` or `[After]` that the weaver calls from a game method. |
| Anchor | A game method rewritten by the weaver so grafts run around it. |
| Test build | The Mono player with `SCION_SLOTS` and `SCION_TRIALS`: swappable slots, trials, virtual time. Never shipped. |
| Final build | The IL2CPP player you ship: grafts are direct calls, no trials code. |
| Recipe | A `TrialRecipe` class that describes one scene to reproduce. |
| Adapter | Your class that turns a recipe into game state and drives the capture. |
| Baseline | The approved result of a recipe, kept in git under `scion/baselines/`. |
| Verdict | `same`, `within`, `px_changed`, `changed`, `new` or `stale`: how a run compares with its baseline. |

The [glossary](/scion/41664431a8f5/reference/glossary/) defines every term the docs use.

## Guarantees and limits

- Grafts change only presentation. Rules change in the game's core through `RuleModifier<T>`, so a rule never
  depends on code the weaver adds. The analyzer SCN003 enforces it.
- Nothing from the test build reaches players: the final build carries grafts as direct calls and refuses test
  defines (SCN121).
- Results are reproducible on the same machine and build: game time advances a fixed step per frame, clocks,
  screen and random sources are virtual, and particles, animators and UI Toolkit are driven by the recipe clock.
- Pixels may differ across GPUs and drivers; baselines are kept per platform, and images are compared with a
  perceptual tolerance.
- The CLI never calls an AI model. Agents read its JSON and files; evaluators that use a model run as separate
  processes or export and import files.
- Approving a result that replaces an approved one needs a person at a terminal.

## See also

- [Requirements](/scion/41664431a8f5/start/requirements/)
- [Install](/scion/41664431a8f5/start/install/)
- [Quickstart](/scion/41664431a8f5/start/quickstart/)
- [For AI agents](/scion/41664431a8f5/agents/)
