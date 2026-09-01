# /milestone-close — Close the milestone

Argument: `/milestone-close`

Run ONCE, in a clean session, after the last phase's `/post-phase`. This step
sits OUTSIDE the phase loop, like pre-flight, and produces no code.

Pre-flight opens the milestone; this closes it. Pre-flight blocks `/spec-create 1`
until a human marks it COMPLETE. This blocks the NEXT milestone's `/plan-init`
the same way — because its output is that milestone's input, and an input
nobody checked is not an input.

You did not run the phases. That is deliberate: this is a state report, not a
recollection.

## Context rules (PRIORITY)

- Read SUMMARIES, never full specs. Twenty specs will not fit and were never
  meant to be re-read.
- Read the plan's phase headings and scope lines, not its full phase bodies.
- Write no recommendations. What to build next is the next plan's job; guessing
  at it here puts an unreviewed opinion into a file the next milestone treats
  as fact.

## Step 0: Resolve the config and check state

Read `.pexec.yml`. Then read `{active}`.

If the last phase in `{plan}` does not show `Status: COMPLETE` in `{active}`,
STOP and say which phase is still open. A milestone with an open phase has
nothing to close.

If `{review}` already exists, STOP. Closing twice overwrites the record of the
first close.

## Step 0b: Requirements and blocked chains

**Requirements.** Read the milestone map's Requirements table for the IDs
assigned to this milestone, then read every summary's `Requirements:` line.
Produce three lists:

- **Satisfied** — the ID appears in a summary whose Status is COMPLETE
- **Not satisfied** — assigned here, no COMPLETE summary carries it
- **Carried by a PARTIAL or HALTED phase** — named separately, because the
  reason differs and so does what happens next

These lists go into the review and then into the map, where the Status column
and the coverage counts get updated. **This command does not edit the map** —
it produces what the map's owner writes, the same way the rest of Step 6 works.

**Blocked chains.** For every summary with `Status: HALTED`, walk `Depends on`
forward and list every phase blocked by it, directly or through a chain.

Report these as **blocked**, never as incomplete. The difference matters:
incomplete work resumes, blocked work needs a decision — replan, cut, or take
the halted phase's fix-or-cut route. A milestone that closes with a blocked
chain reported as "3 phases incomplete" hands the next planner a false problem.

## Step 1: What was built

Read every file in `{summaries}/`, in phase order. Read `{memory_md}`.

Write the "What was built" section from the summaries alone. Public API,
schema and invariant lines come from what the summaries recorded, not from what
the plan intended.

Then, for each phase, compare the summary against the plan's phase scope and
record deviations with the reason the summary or impl-log gives. A phase that
landed something the plan did not ask for, or skipped something it did, is the
most useful line in this document.

## Step 2: Re-run the baseline

Read `{preflight}`. Find the baseline measurement command and the isolation
statement — which metrics this milestone was supposed to move, and which were
supposed to stay unchanged.

Run the same command against the current state. Record both numbers.

- Metrics that were supposed to move: did they, and by how much?
- Metrics that were supposed to stay unchanged: did they?

A baseline that is captured and never compared was never a baseline. If
pre-flight recorded no measurement, write "no baseline captured" — that is
itself a finding for the next milestone's pre-flight.

## Step 3: Triage lessons

Read `{lessons}`. Sort every L-code into three groups, using the criterion the
lessons file already states — **does the mechanism that produces this trap
still exist?**

- **CARRY** — the mechanism is still in the codebase. This trap will recur.
- **RETIRE** — the mechanism was removed or replaced by this milestone.
- **RESOLVED** — the trap was fixed at its source, not worked around.

Write the CARRY entries into the review under `## Lessons to carry forward`,
renumbered from L1. The next milestone's `/plan-init` seeds its lessons file
from this list.

Do not carry an entry just because it looks generally wise. A lesson that could
apply to any project belongs in core-rules, not here, and carrying it forward
spends the 80-line budget on something that was never a trap in this codebase.

## Step 4: Record the external surface

