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

If `{N}` is the first phase, read `{preflight}`. If its status is not
`COMPLETE`, STOP and tell the user which pre-flight items are outstanding.
Baseline measurements cannot be taken after the fix lands.

## Step 1: Read the plan section

Search `{plan}` for Phase {N}. Read only those lines: scope, goal, key files,
validation criteria, noted traps.

## Step 2: Read dependency summaries

Identify the phases this one depends on. For each, read
`{summaries}/phase-{dep}-summary.md`.

If a summary is missing for a phase that IS a dependency, tell the user and
STOP. Implementing against an unrecorded dependency means guessing at its API.

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
