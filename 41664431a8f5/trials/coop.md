---
title: "Co-op trials (1-4 players)"
description: "Run recipes with 1 to 4 players as groups of a host and clients on one port, compare per-turn state hashes, and capture pixels from the host."
type: guide
url: https://jand-s.github.io/scion/41664431a8f5/trials/coop/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/trials/coop.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/trials/parallel.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/configuration.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/player-flags.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Co-op trials (1-4 players)

A recipe with `Players = 2` to `4` runs as a **group**: one host copy and `Players - 1` client copies of the same
test build, connected on a free port the CLI picks. Every process writes the state hash of each turn to its
telemetry; the recipe passes only if all of them agree and none fails. Pixels come from the host. This guide covers
`Players`, the `scion.json` keys `host_args`, `client_args` and `port_arg`, `-scion-role`, and how groups count
toward `-j`.

## When to use

- Write a co-op recipe for every mechanic that crosses the network: turns, shared abilities, joining and leaving.
- Keep solo recipes for presentation: they are cheaper and capture the same pixels.

## Before you begin

- The game takes its own command-line flags to host, join and pick a port. Orchard uses `-orchard-host`,
  `-orchard-join localhost` and `-orchard-port <n>`.
- The adapter writes a `turn` event with the state hash at the end of every turn.

## Steps

### 1. Tell the CLI how to start each role

```json
{"schema_version":1,"trials":{"host_args":["-orchard-host"],"client_args":["-orchard-join","localhost"],"port_arg":"-orchard-port"}}
```

### 2. Write the recipe

```csharp
public sealed class CoopTurn : OrchardRecipe
{
    public CoopTurn()
    {
        Players = 4;
        Description = "Four players each act once; every peer agrees on the state.";
        TimeoutSeconds = 60;
    }
}
```

### 3. Play it by role in the adapter

```csharp
protected override IEnumerator Run(OrchardRecipe r, TrialSession s)
{
    yield return s.WaitUntil(() => Lobby.Connected == r.Players, "all players connected");
    s.BeginCut();
    for (int turn = 1; turn <= r.Players; turn++)
    {
        if (Lobby.LocalPlayer == turn - 1) battle.Issue(Command.Ability("slash", attacker: turn - 1, target: 4));
        yield return s.WaitUntil(() => battle.State.Turn > turn, "turn " + turn);
        s.Event("turn", ("turn", turn), ("hash", battle.State.Hash64.ToString("X16")));
    }
    s.EndCut();
    hash = battle.State.Hash64.ToString("X16");
}
```

`s.Role` tells the adapter whether it is the host (`IsHost`, index 0) or `client:1` to `client:3`.

### 4. Run it

```sh
dotnet scion trial run CoopTurn -j 8
```

With `-j 8`, two groups of four run at once.

## Reference

![Diagram: a host and three clients connected on one port; every process writes per-turn hashes; pixels come from the host.](../../../assets/diagrams/coop-group.svg)

### What the CLI does per group

1. Picks a free port (bind to 0 and release).
2. Starts the host with `player_args`, `host_args`, `port_arg <port>` and `-scion-role host`.
3. Starts each client with `player_args`, `client_args`, `port_arg <port>` and `-scion-role client:<i>`.
4. Gives every process of the group the host's shard and `-scion-players <Players>`.
5. After the group ends, compares the `turn` events of every process.

### Rules

| Rule | Detail |
|---|---|
| Group size | `Players` copies, counted against `-j`. A `-j` smaller than `Players` is raised to `Players`. |
| Selection | A process plays only recipes whose `Players` equals its `-scion-players` (default 1), so solo copies never wait for clients. |
| Clock | The host runs on the virtual clock; clients run on the real clock and follow the host's ticks. |
| Pixels | Only the host captures. Clients write `result.json`, telemetry and logs. |
| Pass | Every process passes, and every `turn` event agrees on `hash` for the same `turn`. |
| Fail | Any process failing or exiting (a client that drops) fails the recipe, naming the process and the turn. |

### Output

Each process has its own `q<i>/` folder; `summary.json` lists them under `copies[]` with `role` and `group`.
A mismatch reads:

```text
CoopTurn r1: turn 3 hash differs: host 9F3A0C11D2E4B5A6, client:2 1B44E07AC9D20F13
```

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| recipe error `turn <n> hash differs` | 1 | Peers disagree on the rules state. | Run `dotnet scion diagnose`; look for rules reading local state. |
| recipe error `client:<i> exited <code>` | 1 | A client crashed or quit. | Read `q<i>/player.log`. |
| `copy_timeout` | 3 | A group made no progress. | Check `host_args`, `client_args` and `port_arg`. |

## Examples

### Only the co-op recipes

```sh
dotnet scion trial run --players 2
dotnet scion trial run --players 4
```

### A two-player recipe with a pass-through flag

```sh
dotnet scion trial run CoopTurn -- -orchard-latency 80
```

Arguments after `--` go to every process of the group and replace the same flag from `player_args`.

## See also

- [Parallel copies](/scion/41664431a8f5/trials/parallel/)
- [Configuration](/scion/41664431a8f5/reference/configuration/)
- [Player command line](/scion/41664431a8f5/reference/player-flags/)
- [Netcode clock](/scion/41664431a8f5/determinism/netclock/)
