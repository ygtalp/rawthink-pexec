# /spec-implement — Build what the spec says

Argument: `/spec-implement <N>`

Run in a clean session. The spec is your only source; you did not write it and
you cannot see the conversation that produced it.

## Context rules (PRIORITY)

- Read the spec ONCE, in full. No chunked reads, no cache re-reads.
- Dependency phases: read SUMMARIES, not full specs.
- Read each file exactly once.
- **Do not use git worktrees.** Work sequentially in one session on `{branch}`.
  Worktrees exist for parallelism; this pipeline is deliberately sequential, and
  a second working tree means a second (stale) code index and a class of tool
  breakage that buys you nothing here.
- **Do not load memory or decision-archive tools.** The spec is the contract.
  Recalled context is how scope drifts.

## Step 0: Verify gate

Read `{phases_dir}/phase-{N}-review.md` — the per-phase verify report. Its
last line is the verdict.

- **`VERDICT: PASSED`** → continue to Step 1.
- **`VERDICT: FAILED`** → STOP. Summarise the blocking violations from that
  file. The spec must be corrected and re-verified.
- **File missing** → STOP. Run `/spec-verify {N}` first.

Read the phase's own review file, not `{active}`. `{active}` holds the current
state of the milestone and gets rewritten; the verdict must not depend on it.

Never implement an unverified or failed spec. This gate is the reason the
verify step has any force.

## Step 1: Read the spec

Read `{phases_dir}/phase-{N}-SPEC.md` in a single pass.

If Open Questions is non-empty, STOP. Unresolved questions get answered by
improvisation at this stage, and improvisation is what the spec exists to
prevent.

## Step 2: Dependency context

Read the summary of each phase listed under "Depends on".

## Step 3: Extract the task list

From Task Breakdown: file, action (CREATE/MODIFY), namespace, dependencies.
Determine the order. Independent tasks run SEQUENTIALLY, not in parallel.

## Step 4: Implement in order

For each task:

1. If MODIFY: read the target file first (understand what is there, including
   changes left by earlier phases)
2. Implement the spec's code exactly
3. Change nothing the spec does not specify
4. Namespace, class name, signatures — exactly as specified

**If a task requires touching something in Scope: OUT — STOP and ask.** Do not
improvise. That boundary is the spec's main defence against unrequested
refactors and helpful additions nobody asked for.

Write `{phases_dir}/phase-{N}-impl-log.md` — ALWAYS, even when nothing
deviated. Record per task: what you changed, and any deviation with its reason.
`post-phase` reads this file to compare intent against reality; "no deviations"
is a finding it needs stated, not an absence it has to infer.

## Step 5: Validate

- `{compile}` → 0 errors
- `{test}` → existing tests pass
- Walk the spec's Validation Criteria

**If `optional_mcp.codegraph` is enabled:** run an affected-tests query on the
changed files and confirm the tests it names actually ran.

## Step 6: Finalize

- Commit on `{branch}`. One small, meaningful commit.
- Update `{active}` to `IMPLEMENT_DONE`.

Do NOT do post-implementation work — summary, decision records, memory files,
lessons. That happens in `/post-phase`, in a clean session, by something that
did not watch the implementation happen.
