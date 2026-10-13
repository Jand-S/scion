---
title: "anchor list, anchor show"
description: "List the methods of the test build that can take grafts, with slots, Anchor ids, docs and promoted grafts, and show one anchor with a ready declaration."
type: cli
url: https://jand-s.github.io/scion/41664431a8f5/cli/anchor/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/cli/anchor.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/grafts/anchors.md
  - https://jand-s.github.io/scion/41664431a8f5/cli/graft.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/api/grafts.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# anchor list, anchor show

`dotnet scion anchor list` reads the test build with Mono.Cecil (without starting it) and lists every method that can
be an anchor: its slot id, its `[Anchor]` id and doc, whether it has a slot in this build, and the promoted grafts on
it. `dotnet scion anchor show <id>` prints one anchor: the signature, the grafts in run order and a graft declaration
that binds to it.

## Synopsis

```text
dotnet scion anchor list [--search <text>] [--type <Type>] [--build <path>]
dotnet scion anchor show <id> [--build <path>]
```

## Description

Eligibility follows the weaver's rules ([Anchors](/scion/grafts/anchors/)): methods that cannot be anchors are left
out. Each eligible method appears twice, once per side (`B__` before, `A__` after).

## anchor list

Lists anchors, sorted by type and method. `--search` matches the type, method, `[Anchor]` id or doc
(case-insensitive); `--type` limits to one type's full name.

## anchor show

`<id>` is an `[Anchor]` id (`presentation.hit`), a slot id (`A__...`) or `Type.Method`. The output has `example`: a
complete graft method that compiles against the anchor, with `ref Call<T>` when the target returns a value.

## Options

| Option | Commands | Meaning |
|---|---|---|
| `--search <text>` | list | Filter by text. |
| `--type <Type>` | list | One type. |
| `--build <path>` | both | The test build. |

## Output

<!-- validate: cli/anchor-show -->
```json
{"schema_version":1,"command":"anchor show","ok":true,"anchor":{"id":"A__Orchard_Presentation_HitPresenter__Play__HitEvent","anchor":"presentation.hit","target":"Orchard.Presentation.HitPresenter.Play","signature":"float Play(HitEvent e)","slot":true,"doc":"Plays one hit and returns how long it lasts.","grafts":[{"name":"Orchard.Grafts.HitFlash.After","kind":"after","priority":0,"observe":false,"promoted":true}],"example":"[After(typeof(HitPresenter), nameof(HitPresenter.Play))]\npublic static void After(HitPresenter presenter, HitEvent e, ref Call<float> c) { }"},"next_step":"dotnet scion graft new MyGraft --after Orchard.Presentation.HitPresenter.Play"}
```

`anchor list` returns `total` and `anchors[]` with `id`, `anchor`, `target`, `signature`, `slot`, `doc` and
`promoted` ([Anchors](/scion/grafts/anchors/) has an example).

## Exit codes

| Code | When |
|---|---|
| 0 | Listed or shown. |
| 2 | No anchor with that id. |
| 3 | No build. |

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| `no_match` | 2 | No anchor matches. | `did_you_mean`; `anchor list --search`. |
| `build_not_found` | 3 | No test build. | `dotnet scion build test`. |
| warning `name_at_run_time` | 0 | A graft names its target with a string built at run time; it cannot be listed. | Use `nameof` or a literal. |

## Examples

### Every camera anchor

```sh
dotnet scion anchor list --type Orchard.Presentation.CameraRig
```

### Show by method name

```sh
dotnet scion anchor show Orchard.Presentation.Hud.ShowTurn
```

## See also

- [Anchors](/scion/grafts/anchors/)
- [graft](/scion/cli/graft/)
- [Grafts API](/scion/reference/api/grafts/)
