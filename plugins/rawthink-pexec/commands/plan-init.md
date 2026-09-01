# /plan-init — Bootstrap a milestone

Scaffolds the execution structure for a milestone. Run ONCE, before phase 1.
This command CREATES files; it does not report on them.

Argument: `/plan-init <milestone>` (e.g. `v3`, `v2.0.0`, `remediation`)

## Step 0: Resolve the config

Everything below reads paths from `.pexec.yml`. Do this first — the other steps
have nothing to resolve against until it exists and is filled in.

**If `.pexec.yml` does not exist:** copy `.pexec.example.yml` to `.pexec.yml`,
set `milestone` to the argument you were given, and then fill the rest WITH the
user — do not leave placeholders behind. Ask for:

- `project.language`
- `project.compile` — the exact command that proves the code is valid
- `project.test` — the exact command that runs the tests
- `project.branch`
- `integration_checkpoints` — files every spec must address or explicitly skip
- `must_read` — files `spec-create` must always read to verify real APIs

Read the repo first and propose answers; ask only where the repo does not
settle it.

**If `.pexec.yml` already exists:** check it was actually filled in. Manual
setup copies the example file, so a config full of example values is the normal
starting state, not an edited one. STOP and ask if any of these are still true:

- `milestone` does not match the argument you were given
- `compile` or `test` still contains `<your ... command>`
- `integration_checkpoints` or `must_read` still contain `<...>` placeholders

Running a milestone with placeholder commands produces a pipeline that reports
success without ever building anything.

Resolve every `{...}` path from this file before continuing.

## Step 0b: Previous milestone gate

**Identify the predecessor first.** Glob `.planning/*-phases/` for directories
other than this milestone's, and treat one as the predecessor only if it
contains a `*-plan.md`. If this project points `paths.review` somewhere outside
`.planning/`, ask the user where the previous milestone's files live — the glob
assumes the default layout and silently finds nothing otherwise.

Milestone names do not sort (`v1`, `v2`, `remediation`, `v3`), so:

- **No such directory** — this is genuinely the first milestone. Continue.
- **Exactly one** — that is the predecessor.
- **More than one** — ask the user which one this milestone follows. Do not
  guess from names or timestamps.

Then check the predecessor for `milestone-review.md`:

- **`STATUS: COMPLETE`** — continue, and read it. Steps 3, 6, 7 and 8 use it.
- **`STATUS: PENDING`** — STOP. Say which milestone is unclosed and that a human
  must review it and mark it COMPLETE.
- **Missing entirely** — the previous milestone was never closed. STOP and offer
  two paths: run `/milestone-close` on it first, or proceed without it and
  accept that this milestone inherits no lessons, no external-surface list and
  no carried assumptions. Do not proceed silently.

The distinction matters. "No review file" does not mean "first milestone" — it
much more often means someone skipped the close, which is exactly what this
gate exists to catch.

A milestone planned on an unreviewed close is planned on an agent's unchecked
report. This is the same gate pre-flight applies at the other end.

## Step 1: Resolve the templates directory

Find the template files. Try in order and use the first that exists:

1. `paths.templates` from `.pexec.yml`, if set — an explicit override
2. `${CLAUDE_PLUGIN_ROOT}/templates/` — installed as a plugin
3. `.claude/pexec-templates/` — installed manually
4. `templates/` relative to this command file — running from a clone

If none of them contains `plan.template.md`, STOP and say exactly this:

> Templates not found. Either set `paths.templates` in `.pexec.yml`, or copy
> the `templates/` directory from the rawthink-pexec repo to
> `.claude/pexec-templates/`.

Do not improvise the templates. A generated `core-rules` that differs by
install method makes every downstream verdict incomparable.

## Step 2: Locate the plan

The plan is normally produced by Claude Code's plan mode. Find it — try in
order:

1. A path given as a second argument
2. `paths.plan` from `.pexec.yml`, if that file already exists
3. `~/.claude/plans/` — plan mode's default output directory
4. `plan.md` at the project root
5. The directory named by `plansDirectory` in `settings.json`, if set

Plan files from plan mode have generated names (`dreamy-orbiting-quokka.md` and
similar), so **never guess**. If more than one candidate turns up, list them —
filename, modified date, first heading, and how many `### Phase` headings each
contains — and ask which one.