List everything this milestone exposed that something outside the codebase can
now depend on:

- HTTP routes and their response shapes
- File formats read or written across a process boundary
- Persisted schemas and artifact formats
- Published package APIs, CLI flags, config keys

Build this from the summaries' "Added/changed public API" and
"Added/changed schema" sections, not from a repo scan. A repo scan finds what
is public; only the record shows what was published to someone.

For each entry, mark whether it has a known external consumer.

The next milestone's `/plan-init` writes this list into the project-specific
BLOCKING rules slot in core-rules. Once something has a consumer, breaking it
is a violation, not a design choice.

## Step 5: Carry the open questions

Two sources:

1. `{preflight}` Step 3 — claims that were checked. Which were false? Which
   were never actually checked?
2. Decisions recorded during the milestone with `epistemic: hypothesis`. A
   hypothesis that was never promoted to `assertion` is an assumption the
   codebase is now built on and nobody confirmed.

Read the `## Decision Record` section of every summary in `{summaries}` and
filter on `epistemic`. If `optional_mcp.rawthink` is enabled, query the archive
for this milestone's decisions as well and merge the two.

Read both; do not pick one. `post-phase` writes the record according to the flag
as it stood at the time, so phases recorded before a mid-milestone toggle live in
only one of the two places. A close that consults one source silently drops every
hypothesis recorded under the other setting — and those are precisely the
assumptions the next milestone is supposed to inherit.

Write these into `## Unverified assumptions carried forward`. This section
becomes the next milestone's pre-flight Step 3 input — the slot that has,
until now, had no producer.

## Step 6: Mark what the map has to absorb

Three of this review's sections have no downstream command that reads them —
the baseline comparison, the deviations, and the questions still open. They are
for a person, and the person needs somewhere to put the consequence.

That place is the milestone map. Under `## For the milestone map`, list only
what changes it:

- a deviation that moves work into or out of a later milestone
- an assumption that turned out false and invalidates a later row
- a question still open that belongs in the map's open decisions
- an estimate this milestone proved wrong, if the same kind of work recurs later

Write the finding, not the fix. "v2's transliteration estimate assumed X, which
was false" belongs here; "reduce v2 to 10 weeks" does not — that is the map
owner's call, and a recommendation written here arrives in the map as a fact.

If nothing changes the map, say so. That is a real and common outcome, and
stating it is what makes the section trustworthy when it is not empty.

**Requirements and blocked chains go in too** — the three requirement lists
from Step 0b and the blocked chains, each with the halted phase that caused it.
These are what the map absorbs; leaving them in the report and out of the
review means they exist only until the session ends.

## Step 7: Write the review

Write `{review}` from `milestone-review.template.md`.

Leave it marked `STATUS: PENDING`. The next milestone's `/plan-init` refuses to
run until a human sets it to COMPLETE.

That gate is the point. Everything above is a report by an agent that read
files; a human deciding "yes, this is what happened" is what makes it usable as
the next milestone's foundation.

## Step 8: Update active context

Overwrite `{active}`:

```
## Milestone: {milestone}
## Status: CLOSED
## Last Action: milestone-close completed
## Time: {date}
## Next: review milestone-review.md, then /plan-init <next-milestone>
```

## Step 9: Report

```
### MILESTONE-CLOSE — {milestone}

**Phases:**            {N} complete | {K} partial
**Deviations:**        {count} recorded
**Baseline:**          re-measured | no baseline captured
**Metrics moved:**     {as expected | unexpected: details}
**Metrics held:**      {held | regressed: details}
**Lessons:**           {C} carry | {R} retire | {S} resolved
**External surface:**  {count} entries, {K} with known consumers
**Open assumptions:**  {count} carried forward
**Review file:**       written, STATUS: PENDING
**For the map:**       {count} findings | nothing changes the map

Next: a human sets STATUS to COMPLETE, updates the milestone map if the review
flagged anything, then /plan-init <next-milestone>
```

Report only. Do not start the next milestone — a closing audit that opens the
next thing has stopped being an audit.
