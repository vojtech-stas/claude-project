# 2026-09-07 — residual close-out and dispatch model pins

**Context.** The operator returned from a quota pause with the v1 sweep half-run and eight owed decisions queued from the prior session. Six were residuals whose machine half was already done and whose only open question was how to accept them; two were about how much model capacity the pipeline spends per dispatch. One item was rejected rather than answered — the operator said the framing itself was wrong — and it was superseded rather than re-asked. Nothing here changes an ADR; the decisions unblock work that was waiting on an answer.

## Decisions

**D229 — The portability PRD/ADR pair strict-stopped at round 3 (#1325). What now?**

- [x] **Authorize one fresh revision and a new critic loop from round 1** — **chosen**. The two blocking findings are enumerable (a missing propagation section against six live citations; a self-inflicted CI deadlock from an executable proof block the draft's own new check would fail by name), and the PRD side already carries a round-3 APPROVE. A fresh loop is cheaper than re-deriving the census.
- [ ] Abandon the pair and re-grill the problem — rejected: the measurement work is sound and would be repeated verbatim.

**D230 — How wide is the regression rider that ADR-0067 D3 imposes?**

- [x] **All fix-type trivial-lane PRs touching runtime code** — **chosen**, overriding the recommendation. Ships as a superseding ADR plus the matching check update. The operator's reasoning: the trivial lane is exactly where a "too small to test" fix slips through, so exempting it inverts the rule's purpose.
- [ ] Only fix-type PRs that close a root-cause capture — the recommended narrower form; rejected as the easier line to draw rather than the right one.

**D231 — Proof binaries in the repo (#823).**

- [x] **No new binaries committed; keep a tracked manifest (path, hash, size) for existing proofs** — **chosen**, so existing references stay checkable without the repo carrying more weight. Aligns with the draft already in hand.
- [ ] Commit proof binaries as they are produced — rejected: unbounded growth for evidence whose value expires.
- [ ] Delete existing proof binaries outright — rejected: live references would break.

**D232 — A stray directory outside the repo, referenced by nothing.**

- [x] **Delete it after a glance** — **chosen**. Verified during application that the tree was already gone; the decision is recorded as answered rather than executed, with the note attached.

**D233 — A residual that can only be witnessed by a real run (round-3 strict-stop escalation).**

- [x] **Accept on the documented behaviour plus the proven I5 mechanism** — **chosen**. The residual closes the first time a real drain run witnesses a round-3 strict-stop; manufacturing the witness would mean fabricating the very evidence the gate exists to demand.
- [ ] Hold the residual open until a synthetic case is constructed — rejected on rule #21 grounds: synthetic evidence in a production ledger is invalid by construction.

**D234 — Two drain-mode residuals (#1376, #1378) awaiting the same kind of witness.**

- [x] **Let the next real drain run be the witness** — **chosen**. #1376 lands through the fix-in-run lane; the run parks and resumes once at a natural boundary; #1378 closes when both record shapes appear in that run's ledger. One real run closes all three obligations at once.
- [ ] Close them on documented behaviour alone — rejected: the ledger record shapes are exactly what is in doubt.

**D245 — Do the three highest-judgment dispatch roles move to a cheaper model?**

- [x] **Keep the existing routing decision unchanged; all three stay on the higher-capability model** — **chosen**. Their deliverable *is* judgment, and a cheap model's failure mode there is a plausible wrong answer rather than an obvious error — the one failure the pipeline cannot catch downstream.
- [ ] Move all three down a tier for the token saving — rejected on that asymmetry.

**Rejected framing.** One queued item asked for per-project capacity limits. The operator rejected the premise rather than choosing an option — *"we need to set what tasks will be done by which model rather than per project limits"* — so it was superseded by a task-routing item rather than re-asked, and a second item was raised for the measurement question the operator asked alongside it. Both are environment-scope, tracked in the operator decision log outside this repo; no repo artifact depends on them.

**Outcome.** D229 authorizes a fresh critic loop on #1325. D230 commissions a superseding ADR widening the regression rider plus its check. D231 commissions the proof manifest. D233 and D234 convert three residuals from "open" to "closes on the next real drain run", which is what makes the queue drain the next thing to run rather than a thing waiting on answers. D245 leaves dispatch routing as it stands.