If `paths.plan` already holds a plan, ask before overwriting.

## Step 3: Copy the plan into the project

**Copy it — do not reference it in place.** Write it to `paths.plan`, named
from the milestone (`{milestone}-plan.md`). Leave the source file untouched.

Prepend a provenance line:

```
<!-- imported by /plan-init on {date} from {source path} -->
```

The copy matters. `~/.claude/plans/` is global and mutable: every later plan
mode run adds files to it, and a plan referenced there can change under a
milestone that is already running. `/post-phase` re-reads the plan on every
phase — across twenty phases, the document phase 20 validates against must be
the same one phase 1 was written from. Copying makes the project the authority
and freezes it.

### Plan drift check

If the predecessor identified in Step 0b has a plan, diff it against the new
plan SECTION BY SECTION and report what disappeared:

- sections present before and absent now
- tables that lost rows
- open decisions that vanished with no recorded closure
- scope statements that are no longer there

Report only; restore nothing. Deleting is often correct — deleting silently is
not. The user confirms each drop as intentional, or restores it.

**Compare sections, not total size.** A plan can grow while losing content: new
sections get added, existing ones are compressed to make room, and the byte count
rises the whole time. Growth is not evidence that nothing was lost.

## Step 4: Validate the plan

Read the copy and confirm it can drive the pipeline:

- [ ] Phases are individually addressable (a `### Phase N` heading or equivalent)
- [ ] Each phase states scope, key files, and validation criteria
- [ ] A dependency order exists (explicit graph or stated ordering)
- [ ] The plan contains NO implementation code

### The greenfield check

**Presence is the trigger.** Resolve `paths.design` and `paths.skeleton_scope`
against this milestone, and look:

- **Neither file exists** — skip this whole check. Normal for any milestone
  after the first, and for a project that already had source when pexec was
  adopted.
- **Both exist** — run the three checks below.
- **One exists and the other does not** — STOP and say which is missing. The
  two are written as a pair by `/greenfield-init`; half a pair means one was
  deleted or never written, and neither absence is safe to assume away.

Both paths must be `{milestone}`-scoped for this to work. If `skeleton_scope`
points at a project-wide file while `design` is per-milestone, every later
milestone sees a scope map with no inventory and stops on a half pair it was
never supposed to have. Say so and stop, rather than guessing which one the
project meant.

Presence rather than a predecessor test, because "there is a predecessor" and
"there is source" are not the same claim. A project can add a second, initially
empty module at v3 — a closed-source counterpart, a new service — and that
module is greenfield even though the project is not. Writing an inventory for
it turns this check back on by itself.

**Check 1 — both are signed.** Neither `{paths.skeleton_scope}` nor
`{paths.design}` may still read `STATUS: PENDING`.

A `PENDING` design inventory means the plan was written before the design was
settled, so its anchors were invented rather than derived. A `PENDING` scope
map means nobody confirmed what the skeleton deliberately left out — and the
`## Left behind deliberately` list is worth exactly what its signature is worth.

**Check 2 — every anchor in a PHASE ENTRY resolves.** Scope this to the
`Source:` and `Key files:` lines inside each phase; do not scan the whole
document.

The rest of the plan legitimately names files that do not exist yet. The
guardrail section points at `{core_rules}` and `{lessons}`, and Steps 6 and 7
of this command are what create them — checking them here rejects a correctly
written plan for a file this command is about to write. Anything outside a
phase entry is prose, not an anchor.

An anchor is mechanically defined, so this is checkable rather than a matter of
judgement:

- A **path anchor** is a backticked token containing `/` and a file extension:
  `` `src/main/java/com/x/Ledger.java` ``. The file must exist.
- A **symbol anchor** is a backticked identifier followed by the word `in` and
  a path anchor: `` `Ledger.append()` `` in `` `src/.../Ledger.java` ``. The
  file must exist and must contain the identifier — **stripped of `()` and of
  any qualifier before the last dot**. `Ledger.append()` is satisfied by a file
  containing `append`; the qualifier is there so a human can read the anchor
  without opening the file, and matching on the qualified form would fail
  against every language that does not repeat the type name at the definition
  site.

