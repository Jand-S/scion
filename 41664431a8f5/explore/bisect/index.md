---
title: "Bisect a regression"
description: "Find the commit that changed a recipe's hash, pixels or a check with scion bisect, building cold at each step and skipping steps that cannot decide."
type: guide
url: https://jand-s.github.io/scion/41664431a8f5/explore/bisect/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/explore/bisect.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/cli/bisect.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/exit-codes.md
  - https://jand-s.github.io/scion/41664431a8f5/determinism/verify.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Bisect a regression

When a recipe was right at an old commit and is wrong now, `dotnet scion bisect` finds the commit that changed it.
It drives `git bisect`: at each step it builds the test player cold, runs the recipe, and compares on what you
choose (`hash`, `pixels` or one check). A step that cannot decide (the build fails, the recipe is flaky) exits 125
and is skipped. It takes about log2(commits) steps.

## When to use

- A `changed` verdict nobody expected, or a check that started failing.
- Not for flaky recipes: fix determinism first, or bisect will skip most steps.

## Before you begin

- A clean working tree (bisect checks out commits).
- The recipe exists at both ends of the range (or comes from `--recipes`).

## Steps

### 1. Run it

```sh
dotnet scion bisect SlashHit --good v1.4.0 --on pixels
```

<!-- validate: cli/bisect -->
```json
{"schema_version":1,"command":"bisect","ok":true,"id":"SlashHit","good":"v1.4.0","bad":"HEAD","on":"pixels","steps":6,"skipped":["8c1f2d0"],"first_bad":{"commit":"5e7a913","subject":"Brighter hit flash material","date":"2026-10-09T17:20:00Z"},"log":".scion/bisect/20261012-110000/log.txt","next_step":"git show 5e7a913"}
```

### 2. Read the result

`first_bad` is the first commit where the recipe differs from `--good`. The log has each step's commit, build time,
verdict and decision.

## Reference

| Option | Default | Meaning |
|---|---|---|
| `<Id>` | | The recipe id. |
| `--good <rev>` | required | A revision where the recipe was right. |
| `--bad <rev>` | `HEAD` | A revision where it is wrong. |
| `--on hash` | `hash` | Compare the state hash with the good revision's. |
| `--on pixels` | | Compare images with the good revision's (same platform). |
| `--on check:<id>` | | Good when that check passes (`check:probe:damage`). |
| `--rounds N` | 2 | Rounds per step; all rounds must agree or the step is skipped. |

### Exit 125

Each step exits 0 (good), 1 (bad) or 125 (skip) to `git bisect run`. 125 means: the build failed, the recipe does
not exist at that commit, or rounds disagree. Too many skips leave a range instead of one commit; `first_bad` is then
null and the log lists the candidates.

### Cost

Every step is a cold build (Orchard: about 35 s on an Apple M5) plus the recipe. Ten commits take about 4 steps.

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| `usage` | 2 | The working tree has changes. | Commit or stash them. |
| `no_match` | 2 | The id does not exist at `--good`. | Use `--recipes` with a DLL that has it. |
| `first_bad: null` | 1 | Too many skipped steps. | Fix determinism or the build at the skipped commits. |

## Examples

### A check that started failing

```sh
dotnet scion bisect KnightFalls --good 3f2a0b1 --on check:order
```

### A rule change between two releases

```sh
dotnet scion bisect FireballArea --good v1.3.0 --bad v1.4.0 --on hash
```

## See also

- [bisect](/scion/cli/bisect/)
- [Exit codes](/scion/reference/exit-codes/)
- [Verify determinism](/scion/determinism/verify/)
