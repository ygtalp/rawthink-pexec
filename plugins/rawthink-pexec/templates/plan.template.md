# {milestone} — {Milestone Title}

<!-- TEMPLATE-HEADER -->
Two uses:

1. **As a target for plan mode.** Point Claude Code's plan mode at this file and
   ask it to produce a milestone plan in this shape. The output then passes
   `/plan-init` validation on the first try.
2. **As the shape `/plan-init` validates.** Step 2 of that command checks a plan
   against the four requirements below.

DELETE THIS BLOCK, INCLUDING BOTH MARKERS, WHEN GENERATING.
<!-- /TEMPLATE-HEADER -->

**Date:**
**Scope:** one or two sentences — what this milestone changes, and what it
deliberately does not.
**Research base:** the derivation document this plan distils, written from
`derivation-doc.template.md`. Name it, then leave it behind: it is a derivation
document, not an execution artifact, and nothing in the pipeline reads it again.
Its unverified-assumptions and known-traps sections are consumed below.
**Design inventory:** greenfield first milestones only — the inventory this
plan's anchors come from, written from `design-inventory.template.md`. Same
status: named, consumed once, never read again. Omit this line on a project
that already had source.
**Current state:** where the codebase is today, in the terms this milestone
will change.

> **This plan contains no implementation code.** `/spec-create` re-derives code
> from live source for every phase. A plan carrying code competes with the
> source and wins arguments it should lose, because the source moved and the
> plan did not.

---

## Context — why this milestone

The one-paragraph argument. What is wrong or missing today, why it is worth a
milestone rather than a single task, and what "done" buys you.

State any principle that governs every phase here, once, instead of repeating
it per phase.

---

## Architecture: {N} tiers, {M} phases

```
Tier 1: {name}   ({what it establishes})     Phases 1-3
    ↓
Tier 2: {name}   ({what it adds})            Phases 4-6
    ↓
Tier 3: {name}   ({what it closes})          Phases 7-9
```

Tiers are optional for short milestones. Use them when phases group naturally
and the groups have an order.

**Ordering rules that must not be broken** — state them explicitly, with the
reason. For example: a measurement phase must precede the phase that changes
what it measures, or the before/after is meaningless.

---

## Unverified assumptions

Claims this plan rests on that nobody has actually checked. From the derivation
document, plus any `epistemic: hypothesis` decisions carried in from the
previous milestone's review.

| Assumption | How to check it | What breaks if false |
|---|---|---|
| {claim} | {the command or test that settles it} | {what this plan gets wrong} |

`/plan-init` copies this table into pre-flight Step 3. Each entry is checked
before phase 1 runs.

An empty table means every claim in this plan was verified against something
that executes. That is rare. If it is empty because nobody looked, pre-flight
has nothing to check and the first false assumption surfaces in phase 6.

---

## Pre-flight (once, manual, outside the phase loop)

Work that must happen before phase 1 and cannot be redone afterwards: backups,
branch creation, snapshots, and above all **baseline measurements**.

If this milestone fixes a correctness or performance defect, capture the
"before" now. Once the fix lands, "how broken was it?" has no answer.

`/plan-init` generates a separate `preflight.md` from this section.

---

## TIER 1 — {name} (Phases 1-3)

**Source:** which findings from the research base this tier addresses.
**Goal:** what is true after this tier that was not true before.

### Phase 1: {Short Imperative Title}

- **Source:** the specific finding, with a file and symbol anchor. Point at
  real code — `ClassName.methodName()` in `path/to/file` — not at a general
  area.
  **Write anchors in a form that can be checked**, not just read:
  - a **path anchor** is a backticked token containing `/` and an extension —
    `` `src/main/java/com/x/Ledger.java` ``
  - a **symbol anchor** is a backticked identifier, the word `in`, then a path
    anchor — `` `Ledger.append()` `` in `` `src/main/java/com/x/Ledger.java` ``.
    The qualifier and `()` are for the reader; the check matches on the last
    segment, so write the form that reads clearly

  A bare type name with no path is not an anchor. It reads like one, survives
  every check, and resolves to nothing.

  **When the phase creates a type that does not exist yet** — normal on a
  greenfield first milestone — there is no real code to point at, and inventing
  one passes the plan checks and fails in `/spec-create` Step 5. Write the
  anchor in two parts instead: the design inventory row the type comes from,
  and a path anchor for the real file this phase attaches it to. If neither
  exists, the phase is not ready to be planned.
