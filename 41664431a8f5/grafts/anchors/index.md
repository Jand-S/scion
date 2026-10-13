---
title: "Anchors"
description: "Find which game methods take grafts, give them stable ids with Anchor, and give them slots in test builds with AnchorScope."
type: guide
url: https://jand-s.github.io/scion/41664431a8f5/grafts/anchors/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/grafts/anchors.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/grafts/write.md
  - https://jand-s.github.io/scion/41664431a8f5/cli/anchor.md
  - https://jand-s.github.io/scion/41664431a8f5/reference/api/grafts.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# Anchors

An anchor is a game method the weaver rewrites so grafts run around it. This guide covers which methods can be
anchors (SCN105), the generated anchor ids (`Anchors.Id`), stable names and descriptions with `[Anchor(Id, Doc)]`,
slots without a graft in test builds with `[assembly: AnchorScope(typeof(T), Members = ...)]`, and the commands that
find them: `dotnet scion anchor list` and `anchor show`. Errors: SCN105, SCN106, SCN107.

## When to use

- Use `anchor list` before writing any graft, to find the method to hang it on and its exact signature.
- Use `[Anchor]` on the presentation methods agents should find first: it gives them a stable id and a one-line doc.
- Use `AnchorScope` on presentation types where new grafts will be injected live, so a draft can bind there without
  a rebuild.

## Steps

### 1. Search the anchors of the build

```sh
dotnet scion anchor list --search hit
```

<!-- validate: cli/anchor-list -->
```json
{"schema_version":1,"command":"anchor list","ok":true,"build":"/Users/alex/orchard/.scion/builds/test/Orchard.app","total":2,"anchors":[{"id":"A__Orchard_Presentation_HitPresenter__Play__HitEvent","anchor":"presentation.hit","target":"Orchard.Presentation.HitPresenter.Play","signature":"float Play(HitEvent e)","slot":true,"doc":"Plays one hit and returns how long it lasts.","promoted":["Orchard.Grafts.HitFlash.After"]},{"id":"B__Orchard_Presentation_HitPresenter__Play__HitEvent","anchor":"presentation.hit","target":"Orchard.Presentation.HitPresenter.Play","signature":"float Play(HitEvent e)","slot":true,"doc":null,"promoted":[]}],"next_step":"create a graft: dotnet scion graft new MyGraft --after Orchard.Presentation.HitPresenter.Play"}
```

`slot: true` means the method has a slot in this test build, so a draft graft can be injected there. `promoted` lists
the grafts already compiled into the game.

### 2. Name the important anchors

```csharp
namespace Orchard.Presentation
{
    public sealed class HitPresenter : MonoBehaviour
    {
        [Anchor(Id = "presentation.hit", Doc = "Plays one hit and returns how long it lasts.")]
        public float Play(HitEvent e) { /* ... */ }
    }
}
```

`[Anchor]` is metadata for tools: the weaver ignores it, so it neither creates an anchor nor changes the IL. The id's
first segment (`presentation`) is the category the project index counts.

### 3. Give presentation types slots

```csharp
[assembly: Scion.AnchorScope(typeof(Orchard.Presentation.HitPresenter))]
[assembly: Scion.AnchorScope(typeof(Orchard.Presentation.Hud), Members = new[] { "ShowTurn", "ShowDamage" })]
[assembly: Scion.AnchorScope(typeof(Orchard.Presentation.CameraRig))]
```

Put these in any file that compiles into the assembly that declares the types (the grafts folder works). Rebuild the
test player; `anchor list` then shows `slot: true` for every eligible method of those types.

## Reference

### Automatic anchors

Every method named by a `[Before]` or `[After]` becomes an anchor in every build. In test builds (`SCION_SLOTS`),
every eligible method covered by an `AnchorScope` also gets a slot, with or without a graft. The final build ignores
`AnchorScope`: there, only methods with a promoted graft change.

### Methods that cannot be anchors

