# {project} — Milestone Map

**Version:** {n}
**Date:**
**Covers:** {period, or "until X ships"}

The layer above the pipeline. It answers one question — **which milestones
exist, in what order, and what constrains them** — and hands each one to a
derivation document.

No command reads this file. Like the derivation document, it is written and read
by people. It is here because the pipeline needs an upstream that holds still,
and because `/milestone-close` produces findings that have to land somewhere.

**Keep the explanatory prose when you fill this in.** This template has no
`TEMPLATE-HEADER` block to strip: every section below, including the short
paragraphs under each heading, is part of the artifact. They are what make the
file usable by someone who did not write it — and by you, a year later, when
the reason a column exists has stopped being obvious. Only the `{placeholder}`
text gets replaced.

## Why this exists

A derivation document scopes ONE milestone. Nothing scopes the sequence.

Without this layer, "what comes after v1" gets answered from memory or from a
strategy document that was written before v1 started and never revised. What
`/milestone-close` learned — a deviation, an assumption that turned out false, a
question the milestone raised and did not answer — has nowhere to go, and the
next milestone is planned as if none of it happened.

This file is where it goes. Update it after every close, before writing the next
derivation document.

**Keep it a map, not a strategy.** Non-engineering work is named here and not
elaborated: budgets, hiring, legal, marketing belong in whatever document you
already keep for them. The moment this file starts arguing a business case it
stops being readable as a map, and the milestone rows get buried.

---

## Purpose

Two or three sentences. What is true when the last milestone on this map ships,
that is not true today. Every milestone below is a step toward this, or it does
not belong here.

## Verified facts

Constraints from outside the codebase that shape the sequence. Sourced.

| Fact | Source | What it constrains |
|---|---|---|
| {fact} | {where it was verified} | {which milestone, or the ordering} |

Facts, not assumptions. An unverified claim is not a constraint yet — it belongs
in the milestone's derivation document, where pre-flight can check it.

## Lanes

Which streams of work exist, and which one the pipeline covers.

| Lane | What it is | In pipeline |
|---|---|---|
| {A} | {description} | no |
| {B} | {engineering} | **yes** |

Exactly one lane should be marked in-pipeline. If two are, they are two projects
sharing a repo, and the milestones below cannot be ordered against each other.

The out-of-pipeline lanes appear in each milestone's derivation document under
"work that shares the calendar" — named, estimated, not planned.

## Capacity unit

**One engineer-week = {N} hours.**

State this even when it feels obvious. The estimates below are meaningless
without it: "8 engineer-weeks" over 9 calendar weeks is comfortable at 15 h/wk
and impossible at 40.

Not a ceiling. More available time shortens the calendar; less stretches it.
**Scope is fixed, the calendar flexes.**

Recalibrate against real hours after the first milestone and update this line.
An estimate that is never checked against a measurement is a guess with a number
on it.

## Requirements

Every capability this map names, given an ID. **The ID is what makes a lost
capability findable.** A capability described in a paragraph and never given an
ID is a capability that can quietly fail to appear in any milestone — and
nothing will report it, because nothing was counting.

Group by area. Keep IDs stable forever: an ID is a handle, not a position.

### {AREA}

| ID | Requirement | Milestone | Status |
|---|---|---|---|
| {AREA}-01 | {what must be true, checkable} | {v1 \| — } | open \| satisfied \| dropped |
| {AREA}-02 | {…} | | |

**Write one row per capability, not per feature description.** If a paragraph
elsewhere in this map names something the system must be able to do — a
component, a guarantee, a surface — it needs a row here. Component lists,
architecture layers and mandatory-interface lists are the usual sources, and
they are the usual place capabilities go missing: they read as delivery
commitments and are structurally nothing of the sort.

### Coverage

**Count only the tables under `## Requirements`.** The `## Out of scope` table
below uses the same row shape — an ID, then prose — so anything that scans the
whole file for ID-shaped rows counts them too and reports a total that is
larger than the requirement set. Two tables, two counts, and the sum of them is
not a number that means anything.

| | Count |
|---|---|
| Total requirements | |
| Assigned to a milestone | |
| **Unassigned** | **⚠ if > 0** |
| Satisfied | |
| Dropped (with reason below) | |

The counts must reconcile: **assigned + unassigned = total**, and the sum of
the ID counts across every milestone's Requirements cell equals *assigned*. If
those two disagree, one of them was read from a shortened cell — see the
Requirements column rule below.

**An unassigned requirement is the finding.** It is not a to-do; it is a
capability nobody owns. Either assign it to a milestone or move it to
`## Out of scope` with a reason — those are the only two resolutions.

**Dropped requirements keep their row.** Deleting the row loses the fact that
the capability was considered; the ID stays, the status becomes `dropped`, and
the reason goes in Out of scope.

## Out of scope

Explicitly excluded. This is what keeps a dropped requirement from returning as
a proposal in three months.

