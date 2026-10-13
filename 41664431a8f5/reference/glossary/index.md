---
title: "Glossary"
description: "Short definitions of every Scion term, from anchor and slot to verdict, warm copy, multiverse and adoption level, with links to the pages that explain them."
type: reference
url: https://jand-s.github.io/scion/41664431a8f5/reference/glossary/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/reference/glossary.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/start/overview.md
  - https://jand-s.github.io/scion/41664431a8f5/grafts/overview.md
  - https://jand-s.github.io/scion/41664431a8f5/trials/overview.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Glossary

Scion uses a small vocabulary consistently across the docs, the CLI output and the code. Each term below has one
definition and a link to the page that explains it. Terms are in alphabetical order.

## Terms

### Adapter

The game's code that plays recipes of a type: `TrialAdapter<TRecipe>` with `Prepare`, `Run`, `Teardown` and
`StateHash`. See [Write an adapter](/scion/41664431a8f5/trials/adapters/).

### Adoption level

How much of Scion a project can use: 1 grafts, 2 trials, 3 multiverse. `doctor` reports it. See
[Find non-determinism](/scion/41664431a8f5/determinism/diagnose/).

### Anchor

A game method the weaver rewrote so grafts run around it. See [Anchors](/scion/41664431a8f5/grafts/anchors/).

### Baseline

The approved hash, frames and images of an id, in `scion/baselines/`. See
[Baselines and approval](/scion/41664431a8f5/trials/baselines/).

### Boundary

The moment between two recipes (and after the queue) when injections apply and leaking state is reset
(`TrialSession.Boundary`).

### Call mode

How final builds reach grafts: a direct call to a generated dispatcher. See [Build modes](/scion/41664431a8f5/grafts/build-modes/).

### Copy

One process of the test build in a run. See [Parallel copies](/scion/41664431a8f5/trials/parallel/).

### Cut

The frames between `BeginCut` and `EndCut`: what a recipe captures.

### Duel

A blind comparison of two variants shown to a person or evaluator. See [A/B comparisons](/scion/41664431a8f5/evaluation/ab-comparison/).

### Eyes

The frozen vision encoder the judge uses for embeddings. See [The Judge](/scion/41664431a8f5/judge/overview/).

### Final build

The player you ship: call mode, no test defines, no lab code.

### Gate

An objective visual check that can fail a recipe (`gate:<id>`). See [Gates](/scion/41664431a8f5/evaluation/gates/).

### Graft

A `static void` method marked `[Before]` or `[After]` that the weaver calls around an anchor. See
[How grafts work](/scion/41664431a8f5/grafts/overview/).

### Group

The copies that play one co-op recipe: a host and its clients. See [Co-op trials (1-4 players)](/scion/41664431a8f5/trials/coop/).

### Head

A small judge model trained on votes to answer one question. See [Train a head](/scion/41664431a8f5/judge/training/).

### Id

One concrete instance of a recipe, with variant, mode and seed: `SlashHit[Distance=3]@360p#7`. See
[Variants, modes and seeds](/scion/41664431a8f5/trials/variants/).

### Live loop

A window of play replayed over and over while injections apply between passes. See
[The live loop](/scion/41664431a8f5/explore/live-loop/).

### Mark

A label on a frame of the cut (`Mark`, `MarkImpact`, `Fx.Mark`).

### Mode

A capture setting applied to every recipe, such as a resolution (`@360p`).

### Mosaic

Several arenas in one process, each with its own camera. See [Mosaic](/scion/41664431a8f5/trials/mosaic/).

### Multiverse

Forking a captured match at a turn and playing variants to see where rules diverge. See
[The multiverse](/scion/41664431a8f5/explore/multiverse/).

### Platform

The part of a baseline that depends on the machine: `osx-metal-1280x720-offscreen`.

### Probe

A number a recipe records (`Probe(key, value)`); checks and specs test it.

### Promoted

A graft compiled into the game (`grafts.promoted`), as opposed to a draft in staging.

### Recipe

A `TrialRecipe` class: the data of one situation to play. See [Write a recipe](/scion/41664431a8f5/trials/recipes/).

### Round

One play of an id; `--rounds 2` plays every id twice.

### Run

One `trial run` and its folder under `trials.out`. See [The run folder](/scion/41664431a8f5/reference/files/run-folder/).

### Run folder

The folder of a run, with one `q<i>/` per copy and the CLI's summaries.

### Sensor

A per-frame column in `frames.tsv` (`Sensor(name, read)`).

### Shard

The share `i/n` of the (id, round) pairs one copy plays.

### Sheet

A labeled grid of captures: `window.png`, `context.png`, `sheet.png`.

### Slot

In test builds, the delegate field through which an anchor calls its grafts; injection replaces it.

### Slot mode

How test builds reach grafts: through slots, on Mono.

### Staging

The drafts folder (`grafts.staging`, `scion/grafts/`).

### Still

One full-size frame saved by name (`Still(name)`).

### Test build

The Mono player with `SCION_SLOTS` and `SCION_TRIALS`. Never shipped.

### Verdict

The comparison of a result with its baseline: `same`, `within`, `px_changed`, `changed`, `new`, `stale`.

### Warm copy

A test build that prepared its adapters and waits idle for work. See [Warm copies](/scion/41664431a8f5/trials/warm-copies/).

## See also

- [What is Scion](/scion/41664431a8f5/start/overview/)
- [How grafts work](/scion/41664431a8f5/grafts/overview/)
- [How trials work](/scion/41664431a8f5/trials/overview/)