Report every anchor that does not resolve, with its phase number, and STOP.
Bare type names with no path are not anchors and are not checked — they are
also not enough, which `plan.template.md` says at the point where they get
written.

**Check 3 — the inventory and the plan agree on phases.** Every type in the
inventory's phase column names a phase that exists in this plan, and every
`CREATE` row's file appears in the plan's file list for that phase. A type
assigned to a phase the plan does not contain is work nobody will do.

The failure all three catch is invisible at every later step until
`/spec-create` Step 5 goes looking for a file that was never there — four
sessions and one wasted verify cycle downstream.

### Coverage and clarity checks

**Requirement coverage.** Read the milestone map's Requirements table and take
every ID whose Milestone column names this milestone. Each one must appear in
at least one phase's `Requirements:` line.

- **Uncovered ID** — STOP, and name it. The map assigned it here and the plan
  does not deliver it; that is the gap this join exists to catch.
- **Phase with an empty `Requirements:` line** — STOP. `none — infrastructure`
  is a valid answer; blank is not, because blank and "forgot" are the same
  character.
- **ID in the plan that the map does not assign here** — report it, do not
  stop. Either the map is stale or the plan has taken on work from a later
  milestone, and both are worth a sentence before phase 1.

Report the counts: assigned / covered / uncovered.

**Clarity.** Every phase answers the four clarity questions, and **every `no`
has a matching row in the plan's `Unverified assumptions` table.** A `no` with
no assumption row is ambiguity that was noticed and then dropped — worse than
ambiguity that was never examined, because the plan now looks checked.

STOP on an unanswered question or an unrouted `no`.

**Prohibitions.** If `{core_rules}` has a prohibitions table, every
`resolved`/`test` row appears as a `Must NOT:` criterion in each phase it
touches. STOP on a prohibition that no phase carries — it is a rule the
project believes in and nothing runs.

If a check fails, report which one and STOP — with one exception.

**If the document is plan-shaped but at the wrong altitude** — one task's
implementation steps rather than a milestone of phases — say so and offer a
choice instead of stopping flat:

> This reads as a single task, not a milestone. Either (a) treat it as the
> source for one phase and write a milestone plan around it, or (b) I can
> reshape its steps into phases using `plan.template.md`.

If it produced three steps, say plainly that a phase pipeline is heavier than
the work needs.

## Step 5: Create the skeleton

Create both directories, even though one of them starts empty:

- `{phases_dir}/` — holds the spec, review and impl-log of every phase
- `{summaries}/` — empty until `/post-phase 1` writes the first summary

`{summaries}/` must exist from the start. `/spec-create` reads it looking for
dependency summaries; an absent directory reads as an error, while an empty one
correctly reads as "this phase has no dependencies yet".

Then write `{active}`:

```
## Active Phase: —
## Status: NOT STARTED
## Last Action: plan-init completed
## Time: {date}
```

## Step 6: Generate core-rules

Copy `{templates}/core-rules.template.md` to `{core_rules}`, **delete every
line between the `<!-- TEMPLATE-HEADER -->` markers**, and fill the
project-specific slots by reading the repo:

- language code rules (money type, formatting, idioms that matter here)
- the "compile" definition — the exact command that proves code is valid
- integration checkpoints — files every spec must address or explicitly skip
- summary format fields that make sense for this project

Ask the user rather than guessing when a slot has no obvious answer. A wrong
rule here propagates into every spec of the milestone.

**If `{paths.skeleton_scope}` exists**, read its `## Modules created` table and
write the dependency direction into the project-specific BLOCKING rules slot,
as:

> An import from {module A} into {module B} is a BLOCKING violation. The
> dependency runs {B} → {A} and not the other way.

**Include test sources in the rule explicitly.** The walking slice needs a real
implementation of everything it touches and the nearest one is often across the
boundary, so the first violation on a split project is almost always a test
wiring the far side in. A rule that says "sources" and means "main sources"
will not catch it, and no build check that inspects main alone will either.

A boundary that exists only in a README is not enforced by anything.
`spec-verify` grades against these tiers; a rule that is not written here is a
rule that is not checked.

