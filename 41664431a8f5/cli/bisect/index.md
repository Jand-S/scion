---
title: "bisect"
description: "Find the first commit that changed a recipe's hash, pixels or a check, building cold at each step and skipping undecidable steps with exit 125."
type: cli
url: https://jand-s.github.io/scion/41664431a8f5/cli/bisect/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/cli/bisect.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/explore/bisect.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/exit-codes.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# bisect

`dotnet scion bisect` runs `git bisect` between a good and a bad revision. At each step it builds the test player
cold, runs one recipe for some rounds and decides good, bad or skip, comparing on the hash, the images or one check.
It prints the first bad commit and a log of every step.

## Synopsis

```text
dotnet scion bisect <Id> --good <rev> [--bad <rev>] [--on hash|pixels|check:<id>] [--rounds N]
```

## Description

1. Requires a clean working tree; records the starting branch to return to.
2. Runs the recipe at `--good` to record the reference (hash, images or check status).
3. `git bisect start <bad> <good>` and `git bisect run` with a step that builds, runs and compares.
4. Each step exits 0 (good), 1 (bad) or 125 (skip: the build failed, the id is missing, rounds disagree).
5. Returns to the starting branch and prints `first_bad`.

## Options

| Option | Default | Meaning |
|---|---|---|
| `<Id>` | required | The recipe id. |
| `--good <rev>` | required | A good revision. |
| `--bad <rev>` | `HEAD` | A bad revision. |
| `--on hash\|pixels\|check:<id>` | `hash` | What decides good or bad. |
| `--rounds N` | 2 | Rounds per step. |

## Output

<!-- validate: cli/bisect -->
```json
{"schema_version":1,"command":"bisect","ok":true,"id":"KnightFalls","good":"3f2a0b1","bad":"HEAD","on":"check:order","steps":4,"skipped":[],"first_bad":{"commit":"9d0c4e2","subject":"Play unit_down before the last hit number","date":"2026-10-10T11:05:00Z"},"log":".scion/bisect/20261012-140000/log.txt","next_step":"git show 9d0c4e2"}
```

## Exit codes

| Code | When |
|---|---|
| 0 | `first_bad` found. |
| 1 | Too many skips: a range instead of one commit. |
| 2 | Dirty tree, bad revision, unknown id. |
| 3 | Build environment errors at the good revision. |
| 125 | Only from a single step, to `git bisect`: skip this commit. |

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| `usage` | 2 | Uncommitted changes. | Commit or stash. |
| `bad_value` | 2 | A revision git does not know. | Check the revision. |
| `no_match` | 2 | The id does not exist at `--good`. | Pass `--recipes` with a DLL that has it. |
| `interrupted` | 3 | Ctrl-C. | The CLI resets `git bisect` and the branch. |

## Examples

### Pixels between releases

```sh
dotnet scion bisect SlashHit --good v1.3.0 --bad v1.4.0 --on pixels
```

### A probe check

```sh
dotnet scion bisect FireballArea --good 3f2a0b1 --on check:probe:damage
```

## See also

- [Bisect a regression](/scion/41664431a8f5/explore/bisect/)
- [Exit codes](/scion/41664431a8f5/reference/exit-codes/)
