---
title: "SCN101-SCN114"
description: "Weaver and graft analyzer diagnostics - method shape, assembly, missing targets, signatures, ineligible anchors, Call usage, overloads, slots and redirects."
type: diagnostics
url: https://jand-s.github.io/scion/41664431a8f5/reference/diagnostics/grafts/
markdown_url: https://jand-s.github.io/scion/41664431a8f5/reference/diagnostics/grafts.md
version: 1.0.0
related:
  - https://jand-s.github.io/scion/41664431a8f5/grafts/write.md
  - https://jand-s.github.io/scion/41664431a8f5/grafts/call-context.md
  - https://jand-s.github.io/scion/41664431a8f5/grafts/anchors.md
---

> Scion 1.0.0 documentation. Index of every page: https://jand-s.github.io/scion/41664431a8f5/llms.txt

# SCN101-SCN114

These diagnostics come from the weaver when Unity compiles, from the analyzers in the IDE, and from `graft check`,
`graft build` and `graft inject` outside Unity. All but SCN111 are errors: the assembly does not compile (or the
draft is not injected) until they are fixed.

## Diagnostics

### SCN101

**Severity:** error · **Source:** weaver, analyzer

**Message:** `Graft {0} must be static void, public or internal, and not generic.` **Hint:** `Declare it as: public
static void {0}(...) in a non-generic class.`

**Cause.** The method marked `[Before]` or `[After]` is an instance method, returns a value, is private or protected,
is generic, or is declared in a generic type or a private nested type. Injected drafts must also be `public` in a
`public` type.

**Fix.**

```csharp
// Before
[After(typeof(HitPresenter), nameof(HitPresenter.Play))]
void After(HitPresenter presenter) { }
// After
[After(typeof(HitPresenter), nameof(HitPresenter.Play))]
public static void After(HitPresenter presenter) { }
```

**Suppress.** Not allowed.

### SCN102

**Severity:** error · **Source:** weaver, analyzer

**Message:** `Target {0} is outside assembly {1}; a graft only weaves targets of its own assembly.` **Hint:** `Put the
graft in a folder with an .asmref to the target's assembly.`

**Cause.** The graft compiles into another assembly than its target, so the weaver cannot rewrite the target.

**Fix.** Move the file into the grafts folder with `{"reference":"Orchard.Runtime"}` in its `.asmref`. In a draft
project compiled outside Unity, turn SCN102 off with `dotnet_diagnostic.SCN102.severity = none`.

**Suppress.** Only in draft projects, as above.

### SCN103

**Severity:** error · **Source:** weaver, analyzer, draft compiler

**Message:** `{0}.{1} does not exist.` **Hint:** `Did you mean '{2}'?`

**Cause.** The target type has no method with that name declared on it (inherited methods do not count).

**Fix.** Use the suggested name, or target the type that declares the method.

```csharp
// Before
[After(typeof(HitPresenter), "Ply")]
// After
[After(typeof(HitPresenter), nameof(HitPresenter.Play))]
```

**Suppress.** Not allowed.

### SCN104

**Severity:** error · **Source:** weaver, analyzer, draft compiler

**Message:** `The signature of {0} does not match {1}.{2}.` **Hint:** `Expected: {3}`

**Cause.** The graft's parameters match no overload of the target: the instance, the target's parameters in order
with the same `in`/`ref`, then optionally `ref Call` or `ref Call<T>`, last.

**Fix.** Copy the expected signature from the hint, or declare a prefix of it.

```csharp
// Before
[After(typeof(HitPresenter), nameof(HitPresenter.Play))]
public static void After(int e) { }
// After
[After(typeof(HitPresenter), nameof(HitPresenter.Play))]
public static void After(HitPresenter presenter, HitEvent e) { }
```

**Suppress.** Not allowed.

### SCN105

**Severity:** error · **Source:** weaver, draft compiler

**Message:** `{0}.{1} cannot have an anchor: {2}.` **Hint:** `Hang the graft on a plain method instead (iterators,
async, RPCs, structs and out parameters have no anchor).`

**Cause.** The target is a constructor, abstract, generic, in a struct, an iterator, `async`, a Netcode RPC, has an
`out` or pointer parameter, returns by reference, or is otherwise ineligible ([Anchors](/scion/grafts/anchors/)).

**Fix.** Target the plain method the iterator, `async` method or RPC calls.

