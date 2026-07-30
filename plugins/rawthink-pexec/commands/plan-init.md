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

## Step 4: Validate the plan

Read the copy and confirm it can drive the pipeline:

- [ ] Phases are individually addressable (a `### Phase N` heading or equivalent)
- [ ] Each phase states scope, key files, and validation criteria
- [ ] A dependency order exists (explicit graph or stated ordering)
- [ ] The plan contains NO implementation code

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

## Step 8: Generate the pre-flight document

Copy `{templates}/preflight.template.md` to `{preflight}`. This one has no
template header — its whole content is addressed to the human who runs
pre-flight, so it survives generation intact.

Fill it from the plan's pre-flight section if it has one. If the milestone
changes anything whose "before" state becomes unrecoverable once fixed — a
correctness bug, a performance regression, a data migration — a baseline
measurement is MANDATORY. Once the fix lands, "how broken was it really?" has
no answer.

Leave it marked `STATUS: PENDING`. `/spec-create` refuses phase 1 until a human
marks it `COMPLETE`.

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
