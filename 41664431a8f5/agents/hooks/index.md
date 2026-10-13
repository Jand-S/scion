---
title: "Hooks"
description: "The four Claude Code hooks of the Scion plugin - session index, graft check after edits, approval guard and baseline guard."
type: reference
url: https://jand-s.github.io/scion/41664431a8f5/agents/hooks/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/agents/hooks.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/agents/setup.md
  - https://jand-s.github.io/scion/41664431a8f5/cli/index-and-guard.md
  - https://jand-s.github.io/scion/41664431a8f5/cli/graft.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Hooks

The Scion plugin registers four Claude Code hooks: `SessionStart` runs `dotnet scion index --hook`, `PostToolUse` on
`Edit|Write` runs `dotnet scion graft check --hook`, `PreToolUse` on `Bash` runs `dotnet scion guard --approve --hook`
and `PreToolUse` on `Edit|Write` runs `dotnet scion guard --hook`. This page gives their configuration, their input and
output, and their failure rules. Each hook finishes in under a second and does nothing outside a Scion project.

## Configuration

The plugin installs this configuration. Without the plugin, put the same block in `.claude/settings.json`:

```json
{
  "hooks": {
    "SessionStart": [
      { "hooks": [{ "type": "command", "command": "dotnet scion index --hook", "timeout": 5 }] }
    ],
    "PostToolUse": [
      { "matcher": "Edit|Write", "hooks": [{ "type": "command", "command": "dotnet scion graft check --hook", "timeout": 5 }] }
    ],
    "PreToolUse": [
      { "matcher": "Bash", "hooks": [{ "type": "command", "command": "dotnet scion guard --approve --hook", "timeout": 5 }] },
      { "matcher": "Edit|Write", "hooks": [{ "type": "command", "command": "dotnet scion guard --hook", "timeout": 5 }] }
    ]
  }
}
```

Each hook reads the event JSON that Claude Code writes on stdin and prints the hook JSON that Claude Code expects on
stdout. Outside a Scion project (no `scion.json` above the working folder), each one exits 0 and prints nothing.

## SessionStart: index

`dotnet scion index --hook` injects the project index as context, written as facts, under 2 KB:

```json
{"hookSpecificOutput":{"hookEventName":"SessionStart","additionalContext":"This project uses Scion 1.0.0 (adoption level 3). Grafts: Assets/Orchard/Grafts (promoted), scion/grafts (drafts). Anchor categories: presentation (14), hud (9), camera (3). Recipes: SlashHit, FireballArea, KnightFalls, CoopTurn (4 players), HudStates, UnitGallery. Commands: dotnet scion anchor list|show, graft new|check|build|inject, trial run|show|approve. Approve only new baselines."}}
```

The index lists categories and counts, not every anchor: the agent asks `anchor list` for details. See
[Project index and manifest](/scion/41664431a8f5/agents/project-index/).

## PostToolUse: graft check

After an `Edit` or `Write`, `dotnet scion graft check --hook` checks the edited file when it is under
`grafts.promoted` or `grafts.staging`, and ignores every other file. Errors return to the agent as context, with
the fix:

```json
{"hookSpecificOutput":{"hookEventName":"PostToolUse","additionalContext":"Scion graft check: scion/grafts/CritShake.cs(7,6): SCN103 HitPresenter.Ply does not exist. Hint: Did you mean 'Play'?"}}
```

A file without problems produces no output. The check reads the test build's DLLs with Mono.Cecil and compiles the
file with Roslyn; it does not start Unity.

## PreToolUse Bash: approval guard

Before a `Bash` call, `dotnet scion guard --approve --hook` looks for `scion trial approve` in the command. Any other
command passes untouched. For an approval it runs the approve as `--dry-run` and decides:

| The approval would replace | Decision | Effect |
|---|---|---|
| Only `new` recipes | allow | The command runs. |
| Any `changed`, `stale`, `within` or `px_changed` recipe | deny | The command does not run. The reason gives the person the review command and the approve command to run at their own terminal. |
| A run made with `--grafts`, or a refused recipe | deny | The command does not run; the reason says why. |

```json
{"hookSpecificOutput":{"hookEventName":"PreToolUse","permissionDecision":"deny","permissionDecisionReason":"SlashHit is changed (image) and replaces a baseline a person approved; only a person approves it, at a terminal. Review: dotnet scion trial show SlashHit --grid. Then the person runs: dotnet scion trial approve SlashHit --reason \"...\""}}
```

The guard denies instead of asking because asking could not help: an agent's shell has no terminal, so the CLI would
refuse the approval anyway (`human_required`, exit 2). The person runs the command from the reason at their own
terminal.

## PreToolUse Edit and Write: baseline guard

`dotnet scion guard --hook` denies any `Edit` or `Write` to a file under `trials.baselines` (`scion/baselines/**`),
`scion.manifest.json` or `GRAFTS.md`, and explains the right command:

```json
{"hookSpecificOutput":{"hookEventName":"PreToolUse","permissionDecision":"deny","permissionDecisionReason":"scion/baselines/ is written only by dotnet scion trial approve. GRAFTS.md and scion.manifest.json are generated: edit the [Anchor] in the .cs file and run dotnet scion gen."}}
```

## Failure rules

| Hook | Time limit | When the CLI fails or times out |
|---|---|---|
| index | under 1 s (5 s timeout) | Fails open: no context, the session starts normally |
| graft check | under 1 s for one file | Fails open: no output |
| approval guard | under 1 s | Fails open: the CLI still refuses with `human_required` |
| baseline guard | under 100 ms (no build access) | Fails closed: an edit under `scion/baselines/` is denied |

Run any hook by hand to see what it does: `echo '{"tool_name":"Write","tool_input":{"file_path":"scion/baselines/SlashHit/baseline.json"}}' | dotnet scion guard --hook`.

## See also

- [index, guard](/scion/41664431a8f5/cli/index-and-guard/)
- [graft](/scion/41664431a8f5/cli/graft/)
- [Baselines and approval](/scion/41664431a8f5/trials/baselines/)
- [Set up your agent](/scion/41664431a8f5/agents/setup/)
