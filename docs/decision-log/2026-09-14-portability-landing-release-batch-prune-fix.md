# 2026-09-14 — the portability stop lifted, the release batch approved, and a silent-work-loss fix promoted

**Context.** One checkpoint answered three owed decisions, all of them gating each other. The release batch could not move while an escalation held the gate shut; the escalation could not be cleared by the pipeline that raised it, because clearing it *was* the thing it stopped; and a fix for a cleanup route that destroys work without checking had been stuck in the captured tier since July because its own description was wrong. All three were answered as recommended. Applying them surfaced a fourth problem nobody had asked about: the clone's own configuration had gone stale in two places, and one of those was making every queue-discovery surface report a false zero.

## Decisions

**D306 — the portability work passed both reviewers on its last allowed round.**

- [x] **Land it** — chosen. Write the decision record, cut the slices, implement, and close the escalation pointing at the published PRD.
- [ ] Land it but leave the escalation open until the work is merged and verified — rejected.
- [ ] Hold until the operator has read the PRD themselves — rejected.

The pair was re-authored from scratch under a fresh round counter, as authorised on 7 September, and both critics returned APPROVE in the same round with nothing outstanding — the first time this pair has passed. The escalation (#1325) was deliberately left open after that pass rather than closed by the run that earned it: landing the work would have resolved a human-escalation item by doing the thing it stopped, which is the outcome the escalation surface exists to prevent. The answer is what converts an approved document into a landing.

**D293 — the 22 changes waiting to be released.**

- [x] **Approve the 22 as a batch** — chosen. Close the blocking escalation, then report the gate state; the operator creates the approval file.
- [ ] Hold the release until every open issue is closed and v1 is tagged — rejected.
- [ ] Approve these and give standing approval to future non-guardrail batches — rejected.

Two properties of this approval were stated before it was given and hold after it: approving is not releasing, because the batch touches guardrail paths and `tools/promote.sh` therefore requires the `.claude/PROMOTE_OK` sentinel that no agent may create (ADR-0070 D4); and the gate was separately held shut by the open escalation above, so the ordering between D293 and D306 was never optional. With #1325 closed, `RELEASE-READY` now reports the gate open on all six conditions. The sentinel remains the only outstanding step, and it remains the operator's own act.

**D295 — a cleanup route deletes scratch copies without checking they are empty.**

- [x] **Promote it to the forward queue now** — chosen. It becomes its own piece of work, and the lifecycle work is held until it lands.
- [ ] Fold it into the lifecycle work as that work's first step — rejected.
- [ ] Close it and write a fresh capture so the critic rules on a body it has never seen — rejected.

`backlog-critic` BLOCKed promotion correctly, on a body that carried a falsified diagnosis. The body was rewritten to carry the measured defect with the original diagnosis preserved verbatim, and the label was deliberately not moved on that rewrite — promoting on a body rewritten after the verdict would be the autopilot grading its own homework. The promotion is therefore a recorded human override of a machine verdict, which is the rescue path the captured tier exists to leave open, rather than a second critic run. The honest cost, named when the option was offered: no check ruled on the corrected body.

## Discovered while applying

Two pieces of this clone's local configuration had drifted to stale absolute spellings of a directory that had been renamed, and neither announced itself:

1. `remote.origin.url` still named the pre-rename repository. GitHub redirects a renamed repository, so fetch, push, and unfiltered issue listing all kept working — but **label-filtered** issue queries resolved against the stale name and returned an empty list with exit 0. Every surface that asks "what is in the queue" — the session-start context injection, the round-3 escalation-discovery instruction, the queue-drain assembly — consequently reported zero open slices, zero open PRs, zero captured items and zero escalations, at a moment when all four were non-empty. The false statement was well-formed, and it is the statement a long unattended run is most likely to act on.
2. `core.hooksPath` pointed at a third spelling of the repo root that exists nowhere, which means the tracked pre-commit and commit-msg hooks had not been running at all.

Both were repaired in local configuration only; nothing tracked changed. The class is captured as a root-cause issue, whose proposed fix is an identity assertion at the source — a check that the resolved remote matches the repository this checkout actually talks to — rather than annotating each downstream consumer. This is the failure mode ADR-0083 D3/D5 names: a signal asserted one state that its observation could not distinguish from a second state consistent with the same evidence.

**Outcome.** #1325 closed and `needs-human` cleared, pointing at PRD #1390. #1044 promoted `captured` → `backlog` with the override recorded on the issue. `RELEASE-READY` PASS; the promotion sentinel is owed by the operator and no agent created it. The lifecycle work stays held behind #1044.

**Pointers.** #1390 (PRD, portability scrub); #1325 (the escalation, now closed); #1044 (`prune` Route A preconditions); #1379 (worktree lifecycle, held); #1396 (root-cause capture for the false-zero queue reports); ADR-0070 D2/D3/D4 (promotion gate and human ack); ADR-0048 D1 / CRI-001 (round-3 strict-stop); ADR-0083 D3/D5 (a signal may only assert what it can observe).
