# /spec-create — Write the phase contract

Argument: `/spec-create <N>`

Writes `{phases_dir}/phase-{N}-SPEC.md`. This file is the ONLY channel between
this step and implementation — nothing you learn here survives except what you
write into it. Run in a clean session.

## Context rules (PRIORITY)

- Read the plan's Phase {N} section ONLY. Search for the heading; do not read
  the whole file.
- For dependency phases read the SUMMARY, never the full spec. Summaries are
  ~30 lines; specs are 1000+.
- Do NOT start research subagents or Explore. The plan already carries the
  research findings.
- Read each file EXACTLY ONCE. No chunked reads, no re-reads from cache. If a
  file is too large, use offset/limit.

These rules exist because this step is the one most prone to running out of
room and stopping early. A half-read spec poisons every step after it, and
there is no conversation to scroll back to.

## Step 0: Gates and prior state

Read `{active}`. Check for remnants of a previous attempt at this phase.

If `{active}` shows `verify FAILED` for phase {N}, read
`{phases_dir}/phase-{N}-review.md` before writing anything. Its blocking
violations are REQUIRED INPUT: the new spec must address each one explicitly,
and you must say where. Rewriting from the plan alone reproduces the same spec
and earns the same verdict.

If `{active}` carries three or more `verify FAILED` lines for phase {N}, STOP.
Three rejections is not a spec problem — the phase scope is probably wrong, and
that is a question for the plan, not another rewrite.

If `{N}` is the first phase, read `{preflight}` and check TWO things:

1. **Status is `COMPLETE`.**
2. **No row is blank.** Every row in the claim-verification table has one of
   `HOLDS` / `FALSE` / `ACCEPTED AS RISK`, and every open-decision checkbox is
   ticked with an outcome.
3. **No evidence cell is empty.** A `HOLDS` with nothing in Evidence is an
   opinion in the shape of a result; for an accepted risk, the evidence is why
   it could not be settled.

STOP on any of the three, and say which rows are outstanding.

The second check is not redundant. A status line is one keystroke and a table
is twenty rows; signing the first without finishing the second is the normal
way this gate fails, and it fails silently — the milestone starts, and the
unchecked claim surfaces in phase 6 as a bug nobody can trace to a decision.

Report the `ACCEPTED AS RISK` count in your own report even when you proceed.
Phase 1 starting on four accepted risks is a fact the next steps should be able
to read without opening the pre-flight file.

Baseline measurements cannot be taken after the fix lands.

## Step 1: Read the plan section

Search `{plan}` for Phase {N}. Read only those lines: scope, goal, key files,
validation criteria, noted traps.

**Carry the phase's `Requirements:` line into the spec's provenance section.**
It is what `/post-phase` writes into the summary and what `/milestone-close`
reads to mark requirements satisfied. An ID that never reaches the summary is
an ID nothing can close.

## Step 2: Read dependency summaries

Identify the phases this one depends on. For each, read
`{summaries}/phase-{dep}-summary.md`.

If a summary is missing for a phase that IS a dependency, tell the user and
STOP. Implementing against an unrecorded dependency means guessing at its API.

**Check the Status line of every dependency summary.** If any dependency —
directly, or through a chain of dependencies — has `Status: HALTED`, STOP.
Report which phase halted, the question it answered, and the answer, and name
the chain that leads from it to this phase.

This phase is **blocked**, not failed and not late. A halted dependency means
the premise this phase was going to build on did not survive contact with
reality. Writing a spec anyway produces work that is coherent and pointless,
and it is cheaper to say so now than to discover it in verification.

What happens next is a human decision — replan the milestone, cut this phase,
or take the fix-or-cut route the plan named for the halted phase. None of those
is `/spec-create`'s call.

If this phase has no dependencies — normally the first phase of a milestone —
there is nothing to read here. An empty `{summaries}` directory is expected at
that point and is not an error.

## Step 3: Read lessons (anti-hallucination)

Read `{lessons}`. These are traps discovered in THIS codebase — you can
reproduce them, because you are about to re-derive code from live source.

Do not repeat any of them. `spec-verify` treats a repeated lesson as a
BLOCKING violation.

## Step 3b: Check what was already rejected

**Always:** the dependency summaries you already read in Step 2 carry a
`## Decision Record` section — use those. For non-dependency phases, extract ONLY
that section from their summaries; do not read the summaries in full.

