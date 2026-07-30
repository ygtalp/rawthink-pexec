# /spec-verify — Independent verification

Argument: `/spec-verify <N>`

**You did not write this spec.** You are an independent verifier. Your job is
not to approve it — it is to try to break it.

Run in a clean session. That isolation is the point: a verifier that shares
context with the author inherits the author's blind spots and confirms its own
omissions.

## Context rules (PRIORITY)

- Read the spec ONCE, in full.
- Read the plan's Phase {N} section only.
- Verify APIs INLINE — do NOT start subagents. Read the files yourself.
- Read each file exactly once. No re-reads from cache.
- **Do not load memory or decision-archive tools in this session.** If a gap in
  the spec can be excused by a past decision, that is rationalisation, not
  verification. The spec must stand on its own against the current source.

## Step 1: Read the spec

Read `{phases_dir}/phase-{N}-SPEC.md` in a single pass.

## Step 2: Compare against the plan

Search `{plan}` for Phase {N}.

- Does the spec match the planned scope?
- Anything in the plan MISSING from the spec? (especially validation criteria)
- Anything in the spec ABSENT from the plan? (scope creep)

## Step 3: Code quality

Scan every code block:

- [ ] Pseudocode: `...`, `TODO`, "implement", table-formatted code
- [ ] Empty or missing method bodies
- [ ] Signature declared with no implementation
- [ ] Language-specific violations from `{core_rules}`

Report with line numbers.

## Step 4: API verification (INLINE)

For every method call, property and field the spec references:

- Read the source file (once each)
- Does the member actually exist?
- Does the signature match — parameter count, types, return type?

Report as `FILE:LINE — expected vs actual`.

Verification comes only from real source. Documentation is not a reference;
docs drift from code, and a spec verified against drifted docs is unverified.

## Step 5: Namespace / package verification

For every type the spec defines or uses, compare the declared namespace against
the real source file. Use what you read in Step 4; do not re-read.

## Step 6: Integration checkpoints

For each entry in `integration_checkpoints`, confirm the spec either updates it
or states "NO CHANGES NEEDED" with a rationale. Report omissions.

## Step 7: Internal consistency and lessons

- [ ] File Summary table matches Task Breakdown (counts, actions)
- [ ] Dependency Graph matches the per-task dependencies
- [ ] Same file modified by multiple tasks — do they conflict?
- [ ] Scope: OUT present and specific?
- [ ] Open Questions EMPTY? (a non-empty list blocks implementation)
- [ ] **Any lesson from `{lessons}` reproduced?** Check each L-code.

The lessons check is why lessons entries carry L-codes: an entry you cannot
cite is an entry you cannot enforce.

## Step 8: Verdict

```
### SPEC VERIFY — Phase {N}

**BLOCKING violations:**        {count}
{details}

**Strong-preference gaps:**     {count}
{details}

**API problems:**               {count}
{details}

**Namespace problems:**         {count}
{details}

**Lessons reproduced:**         {count}
{L-codes and where}

**VERDICT:**
- PASSED — 0 blocking violations, implementation may proceed
- FAILED — {N} blocking violations, spec must be corrected
```

Write the full report to `{phases_dir}/phase-{N}-review.md`. The last line of
that file must be exactly `VERDICT: PASSED` or `VERDICT: FAILED`.

Then append one line to `{active}`:
`Phase {N}: verify {PASSED|FAILED} — see phase-{N}-review.md`

The verdict lives in its own per-phase file for a reason: `{active}` is
rewritten by later steps, and a verdict that can be overwritten by the step it
is supposed to gate is not a gate.

Verify and report only. Do not fix the spec — a verifier that edits becomes an
author, and there is no one left to check the edit.