A graft on one of these is [SCN105](/scion/reference/diagnostics/grafts/#scn105), with the reason; `AnchorScope`
skips them silently:

- constructors, abstract methods and methods without a body;
- generic methods and methods of generic types;
- methods of a struct;
- iterators and `async` methods (the anchor would only catch the creation of the state machine);
- Netcode for GameObjects RPCs (their prologue returns early on the sending side);
- methods with `[RuntimeInitializeOnLoadMethod]` or `[BurstCompile]`, compiler-generated methods and names that
  start with `__`;
- methods with an `out` or pointer parameter, or that return by reference, a ref struct or a pointer;
- methods of private or protected nested types, or whose parameter or return types generated code cannot access.

Hang the graft on the plain method the iterator, `async` method or RPC calls.

### Anchor ids

Each side of an anchor has an id: `B__` (before) or `A__` (after), the type's full name, the method name and the
simple names of the parameter types, with every character outside `[A-Za-z0-9_]` turned into `_`. `Anchors.Id`
computes it the same way the weaver and the injection do:

```csharp
Anchors.Id(true, "Orchard.Presentation.HitPresenter", "Play", new[] { "HitEvent" })
// "A__Orchard_Presentation_HitPresenter__Play__HitEvent"
```

Overloads whose ids collide (for example `List<int>` and `List<string>` parameters) are
[SCN106](/scion/reference/diagnostics/grafts/#scn106) when a graft targets them, and get no slot in a scope.

### AnchorScope rules

| Rule | Detail |
|---|---|
| Where | In the assembly that defines the type; elsewhere it is [SCN107](/scion/reference/diagnostics/grafts/#scn107). |
| `Members` omitted | Every eligible method of the type, both sides. |
| `Members` given | Only those names; an empty array gives none. Use compiled names: a property getter is `get_Name`. |
| A name that does not exist | [SCN103](/scion/reference/diagnostics/grafts/#scn103) with the closest name. |
| A name whose overloads are all ineligible or collide | SCN105 or SCN106 (no file and line: it comes from an assembly attribute). |
| Several attributes for one type | They add up. |
| Final build | Ignored. |

### Commands

| Command | Use |
|---|---|
| `dotnet scion anchor list [--search <text>] [--type <Type>]` | Eligible anchors of the build, with slot, `[Anchor]` id, doc and promoted grafts |
| `dotnet scion anchor show <id>` | One anchor: signature, grafts in run order, a ready graft declaration |

`<id>` is an `[Anchor]` id (`presentation.hit`), a slot id (`A__...`) or `Type.Method`. See
[anchor list, anchor show](/scion/cli/anchor/).

## Errors

| Code | Exit | Cause | Next action |
|---|---|---|---|
| `SCN105` | compile error | The target cannot be an anchor. | Hang the graft on the method it calls. |
| `SCN106` | compile error | Overloads collide on one anchor id. | Rename one overload or change a parameter type name. |
| `SCN107` | compile error | `AnchorScope` names a type of another assembly. | Move the attribute into that assembly. |
| `SCN113` | 1 (`graft build`, `graft inject`) | The draft targets a method with no slot in this build. | Add its type to `AnchorScope`, rebuild, inject again. |
| `no_match` | 2 | `anchor show` found no anchor with that id. | Use `did_you_mean`, or `anchor list --search`. |

## Examples

### Every anchor of one type

```sh
dotnet scion anchor list --type Orchard.Presentation.CameraRig
```

### A scope limited to two methods

```csharp
[assembly: Scion.AnchorScope(typeof(Orchard.Presentation.DamageNumbers), Members = new[] { "Spawn", "SpawnHeal" })]
```

## See also

- [anchor list, anchor show](/scion/cli/anchor/)
- [Write a graft](/scion/grafts/write/)
- [Live injection](/scion/grafts/injection/)
- [Grafts API](/scion/reference/api/grafts/)
