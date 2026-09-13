# 2026-09-09 — context trim, guard verification, and repo cleanup

Eight decisions answered in one checkpoint. All eight were raised on 2026-08-23 and carried on a per-problem card; the 17-day gap between raising and answering matters for one of them, noted below.

## Context

A single session in late August surfaced two clusters at once: a set of one-directional guards that each passed their own check while the thing they protected was broken, and a set of measurements showing the always-loaded context bill was both larger and less trimmable than the approving decision assumed. The cleanup items are unrelated housekeeping that had accumulated behind them.

## Decisions

- [x] **D102 — Demote four rule scopes from GLOBAL to AREA.** A: do it as its own PRD, with an ADR-0073 amendment and the test update. Considered: (B) fold it into the existing prose-trim PRD; (C) leave the scopes global. Rejected B because the change is not prose — the scope map encodes ADR-0073 D3 and the generator's test asserts that split as a contract, so it breaks tests by design. Tracked as #1273, rescued to `backlog` on this decision.
- [x] **D103 — Wall-clock tests.** A: mark at the source with a pytest marker and let CI deselect, instead of listing them in the quarantine file. Considered: (B) keep the hand-maintained list. Rejected because the list enumerates the flaky tests someone noticed, not the flaky set — a mechanical criterion filtered it correctly and still returned a wrong answer. Accepted trade-off: deselected tests stop running in CI. Tracked as #1385.
- [x] **D104 — Guard verification.** A: promote "a guard is not verified until it has been shown to fail on purpose" to a numbered rule with a mechanical check, per rule #23. Considered: (B) leave it as guidance. Rejected because nine guards in one session passed their own checks while broken, and a negative control caught them in under a minute once run. Binds forward only; no retrospective sweep. Tracked as #1386.
- [x] **D105 — A captured item that was really backlog.** A: rescue it — remove `captured`, add `backlog`. Applied to #1278.
- [x] **D106 — Prose-trim shortfall.** A: accept the −21.7% against the −25% that was approved, and let the record carry the correction. Considered: (B) go after the remaining gap now. Rejected as scope the D102 PRD already covers more cleanly. The PRD had already auto-closed on machine PASS per ADR-0040 D2, so this decision is about the record, not the closure.
- [x] **D107 — Promotion ack.** A: ack the batch. **Recorded, deliberately not executed.** The ack covered three commits and was 17 days old at answering time; the development branch is now 22 commits ahead of the released version, so acting on it would promote 19 unreviewed commits. The approval sentinel was not created — creating it is the operator's act alone, per ADR-0070 D4. Superseded by a new dated item on the current batch.
- [x] **D108 — Orphaned local commits.** A: drop them — reset local `develop` to `origin/develop`. Verified already satisfied at answering time: local `develop` was level with its remote, and the five orphan commits had been carried onto a rescue branch, whose SHA the D112 manifest preserves.
- [x] **D112 — Stale local branches.** A: delete all but `main` and `develop`, restoring from the manifest if ever needed. Executed: 493 of 495 deleted. The two survivors are checked out in live agent worktrees and were correctly left alone. A full restore manifest — every branch with its SHA, upstream, last commit and orphan-commit count — was written before any deletion, because 166 of the branches carried commits reachable from no remote.

## Pointers

- Card: the standing project Decision Inbox, revision `2026-09-09-inbox-*`.
- Issues: #1273 (D102), #1385 (D103), #1386 (D104), #1278 (D105).
- ADRs touched by the follow-on work: ADR-0073 (D102), ADR-0067 (D103), ADR-0056 (D104), ADR-0070 (D107).
