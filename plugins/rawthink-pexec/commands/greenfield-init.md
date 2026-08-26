# /greenfield-init — Prepare a project that has no code yet

Argument: `/greenfield-init`

Run ONCE, per project, before the first `/plan-init`. Produces two documents
and no code.

## When this is needed — and when it is not

`/spec-create` re-derives implementation from live source every time, and
`plan.template.md` asks each phase to anchor on real code. Both assume a
codebase exists.

On a greenfield project neither holds. Plan mode has nothing to derive
structure from, so it invents type names and file paths — and nothing catches
it. `/plan-init` Step 4 checks that the plan carries no implementation *code*;
it does not check that the anchors point at anything real. The invented anchor
passes validation and fails in `/spec-create` Step 5, several sessions later.

**Skip this command entirely if the project already has source.** There, the
codebase is the design document and this would duplicate it.

## What it does not do

**It writes no code.** The skeleton is built afterwards, in ordinary sessions,
from the scope map this command scaffolds. Writing code without a spec is what
the rest of the pipeline exists to prevent, and a bootstrap command is not an
exemption.

## Step 0: Resolve the config

Read `.pexec.yml`. If it does not exist, copy `.pexec.example.yml` to
`.pexec.yml` and set `milestone` to the first milestone name — you can leave
`compile`, `test`, `must_read` and `integration_checkpoints` as placeholders
for now. They cannot be answered before a skeleton exists; filling them is the
last thing this bootstrap produces, not the first.

Confirm `paths.skeleton_scope` and `paths.design` are set. If the file predates
this command, add them with the defaults from `.pexec.example.yml`.

## Step 1: Resolve the templates directory

Same four-way resolution as `/plan-init` Step 1, in this order:

1. `paths.templates` from `.pexec.yml`, if set
2. `${CLAUDE_PLUGIN_ROOT}/templates/` — installed as a plugin
3. `.claude/pexec-templates/` — installed manually
4. `templates/` relative to this command file — running from a clone

If none of them contains `skeleton-scope.template.md`, STOP and say so.

## Step 2: Check the upstream

Look for a milestone map and a derivation document for the first milestone.

- **Both present** — good. Name them in the report; the scope map is filled
  from them.
- **Map missing** — say so and continue. A scope map can be built from a
  derivation document alone; it will just have no sequence around it.
- **Both missing** — STOP. There is nothing to draw a skeleton from, and a
  skeleton drawn from nothing is an architecture invented at the worst possible
  moment. Point at `milestone-map.template.md` and
  `derivation-doc.template.md`.

## Step 3: Scaffold the scope map

Copy `{templates}/skeleton-scope.template.md` to `{paths.skeleton_scope}` and
strip the template header block.

Then fill what the upstream settles, and only that:

- one row per capability named in the map or the derivation document
- the module each belongs to, if the upstream says
- open/closed side, if the upstream draws that boundary

Leave the "In the skeleton as" column EMPTY. That column is the decision this
bootstrap exists to force, and pre-filling it with a guess is how a capability
quietly becomes someone's assumption.

Leave it marked `STATUS: PENDING`. `/plan-init` refuses to run until a human
signs it, the same way pre-flight blocks `/spec-create 1`.

## Step 4: Scaffold the design inventory

Copy `{templates}/design-inventory.template.md` to `{paths.design}` and strip
the header. Leave every table empty and `STATUS: PENDING`.

**Scaffold both or neither.** `/plan-init` treats a lone scope map or a lone
inventory as an error, because half a pair means one of them was deleted or
never written and there is no safe way to tell which.

It cannot be filled yet. Its input is the skeleton's live source, which does
not exist until Step 5 is done by hand.

## Step 5: Report the sequence

This command ends here. What follows is manual, in this order, and the order is
load-bearing:

```
1. fill    {paths.skeleton_scope}     ← from the map and derivation doc
2. build   the walking skeleton       ← ordinary sessions, no pexec
3. fill    {paths.design}             ← reads the skeleton's live source
4. fill    .pexec.yml                 ← compile, test, must_read, checkpoints
5. run     /plan-init <milestone>
```

Step 2 has no command and should not have one. A skeleton is a thin end-to-end
slice that runs — the thinnest path through the system that a test can walk.
Deciding what belongs in that slice is a design judgement, and the scope map is
where it gets recorded, not automated.

Step 4 is last because its four answers come from the skeleton: `compile` and
`test` are the commands that actually build and test it, `must_read` is its
core type definitions, and `integration_checkpoints` are its wiring points.
Answering them earlier means answering them from imagination.

```
### GREENFIELD-INIT

**Config:**        created | verified
**Templates:**     {resolved path} (rule {which of the four matched})
**Upstream:**      map: {found|missing} | derivation: {found|missing}
**Scope map:**     scaffolded, {N} capability rows, STATUS: PENDING
**Design:**        scaffolded, empty, STATUS: PENDING
**Next:**          fill the scope map, then build the skeleton

GREENFIELD_INIT_DONE
```

Do not write code. Do not write the plan. Do not fill the design inventory.
