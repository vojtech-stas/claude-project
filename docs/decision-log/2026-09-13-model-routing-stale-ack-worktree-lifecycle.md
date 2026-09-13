# 2026-09-13 — model routing, a stale release ack, and worktree lifecycle

**Context.** One checkpoint answered five owed decisions. Three concern this repo and are recorded here; two concern the operator's machine-wide setup and are recorded outside it. The first of the three was not answered with an option at all — the operator rejected the question's framing — so it is closed as rejected and the direction given instead is recorded as its own decision. The other two were answered as recommended, and preparing the evidence for one of them changed what its answer means.

## Decisions

**D257 — which model writes the code for a slice?**

- [ ] Move the implementer to the mid tier; reviewer and tester stay on the top model — rejected.
- [ ] Try it for one batch and decide on the numbers — rejected.
- [ ] Keep the implementer on the top model permanently — rejected.
- [x] **None of the above.** The operator replaced the single-agent question with a whole-fleet assignment, recorded as **D292**: the controlling session runs the top controller model; implementation and everything done frequently runs the mid tier; the premium model is reserved for audits and for the occasional wide pass whose job is to find what the controller missed.

This is not the first option under another name. That option moved one agent down one tier; the direction reassigns every tier including the controller's own, and it reverses a standing 2026-08-30 direction that had kept the orchestrator on the premium model precisely because everything below it had already been pinned down.

**The test clause is an obligation, not a caveat.** The direction ends *"if we see more rejection-rate we will improve the coding model."* No rejection-rate figure exists today, so "more" has nothing to be more than, and the clause would be unfalsifiable the moment the routing changed. A baseline — critic BLOCK rate per dispatched implementation over a named window — is therefore owed as part of D292 rather than noticed afterwards.

**D284 — the promotion ack went stale.**

The operator approved a release of 3 changes on 2026-08-23. Measured at answer time: 22 commits on `develop` that `main` has never seen (`origin/main` f7378de..`origin/develop` 3c1356b), 39 files, +4690/-461.

- [x] **Show the 22 first, then approve** — chosen.
- [ ] Approve all 22 now on the operator's read of them — rejected.
- [ ] Hold the release until the v1 issue sweep finishes — rejected.

Two facts surfaced while preparing that summary, both of which change what an approval means:

1. The batch touches guardrail paths — three hook scripts and `tools/ci-checks.sh` — so `tools/promote.sh` requires the `.claude/PROMOTE_OK` human-ack sentinel. Per [ADR-0070](../../decisions/0070-two-tier-autonomous-delivery.md) D4 no agent creates it; it stays the operator's own act.
2. The release gate is independently held shut. `python dashboard/health.py --check RELEASE-READY` reports WARN — condition (e), one open `needs-human` item (#1325). The promotion would refuse whatever the ack says.

So what is being sought is a standing approval of content, not a release; and the escalation on #1325 is the actual blocker. Recorded here because the 2026-08-23 ack went stale the same way, and an approval whose dependency is unstated is the mechanism by which it happens again.

**D286 — scratch copies of the repo pile up because nothing ever calls the cleanup.**

- [x] **Retirement becomes part of the job** — chosen. A worktree is recorded when it is created and retired by the step whose work it served; anything holding unsaved work is reported rather than deleted.
- [ ] Leave the design alone and give the existing cleanup an automatic cadence — rejected on measurement: `prune`'s "landed" route tests whether the branch has a merged PR, and none of the leftover copies measured had one — so a scheduled run would remove nothing, while the one route it does fire on skips every safety check that the other route applies.
- [ ] Refuse to start new work while too many un-retired copies exist — rejected: it converts a cleanup problem into a work stoppage.

**Two constraints this answer inherits, both established by measurement rather than by design intent.**

*Ordering.* The retirement step must not land before the `prune` Route A precondition fix on #1044. Giving reclamation an automatic caller makes the unguarded route fire far more often than it does today, so shipping the caller first would convert a dormant gap into a live one.

*Acceptance criterion.* Retirement must count unreachable commits, not just ask whether a branch is merged: `git rev-list --count <worktree HEAD> --not --branches --remotes`. A detached worktree's commits are reachable only through its own HEAD, so removing the directory orphans them and deleting the branch is irrelevant; work on a named branch survives directory removal and dies one step later, when the branch is deleted. The existing tool does both in one breath. Measured at answer time: twelve such commits across six copies, ten of which exist in no other form.

**Outcome.** D292 recorded as its own decision with the routing and the baseline obligation. D284's summary of the 22 raised as the next operator ask, with the guardrail-ack and gate-held facts stated up front; no sentinel created. D286 queued behind #1044 with the reachability check as an acceptance criterion rather than an implementation note.

**Pointers.** #1379 (worktree lifecycle, owning issue); #1044 (`prune` Route A preconditions); #1325 (the escalation holding the release gate); [ADR-0070](../../decisions/0070-two-tier-autonomous-delivery.md) D2/D3/D4 (promotion gate and human ack); [ADR-0058](../../decisions/0058-worktree-isolation-as-asserted-interface.md) D3 (guard is ff-only and loud).
