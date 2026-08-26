# rawthink-pexec

Phase-by-phase execution for AI coding agents. Six commands, one contract per
phase, a verification gate the implementer cannot skip, and two manual gates
that bracket the milestone. Plus one command that runs once, on a project that
has no code yet.

**No runtime, no dependencies, no MCP server.** The whole thing is markdown.

---

## The problem

You hand an agent a plan. It writes code. Somewhere between "here is the plan"
and "here is the code" it added something you did not ask for, missed a caller
you did not know about, or quietly answered a question it should have asked.

Not because it is careless. Because a single long session has no seam where
anyone checks the work, and by the time the diff appears the reasoning that
produced it has scrolled away.

## The shape

```
   milestone map           upstream — which milestones exist, in what order
   ↓
   derivation doc          one milestone's argument, drawn from the map
   ↓
/greenfield-init           once, only if there is no codebase yet
   ↓
   skeleton scope          which capabilities exist on day one, and as what
   ↓
   the skeleton            built by hand — a thin slice that runs
   ↓
   design inventory        types, signatures, files — what the plan anchors on
   ↓
/plan-init <milestone>     once — import the plan, scaffold, generate rules
   ↓
   pre-flight              once, manual — baselines, backups, claim checks
   ↓
/spec-create N             write the contract          ← new session
/spec-verify N             try to break it             ← new session
/spec-implement N          build exactly that          ← new session
/post-phase N              audit, summarise, record    ← new session
   ↓
   repeat for N+1
   ↓
/milestone-close           once, after the last phase  ← new session
   ↓
   milestone review        manual gate — blocks the next /plan-init
   ↓
   update the map          findings land, next milestone is planned from them
   ↺

**Each step runs in its own session.** That is the load-bearing decision.

A verifier that shares context with the author inherits the author's blind
spots and confirms its own omissions. A closing audit that watched the
implementation happen will report what it remembers, not what is on disk. The
session boundary is what makes each step an independent observer, and it is
cheaper and cleaner than any prompt instructing a model to "be objective".

The cost is that nothing carries over except what is written down. That is also
the point.

## Install

### Plugin — you, everywhere (recommended)

```
/plugin marketplace add ygtalp/rawthink-pexec
/plugin install rawthink-pexec@rawthink
/reload-plugins
```

CLI equivalents, for scripting:

```bash
claude plugin marketplace add ygtalp/rawthink-pexec
claude plugin install rawthink-pexec@rawthink --scope user
```

Then, in your project:

```bash
cp .pexec.example.yml .pexec.yml   # edit milestone, language, build/test commands
```

### Plugin — a whole team, zero commands

Commit this to your project's `.claude/settings.json`:

```json
{
  "extraKnownMarketplaces": {
    "rawthink": {
      "source": { "source": "github", "repo": "ygtalp/rawthink-pexec" }
    }
  },
  "enabledPlugins": { "rawthink-pexec@rawthink": true }
}
```

A teammate clones the repo, opens Claude Code in it, trusts the folder — and
the plugin installs itself. Nothing to type.

### Manual — no marketplace

```bash
git clone https://github.com/ygtalp/rawthink-pexec
cp rawthink-pexec/plugins/rawthink-pexec/commands/*.md   <your-project>/.claude/commands/
cp -r rawthink-pexec/plugins/rawthink-pexec/templates    <your-project>/.claude/pexec-templates
cp rawthink-pexec/.pexec.example.yml                     <your-project>/.pexec.yml
```

### Vendored — offline or restricted environments

Same as manual, but commit the copied files to your repo. No network, no
marketplace, no plugin system. In locked-down environments where administrators
restrict marketplaces via `strictKnownMarketplaces`, this is the path that
always works.

---

**Then, in every case:**

```
# greenfield only, once per project:
/rawthink-pexec:greenfield-init
# fill the scope map, build the skeleton, fill the design inventory, then:

/rawthink-pexec:plan-init <milestone>
# complete pre-flight, then:
/rawthink-pexec:spec-create 1
# ... phases ...
# after the last phase:
/rawthink-pexec:milestone-close
```

Plugin commands are namespaced `/<plugin>:<command>`. Typing the short form
(`/plan-init`) finds them too, as long as nothing else in your setup uses the
same name. Installed manually rather than as a plugin, the short form is the
only form.

`/plan-init` finds the templates in whichever of the four layouts you used, so
all of them behave identically from here on.

## Where the plan comes from

Above the plan sit documents that no command reads. They are written and read
by people, and they are here because the pipeline needs an upstream that holds
still.

The **milestone map** (`templates/milestone-map.template.md`) answers which
milestones exist, in what order, and what constrains them. One row becomes one
derivation document. It is also where `/milestone-close` findings land — the
deviations, the false assumptions and the questions a milestone raised and did
not answer, none of which any command consumes.

The plan is derived from a **derivation document** — one milestone's row, worked
out in a session that can search, from `templates/derivation-doc.template.md`.
Plan mode derives structure from live source: file layout, symbol names, the
call graph. It cannot derive what the domain requires, why one approach beat
another, or which claims nobody has checked.

Two of that document's sections are consumed downstream: unverified assumptions
become pre-flight's claim checks, and code-anchored traps seed the lessons file.
The rest is read once, by whoever writes the plan, and never again.

### When there is no codebase yet

`/spec-create` re-derives implementation from live source, and each phase is
expected to anchor on real code. A greenfield first milestone has neither, so
plan mode invents anchors — and `/plan-init` Step 4 does not catch it, because
it checks that the plan carries no implementation code, not that its anchors
resolve. The invented anchor fails four sessions later, in `/spec-create`.

`/greenfield-init` scaffolds the two documents that close that gap:

The **skeleton scope** (`templates/skeleton-scope.template.md`) records which
planned capabilities exist in the first skeleton and as what — `interface`,
`walking`, or `absent` with the milestone named. There is no fourth value,
because "later" is how a capability disappears. It also names the walking
slice: the thinnest end-to-end path a test can follow, chosen so that it
exercises whatever claim the architecture rests on.

You then build that skeleton by hand, in ordinary sessions. There is no command
for it and there should not be: what belongs in the first slice is a design
judgement, and the scope map is where it is recorded rather than automated.

The **design inventory** (`templates/design-inventory.template.md`) is written
after the skeleton exists and reads it. Type names, signatures, field lists,
file tree — the design that a brownfield project gets for free from live
source. Every type traces to a planned capability and to a phase; a row missing
either is scope that entered through design instead of planning.

Both are consumed once and never read again, like the derivation document.
Neither carries implementation.

`/plan-init` refuses to run while the inventory is `PENDING`, and pulls its
assumptions into pre-flight — those are assumptions the architecture is already
built on.

Skip all of this on a project that already has source. There the codebase is
the design document, and an inventory would only be a worse copy of it.

### Writing the plan

Then write the plan with Claude Code's plan mode. Point plan mode at
`templates/plan.template.md` and ask for a milestone plan in that shape — the
output then passes `/plan-init` validation on the first try.

`/plan-init` finds that plan (plan mode leaves it in `~/.claude/plans/` under a
generated name), **copies** it into your project, and works from the copy.

Run `/plan-init` in the same session that produced the plan. It is not a
judgement step, so it does not need a clean context — and the session that just
wrote the plan knows which file it is, while a fresh one sees only a directory
of generated names. Session isolation starts at `/spec-create`.

The copy is not tidiness. `~/.claude/plans/` is global and mutable — every
later plan mode run adds to it. `/post-phase` re-reads the plan on every phase,
so across twenty phases the document phase 20 validates against must be the one
phase 1 was written from. Copying freezes it and makes the project authoritative.

## Six channels

Everything that survives between steps lives in one of these:

| Channel | Carries | Lifetime |
|---|---|---|
| `phase-{N}-SPEC.md` | the contract | the phase |
| `phase-{N}-review.md` | the verify verdict | the phase |
| `phase-{N}-impl-log.md` | what actually happened | the phase |
| `activeContext.md` | current milestone state | the milestone |
| `summaries/phase-{N}-summary.md` | what a later phase can rely on | forever |
| `milestone-review.md` | what the next milestone starts from | across milestones |

The verdict has its own file rather than living in `activeContext.md`.
`activeContext.md` is a status file that later steps rewrite, and a gate whose
verdict can be overwritten by the step it gates is not a gate.

The summary is a ~30 line compression of a 1000+ line spec. That ratio is what
makes a twenty-phase milestone survivable — later phases read summaries, never
full specs.

## Two guardrail files

**core-rules** — the format contract. Violation tiers, spec section order,
summary format, what "compile" means in this project. Generated from a template
by `plan-init`; mostly universal, with project-specific slots.

**lessons** — traps discovered in *this* codebase, each with an L-code. Starts
empty. `spec-create` reads it before writing; `spec-verify` treats a reproduced
lesson as a blocking violation; `post-phase` appends new ones. Capped at 80
lines, because it is read at the start of every spec-create.

The L-code is not bookkeeping. `spec-verify` enforces lessons by citing them —
an entry with no code cannot be cited, and an entry that cannot be cited is not
enforced.

## Two manual gates

Pre-flight and the milestone review are the only two files a human must sign.
Everything else is written and read by agents.

They bracket the milestone. Pre-flight blocks `/spec-create 1` until state that
cannot be recovered later has been captured. The milestone review blocks the
NEXT `/plan-init` until someone confirms what actually happened.

The symmetry is the design. A milestone that starts without a baseline cannot
prove it worked; a milestone planned on an unreviewed close is planned on an
agent's unchecked report. Both failures are silent, and both surface far
downstream of where they were introduced.

Closing a milestone also produces findings that no command consumes: what
deviated, what the baseline says now, what turned out false, what is still open.
Those go to the milestone map, and the next milestone is planned from the
updated map rather than from the one written before any of it was known. That
step is manual and it is the only thing that closes the loop.

`/milestone-close` is a separate command rather than a branch inside
`/post-phase` because the two have different jobs and different frequencies.
`post-phase` runs after every phase and audits one diff. `milestone-close` runs
once, reads every summary, re-measures the baseline, triages the lessons, and
records what the next milestone inherits.

## Why the plan carries no code

`spec-create` re-derives implementation from live source every time. A plan
carrying code competes with the source and wins arguments it should lose,
because the source moved and the plan did not.

The cost of that choice is that re-derivation can reintroduce a bug the project
already fixed. That is exactly what the lessons file is for. High-level plan →
re-derivation → lessons as the guardrail: the three follow from each other.

## Configuration

Everything project-specific lives in `.pexec.yml`, core-rules and lessons. The
command files are identical across projects — if you find yourself editing one
for a single project, that difference belongs in config.

`paths.skeleton_scope` and `paths.design` are the only greenfield-specific
keys. They can stay: `/plan-init` checks the inventory for the first milestone
only, and retires the check once a predecessor milestone exists.

See `examples/` for C#/Unity, Java/Spring and Python configurations.

## Optional integrations

Both default to off. The pipeline is complete without them.

**codegraph** — code intelligence. `spec-create` asks it what a change reaches
and pastes the answer into the spec's Blast Radius, so the file list comes from
a tool rather than from the model's reading stamina. `spec-implement` uses it to
confirm the right tests ran.

**rawthink-mcp** — decision archive. `post-phase` records what was decided, what
was rejected, and why. Code tells you what was chosen; nothing tells you what was
considered and dropped, which is the question that actually gets asked later.
`spec-create` queries it before proposing an approach, so a settled question is
not re-litigated ten phases on.

The record also goes into the phase summary either way. The archive answers
queries across milestones; the summary is what the next phase actually reads.
Writing to only one of them would mean enabling the integration made the
pipeline see less.

Neither is loaded during `spec-verify` or `spec-implement`. Verification that
can excuse a gap by citing a past decision is rationalising, not verifying, and
recalled context during implementation is how scope drifts.

## Security

Claude Code plugins run with your privileges, and Anthropic does not audit
third-party marketplace content. Read what you install.

For this one that is a short read: **there is no executable code in this
repository.** Twenty markdown and JSON files plus four YAML examples, no
scripts, no hooks, no MCP server, no post-install step. Everything it does, it
does by instructing an agent you are already running — and you see every action
in your transcript.

## Status

Extracted from three projects run this way: a Unity game, a Java/Spring
microservices platform, and a Python MCP server. Validated end to end on two
further demo projects. The structure is stable across all of them; only the
config differs.

The greenfield layer is newer and has one origin rather than three: a project
where the plan carried no anchors because there was no source to derive them
from, and the gap was found before the first phase rather than during it.

MIT.
