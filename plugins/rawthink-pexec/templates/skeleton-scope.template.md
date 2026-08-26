# {project} — Skeleton Scope

<!-- TEMPLATE-HEADER -->
Scaffolded by `/greenfield-init` from `skeleton-scope.template.md`.
Filled by a person, before the skeleton is built.
DELETE THIS BLOCK, INCLUDING BOTH MARKERS, WHEN GENERATING.
<!-- /TEMPLATE-HEADER -->

**STATUS: PENDING**

Written ONCE, per project, before any code exists. It answers one question —
**which of the planned capabilities exist in the first skeleton, and as what**
— and it is the only record of what was deliberately left out.

Two things read this file. `/plan-init` refuses to run while it is `PENDING`,
and takes the dependency direction from `## Modules created` into the BLOCKING
rules. Everything else in it is written and read by people.

The signature is not ceremony. The `## Left behind deliberately` list is worth
exactly what its signature is worth: an unsigned omission list is
indistinguishable from one nobody wrote.

## Why this exists

A skeleton is an extraction: you take an architecture described in documents
and draw module boundaries from it. Extraction is where content dies quietly.
A capability gets summarised into a neighbouring one, a constraint is assumed
to be "obvious later", and nothing downstream knows either happened.

The plan and the derivation document argue *what* the system does. This file
records *what exists on day one and what does not*, one row at a time, so that
the difference is reviewable instead of remembered.

**It is also the last honest moment.** Once the skeleton is built, its module
boundaries become the live source that plan mode derives from, and every
milestone after this one inherits them. A boundary drawn wrong here is not a
bug; it is a premise.

---

## Capability map

One row per capability named in the milestone map or the derivation document.

| Capability | Layer / module | Open / closed | In the skeleton as | Which milestone |
|---|---|---|---|---|
| {what the system must be able to do} | {module} | {side} | {see below} | {v1, v2...} |

**The "In the skeleton as" column takes exactly three values:**

- **`interface`** — the type and signature exist; the body is real but minimal
- **`walking`** — part of the end-to-end slice that runs and is tested
- **`absent — {milestone}`** — not in this skeleton, and the milestone that
  brings it is named

There is no fourth value. A row reading "later" or "TBD" is a row that will be
lost: nothing downstream distinguishes it from a capability nobody ever wrote
down.

**Every row needs a milestone.** A capability owned by no milestone is either
out of scope — in which case it belongs below, not here — or it is work that
will surprise someone.

## The walking slice

One paragraph. The thinnest path through the system that a test can walk, end
to end, using one real example of each thing it touches.

Name it concretely: which input, which single rule or transformation, which
output, which assertion. "A minimal working pipeline" is not a slice; it is a
wish.

**Choose the slice so that it exercises the architecture's central claim.** If
the product rests on a thesis — that one definition can drive two directions,
that a decision can be replayed, that a boundary holds under load — the slice
is where that thesis first has to survive contact with a compiler. Discovering
it does not work is cheap here and expensive in phase 5.

## Modules created

| Module | Purpose | Open / closed | Depends on | Must never import |
|---|---|---|---|---|

The last column is what `/plan-init` turns into a BLOCKING rule. Leave it empty
only if the project genuinely has no one-way boundary.

**Create no empty modules.** A module with no capability mapped to it in this
milestone is a directory that signals more than it delivers, and it will be
read as an intention by everyone who opens the repo. Create it in the
milestone that needs it.

**State the dependency direction and keep it enforceable.** If the project
splits along a licensing or distribution boundary, that split is a build-level
constraint, not a README sentence — a dependency that runs the wrong way is
found at publication time, when it is a legal problem rather than a design one.

**Check test scope too.** The walking slice needs a real implementation of
everything it touches, and the nearest one is often on the other side of the
boundary. Wiring it into the open module's test is the first violation almost
every split project makes, and no build check that only inspects main sources
will see it. Give the open side a test double instead — writing one is also the
first proof that the boundary is an interface and not a habit.

## Integration unknowns

Questions that must be settled BEFORE the module they affect is drawn. Each one
is an integration fact about a dependency, not a domain fact — vendor library
API surface, canonicalisation guarantees, storage-level constraints, how two
toolchains coexist in one build.

| Unknown | Which module it blocks | How it will be settled |
|---|---|---|

Settle these by running something. A dependency's documentation describes what
it intends to expose; only calling it shows what it actually does, and the
skeleton is where that difference becomes structural.

An unknown that is still open when its module is drawn does not disappear — it
becomes a design decision made on a guess. Move it to the design inventory's
unverified-assumptions section so that pre-flight checks it before phase 1.

## Config this skeleton must be able to answer

The skeleton is finished when `.pexec.yml` can be filled from it, without
imagination:

- [ ] `project.compile` — the exact command that builds it
- [ ] `project.test` — the exact command that runs its tests, and at least one
      test passes
- [ ] `must_read` — its core type definitions
- [ ] `integration_checkpoints` — its wiring points

If any of these still needs a guess, the skeleton is not finished. These four
answers are what every later phase is validated against.

## Left behind deliberately

What the plan or derivation document names that this skeleton does not carry,
one line each, with the reason.

- {capability} — {why not now}

An omission with no reason returns as a proposal in three months, and nobody
can tell whether it was considered and declined or simply missed.

---

**Completion:** set STATUS to COMPLETE above, with date and who filled it.
