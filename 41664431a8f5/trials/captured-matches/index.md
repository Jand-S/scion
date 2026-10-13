---
title: "Turn any match into a recipe"
description: "Press F9 in a test build to save the setup, seed and command log of the current match, and replay it as a CapturedMatch recipe."
type: guide
url: https://jand-s.github.io/scion/41664431a8f5/trials/captured-matches/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/trials/captured-matches.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/reference/files/manifest.md
  - https://jand-s.github.io/scion/41664431a8f5/explore/multiverse.md
  - https://jand-s.github.io/scion/41664431a8f5/explore/rare-moments.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Turn any match into a recipe

Anything that happens while someone plays a test build can become a recipe. Press **F9**: the game saves the
match's setup, seed and command log to `scion/recipes/captured/<name>.match.json`. Every saved match is a
`CapturedMatch` recipe that replays the commands and captures the result like any other recipe. The game implements
`IMatchCapture` once to save and restore its setup.

## When to use

- Capture a bug the moment you see it: the recipe reproduces it for an agent and guards the fix.
- Capture a moment that looks great, so later changes cannot break it unnoticed.
- Captured matches are also the starting points of the multiverse and the live loop.

## Before you begin

- The rules take commands and are deterministic from a seed (level 2 or more, see
  [Find non-determinism](/scion/determinism/diagnose/)).

## Steps

### 1. Implement IMatchCapture

```csharp
using Scion.Trials;

[assembly: MatchCapture(typeof(Orchard.Trials.OrchardMatchCapture))]

namespace Orchard.Trials
{
    public sealed class OrchardMatchCapture : IMatchCapture
    {
        public bool CanCapture => Battle.Current != null;
        public int Seed => Battle.Current.Setup.Seed;
        public int Players => Battle.Current.Setup.Players;
        public string SaveSetup() => JsonUtility.ToJson(Battle.Current.Setup);
        public IEnumerable<(int Turn, int Player, string Data)> Commands() =>
            Battle.Current.Log.Select(c => (c.Turn, c.Player, c.ToJson()));

        public IEnumerator Replay(MatchData match, TrialSession s)
        {
            var battle = Battle.Start(JsonUtility.FromJson<BattleSetup>(match.Setup));
            yield return s.WaitUntil(() => battle.Ready, "battle ready");
            s.BeginCut();
            foreach (var c in match.Commands)
            {
                battle.Issue(Command.FromJson(c.Data));
                yield return s.WaitUntil(() => battle.Idle, "turn " + c.Turn);
            }
            s.EndCut();
        }

        public string StateHash => Battle.Current?.State.Hash64.ToString("X16");
    }
}
```

### 2. Capture

Play the test build and press F9. The game shows `Saved match knight-dies-to-crit` and writes:

```text
scion/recipes/captured/knight-dies-to-crit.match.json
```

The name is typed in a small prompt (Enter keeps the suggested name).

### 3. Replay it

```sh
dotnet scion trial list
dotnet scion trial run "CapturedMatch[match=knight-dies-to-crit]"
```

## Reference

### The file

<!-- validate: match -->
```json
{"schema_version":1,"name":"knight-dies-to-crit","captured_at":"2026-10-11T16:02:10Z","build_id":"4c49a1e0b2f3","source":"f9","seed":7,"players":1,"setup":{"map":"orchard-02","units":["knight","skeleton","skeleton"]},"commands":[{"turn":1,"player":0,"data":"{\"ability\":\"slash\",\"attacker\":0,\"target\":1}"},{"turn":2,"player":1,"data":"{\"ability\":\"bite\",\"attacker\":1,\"target\":0}"}],"turns":2,"feature":null}
```

`source` is `f9`, `hunt` (found by the [rare-moment hunter](/scion/explore/rare-moments/)) or `manual`. Matches the
hunter finds go to `scion/recipes/found/` until someone moves them to `captured/`.

### The recipe

`CapturedMatch` is a recipe of the package with one variant per file, `CapturedMatch[match=<name>]`. Its adapter
calls `IMatchCapture.Replay`. Baselines, checks and approval work as for any recipe.

### Options

| Where | Setting |
|---|---|
| Key | F9 by default; change it with `[assembly: MatchCapture(typeof(OrchardMatchCapture), Key = KeyCode.F8)]`. |
| Folder | `scion/recipes/captured/` (`trials.recipes_staging` + `captured/`). |
| Player flag | `-scion-match <file>` plays one match file directly. |

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| recipe error `replay diverged at turn <n>` | 1 | The rules changed since the capture, or are not deterministic. | Compare with `dotnet scion multiverse run`; recapture if the change is intended. |
| `no_match` | 2 | No match file with that name. | `trial list` shows captured matches. |

## Examples

### Guard a bug fix

```sh
dotnet scion trial run "CapturedMatch[match=knight-dies-to-crit]" --rounds 2
dotnet scion trial approve --new --reason "regression guard for the crit death bug"
```

### Play one match file in a window

```sh
.scion/builds/test/Orchard.app/Contents/MacOS/Orchard -scion-trials -scion-out /tmp/m -scion-match scion/recipes/captured/knight-dies-to-crit.match.json
```

## See also

- [The multiverse](/scion/explore/multiverse/)
- [Rare-moment hunter](/scion/explore/rare-moments/)
- [Recipes API](/scion/reference/api/recipes/)
