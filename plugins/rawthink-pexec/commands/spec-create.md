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