**Additionally, if `optional_mcp.rawthink` is enabled:** query the decision
archive for decisions whose `touches` overlaps this phase's key files, and read
their `rejected` fields. This reaches decisions from earlier milestones, which
summaries in this milestone cannot carry.

**Read both; do not pick one.** `post-phase` writes the record according to the
flag as it stood at the time, so a phase recorded under one setting is invisible
to a reader that consults only the other. Toggling the integration mid-milestone,
in either direction, would otherwise blind this step to every phase recorded
before the switch — which is exactly when a rejected approach comes back.

At phase 1 there is nothing to read here — `{summaries}` is empty and the
archive holds nothing for this milestone. That is expected, not an error.

This step adds one short section per phase, not a second pass over files you
have already read. The context rules above still hold: each file once, and
nothing read whole that can be read in part.

Do not propose an approach a prior phase rejected. If you believe the constraint
that eliminated it no longer holds, say so in the spec's Provenance section and
name the decision you are overriding. Silently re-proposing it is how a settled
question gets re-litigated in phase 12.

A rejected alternative is not a lesson. Lessons are traps in the code; this is a
design choice that was considered and declined. `spec-verify` does not gate on
these — which is exactly why reading them here matters.

## Step 4: Read format reference

- `{core_rules}` → "Spec File Format" and the violation tiers
- The previous completed phase's summary, as a depth and tone reference — if
  one exists. For the first phase of a milestone there is none; `{core_rules}`
  alone is the format authority, and the section list below is the contract.

**If `{core_rules}` has a prohibitions table**, take every `resolved`/`test`
row that touches this phase and write it into the spec's validation criteria as
a **negative criterion** — a check that fails if the forbidden thing is
present. A `resolved`/`judgment` row goes in as a criterion the verifier reads
for and answers explicitly.

The plan's `Must NOT:` line names which prohibitions apply to this phase. If it
names one the core rules do not carry, say so: the two disagree, one of them is
stale, and a prohibition that exists in only one of them is enforced by
neither.

## Step 5: Read real source (API verification)

Read every file in `must_read` from `.pexec.yml`, plus the phase's own key
files from the plan, plus any abstract base class whose signature you will
override.

**If `optional_mcp.codegraph` is enabled:** run an impact query on the symbols
this phase touches and paste the result into the spec's Blast Radius section.
Take the file list from the tool, not from your own reading stamina.

Report which files you read and why. Do not read research documents or `docs/`
— API verification comes only from real source.

## Step 6: Write the spec

Write `{phases_dir}/phase-{N}-SPEC.md` following the format in `{core_rules}`.

Required sections:

1. **Provenance** — which plan items / findings this phase implements
2. **Depends on** — phases and their status
3. **Scope: IN** — files and symbols this phase may change
4. **Scope: OUT** — what it explicitly must not touch
5. **Blast Radius** — what a change here reaches (tool output, not memory)
6. **File Summary** — table: file / action / namespace / why
7. **Task Breakdown** — per task: file, action, namespace, dependencies, and
   COMPLETE implementation code
8. **Dependency Graph** — ordering between tasks
9. **Validation Criteria** — measurable, 15+ items preferred
10. **Integration Checkpoints** — each one updated, or "NO CHANGES NEEDED" + rationale
11. **Open Questions** — must be EMPTY before implementation
12. **Rollback** — how to undo this phase

Every code block must be complete and compilable. Pseudocode, table-formatted
code, `...` elisions and empty method bodies are forbidden.

Scope: OUT is not decoration. It is the instruction the implementer checks
before touching anything it did not expect to touch.

## Step 7: Self-verify (INLINE — no subagents)

You already read the files. Review your own spec without re-reading.

### BLOCKING — any one of these fails the spec

- [ ] Pseudocode present? (grep: TODO, `...`, "implement", empty bodies)
- [ ] Any API referenced that you did not verify in Step 5?
- [ ] Namespace/package matches the real source?
- [ ] Override signatures match their base?
- [ ] Plan scope exceeded?
- [ ] Any lesson from `{lessons}` reproduced?

### STRONG PREFERENCE — justify if missing

- [ ] Validation criteria 15+ and measurable
- [ ] Integration checkpoints all addressed
- [ ] Scope: OUT explicitly listed
- [ ] Rollback stated

Write the outcome to `{active}`.

Do not implement. Do not commit. This step produces one file.
