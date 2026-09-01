# /post-phase — Close the phase

Argument: `/post-phase <N>`

Run in a clean session AFTER implementation. You did not watch the
implementation happen — that is deliberate. This step is an independent audit,
not a self-report.

## Step 1: Re-acquire context

Read:

1. `{phases_dir}/phase-{N}-SPEC.md` — what was supposed to happen
2. `{phases_dir}/phase-{N}-impl-log.md` — what the implementer says happened.
   If it is missing, note that in the report and audit from the diff alone —
   an absent log is itself worth recording.
3. `{phases_dir}/phase-{N}-review.md` — the verify verdict and what it flagged
4. `{active}` — last recorded state
5. The plan's Phase {N} section — original scope
6. `git log --oneline -20` — what actually landed
7. Dependency summaries

## Step 2: Verify the implementation

For every task in the spec's Task Breakdown:

- CREATE: does the file exist?
- MODIFY: read the file — was the change actually made?
- Are the specified public APIs present with the specified signatures?
- Schema/migration changes: registered where they need to be registered?

Small gaps (missing import, missing comment) you MAY fix.
Large gaps (an entire task unimplemented) you REPORT only. Do not quietly
complete work the implementer skipped — that hides the failure from the record.

## Step 3: Scan for leftovers

In the changed files:

- `TODO`, "implement", `NotImplementedError` / `UnsupportedOperationException`
- Empty method bodies, placeholder `pass`, `...`
- Any project-specific red flag listed in `{core_rules}`

## Step 4: Write the phase summary

Write `{summaries}/phase-{N}-summary.md` per the format in `{core_rules}`.
Target ~30 lines.

**Build it from the REAL files, not from the spec.** Read the changed
entities, services, configs, migrations and tests; verify signatures against
source. The summary becomes the input for every later phase — a summary copied
from the spec propagates the spec's errors forward instead of catching them.

**Set the Status line honestly.** `COMPLETE`, `PARTIAL ({K} incomplete)`, or
**`HALTED`**.

Use HALTED when the phase ran, answered the question it existed to answer, and
the answer means the work it was gating cannot proceed. Add the two-line
`## Halted` block from `{core_rules}`: the question and the answer. This is a
success — the phase did its job — and marking it PARTIAL instead hides a
blocked chain behind what looks like ordinary unfinished work.

**Write the Requirements line** from the spec's provenance: the map IDs this
phase covered, or `none — infrastructure`. `/milestone-close` reads this line
to mark requirements satisfied; an ID that never reaches a summary cannot be
closed by anything.

## Step 5: Record the decision (if `optional_mcp.rawthink` is enabled)

Record a structured decision entry for this phase:

```
type:       decision
domain:     {project domain}
epistemic:  assertion | hypothesis | unknown
visibility: private | shareable
decided:    what was chosen
rejected:   what was considered and NOT chosen, and why   ← the valuable part
because:    the constraint that forced it
touches:    files/symbols (shared key with the code index)
supersedes: prior decision id, if this replaces one
```

Derive this from the summary and the spec, not from conversation. `rejected`
matters most: "we used X" is readable from the code forever; "we did not use Y,
because Z" exists nowhere else and is exactly what gets asked six months later.

If a prior decision is now wrong, mark it invalidated with a pointer to this
one. Do not delete it. An archive that forgets its own reversals cannot answer
"why did we change our mind".

Write these fields into the summary under a `## Decision Record` heading
REGARDLESS of whether the archive is enabled. The duplication is deliberate.

`/spec-create` reads summaries; nothing reads the archive during a phase. If the
record only goes to the archive, enabling the integration REMOVES the rejected
alternatives from the only place a later phase would find them — the optional
tool would make the pipeline see less, not more.

The archive is for cross-milestone recall and queries. The summary is for the
next phase. Both are needed, and the pipeline must not require the tool.

## Step 6: Update the milestone memory file

Append to `{memory_md}`:

- Phase {N} DONE line (commit hash, one-line summary)
- Important patterns / APIs / invariants introduced
- Move the "NEXT" pointer to the following phase

## Step 7: Update lessons

If implementation surfaced a NEW trap — a wrong API in the spec, a
non-existent field, a wrong namespace, an inconsistency fixed during compile —
append it to `{lessons}` with the next L-code.

Format: `**L{n}** — {trap}. {why it happens}. {what to do instead}.`

Every entry MUST have an L-code. `spec-verify` enforces lessons by citation; an
entry with no code cannot be cited, and an entry that cannot be cited is not
enforced.

If the file exceeds 80 lines, remove entries that are resolved or no longer
apply. This file is read at the start of every `spec-create`; unbounded growth
turns the guardrail into a context tax.

No new traps? Skip this step.

## Step 8: Update active context

```
## Active Phase: {N}
## Status: COMPLETE
## Last Action: post-phase checks completed
## Time: {date}
## Next: Phase {N+1}
```

**If phase {N} is the last phase in `{plan}`, write `## Next: /milestone-close`
instead.** There is no phase {N+1}. Pointing at one leaves the milestone looking
open when it is finished, and the closing step never gets run.

Rewriting `{active}` is correct — it is a status file, not a log. Verdicts and
reports live in their own per-phase files and are not affected.

## Step 9: Report

```
### POST-PHASE — Phase {N}

**Implementation:**   COMPLETE | {K} tasks incomplete
**Leftovers:**        CLEAN | {K} issues
**Deviations:**       {from impl-log, with reasons}
**Summary:**          written from source
**Decision record:**  written | skipped (rawthink disabled)
**Memory file:**      updated
**Lessons:**          {L-codes added} | no new traps
**Active context:**   COMPLETE

POST_PHASE_DONE
```
