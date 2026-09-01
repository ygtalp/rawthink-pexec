# {milestone} — Design Inventory

<!-- TEMPLATE-HEADER -->
Scaffolded by `/greenfield-init` from `design-inventory.template.md`.
Filled after the skeleton exists, before plan mode runs.
DELETE THIS BLOCK, INCLUDING BOTH MARKERS, WHEN GENERATING.
<!-- /TEMPLATE-HEADER -->

**STATUS: PENDING**

**Skeleton scope:** {the scope map this is drawn from}
**Derivation document:** {the milestone's argument}
**Live source read:** {the skeleton modules this inventory was derived against}

Written ONCE, for the first milestone of a greenfield project, between the
derivation document and plan mode.

`/plan-init` refuses to run while this file is `PENDING`, and checks that every
anchor in the plan resolves against real files. It looks for this file by
presence, not by milestone number: writing one for a later milestone — a new
module that starts empty, a closed-source counterpart — turns the check back on
for that milestone by itself.

## Why this exists

A plan phase is expected to anchor: *"Source: `ClassName.methodName()` in
`path/to/file`"*. On a brownfield project that anchor is free — plan mode reads
live source, sees the existing structure, and names the target types by
transforming it. The design is derived, not invented.

On a greenfield project the first milestone has almost nothing to transform.
The skeleton supplies boundaries and a walking slice; everything the milestone
adds beyond that slice has no source to be derived from. Plan mode will either
invent anchors or write phases with none, and both fail later — the first in
`/spec-create` Step 5, the second as a spec that cannot say what it is
building.

This file is that design, produced deliberately and once, so that plan mode has
something real to point at.

**This is a planning input, not an execution artifact.** Nothing in the
pipeline reads it after `/plan-init`. By phase 3 the live source contains what
phases 1 and 2 built, and `/spec-create` derives from that — at which point
this document is stale by construction. It is written to be left behind, like
the derivation document above it.

**No bodies.** Type names, signatures, field names and file paths — yes. An
algorithm, a condition, a loop — no. A document carrying implementation
competes with the source and wins arguments it should lose. The rule is the
same one the plan follows; the line sits between *design* and *implementation*,
not between *prose* and *code*.

---

## 1. Modules

| Module | Layer | Open / closed | Exists in skeleton |
|---|---|---|---|

Only modules this milestone touches. A module nothing is assigned to does not
belong here yet.

## 2. Type inventory

The core of this document.

| Type | Module | Role | From which planned capability | CREATE / MODIFY | Phase |
|---|---|---|---|---|---|

**Three rules, each closing a specific failure:**

1. **Every type gets a phase.** A row with an empty phase column is a type
   nobody asked for. It will be built anyway — by whichever phase happens to
   need something like it — and then built again, differently, by the next one.
2. **Every type traces to a planned capability.** The column is not decoration:
   a type that traces to nothing is scope that entered through design rather
   than through planning, and no review step downstream will catch it.
3. **MODIFY means the skeleton already has it.** CREATE means this milestone
   introduces it. Getting this wrong sends a phase looking for a file that is
   not there.

## 3. Core signatures

Interfaces and abstract types only — not the internal methods of concrete
classes. Those are `/spec-create`'s work, derived from source at the time.

| Signature | Module | What it does |
|---|---|---|
| `{return} {name}({params})` | {module} | {one sentence} |

Include a signature here only if something else in this milestone has to
compile against it. That test keeps the section from becoming a second, worse
copy of the codebase.

**Where a decision is still open, keep the signature agnostic to it** and say
which open decision it is dodging. A parameterised interface costs one
indirection now; a hardcoded choice costs every caller when the decision lands
the other way.

## 4. Field lists

For record, message and configuration types whose shape is fixed by the
milestone's argument rather than by implementation convenience.

**{Type name}**

| Field | Type | Constraint |
|---|---|---|

Copy shapes the plan or the derivation document already states, exactly. Where
one of those documents names a record's fields, that list is the contract — an
inventory that quietly drops a field from it has changed the product.

**A field that another part of the system will version, calibrate or replay
against gets its own name.** Folding it into a general-purpose "context" or
"trace" field makes it unqueryable at exactly the moment it matters: when
something changed and a past result has to be reconstructed as it was.

## 5. File tree

```
Files to create:
  {module}/
    {path}/{Type}.{ext}

Files to modify:
  {path}/{Type}.{ext}      — {what changes}
```

The MODIFY list comes from the skeleton. If a file appears here that the
skeleton does not contain, the inventory and the scope map disagree, and one of
them is wrong.

---

## Design decisions resting on unverified assumptions

**Required.** Copied into the plan's `Unverified assumptions` table, then into
pre-flight Step 3, where each is checked by running something before phase 1.

| Design decision | The assumption under it | Evidence for it | What breaks if false |
|---|---|---|---|

**Evidence is what the assumption currently rests on** — the walking slice test
that passed, the file that shows the pattern, the vendor page that was read.
Write `none` where there is none: an assumption with no evidence is the highest
priority row in pre-flight, and it can only be prioritised if it is visible.

**Every consequence is concrete.** "Could cause issues" is not a consequence;
"`X-core`'s central interface changes and every phase touches it" is. A vague
consequence cannot be weighed against the cost of checking it, so it never gets
checked.

At least one row is expected. A design derived from a skeleton inherits every
assumption the skeleton made — starting with the assumption that whatever the
walking slice proved at one example also holds at the scale the phases need.

A design with no assumptions listed has not been examined; it has been
transcribed.

## Left behind deliberately

What this milestone's scope names that the inventory does not carry, one line
each, with the reason.

- {capability} — {why not}

---

**Completion:** set STATUS to COMPLETE above, with date and who filled it.