**Suppress.** Not allowed.

### SCN106

**Severity:** error · **Source:** weaver

**Message:** `Overloads of {0}.{1} collide on anchor id {2}.` **Hint:** `Rename one overload or give its parameter
types distinct names.`

**Cause.** Two overloads produce the same anchor id because their parameter types have the same simple names
(`List<int>` and `List<string>`).

**Fix.** Rename one overload, or wrap a parameter in a type with a distinct name.

**Suppress.** Not allowed.

### SCN107

**Severity:** error · **Source:** weaver

**Message:** `[assembly: AnchorScope(typeof({0}))] in {1} points outside the assembly.` **Hint:** `Declare the scope
in the assembly that defines {0}.`

**Cause.** An `AnchorScope` names a type of another assembly.

**Fix.** Move the attribute to a file that compiles into the type's assembly (the grafts folder works).

**Suppress.** Not allowed.

### SCN108

**Severity:** error · **Source:** weaver, analyzer

**Message:** `{0} calls {1}, but only a [Before] that is not an observer may skip.` **Hint:** `Use a [Before] without
Observe = true, or just read c.Skipped.`

**Cause.** `Skip()` or `Return()` in an `[After]` or an observer.

**Fix.** Move the skip to a non-observer `[Before]`.

**Suppress.** Not allowed.

### SCN109

**Severity:** error · **Source:** weaver, analyzer

**Message:** `{0} takes {1}, but {2}.{3} returns {4}.` **Hint:** `Use ref Call<{4}> (or ref Call when the target
returns void).`

**Cause.** `Call<T>` with a `T` other than the return type, or `Call` on a target that returns a value.

**Fix.**

```csharp
// Before
public static void After(HitPresenter presenter, HitEvent e, ref Call<int> c) { }
// After
public static void After(HitPresenter presenter, HitEvent e, ref Call<float> c) { }
```

**Suppress.** Not allowed.

### SCN110

**Severity:** error · **Source:** analyzer

**Message:** `{0} calls graft {1} directly; only the weaver calls grafts.` **Hint:** `Move the shared code to a normal
helper and call that.`

**Cause.** Game code refers to graft code. Deleting the grafts folder must leave a game that compiles.

**Fix.** Move the shared code out of graft code. Test assemblies are exempt.

**Suppress.** Not allowed.

### SCN111

**Severity:** warning · **Source:** weaver

**Message:** `Grafts {0} on anchor {1} all use Call.Tag; they share one value.` **Hint:** `Let one graft own Tag, or
keep the state in your own field.`

**Cause.** Two or more befores, or two or more afters, of one anchor read or write `Tag`.

**Fix.** Let one pair own `Tag`.

**Suppress.** `#pragma warning disable SCN111` when the sharing is intended.

### SCN112

**Severity:** error · **Source:** weaver, analyzer

**Message:** `{0} matches more than one overload of {1}.{2} by prefix: {3}.` **Hint:** `Declare all parameters so
exactly one overload matches.`

**Cause.** The graft declares a prefix of parameters that fits several overloads.

**Fix.** Declare the full parameter list of the overload you want.

**Suppress.** Not allowed.

### SCN113

**Severity:** error · **Source:** draft compiler (`graft build`, `graft inject`)

**Message:** `{0}.{1} has no slot in this build.` **Hint:** `Add its type to [assembly: AnchorScope], run dotnet scion
build test and inject again.`

**Cause.** A draft targets a method that has neither a promoted graft nor an `AnchorScope` in this test build.

**Fix.** Add `[assembly: AnchorScope(typeof(T))]`, rebuild the test player, inject again.

**Suppress.** Not allowed.

### SCN114

**Severity:** error · **Source:** weaver

**Message:** `Redirect target {0}.{1} does not exist in Scion.Runtime.` **Hint:** `Keep the redirect targets in
Scion.Runtime under SCION_TRIALS, or fix TrialsReads in ScionWeaver.cs.`

**Cause.** The weaver and `Scion.Runtime` come from different package versions, so a redirect target is missing.

**Fix.** Reinstall the package so all its assemblies match.

**Suppress.** Not allowed.

## See also

- [Write a graft](/scion/grafts/write/)
- [Call and Call\<T\>](/scion/grafts/call-context/)
- [Anchors](/scion/grafts/anchors/)
- [Diagnostics](/scion/reference/diagnostics/)