| Requirement / capability | Reason | Reopen if |
|---|---|---|
| {ID or name} | {why not} | {what would change the answer} |

## Milestones

Each row becomes one derivation document, then one plan, then one `/plan-init`.

| # | Milestone | Delivers | Requirements | Est. | Calendar | Depends on |
|---|---|---|---|---|---|---|
| v1 | {name} | {what is true after} | {AREA-01, AREA-02} | {N} e-wk | {span} | — |

**The Requirements column is the join.** It is what lets `/plan-init` check that
a milestone's plan covers what the map assigned to it, and what lets
`/milestone-close` mark requirements satisfied. A milestone with an empty
Requirements cell delivers nothing that anyone is counting.

**Write every ID in full, comma-separated: `SUB-01, SUB-04, SUB-05`.** Not
`SUB-01,04,05`. The shortened form is the natural thing to write — it is
shorter, it reads fine, and a person parses it without noticing — which is
exactly why it is dangerous: anything scanning for IDs finds one where there
are three, reports full coverage, and the gap this column exists to catch
passes silently. Prefer a long cell to a short one; the cell is read by a
counter more often than by a person.
| v2 | {name} | {what is true after} | {N} e-wk | {span} | v1 |

**Load check:** divide estimate by calendar weeks. A row above 1.0
engineer-weeks per calendar week is overloaded against the unit above, and the
overload is easier to fix here than three months in.

### {v1} — {name}

**Why this one first:** the argument for its position in the sequence, not for
the work itself. The work is argued in its derivation document.

**Out of scope:** what belongs to a later milestone, and which one.

*(repeat per milestone)*

## Critical path

```
{decision} ──▶ {what it unblocks} ──▶ {and then}
     │
     └──▶ {parallel branch}
```

**Can run in parallel:** {list}
**Strictly ordered:** {list}
**The link that breaks most often:** {which, and what it costs}

A dependency you cannot draw is a dependency you have not thought about. Redraw
this after every close; a path that was true at v1 rarely survives to v3
unchanged.

## Open decisions

| # | Decision | Due | If not made | Candidates |
|---|---|---|---|---|
| D1 | {question} | {milestone/date} | {consequence} | {the options} |

**The candidates column is not optional.** A decision whose option set has been
lost cannot be made — you are back to rediscovering the alternatives before you
can choose between them. This is the single most common way an open decision
becomes permanently open.

**If the options are not known yet, that is the finding — write it.**
`unknown — enumerate the options before deciding` is a valid entry and an
honest one; a blank cell is neither. It says the same thing while looking like
an oversight, and the next reader cannot tell whether the options were lost or
never existed.

Record each of these in the decision archive as an open question. This file is
read by people; the archive is what a planning session queries.

## Closed decisions

| # | Decision | Outcome | Because | Closed in |
|---|---|---|---|---|
| D{n} | {question} | {what was chosen} | {the constraint that forced it} | {version} |

Never delete a closed decision — move it here. The `because` column is the whole
point: "we chose X" is recoverable from the code forever, "we did not choose Y,
because Z" exists nowhere else and is the first thing asked when the choice is
questioned.

Record these in the archive too, with `rejected` filled in.

## Risks

| Risk | Early signal | Response |
|---|---|---|

The signal column is what makes this useful. A risk with no observable early
signal is a worry, and worries do not belong on a map.

## Triggered packages

Work that is not scheduled and not cancelled — it happens if something happens.

| Package | Trigger | Cost if triggered |
|---|---|---|

Without this section, conditional work gets deleted as "out of scope" and comes
back as a surprise, or gets scheduled and blocks on a trigger that never fires.

---

## Revision log

Updated after every `/milestone-close`, before the next derivation document.

### {version} — {date}, after {milestone} closed

**From the milestone review:**
- Deviations that change a later milestone: {what, and which row moved}
- Assumptions that turned out false: {what, and what it invalidates}
- Questions still open at close: {carried into which row, or into open decisions}

**Changed:** {rows, estimates, ordering}
**Added:** {new milestones or decisions}
**Removed:** {what, and why — never "cleaned up"}

### Revision rules

Four rules, each earned from a specific way this file decays:

1. **Closing a decision and simplifying the document are separate passes.**
   Tidying a table while recording an outcome is how rationale unrelated to that
   decision gets swept out with it.

2. **Diff by section, not by size.** A map can grow while losing content: new
   sections get added, existing ones compressed to make room, and the byte count
   rises the whole time. Growth is not evidence that nothing was lost.

3. **Rationale goes to the archive, not just here.** Every revision that closes a
   decision writes `decided` / `rejected` / `because` to the decision archive.
   When this file is later compressed — and it will be — the archive still
   answers "why".

4. **Never remove an open decision's candidate set.** Removing the options makes
   the decision unmakeable. If a candidate is eliminated, move it to the closed
   table with its reason; do not delete the row.
