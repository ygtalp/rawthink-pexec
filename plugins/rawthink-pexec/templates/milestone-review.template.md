# {milestone} — Milestone Review

**STATUS: PENDING**

Written ONCE by `/milestone-close`, after the last phase. Outside the phase
loop; produces no code.

`/plan-init` for the NEXT milestone refuses to run until a human sets STATUS to
COMPLETE.

## Why this exists

Pre-flight captures state that cannot be recovered once a milestone starts.
This captures state that cannot be recovered once a milestone ends.

A summary answers "what did phase 7 do". Nothing answers "what does the next
milestone start from" — and by the time that question is asked, the session
that could have answered it is gone and the plan has moved on. The inputs are
all still on disk; what is missing is the one pass that reads them together.

The gate is deliberate. Everything below is written by an agent that read
files. A human confirming "yes, this is what happened" is what makes it usable
as the next milestone's foundation rather than one more generated document.

---

## What was built

*(from summaries, not from the plan — the plan says what was intended)*

**Phases:** {N} complete, {K} partial

### Public API added or changed
- {signature} — {what it does} (phase {N})

### Schema / config / persisted state
- {change} (phase {N})

### Invariants later work must not break
- {invariant} (phase {N})

## Deviations from plan

| Phase | Plan said | What landed | Reason |
|---|---|---|---|
| {N} | {scope line} | {summary line} | {from summary or impl-log} |

State the reason as recorded, not as reconstructed. "No reason recorded" is a
valid and useful entry.

## Baseline comparison

**Measurement command:** `{from preflight}`

| Metric | Before | After | Expected to |
|---|---|---|---|
| {name} | {value} | {value} | move / hold |

**Metrics that moved as expected:** {list}
**Metrics that held:** {list}
**Unexpected movement:** {list, or none}

If pre-flight captured no baseline, write that here. It is a finding for the
next milestone's pre-flight, not an omission to hide.

## Lessons to carry forward

*(CARRY entries only — the mechanism that produces the trap still exists)*

Renumbered from L1. `/plan-init` seeds the next milestone's lessons file from
this list.

- **L1** — {trap}. {why it happens}. {what to do instead}.

**Retired:** {old codes and why the mechanism is gone}
**Resolved:** {old codes and where the fix landed}

## External surface

Everything something outside the codebase can now depend on. The next
`/plan-init` writes this into the project-specific BLOCKING rules in
core-rules.

| Surface | Kind | Known consumer |
|---|---|---|
| {route / format / schema / API} | {kind} | yes / no / unknown |

"Unknown" is not "no". An entry that might have a consumer is treated as
breaking until someone checks.

## Unverified assumptions carried forward

Feeds the next milestone's pre-flight Step 3 — the claim-verification list.

| Assumption | Source | Why it still matters |
|---|---|---|
| {claim} | pre-flight §3 unchecked / decision recorded as `hypothesis` | {what breaks if false} |

A decision recorded as `hypothesis` and never promoted to `assertion` is an
assumption the codebase is now built on and nobody confirmed. Those are the
expensive ones: they were flagged as uncertain at the time and the flag was
never cleared.

## Still open at close

- {question the milestone raised and did not answer}

Questions only. No recommendations — what to do about them is the next plan's
job, and an opinion written here arrives in that plan as a fact.

## For the milestone map

The three sections above — baseline comparison, deviations, questions still open
— have no command that reads them. This is where their consequence goes.

List only what changes the map:

| Finding | What it changes on the map |
|---|---|
| {deviation / false assumption / open question / wrong estimate} | {which row, decision or estimate} |

**"Nothing changes the map"** is a valid entry and a common one. Saying it
explicitly is what makes this section worth reading when it is not empty.

Findings, not fixes. What to do about them is the map owner's decision, and a
recommendation written here arrives in the map as a settled fact.

---

**Completion:** set STATUS to COMPLETE above, with date and who reviewed it.
