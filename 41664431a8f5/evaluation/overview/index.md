---
title: "The evaluation funnel"
description: "How Scion judges a change in stages - the state oracle and objective gates block, taste triage only orders, and a person makes the final pick."
type: concept
url: https://jand-s.github.io/scion/41664431a8f5/evaluation/overview/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/evaluation/overview.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/evaluation/gates.md
  - https://jand-s.github.io/scion/41664431a8f5/judge/overview.md
  - https://jand-s.github.io/scion/41664431a8f5/evaluation/evaluators.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# The evaluation funnel

An agent can produce ten versions of a hit effect in minutes; nobody can review ten versions of everything. Scion
evaluates in a funnel: cheap, certain stages first, each cutting what reaches the next. The **state oracle** and the
**objective gates** may block a change; **taste triage** (the judge, evaluators) only orders what is left; a
**person** picks. No model ever approves a change.

## How it works

![Diagram: four stages from wide to narrow: state oracle, objective gates, taste triage, a person picks.](../../../assets/diagrams/funnel.svg)

### 1. State oracle (blocks)

What the rules say happened: the state hash, checks, probes, the turn log. A graft that changes the hash is wrong
whatever it looks like. Cost: free, already in every run. See [Checks](/scion/trials/checks/).

### 2. Objective gates (block)

Measurable visual rules that do not need taste: text that overflows its box, contrast under WCAG, flashes that risk
seizures, a regression against the baseline, telemetry that says something happened while the pixels show nothing.
A gate is a check with the id `gate:<name>`; a failing gate fails the recipe. See [Gates](/scion/evaluation/gates/).

### 3. Taste triage (orders)

What remains is a matter of feel: is the hit heavy enough, is the HUD clean? Judge heads score images and sequences
in milliseconds; evaluators (scripts or agents) add scores from files. Scores never fail a run (`gate: false`): they
sort variants and pick which pairs a person should see. See [The Judge](/scion/judge/overview/) and
[Evaluators](/scion/evaluation/evaluators/).

### 4. A person picks

The person sees few, well-chosen comparisons: blind A/B pairs aligned on the impact frame
([A/B comparisons](/scion/evaluation/ab-comparison/)). Their vote approves the baseline and trains the judge.

### Where the inputs come from

| Stage | Reads |
|---|---|
| State oracle | `result.json` (`hash`, `checks`, `probes`), `telemetry.jsonl` |
| Gates | frames, `frames.tsv`, the UI tree in test builds, telemetry |
| Taste | `window.png`, `context.png`, the motion pack, `metrics.json` |
| Person | `trial show --grid`, `preview.gif`, the judge's ranking |

## Key terms

| Term | Meaning |
|---|---|
| State oracle | The rules' own record: hash, checks, probes. |
| Gate | An objective visual check that can fail a recipe. |
| Triage | Ordering variants by predicted taste; never blocks. |
| Head | A small judge model that answers one question. |
| Evaluator | An external scorer (process or agent) wired through files. |
| Duel | A blind A/B comparison shown to a person. |

## Guarantees and limits

- Only stages 1 and 2 change exit codes. A low judge score is never a failure.
- Gates measure what they can measure: a gate that passes does not mean the screen looks good.
- The judge is trained on the team's own votes; it predicts their taste, not general quality.
- A person approves every replaced baseline ([Baselines and approval](/scion/trials/baselines/)).

## See also

- [Gates](/scion/evaluation/gates/)
- [The motion pack](/scion/evaluation/motion-pack/)
- [The Judge](/scion/judge/overview/)
- [Evaluators](/scion/evaluation/evaluators/)
