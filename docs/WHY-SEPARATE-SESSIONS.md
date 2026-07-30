# Why each step runs in its own session

The pipeline could run as one command. It does not, and this is the decision
everything else follows from.

## Verification cannot share context with authoring

A verifier running in the session that just wrote the spec is marking its own
homework. It inherits the same reading of the source, the same assumptions, and
crucially the same *omissions* — the files it did not open are files it does not
know it did not open.

A fresh session sees only the spec and the repo. If the spec does not stand on
its own, that is now observable rather than filled in from memory.

This is also why `spec-verify` loads no memory tools. A gap that can be excused
by a past decision has not been verified; it has been rationalised.

## The closing audit should not have watched the work

`post-phase` compares intent against reality. A session that watched the
implementation happen will report what it remembers doing, which is the same
source that produced the deviation in the first place.

Reading the diff cold is the whole value.

## What this costs

Nothing carries over except what is written down. Every step rebuilds its
understanding from files.

That cost is real: four sessions per phase, each paying startup context, each
re-reading. It is also why the summary compression matters, why the spec must
be complete, and why Open Questions must be empty before implementation — there
is no conversation to scroll back to.

## What it buys

Four independent observations of the same phase, none of which can quietly
inherit another's blind spot.

## When to collapse steps

`spec-implement` and `post-phase` can share a session if you accept a weaker
audit — post-phase's job is to check the implementation, and having watched it
is a disadvantage but not a disqualification.

`spec-create` → `spec-verify` must never be collapsed. That boundary is where
an incomplete spec gets caught, and an incomplete spec is the single most
expensive failure in this pipeline: it is the input to everything downstream.
