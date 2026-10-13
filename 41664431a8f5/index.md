---
title: "Scion"
description: "Graft new code onto a Unity game, test it in the real build and judge the result, with a CLI built for AI agents."
type: landing
url: https://jand-s.github.io/scion/41664431a8f5/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/index.md
version: 1.0.0
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Scion

Scion 1.0.0 is a Unity 6 package (`me.jandson.scion`) and a .NET command line (`dotnet scion`). Your game stays
the trunk; new code arrives as small branches grafted onto it at compile time, and every change is played,
captured and compared in the real player build before it ships.

## Install

With Claude Code, add the plugin and ask it to set Scion up:

```sh
/plugin marketplace add Jand-S/scion
/plugin install scion@scion
```

By hand, add the package and the CLI, then check the setup:

```sh
openupm add me.jandson.scion
dotnet new tool-manifest
dotnet tool install --local Scion.Cli --version 1.0.0
dotnet scion init --existing
dotnet scion doctor
```

The [install page](/scion/start/install/) covers every path, including other agents and plain Git URLs.

## Grafts

Static C# methods marked `[Before]` or `[After]` hang onto existing game methods, woven into the IL at compile
time. Final builds call them directly; test builds swap them live in about a second.
[How grafts work](/scion/grafts/overview/).

## Trials

Deterministic recipes play in parallel copies of the real test player, headless, on a virtual clock. Each run
records a state hash, frames and images, runs checks and compares everything with approved baselines.
[How trials work](/scion/trials/overview/).

## Judge

Frozen vision models turn every capture into numbers once; small heads trained in seconds on your own votes
rank results by taste. The Judge filters and orders; a person always makes the final choice.
[The Judge](/scion/judge/overview/).

## Exploration

Fork a deterministic match at any turn, loop a moment of play while you tune it, bisect regressions, hunt rare
states and search tunables with guided duels. [The multiverse](/scion/explore/multiverse/).

## For AI agents

Every command prints one JSON object with a `next_step`. Agents start at
[/scion/llms.txt](/scion/llms.txt), which lists every page as Markdown, and at the
[For AI agents](/scion/agents/) section.
