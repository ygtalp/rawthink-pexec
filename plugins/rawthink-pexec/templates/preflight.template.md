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

- [ ] Assumptions in the plan that were never actually checked — check them now
- [ ] Anything found to be false: correct the plan before phase 1

## 4. Project-specific

- [ ] {add items particular to this milestone}

---

**Completion:** set STATUS to COMPLETE above, with date and who ran it.
