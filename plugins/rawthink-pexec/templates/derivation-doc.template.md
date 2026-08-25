# {milestone} — Derivation Document

**Research base for:** {milestone plan}
**Date:**
**Milestone map:** the map this milestone is drawn from, and which row. Name it,
then leave it behind: take this milestone's row and the lanes marked
in-pipeline, and nothing else. Decisions and open questions come from the
archive, not from the map — the map is read by people, the archive by planning
sessions.
**Sources:** {transcripts, audits, specs, vendor docs — name them}
**Left behind deliberately:** what the map's row mentions that this document
does not carry, and why. One line each.

Written BEFORE the plan, in a session that can search. The plan is derived from
this; nothing in the pipeline reads this file again.

## Why this exists

Plan mode derives structure from live source: file layout, symbol names, the
call graph. It cannot derive what the domain requires, why one approach was
chosen over another, or which claims nobody has checked yet.

That is what this document carries. It is the gap between "here is the
codebase" and "here is what we are going to do to it".

**No code.** Not an example, not a signature, not a schema. A document carrying
code competes with the source and wins arguments it should lose, because the
source moved and the document did not. This document carries argument and
evidence; the spec carries code.

**Record what you left behind.** Extraction is where content dies quietly: a
scope qualifier drops, a constraint gets summarised away, and nothing downstream
knows it existed. The header line above costs one line per omission and makes
the loss reviewable instead of invisible.

Two sections below are not optional, because two things downstream consume them
and nothing else produces them:

- **Unverified assumptions** → the plan's assumptions section → pre-flight
  Step 3
- **Known traps** → `/plan-init` Step 7 → the lessons file

---

## 1. What we are building and why now

One or two paragraphs. What is true after this milestone that is not true
today, and what makes this the right moment.

If the milestone depends on an external date — a regulatory deadline, a
dependency's end of life, a partner's timeline — state it here **and state what
remains true after that date passes**. A milestone justified only by a deadline
has no argument left the day after.

## 2. Domain facts

Verified, with sources. Everything the plan will assume about the world outside
the codebase.

| Fact | Source | Why it constrains us |
|---|---|---|
| {fact} | {where it was verified} | {what it rules in or out} |

Separate what a specification says from what an implementation does. They
disagree more often than either document admits.

## 3. The architectural argument

The claim this milestone rests on, stated so it could be wrong.

**Claim:** {the thesis}

**Alternatives considered and rejected:**

| Alternative | Why not |
|---|---|
| {approach} | {the constraint that eliminated it} |

Fill this in even when the choice feels obvious. "We did not use Y, because Z"
is the thing nobody can reconstruct six months later, and it is the first thing
asked when the choice is questioned. Record each of these in the decision
archive as well — this document is not read again, and the archive is.

## 4. External dependencies

What the milestone builds on that is not ours.

| Dependency | Version | License | What we rely on it for |
|---|---|---|---|

Check the license against the actual artifact, not the project's website. A
dependency that changed license between releases is a discovery you want now,
not in phase 9.

## 5. Known traps

Code-anchored. `/plan-init` seeds the lessons file from this section, and
`/spec-verify` enforces lessons by citing them — **a trap with no code anchor
cannot be cited, and one that cannot be cited is not enforced.**

- {trap} — {mechanism that produces it} — {what to do instead} — anchor:
  `{Class.method()}` in `{path}` or `{dependency API}`

"Be careful with X" is not a trap. Name the mechanism.

## 6. Unverified assumptions

**Required.** Copied into the plan, then into pre-flight Step 3, where each one
is checked before phase 1 runs.

| Assumption | How to check it | What breaks if false |
|---|---|---|

Everything above that came from a document rather than from running something
belongs here. Vendor documentation describes intent; only execution shows
behaviour.

Be specific about how to check. "Verify the API" is not checkable; "call
`{method}` with `{input}` and confirm `{output}`" is.

## 7. Out of scope

What this milestone will not touch, and why. Split into two kinds:

**Deferred** — will be done, in a later milestone. Say which.

**Excluded** — will not be done, with the reason. An exclusion without a reason
comes back as a proposal in three months.

## 8. Work that shares the calendar

Anything that consumes the same weeks but is not in this pipeline — customer
delivery, external audits, work whose requirements are not knowable yet.

| Work | Estimated load | Why it is not planned here |
|---|---|---|

This is not padding. Work whose requirements are unknown at planning time
cannot be specified, and a phase validated against a plan written blind is
validated against nothing. Naming it keeps it out of the pipeline while keeping
the calendar honest.