- **Requirements:** the map IDs this phase covers, or `none — infrastructure`.
  Every ID the map assigned to this milestone appears against at least one
  phase; `/plan-init` counts them and stops on a gap. An infrastructure phase
  that covers no requirement says so explicitly — the difference between
  "covers nothing" and "forgot to write it" is not recoverable later.
- **Depends on:** phases this one needs, or "none". A dependency that finished
  `halted` blocks this phase — see the halted note below.
- **Goal:** one sentence. What is true when this phase is done.
- **What to do:**
  - The change, described at the level of intent, not implementation.
  - Constraints that must hold — what must keep working unchanged.
  - Tests or validation this phase must add.
- **Key files:** the files `/spec-create` should read first. Being generous
  here is cheap; being wrong is not.
- **⚠️ Traps:** anything known to go wrong in this area. These seed the lessons
  file when `/plan-init` runs.
- **Validation:** measurable criteria, 15+ preferred. Each one either passes or
  fails — "works correctly" is not a criterion.
- **Must NOT:** the prohibitions from `{core_rules}` that apply to this phase,
  each written as a negative criterion the verifier can check. A prohibition
  that is never turned into a criterion is a rule nobody runs.
- **Clarity check:** four yes/no questions, answered honestly. **Any `no`
  becomes a row in `## Unverified assumptions`** — that is the whole mechanism;
  the questions exist to route ambiguity into the assumption pipeline rather
  than to produce a grade.
  - *Goal* — is it a state change with a before and an after, rather than
    "improve X"?
  - *Boundaries* — is there an explicit out-of-scope line for this phase?
  - *Constraints* — named, or explicitly "none beyond project conventions"?
  - *Validation* — does every criterion resolve to pass or fail, with no
    "works well" or "looks reasonable"?
- **Fix-or-cut:** for risky phases, what happens if it cannot be finished this
  cycle. An honestly removed feature is defensible; a half-finished one is not.

### Phase 2: {Title}

(same shape)

---

## A dependency that halted

A phase can finish `halted`: it ran, it answered its question, and the answer
means the work it was gating cannot proceed. A spike that returns "no" is the
clearest case. **That is a success, not a failure** — the phase did exactly
what it was for.

Halted propagates over `Depends on`. Every phase that depends on a halted
phase, directly or through a chain, is **blocked**: `/spec-create` stops rather
than writing a spec for work whose premise just failed, and `/milestone-close`
reports it as blocked rather than as ordinary incomplete work.

**Plan for it where the risk is real.** If a phase exists to answer a question
that could come back "no", say in its `Fix-or-cut` what the milestone does
then. A blocked chain discovered at close is a surprise; a blocked chain with a
written answer is a decision.

## Phase Execution Order

```
Phase 1  {short label} ─┐
Phase 2  {short label}  │ (independent — any order)
Phase 3  {short label} ─┘
    ↓ (Tier 1 complete)
Phase 4  {short label}   (needs 1)
    ↓
Phase 5  {short label}   (needs 4)
```

Say where you can stop and still have something coherent. A milestone you can
abandon halfway is worth more than one you cannot.

---

## Verification Strategy

**Per phase:** what `/spec-implement` runs to prove a phase is done — the exact
compile and test commands.

**Milestone level:** what proves the whole thing worked. Full build, an
end-to-end check, a grep that must come back empty, a measurement compared
against the pre-flight baseline.

---

## Success Criteria

Numbered, checkable at the end. Written so someone else could verify them
without asking you what you meant.

1.
2.

---

## Out of pipeline

Work that shares this milestone's calendar but is not planned here.

| Work | Estimated load | Why it is not planned here |
|---|---|---|

Requirements that are not knowable at planning time cannot be specified, and
`/post-phase` validating against a plan written blind validates nothing. Naming
the work keeps it out of the pipeline while keeping the calendar honest.

Leave empty if the milestone owns all of its weeks.

---

## Risk Assessment

| Risk | Mitigation |
|---|---|
| {what could go wrong} | {what makes it survivable} |

---

## Format & Guardrails (external)

Spec format, violation tiers and the empirical trap list live outside this
plan, so it stays purely the plan:

- `{core_rules}` — spec file format, summary format, the ⛔/⚠️/✅ tiers
- `{lessons}` — code-anchored traps, L-coded, that `/spec-create` must not
  reproduce and `/spec-verify` treats as blocking

Both are generated by `/plan-init`.