**Fill the prohibitions table.** Every blocking rule that is a must-NOT gets a
row: what it forbids, which requirement or architectural claim it protects, and
whether a test can check it or only judgment can. Take the `Protects` column
from the milestone map's requirement IDs where one applies; write
`architecture` where the thing protected is a structural claim with no ID.

Then write the coverage line. A prohibition with `⚠ UNRESOLVED` status goes
into the plan's `Unverified assumptions` — it is a rule the project believes in
and cannot currently check, which is exactly what that table is for.

If the previous milestone's review has an `## External surface` section, write
its entries into the project-specific BLOCKING rules slot, as:

> Breaking {surface} is a BLOCKING violation. It was published in {milestone}
> and has a known consumer.

Include entries marked "unknown consumer" too — unknown is not no.

A repo scan finds what is public; only the milestone record shows what was
published to someone. `plan-init` cannot derive this by reading code.

## Step 7: Generate an empty lessons file

Copy `{templates}/lessons.template.md` to `{lessons}` and strip the header
block.

It starts EMPTY. This file holds only empirically discovered traps, appended by
`/post-phase` as they are found.

**Two legitimate ways it starts non-empty:** the plan's per-phase `⚠️ Traps`
entries, and a research or audit document that already lists code-anchored
pitfalls. Seed those now, each with an L-code. This matters most when the plan
is high-level — `/spec-create` re-derives code that could reintroduce exactly
those pitfalls.

**A third way, from the previous milestone:** if its review has a
`## Lessons to carry forward` section, seed from it and keep the numbering the
review assigned. Those entries were triaged at close by asking whether the
mechanism that produces each trap still exists. A CARRY entry describes a
mechanism still in this codebase — starting with an empty lessons file means
rediscovering it, at the cost of one more failed phase.

## Step 8: Generate the pre-flight document

Copy `{templates}/preflight.template.md` to `{preflight}`. This one has no
template header — its whole content is addressed to the human who runs
pre-flight, so it survives generation intact.

Fill Step 3 (claim verification) from the plan's **Unverified assumptions**
table, plus anything under `## Unverified assumptions carried forward` in the
previous milestone's review, plus — if `paths.design` is configured — the
design inventory's `## Design decisions resting on unverified assumptions`
table. Those are assumptions the architecture is already built on, which makes
them the expensive kind. If both are empty, say so in the report — an empty
claim-verification table is a finding, not a clean bill of health.

Fill the rest from the plan's pre-flight section if it has one. If the milestone
changes anything whose "before" state becomes unrecoverable once fixed — a
correctness bug, a performance regression, a data migration — a baseline
measurement is MANDATORY. Once the fix lands, "how broken was it really?" has
no answer.

**Carry the milestone's requirement IDs into Step 5 (project-specific).** One
checkbox per ID assigned to this milestone, so that pre-flight and close read
against the same list. This is not duplication: the plan says which phase
covers an ID, pre-flight says the list was seen before phase 1 started.

**Separate claims from open decisions.** Step 3 is for things that can be
settled by running something. A row that reads "decide X" is not a claim — it
is an open decision, and it belongs in Step 4 as its own checkbox with the
milestone it is due in. Mixing the two produces a claim-verification table
whose rows cannot be checked by anything, which is how a table ends up signed
with most of it unresolved.

Leave it marked `STATUS: PENDING`. `/spec-create` refuses phase 1 until every
row in Steps 3 and 4 has an outcome AND a human marks it `COMPLETE`. Both
conditions — the signature alone was never meant to be the gate.

Generated files must contain no template instructions. If the words "template"
or "slot" survive into `{core_rules}` or `{lessons}`, the strip step was missed.

## Step 9: Report

```
### PLAN-INIT — {milestone}

**Config:**      created  |  verified  ({fields still needing edits})
**Templates:**   {resolved path} (rule {which of the four matched})
**Plan:**        copied from {source} → {paths.plan} ({N} phases detected)
**Skeleton:**    created
**core-rules:**  generated ({M} slots filled, {K} need review)
**lessons:**     empty  |  {S} seeded from {source}
**pre-flight:**  PENDING — complete before /spec-create 1
**Next:**        review {core_rules}, then run pre-flight

PLAN_INIT_DONE
```

Do not implement anything. Do not write specs. Do not write the plan.
