---
title: "Baselines and approval"
description: "How runs are compared with approved baselines, what each verdict means, who may approve what, and how baselines are stored per platform."
type: guide
url: https://jand-s.github.io/scion/41664431a8f5/trials/baselines/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/trials/baselines.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/cli/trial-approve.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/files/baselines.md
  - https://jand-s.github.io/scion/41664431a8f5/trials/image-comparison.md
  - https://jand-s.github.io/scion/41664431a8f5/agents/hooks.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Baselines and approval

A baseline is the approved result of one id: its state hash, frame count and, per platform, its frame hashes and
images. Every `trial run` compares each result with `scion/baselines/` and gives a verdict (`same`, `within`,
`px_changed`, `changed`, `new`, `stale`). `trial approve` turns a run into the new baseline. Agents may approve only
`new`; replacing a baseline someone approved needs a person at an interactive terminal.

## When to use

- Approve after an intended change, once the run shows exactly the change you meant.
- Never approve to make a failure go away: a failed recipe cannot be approved.

## Steps

### 1. Run and read the verdicts

```sh
dotnet scion trial run
dotnet scion trial show --changed
```

<!-- validate: cli/trial-show -->
```json
{"schema_version":1,"command":"trial show","ok":true,"run":"/Users/alex/orchard/.scion/runs/20261011-143200","count":1,"recipes":[{"id":"SlashHit","rounds":{"ok":2,"failed":0},"hashes":["9F3A0C11D2E4B5A6"],"frames":[46],"vs_approved":"changed","report":"q0/SlashHit/r1/report.md","images":["q0/SlashHit/r1/window.png"],"reasons":["image"],"stale":[],"px_diff":0.12}],"next_step":"look at q0/SlashHit/r1/window.png; if intended, a person approves: dotnet scion trial approve SlashHit --reason \"...\""}
```

### 2. Compare side by side

```sh
dotnet scion trial show SlashHit --grid
```

`--grid` writes one image per artifact with three columns: baseline, current, difference.

### 3. Approve

```sh
dotnet scion trial approve SlashHit --reason "crit shake is stronger"
```

<!-- validate: cli/trial-approve -->
```json
{"schema_version":1,"command":"trial approve","ok":true,"run":"/Users/alex/orchard/.scion/runs/20261011-143200","approved":1,"unchanged":0,"baselines":[{"id":"SlashHit","was":"changed","round":1,"copy":0,"dir":"scion/baselines/SlashHit"}],"staged":true,"refused":[],"next_step":"git commit -m \"Approve SlashHit: crit shake is stronger\""}
```

`approve` writes the files and runs `git add` on them; it never commits.

## Reference

### Verdicts

| Verdict | When |
|---|---|
| `same` | Hash, frames and images match. |
| `within` | Images differ inside the recipe's `Tolerance`. |
| `px_changed` | Hash and frames match, frame hashes differ, and the recipe compares no images (`Tolerance = null`). Informative. |
| `changed` | The hash differs, frames differ by more than one, an image is outside the tolerance, or the rule digest changed. |
| `new` | The id has no `baseline.json`. |
| `stale` | `baseline.json` exists but not for this platform, or fps, player args or Unity differ. Images and frame hashes are suspended; the hash is still compared, so a different hash is `changed`. |

`reasons` lists what triggered `changed`: `hash`, `frames`, `px`, `image`, `rule_hash`. `stale` lists why:
`platform`, `fps`, `player_args`, `unity`.

### Who approves what

| Verdict | Agent (no TTY) | Person at a terminal |
|---|---|---|
| `new` | yes | yes |
| `changed`, `stale`, `within`, `px_changed` | no: exit 2, `human_required` | yes, after a confirmation prompt |
| failed recipe | no | no |
| run with `--grafts` or `--warm` | no | no |

Approval is all or nothing: if one id is refused, nothing is written and `refused[]` says why. The `guard` hook
asks the person before an agent's `trial approve` and denies edits under `scion/baselines/`
([Hooks](/scion/agents/hooks/)).

### Storage

```text
scion/baselines/SlashHit/
  baseline.json                         hash, frames, fps, player args, rule digest, approval
  osx-metal-1280x720-offscreen/
    platform.json                       size, Unity, tolerance, images
    frames.tsv                          per-frame hashes
    window.png  context.png  before.png
    unstable.png                        learned mask, optional
```

Store PNGs with Git LFS. Approving on one platform updates `baseline.json` and that platform's folder; other
platforms become `stale` until someone approves on each.

### Rule digest

`rule_digest` is a 12-hex digest of the files under `rule_paths` in `scion.json` (`Assets/Orchard/Core`,
`Assets/Orchard/Data`), uncommitted changes included. When it differs from the baseline's, the verdict is `changed`
with reason `rule_hash`, even if the hash happens to match: a rule change must be reviewed.

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| `human_required` | 2 | Approval of a non-`new` verdict without a TTY. | A person runs the command at a terminal. |
| `refused` | 1 | An id failed, or the run used `--grafts` or `--warm`. | Fix it, run cold, approve again. |
| `no_match` | 2 | The id is not in the run. | `trial show` lists the ids of the run. |
| warning `not_staged` | 0 | `git add` failed (no repository, LFS missing). | Stage the files yourself. |

## Examples

### Approve every new recipe (agents may)

```sh
dotnet scion trial approve --new --reason "first baseline for the gallery"
```

### Preview what would be approved

```sh
dotnet scion trial approve --changed --reason "new palette" --dry-run
```

## See also

- [trial approve](/scion/cli/trial-approve/)
- [baseline.json and platform.json](/scion/reference/files/baselines/)
- [Image comparison](/scion/trials/image-comparison/)
- [Hooks](/scion/agents/hooks/)
