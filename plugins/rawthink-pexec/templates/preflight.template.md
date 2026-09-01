# {milestone} — Pre-Flight

**STATUS: PENDING**

Run ONCE, manually, before `/spec-create 1`. This is outside the phase loop and
produces no code.

`spec-create` refuses phase 1 until a human sets STATUS to COMPLETE.

## Why this exists

Some state cannot be recovered once the milestone starts. A baseline
measurement taken after the fix is not a baseline. A backup taken after the
migration is not a backup.

If this milestone changes anything whose "before" state answers a question you
will want answered later — how broken was it, how slow was it, what did the
data look like — capture it here.

---

## 1. Backup / branch

- [ ] Working copy backed up
- [ ] Milestone branch created: `{branch}`
- [ ] Data store snapshot taken (if the milestone touches persistent state)

## 2. Baseline measurement

**Required if the milestone fixes a correctness or performance defect.**

- [ ] Measurement command run against the CURRENT (unfixed) state
- [ ] Output stored at: `{path}`
- [ ] Isolation noted: which metrics this milestone should move, and which must
      stay unchanged

State the second item explicitly. If phase 2 and phase 3 both move the same
metric, neither one's effect is measurable, and the ordering rule that keeps
them separate belongs in the plan.

## 3. Claim verification

Populated by `/plan-init` from the plan's **Unverified assumptions** table, plus
anything carried forward from the previous milestone's review.

| Assumption | How to check it | Outcome |
|---|---|---|
| {claim} | {command or test} | {one of the three below} |

**Every row ends in exactly one of three outcomes. There is no fourth, and
there is no blank.**

- **`HOLDS`** — checked by running something. Say what was run.
- **`FALSE — plan corrected: {what changed}`** — the check failed and the plan
  was fixed. Record the correction here; a plan quietly edited after a failed
  check loses the fact that the check happened.
- **`ACCEPTED AS RISK — {who accepted it, and what breaks if it is false}`** —
  the claim could not be settled now. This is the only way an unchecked
  assumption survives into phase 1, and it costs a named person and a named
  consequence.

The third outcome exists because some claims genuinely cannot be settled yet: a
vendor has not answered, the counterparty does not exist, the tool is not
available. Pretending otherwise produces a table signed with most of it blank,
which reads as verified and is not.

**But it is not a way through.** A milestone that starts with most of its
claims accepted as risk is a milestone built on assumptions, and the count
belongs in the sign-off below where it can be read at a glance rather than
inferred from a table nobody re-opens.

- [ ] Every row has one of the three outcomes — no blanks
- [ ] Every `HOLDS` was reached by RUNNING something, not by reading
      documentation
- [ ] Every `ACCEPTED AS RISK` names a person and a consequence

An empty table means the plan declared no unverified assumptions. Confirm that
is true, rather than that nobody filled it in.

## 4. Open decisions due before phase 1

Decisions this milestone cannot start without. These are not claims — nothing
can be run to settle them; someone has to choose. `/plan-init` puts them here
rather than in Step 3, because a claim-verification row that cannot be checked
by anything is a row that gets signed unresolved.

- [ ] {decision} — chosen: {outcome}, because {reason}

Each one is either **decided** or the milestone does not start. There is no
"accepted as risk" here: an undecided decision is not a risk you carry, it is a
question that will be answered by whoever writes the first spec that needs it,
silently and probably wrongly.

## 5. Project-specific

- [ ] {add items particular to this milestone}

---

## Sign-off

Fill these from the tables above. They are counts, not prose — the point is
that a milestone's real starting position is readable in one line.

| | Count |
|---|---|
| Claims checked, `HOLDS` | |
| Claims `FALSE`, plan corrected | |
| Claims `ACCEPTED AS RISK` | |
| Open decisions closed | |
| **Rows still blank** | **must be 0** |

**Completion:** set STATUS to COMPLETE above, with date and who ran it.

`/spec-create` checks both: the status line AND that no row is blank. A
signature over an unfinished table was the failure this gate was written to
prevent, and a gate that only reads the signature does not prevent it.
